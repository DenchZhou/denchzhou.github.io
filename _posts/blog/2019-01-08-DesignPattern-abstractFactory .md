---
layout: post
title: 设计模式之抽象工厂
categories: DesignPattern
description: Java设计模式总结之抽象工厂模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 创建型
### 抽象工厂设计模式（Abstract Factory Pattern）
围绕一个超级工厂而创建其它工厂。

***应用场景***
当要系统有一类产品，并且系统只消耗其中一种产品。例如形状有形状的工厂，颜色有颜色的工厂。其中一个抽象的大工厂包含形状和颜色的两个工厂。

![image.png](https://upload-images.jianshu.io/upload_images/14607771-cece009fea9fa0fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***优点***
当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。

***缺点***
不容易扩展

***代码如下***
```
//图形接口及其实现
public interface Shape {
	void draw();
}
public class Circle implements Shape {
	@Override
	public void draw() {
		System.out.println("我是圆形");
	}
}
public class Square implements Shape {
	@Override
	public void draw() {
		System.out.println("我是正方形");
	}
}

//颜色接口及其实现
public interface Color {
	void getColor();
}
public class Blue implements Color {
	@Override
	public void getColor() {
		System.out.println("我是蓝色");
	}
}
public class Red implements Color {
	@Override
	public void getColor() {
		System.out.println("我是红色");
	}
}

//抽象工厂可以生产颜色或者图形
public abstract class AbstractFactory {
	public abstract Color getColor(String color);
	public abstract Shape getShape(String shape);
}
//图形类工厂
public class ShapeFactory extends AbstractFactory{	
	public Shape getShape(String shape) {	
		if("circle".equals(shape)) 
			return new Circle();	
		if("square".equals(shape)) 
			return new Square();
		return null;
	}

	@Override
	public Color getColor(String color) {
		return null;
	}
}
//颜色工厂
public class ColorFactory extends AbstractFactory{
	public Color getColor(String color) {
		if("red".equals(color)) {
			return new Red();
		}
		if("blue".equals(color)) {
			return new Blue();
		}
		return null;
	}

	@Override
	public Shape getShape(String shape) {
		return null;
	}
}
//工厂类生产者
public class FactoryProducer {
	
	public static AbstractFactory getFactory(String choice){
      if(choice.equalsIgnoreCase("shape")){
         return new ShapeFactory();
      } else if(choice.equalsIgnoreCase("color")){
         return new ColorFactory();
      }
      return null;
   }
}
```
