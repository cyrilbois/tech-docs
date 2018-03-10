---
title:  Java并发编程的艺术 -02 Java并发机制的底层实现原理
description: Java并发机制的底层实现原理
...

读Java 并发编程的艺术第二章 - Java并发机制的底层实现原理

# volatile的应用
volatile是轻量级的synchronized, 不会引起线程上下文的切换。 
>volatile 只能保证其修饰变量的原子操作的数据一致，一些比如num++ 等都是复合操作，仅仅靠volatile修饰变量是无法保证并发情况下的数据一致的。

> synchronized 会引起上线文切换? 

以下示例暂时了非原子操作的时候，volatile无法保证数据
```java

public class RaceConditionFixingUsingVolatile {
	public  static void main(String[] args){
		Counter counter = new Counter();
		Thread t1 = new Thread(new Runnable() {			
			@Override
			public void run() {
				for(long i=0;i<1000L;i++){
					counter.decrease();
				}
			}
		});
		
		Thread t2 = new Thread(new Runnable() {			
			@Override
			public void run() {
				for(long i=0;i<1000L;i++){
					counter.increase();
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
		System.out.println(counter.getCount()); // 不会打印出 0 因为 count++ 和 count-- 不是原子操作
	}
}

class Counter {
	private volatile Long count = 0L;

	public void decrease() {
		count--;
	}

	public void increase() {
		count++;
	}

	public Long getCount() {
		return count;
	}
}


```



