---
layout: post
title: 设计模式之桥接模式
categories: DesignPattern
description: Java设计模式总结之桥接模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 桥接模式（Bridge Pattern）
桥接模式（Bridge），将抽象部分与它的实现分离，并不是说让抽象类与其派生类分离，而是用抽象类和它的派生类用来实现自己的对象。

***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-ab177d96a215f64a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



***优点***
抽象和实现的分离；优秀的扩展能力；实现细节对客户透明。

***缺点***
会增加系统的理解与难度

***使用场景***
不希望使用继承或因为多层次继承导致系统类的个数急剧增加的系统；一个类存在两个独立变化的维度，且这两个维度都需要进行扩展



***代码如下***
软件抽象类及相应的实现类
```
//软件类（Abstraction）
public abstract class Soft {

	public abstract void run();

}
//RefinedAbstraction
public class Game extends Soft{

	@Override
	public void run() {
		System.out.println("运行手机游戏");
	}
}
public class AddressList extends Soft{

	@Override
	public void run() {
		System.out.println("运行通讯录");
	}
}
```
//手机品牌抽象类（Implementor）
```
public abstract class PhoneBrand {
	
	public Soft soft;   //引用了软件类 类似桥梁，将两个抽象类连通
	
	public void setSoft(Soft soft) {
		this.soft = soft;
	}
	
	public abstract void run();
}
//品牌M
public class BrandM extends PhoneBrand{

	@Override
	public void run() {
		super.soft.run();
		System.out.println("品牌M");
	}
}
//品牌N
public class BrandN extends PhoneBrand{

	@Override
	public void run() {
		super.soft.run();
		System.out.println("品牌N");
	}
}
```
测试类
```
public class JavaDemo {
	public static void main(String[] args) {
		PhoneBrand brand1 = new BrandM();
		brand1.setSoft(new Game());
		brand1.run();
		
		brand1.setSoft(new AddressList());
		brand1.run();
		
		PhoneBrand brand2 = new BrandN();
		brand2.setSoft(new Game());
		brand2.run();
		
		brand2.setSoft(new AddressList());
		brand2.run();
		
		
	}
}
```
输出结果如下
```
运行手机游戏
品牌M
运行通讯录
品牌M
运行手机游戏
品牌N
运行通讯录
品牌N
```

***注意***
本文参考了书籍《大话设计模式》，和菜鸟教程设计模式相关部分资料

