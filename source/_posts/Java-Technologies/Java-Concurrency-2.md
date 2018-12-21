---
title:  Java并发编程的艺术 -02 Java并发机制的底层实现原理
description: Java并发机制的底层实现原理
...

读Java 并发编程的艺术第二章 - Java并发机制的底层实现原理

# volatile关键字
volatile是轻量级的synchronized, 不会引起线程上下文的切换。 
volatile 只能保证其修饰变量的原子操作的数据一致，一些比如num++ 等都是复合操作，仅仅靠volatile修饰变量是无法保证并发情况下的数据一致的; 因此volatile不能像synchronized那样普遍用于线程安全。

线程安全的场景，一般来说我们很容易想到类似购票系统这种场景(安全计数器), 但是由于上面介绍的原因，仅仅靠volatile关键字是做不到'安全计数'的。 关于具体使用场景可以参考：文章： http://blog.csdn.net/hxpjava1/article/details/55188908 。

> synchronized 会引起上线文切换? 

以下示例说明了非原子操作的时候，volatile无法保证数据
```java

public class NonAtomicOperationOnVolatile {
	static volatile int count = 0;

	public static void main(String[] args) {
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				for (int i = 0; i < 10000; i++) {
					count--;
				}
			}
		});

		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				for (int i = 0; i < 10000; i++) {
					count++;
				}
			}
		});

		t1.start();
		t2.start();
		try {
			t1.join();
			t2.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(count); //  不是 0
	}
}

```

# synchronized 关键字
synchronzied是偏重量级，但是在并发不是很大的情况下的优选实现方式。Java中的每一个对象都可以作为锁：
- 对于实例方法：锁是当前对象
- 对于静态方法：锁是当前类的Class 对象
- 对于同步方法块： 锁是synnchronized括号里的对象



