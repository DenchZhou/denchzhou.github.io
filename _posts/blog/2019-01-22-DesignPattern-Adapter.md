---
layout: post
title: 设计模式之适配器模式
categories: DesignPattern
description: Java设计模式总结之适配器模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 适配器设计模式（Adapter Pattern）
适配器模式，将一个类的接口转换成客户希望的另一个接口。适配器模式使得原本由于接口不兼容而不能一起工作的类可以一起工作。例如，各国使用的电压各不相同，有110V和220V等，但是我们的笔记本却可以直接使用。原因是电源适配器把电源转换成我们所需要的电压。

***优点***
可以让任何两个没有关联的类一起运行；提高了类的复用；增加了类的透明度；灵活性好

***缺点***
过多使用会导致系统变乱

***使用场景***
有动机的修改一个正常运行的系统接口，这时应该考虑适配器模式

***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-e8a33a40180a6e14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***代码如下***
目标类
```
public class Target {
	public void request() {
		System.out.println("普通请求");
	}
}
```
需要适配的类
```
public class Adaptee {
	public void SpecificRequest() {
		System.out.println("特殊请求！");
	}
}
```
适配器类
```
public class Adapter extends Target{ //继承目标类
	private Adaptee adaptee = new Adaptee();
	
	@Override
	public void request() {
		adaptee.SpecificRequest(); //实际调用适配类
	}
}
```
客户端调用
```
public class Client {
	public static void main(String[] args) {
		Target target = new Adapter();
		target.request();
	}
}
```
输出如下
```
特殊请求！
```
***注意***
本文参考了书籍《大话设计模式》，和菜鸟教程设计模式相关部分资料

