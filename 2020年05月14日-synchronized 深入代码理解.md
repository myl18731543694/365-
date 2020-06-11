### synchronized 深入代码理解

```java
package 面试题.八锁;

import java.util.concurrent.TimeUnit;

class LockTest1 {

	public synchronized void put() {
		System.out.println(Thread.currentThread().getName() + "\tput方法 synchronized");

		try {
			TimeUnit.SECONDS.sleep(2); 
			System.out.println(Thread.currentThread().getName() + "\t线程结束 synchronized");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public synchronized void put1() {
		System.out.println(Thread.currentThread().getName() + "\tput1方法 synchronized");

		try {
			TimeUnit.SECONDS.sleep(2); 
			System.out.println(Thread.currentThread().getName() + "\t线程结束 synchronized");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static synchronized void put2() {
		System.out.println(Thread.currentThread().getName() + "\tput2方法 static synchronized");

		try {
			TimeUnit.SECONDS.sleep(2);
			System.out.println(Thread.currentThread().getName() + "\t线程结束  static synchronized");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static synchronized void put22() {
		System.out.println(Thread.currentThread().getName() + "\tput22方法 static synchronized");

		try {
			TimeUnit.SECONDS.sleep(2);
			System.out.println(Thread.currentThread().getName() + "\t线程结束  static synchronized");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public void put3() {
		System.out.println(Thread.currentThread().getName() + "\tput3方法 普通");

		try {
			TimeUnit.SECONDS.sleep(2);
			System.out.println(Thread.currentThread().getName() + "\t线程结束 普通");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

}

public class EightLockTest {
	public static void main(String[] args) {

		LockTest1 lockTest1 = new LockTest1();
		LockTest1 lockTest2 = new LockTest1();

		new Thread(() -> {
			lockTest1.put();
		}, "AAA").start();

		new Thread(() -> {
			lockTest2.put1();
		}, "BBB").start();
	}
	
	public static void lockProve9() {

		LockTest1 lockTest1 = new LockTest1();
		LockTest1 lockTest2 = new LockTest1();

		new Thread(() -> {
			lockTest1.put2();
		}, "AAA").start();

		new Thread(() -> {
			lockTest2.put22();
		}, "BBB").start();
	}
	
	public static void lockProve8() {

		LockTest1 lockTest1 = new LockTest1();

		new Thread(() -> {
			lockTest1.put();
		}, "AAA").start();

		new Thread(() -> {
			lockTest1.put1();
		}, "BBB").start();
	}
	
	public static void lockProve7() {

		LockTest1 lockTest1 = new LockTest1();

		new Thread(() -> {
			lockTest1.put();
		}, "AAA").start();

		new Thread(() -> {
			lockTest1.put3();
		}, "BBB").start();
	}
	
	public static void lockProve6() {

		LockTest1 lockTest1 = new LockTest1();
		LockTest1 lockTest2 = lockTest1;

		new Thread(() -> {
			lockTest1.put2();
		}, "AAA").start();

		new Thread(() -> {
			lockTest2.put();
		}, "BBB").start();
	}
	
	public static void lockProve5() {

		LockTest1 lockTest1 = new LockTest1();
		LockTest1 lockTest2 = new LockTest1();

		new Thread(() -> {
			lockTest1.put2();
		}, "AAA").start();

		new Thread(() -> {
			lockTest2.put2();
		}, "BBB").start();
	}
	
	public static void lockProve4() {

		LockTest1 lockTest1 = new LockTest1();

		new Thread(() -> {
			lockTest1.put2();
		}, "AAA").start();

		new Thread(() -> {
			lockTest1.put();
		}, "BBB").start();
	}

	public static void lockProve3() {

		LockTest1 lockTest1 = new LockTest1();
		LockTest1 lockTest2 = lockTest1;

		new Thread(() -> {
			lockTest1.put();
		}, "AAA").start();

		new Thread(() -> {
			lockTest2.put();
		}, "BBB").start();
	}

	public static void lockProve2() {
		LockTest1 lockTest1 = new LockTest1();
		LockTest1 lockTest2 = new LockTest1();

		new Thread(() -> {
			lockTest1.put();
		}, "AAA").start();

		new Thread(() -> {
			lockTest2.put();
		}, "BBB").start();
	}

	public static void lockProve1() {
		LockTest1 lockTest1 = new LockTest1();

		new Thread(() -> {
			lockTest1.put();
		}, "AAA").start();

		new Thread(() -> {
			lockTest1.put();
		}, "BBB").start();
	}
}

```

