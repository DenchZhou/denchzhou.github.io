---
layout: post
title: 设计模式之单例模式
categories: DesignPattern
description: Java设计模式总结之单例模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。

### 单例设计模式
某个类在被创建的过程中只允许存在一个实例，并且要提供一个访问到它的全局点。
优点：避免了过多的创建和销毁实例，造成了内存的额外开销；同时可以用在避免资源的多重占用。
#### 饿汉式单例模式
```
public class HunSingleton {
	private HunSingleton() {}
	private static HunSingleton singleton = new HunSingleton();
	
	public static HunSingleton getSingleton(){
		return singleton;
	}
}
```
首先，构造器私有保证无法被new出来，其次提前创建好一个对象，通过get方法返回。由于已经直接实例化，因此线程安全。
#### 懒汉式单例模式(非线程安全)
```
public class LazySingleton {
	
	private LazySingleton() {}
	
	private static LazySingleton lazySingleton;
	
	public static LazySingleton getSingleton() {
		
		if(lazySingleton == null) { //当两个线程同时走完这步的时候，可能会创建两个对象
			lazySingleton = new LazySingleton();
		}
		
		return lazySingleton;
	}
}
```
懒汉式相比于饿汉式，优点是可以延迟创建，需要的时候才创建实例对象，不过存在线程安全的问题。
#### 懒汉式单例模式（线程安全）
为了解决线程安全问题可以直接加上synchronized关键字，不过锁粒度的增大可能导致性能问题。不推荐使用
```
public class LazySingleton {
	private LazySingleton() {}	
	private static LazySingleton lazySingleton;
	
	public static synchronized LazySingleton getSingleton() {
		
		if(lazySingleton == null) {
			lazySingleton = new LazySingleton();
		}
		
		return lazySingleton;
	}
}
```
#### 双重锁校验（线程安全）

```
public class LazySingleton {
	
	private LazySingleton() {}
	
	private static volatile LazySingleton lazySingleton;
	
	public static LazySingleton getSingleton() {
		
		if(lazySingleton == null) {
			synchronized (LazySingleton.class) {
				if(lazySingleton == null) { 
					lazySingleton = new LazySingleton();
				}
			}
		}
		return lazySingleton;
	}
}
```
双重锁校验是基于懒汉式，通过减小锁的粒度来提高性能。第二次判断是预防两个线程a b同时通过了第一个判断，假设a线程获得了锁，创建了对象离开，b线程此时也获得了对象，进行判断，非空直接离开，避免了再次创建。
同时有一个需要注意的点，比较要有***volatile***关键字，原因是不加volatile关键字，线程a实例对象的时候可能存在之灵重排，而为完全实例成功的时候，线程b误以为已经实例完成而离开，此时调用将发生错误。关于指令重排可以看volatile相关知识
#### 静态内部类实现
```
public class StaticSingleton {
	
	private StaticSingleton() {}
	
	private static class SingletonHolder {
        private static final StaticSingleton INSTANCE = new StaticSingleton();
    }
	
	public static StaticSingleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
	
	
}
```
只有当调用getInstance()方法的时候，才会被加载。由于静态类只能被加载一次又保证了线程安全。该方法同时具有延迟加载和通过jvm保证线程安全的机制。
#### 枚举实现
以上所有单例模式都无法避免的一个问题是，通过反射，依然可以创建出两个相同的对象。对此，通过枚举来实现单例可以有效的避免这个问题。
```
public enum Singleton {
	INSTANCE;
}
```