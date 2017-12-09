title: Android.os.SystemClock 介绍
author: ccbu
tags: []
categories:
  - android
date: 2017-12-06 22:01:00
---

Android.os.SystemClock 介绍
----

**Class Overview**

Core timekeeping facilities.

Three different clocks are available, and they should not be confused:

- System.currentTimeMillis() is the standard "wall" clock (time and date) expressing milliseconds since the epoch. The wall clock can be set by the user or the phone network (see setCurrentTimeMillis(long)), so the time may jump backwards or forwards unpredictably. This clock should only be used when correspondence with real-world dates and times is important, such as in a calendar or alarm clock application. Interval or elapsed time measurements should use a different clock. If you are using System.currentTimeMillis(), consider listening to the ACTION_TIME_TICK, ACTION_TIME_CHANGED andACTION_TIMEZONE_CHANGED Intent broadcasts to find out when the time changes.  
该时间是基于世界时间的，它返回的是从January 1, 1970 00:00:00 UTC到现在时间已经逝去了多多少millisecond，当我设置Android手机的系统时间时，会应该影响该值。

- uptimeMillis() is counted in milliseconds since the system was booted. This clock stops when the system enters deep sleep (CPU off, display dark, device waiting for external input), but is not affected by clock scaling, idle, or other power saving mechanisms. This is the basis for most interval timing such asThread.sleep(millls), Object.wait(millis), and System.nanoTime(). This clock is guaranteed to be monotonic, and is the recommended basis for the general purpose interval timing of user interface events, performance measurements, and anything else that does not need to measure elapsed time during device sleep. Most methods that accept a timestamp value expect the uptimeMillis() clock.  
它表示的是手机从启动到现在的运行时间，且不包括系统sleep(CPU关闭)的时间，很多系统的内部时间都是基于此，比如Thread.sleep(millls), Object.wait(millis), and System.nanoTime()

- elapsedRealtime() is counted in milliseconds since the system was booted, including deep sleep. This clock should be used when measuring time intervals that may span periods of system sleep.  
它表示的是手机从启动到现在的运行时间，且包括系统sleep(CPU关闭)的时间

**There are several mechanisms for controlling the timing of events(有几种机制控制事件发生的时间):**
- Standard functions like Thread.sleep(millis) and Object.wait(millis) are always available. These functions use the uptimeMillis() clock; if the device enters sleep, the remainder of the time will be postponed until the device wakes up. These synchronous functions may be interrupted withThread.interrupt(), and you must handle InterruptedException.  
标准的方法像Thread.sleep(millis) 和 Object.wait(millis)总是可用的，这些方法使用的是uptimeMillis()时钟，如果设备进入深度休眠，剩余的时间将被推迟直到系统唤醒。这些同步方法可能被Thread.interrupt()中断，并且你必须处理InterruptedException异常。

- SystemClock.sleep(millis) is a utility function very similar to Thread.sleep(millis), but it ignores InterruptedException. Use this function for delays if you do not use Thread.interrupt(), as it will preserve the interrupted state of the thread.  
SystemClock.sleep(millis)是一个类似于Thread.sleep(millis)的实用方法，但是它忽略InterruptedException异常。使用该函数产生的延迟如果你不使用Thread.interrupt()，因为它会保存线程的中断状态。

- The Handler class can schedule asynchronous callbacks at an absolute or relative time. Handler objects also use the uptimeMillis() clock, and require anevent loop (normally present in any GUI application).  
Handler可以在一个相对或者绝对的时间设置异步回调，Handler类对象也使用uptimeMillis()时钟，而且需要一个loop(经常出现在GUI程序中)。

- The AlarmManager can trigger one-time or recurring events which occur even when the device is in deep sleep or your application is not running. Events may be scheduled with your choice of currentTimeMillis() (RTC) or elapsedRealtime() (ELAPSED_REALTIME), and cause an Intent broadcast when they occur.  
AlarmManager可以触发一次或重复事件，即使设备深度休眠或者应用程序没有运行。事件可以选择用 currentTimeMillis或者elapsedRealtime()(ELAPSED_REALTIME)来设置时间，当事件发生会触发一个广播。

**常用函数**

| return | methods | info |
| ---- | ---- | ---- |
|static long|currentThreadTimeMillis()|Returns milliseconds running in the current thread.|
|static long|elapsedRealtime()|Returns milliseconds since boot, including time spent in sleep.|
|static long|elapsedRealtimeNanos()|Returns nanoseconds since boot, including time spent in sleep.|
|static boolean|setCurrentTimeMillis(long millis)|Sets the current wall time, in milliseconds.|
|static void|sleep(long ms)|Waits a given number of milliseconds (of uptimeMillis) before returning.|
|static long|uptimeMillis()|Returns milliseconds since boot, not counting time spent in deep sleep.|

**注释：**

1、public static long currentThreadTimeMillis () 返在当前线程运行的毫秒数。

2、public static long elapsedRealtime () 返回系统启动到现在的毫秒数，包含休眠时间。

3、public static long elapsedRealtimeNanos () 返回系统启动到现在的纳秒数，包含休眠时间。

4、public static boolean setCurrentTimeMillis (long millis) 设置当前wall time，要求调用进程有许可权限。返回是否成功。

5、public static void sleep (long ms) 等待给定的时间。和Thread.sleep(millis)类似，但是它不会抛出InterruptedException异常。事件被推迟到下一个中断操作。该方法直到指定的时间过去才返回。

6、public static long uptimeMillis () 返回系统启动到现在的毫秒数，不包含休眠时间。就是说统计系统启动到现在的非休眠期时间。