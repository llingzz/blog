---
title: Jenkins自动编译部署
date: 2022-10-17 16:43:23
categories: Jenkins
tags: Jenkins
---

目前的项目中c++后端服务在编译部署测试服时还是采用笨拙的方案：本地编译再手动上传到测试服务器

如果直接再测试服务器上部署一套Jenkins后台，即可实现自动化编译和部署。安装Jenkins和相关环境配置在这不赘述，官方文档有很详细的说明。基于当前项目配置步骤如下：

1. 新建freestyle project

2. 勾选This project is parameterized，并新建**Choice Parameter**和**String Parameter**两个参数，前者表示选择要部署的具体服务**svrname**，后者表示具体服务代码路径**svn_path**，设定好自定义的工作空间用来**checkout**代码（这里使用的svn，需要安装对应的svn的Jenkins插件）

3. 在**Subversion**下配置**svn**配置，主要由两部分组成：**Repository URL**和**Credentials**，前者为svn路径，后者为svn账号权限（需要在Jenkins中配置），**Local module directory**下设定对应存放代码的本地目录**.\${svrname}\svn**，**Repository depth**选择**infinity**即可。部分服务可能不仅仅需要这个代码路径，同理添加配置即可。最后此模块选项**Check-out Strategy**按需设定就好，这里选择的是**Use 'svn update' as much as possible**

4. 接下来添加**Build Steps**，使用**MSBuild**(需要Jenkins插件支持)，设定**MSBuild Version**为**vs2013**，**MSBuild Build File**设定为**.\${svrname}\svn\${svrname}\${svrname}.sln**解决方案的文件目录，命令行参数**Command Line Arguments**：/t:Rebuild /p:Configuration=ReleaseS

5. 添加构建后命令行操作****Execute Windows batch command****：

   `set bak_path=%WORKSPACE%\%svrname%\backup\%BUILD_NUMBER%`
   `set src_path=%WORKSPACE%\%svrname%\svn\%svrname%\ReleaseS`
   `mkdir %bak_path%`
   `cd E:\ServerController\ServerController\%svrname%`
   `copy %svrname%.exe %bak_path%`
   `copy %svrname%.pdb %bak_path%`
   `sc config %svrname% start=DISABLED`
   `rem sc stop %svrname%`
   `taskkill /f /im %svrname%.exe /T`
   `ping 127.0.0.1 -n 15 >nul`
   `sc query %svrname%`
   `copy %src_path%\%svrname%.exe /y`
   `copy %src_path%\%svrname%.pdb /y`
   `sc config %svrname% start=DEMAND`
   `sc start %svrname%`
   `ping 127.0.0.1 -n 15 >nul`
   `sc query %svrname%`

6. 保存应用即可

PS：Jenkins内包含丰富的插件，可以用来实现复杂的自动化操作，这里只是一个非常简单的应用。



