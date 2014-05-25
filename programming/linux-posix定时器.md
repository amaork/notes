
# Linux POSIX定时器


## 一、概述

在设计软件的时候，有时需要在同一个软件中实现多个定时器，来完成不同的工作。Linux下有各种不同方案可以实现这个功能，比较常见的有以下几种：

1. 使用setitimer 发送不同的信号可以实现3个定时器
2. 使用小粒度的定时器，在定时器中通过软件计数器实现无限多的定时器
3. 使用多线程或多进程定时发送信号，如USER1，USER2信号，实现2个定时器
4. 使用POSI标准定时器timer_create, timer_seting


本文档将重点讲述最后一种实现方案，即使用POSIX标准定时器，实现软件中多个定时器共存的功能。

## 二、POSIX定时器实现


1. 在系统中创建一个定时器 

		int timer_create(clockid_t clockid, struct sigevent *restrict evp, timer_t *restrict timerid);
 
 		clockid 定时器参考的类型

		CLOCK_REALTIME :Systemwide realtime clock.
		CLOCK_MONOTONIC:Represents monotonic time. Cannot be set.
		CLOCK_PROCESS_CPUTIME_ID :High resolution per-process timer.
		CLOCK_THREAD_CPUTIME_ID :Thread-specific timer.
		CLOCK_REALTIME_HR :High resolution version of CLOCK_REALTIME.
		CLOCK_MONOTONIC_HR :High resolution version of CLOCK_MONOTONIC.
		 
		evp 要创建的定时器的属性

		union sigval
		{
			int sival_int; //integer value
			void *sival_ptr; //pointer value
		}

		struct sigevent
		{
			int sigev_notify; //notification type
			int sigev_signo; //signal number
			union sigval   sigev_value; //signal value
			void (*sigev_notify_function)(union sigval); 
			pthread_attr_t *sigev_notify_attributes;
		}
		

		SIGEV_SIGNAL:发送由evp->sigev_sino指定的信号到调用进程，evp->sigev_value的值将被作为siginfo_t结构体中 si_value的值。

		SIGEV_NONE：什么都不做，只提供通过timer_gettime和timer_getoverrun查询超时信息。

		SIGEV_THREAD: 以evp->sigev_notification_attributes为线程属性创建一个线程，在新建的线程内部以 evp->sigev_value为参数调用evp->sigev_notification_function。
timerid  创建成功返回定时器的ID

2. 设置定时器的时间

		int timer_settime(timer_t timerid, int flags,  const struct itimerspec *restrict value,struct itimerspec *restrict ovalue)

timer_settime负责启动或停止timer_create创建的定时器。参数flag说明定时器使用的是相对时间还是绝对时间。相对时间与 POSIX:XSI定时器使用的策略类似，而绝对时间的精确度更高。参数vaule指向的值来设置timerid指定的定时器。如果ovalue不为 NULL，timer_settime就将定时器以前的值放在ovalue指定的位置上。如果定时器正在运行，那么*ovalue的成员it_value 非零，并包含了定时器到期之前剩余的时间。

TIMER_ABSTIME表示绝对时间；如果flag没有设定为TIMER_ABSTIME,则定时器从调用开始在it_value内超时；如果 设定为TIMER_ABSTIME,该函数表现为时间直到下一次超时被设定为it_value指定的绝对时间和与timerid相联的时钟值的差值。定时 器的再装由value的it_interval成员值来设定。
value 中存放要设置的新的定时器的时间，如果为0则停止定时器。
ovalue 中保存设置之前的定时器的时间，可以设置为NULL保存
	