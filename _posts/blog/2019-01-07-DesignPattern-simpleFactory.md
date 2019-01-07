---
layout: post
title: 设计模式之工厂模式
categories: DesignPattern
description: Java设计模式总结之工厂模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 创建型
### 工厂设计模式
工厂设计模式是Java中非常常见的设计模式之一。它提供了一种创建对象的最佳方式。在工厂设计模式中，我们不需要向客户端提供创建的逻辑，只需通过使用一个共同的接口来指向新创建的对象

***使用场景***
定义一个创建对象的接口，让它的子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。例如购买一件商品，不需要了解它是怎样被制作出来的，只需要通过超市进行购买。

***优点***
创建对象只需要直到名称；扩展性高，想增加产品仅需添加一个工厂类；屏蔽性好，调用者无需直到创建过程

***缺点***
添加产品需要添加工厂类，若数量过多则增加了系统的复杂度。

***代码如下***
```
//图形接口
public interface Shape {
	void draw();	
}
//圆形的实现类
public class Circle implements Shape {
    @Override
	public void draw() {
		System.out.println("我是圆形");
	}
}
//正方形实现类
public class Square implements Shape {
	@Override
	public void draw() {
		System.out.println("我是正方形");
	}
}
//工厂方法  提供getShape  根据调用者传入的名字进行相应的创建
public class ShapeFactory {
	
	public static Shape getShape(String shape) {
		
		if("circle".equals(shape)) 
			return new Circle();
		
		if("square".equals(shape)) 
			return new Square();
		
		
		return null;
	}
	
}
//主测试类  输出圆形
public class JavaDemo {
	
	public static void main(String[] args) {
		
		Shape shape = ShapeFactory.getShape("circle");
		shape.draw();
		
	}
	
}
```