---
title:  Java并发编程的艺术-01-并发编程的挑战
description: Java并发编程的艺术-01-并发编程的挑战
...

**并发编程的目的是为了让程序运行的更快**

# 知识点汇总
- 并非启动更多线程程序并发就最大
- 多线程不一定就快，线程的创建和上下文切换需要成本
- 减少上下文切换
	- 无锁并发 - 通过ID将数据分开
	- CAS算法； 使用Java的Atomic包
	- 使用最少线程；避免创建不必要的线程导致太多线程等待

## 死锁示例
下面的示例一般不会出现在大家的代码中，只做展示。
> 很多面试会有考死锁，简单来说当线程同时需要获取多个锁就很容易出现死锁

```java
public class DeadLock {
	public static String A = "A";
	public static String B = "B";	
	public static void main(String[] args) {
		Thread t1 = new Thread(new Runnable() {			
			@Override
			public void run() {
				synchronized (A) {
					System.out.println("T1 Locked A");
					try {
						Thread.sleep(1000);
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					synchronized (B) {
						System.out.println("T1 Locked B");
					}
				}
			}
		});
		
		Thread t2 =new Thread(new Runnable() {
			
			@Override
			public void run() {
				synchronized (B) {
					System.out.println("T2 Locked B");
					synchronized (A) {
						System.out.println("T2 Locked A");
					}
				}
			}
		});
		
		
		t1.start();
		t2.start();
	}

}
```

## 避免死锁的出现
- 避免一个线程同时获取多个锁
- 

## 资源限制
当资源(cpu)的利用率已经很高，达到或者接近100%的时候，增加线程反而导致程序运行的更慢。
