---
layout: post
title: 设计模式之装饰者模式
categories: DesignPattern
description: Java设计模式总结之装饰者模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 装饰者模式（Decorator Pattern）
装饰者模式，动态地给一个对象添加一些额外的指责，就增加功能来说，装饰者模式比生成子类更为灵活。

***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-a9eee867e429cf7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***优点***
装饰类和非装饰类可以独立发展，不会相互耦合。

***缺点***
多层装饰比较复杂。

***使用场景***
拓展一个类的功能；动态增加、撤销功能。



***代码如下***
Person类（ConcreteComponent）
```
public class Person {
	
	private String name;
	
	public Person(String name) {
		this.name = name;
	}
	
	public Person() {}
	
	public void show() {
		System.out.println("装扮的"+name);
	}
}
```
装饰类Decorator
```
public class Decorator extends Person{

	protected Person person;
	
	public void Decorator(Person person) {
		this.person = person;
	}
	
	public Decorator() {}
	
	public void show() {
		if(person != null) {
			person.show();
		}
	}

}
```
具体的服饰类 Jeans和Tshirt
```
public class Jeans extends Decorator{
	
	public Jeans() {
	}
	@Override
	public void show() {
		System.out.println("牛仔裤");
		super.show();
	}
}
public class Tshirt extends Decorator{

	public Tshirt() {
	}
	@Override
	public void show() {
		System.out.println("T恤");
		super.show();
	}

}
```
测试Demo
```
public class TestDecorator {
	//装饰者模式测试类
	public static void main(String[] args) {
		
		Person person = new Person("jack");
		
		Tshirt t = new Tshirt();
		//t.show();
		Jeans j = new Jeans();
		t.Decorator(person);
		//t.show();
		j.Decorator(t);
		j.show();
	}
}
```
输出结果如下
```
牛仔裤
T恤
装扮的jack
```
***注意***
本文参考了书籍《大话设计模式》，和菜鸟教程设计模式相关部分资料

