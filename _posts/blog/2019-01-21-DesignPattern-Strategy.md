---
layout: post
title: 设计模式之策略模式
categories: DesignPattern
description: Java设计模式总结之策略模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 行为型
### 策略设计模式(Stragety)
定义了算法家族，分别封装，让它们之间可以相互替换，此模式让算法的变化不会影响到使用算法的客户。

***优点***
算法可以自由切换，避免了多重if判断，扩展性良好

***缺点***
策略类会增多，并且向外界暴露

***使用场景***
一个类需要在多种算法之间切换，例如商场卖商品，各种打折，八折，七折等等或者各种满减，此时促销算法类可以当成一个策略类。

***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-2ab4638ee2e1bc45.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***代码如下***
支付方法的父类
```
public abstract class CashSuper {
	public abstract double cash(double money);
}
```
两个子类
```
public class CashA extends CashSuper{
	@Override
	public double cash(double money) {
		System.out.println("没有打折:当前价格为："+money);
		return money;
	}
}
public class CashB extends CashSuper{
	private double account = 0;
	public CashB(double account) {
		this.account = account;
	}
	public double cash(double money) {
		System.out.println("当前折扣为："+account+" 折后价格为："+money*account);
		return money*account;
	}
}
```
使用策略类
```
public class StrategyContent {

	private CashSuper cashSuper;
	
	public StrategyContent(CashSuper cashSuper) {
		this.cashSuper = cashSuper;
	}
	
	public double contentInterface(double money) {
		return cashSuper.cash(money);
	}
	
}
```
测试类
```
public class StrategyDemo {
	
	public static void main(String[] args) {
		StrategyContent sc = new StrategyContent(new CashA());
		sc.contentInterface(100);
		StrategyContent sc2 = new StrategyContent(new CashB(0.7));
		sc2.contentInterface(200);
	}
}
```
输出结果如下
```
没有打折:当前价格为：100.0
当前折扣为：0.7 折后价格为：140.0
```

***注意***
本文参考了书籍《大话设计模式》，一个菜鸟教程设计模式相关部分资料

