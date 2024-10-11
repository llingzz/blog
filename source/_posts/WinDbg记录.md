---
title: WinDbg记录
date: 2021-11-04 15:17:08
categories: Windows
tags: Windows
---

1.查看堆内存
!heap -s
2.查看堆内存百分比
!heap -stat -h 003b0000
3.打印堆内存块
!heap -stat -h 003b0000 -grp S 1000
4.查看大小为0x14内存的地址
!heap -flt s 14
5.查看堆内存申请堆栈 
!heap -p -a 0x000001ab

6.拉取dump文件
[windbg]attach进程之后
.dump /ma C:\Users\xxx\Desktop\dump\1.dmp
[ntsd.exe]
ntsd -pv -p PID
.dump /ma C:\Users\xxx\Desktop\dump\aaa.dmp

7.umdh分析内存泄漏
cd C:\Program Files (x86)\Debugging Tools for Windows (x86)
set _NT_SYMBOL_PATH=C:\xxx\xxx\xxx\tcbychunksvr;srv*E:\symbol*http://msdl.microsoft.com/download/symbols
gflags -i TcByChunkSvr.exe +ust
umdh -pn:TcByChunkSvr.exe -f:d:\1.log
(隔一段时间之后)
umdh -pn:TcByChunkSvr.exe -f:d:\2.log
umdh -d D:\Snap1.log D:\Snap2.log -f:d:\result.txt

8.windows死锁相关
1). ~*kb 查看当前所有线程堆栈，从堆栈中找出可能有锁问题的线程(例如堆栈：ntdll!RtlEnterCriticalSection)
2). !cs 00403370(临界区地址，可以从堆栈的参数列表找到)
LockCount:x x个线程在等待它
OwningThread 表示拥有这个临界区的线程ID
RecursionCount 表示拥有线程调了几次
LockSemaphore 实际上是一个自复位事件
SpinCount 旋转锁的设置，单CPU下忽略
3). ~~[OwningThread] 查看当前拥有锁的线程的堆栈(如下)
	5  Id: aec.43c8 Suspend: 1 Teb: 0054f000 Unfrozen
		  Start: ucrtbased!register_onexit_function+0x140 (009d6c40) 
		  Priority: 0  Priority class: 32  Affinity: f
4). ~5kb 切换到5号线程(或者 ~4s;kv;)观察该线程是否也有锁等待,如果有,继续2)操作,观察该线程等待的锁属于哪个线程,如果线程和1)线程一样,即存在死锁
5). !locks 查看当前所有处于锁定状态的锁

9.输出到log
1). .logopen D:\xxx\Desktop\dump\chunksvr\dbglog.txt
2). kb 当前线程堆栈
3). $<D:\xxx\Desktop\dump\chunksvr\command.txt 执行command.txt中的所有命令
3). .logclose

10.以结构解析
dt CSocketBuffer 0x000001ab
db 0x000001ab L0x18 bytes形式打印内存0x000001ab长度为0x18

11.gflags开启页内存堆检测
gflags.exe –p /enable HttpServer.exe /full
gflags.exe -p /disable HttpServer.exe