---
layout: post
title: 设计模式之原型模式
categories: DesignPattern
description: Java设计模式总结之原型模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 创建型
### 原型模式（Prototype Pattern）
用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。
![image.png](https://upload-images.jianshu.io/upload_images/14607771-12cc595f3fd60072.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***优点***
提高性能

***缺点***
已有的类没有写相应的clone方法，较为麻烦

***使用场景***
资源优化场景；类初始化需要很多资源；性能和安全有要求的场景

***代码如下***
原型类
```
public abstract class Prototype implements Cloneable{
	
	private String id;
	
	public Prototype(String id) {
		this.id = id;
	}
	
	public String getId() {
		return this.id;
	}
	
	public Prototype clone() throws CloneNotSupportedException {
	
		return (Prototype) super.clone();
		
	}
	
}
```
具体原型类
```
public class CreateProtype extends Prototype{
	
	public CreateProtype(String id) {
		super(id);
	}
}
```
客户端代码
```
public class JavaDemo {
	
	public static void main(String[] args) throws CloneNotSupportedException {
		
		Prototype p1 = new CreateProtype("1");
		Prototype p2 = p1.clone();
		System.out.println(p1 == p2);
	}
	
}
```
输出结果为false，说明并不是传值引用 ，两个对象有自己的地址。

***注***
本文参考了菜鸟教程和书籍大话设计模式。
