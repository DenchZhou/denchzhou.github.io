---
layout: post
title: 设计模式之模板方法模式
categories: DesignPattern
description: Java设计模式总结之模板方法模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 行为型
### 模板模式（Template Pattern）
模板方法模式，定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-f2f544f2cadaa447.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

***优点***
封装不变部分，扩展可变部分；提取公共代码，便于维护；行为由父类控制，子类实现

***缺点***
每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。

***使用场景***
重要，复杂的方法，例如框架的骨架；有多个子类共有方法，且逻辑相同。


***代码如下***
游戏抽象类，定义了三个抽象方法及结构图中的AbstractClass
```
public abstract class Game {
	abstract void initialize();
	abstract void startPlay();
	abstract void endPlay();
	
	//模板
	public final void play() {
		//初始化游戏
		initialize();
		
		//开始游戏
		startPlay();
		
		//结束游戏
		endPlay();
	}
}
```
具体的实现类ConcreateClass
```
public class Cricket extends Game{

	@Override
	void initialize() {
		System.out.println("Cricket Game Finished!");
	}

	@Override
	void startPlay() {
		System.out.println("Cricket Game Initialized! Start playing.");
	}

	@Override
	void endPlay() {
		System.out.println("Cricket Game Started. Enjoy the game!");
	}
	
}
```
主要测试类
```
public class JavaDemo {
	
	public static void main(String[] args) {
		Game game = new Cricket();
		game.play();
	}
	
}
```
测试结果如下
```
Cricket Game Initialized! Start playing.
Cricket Game Started. Enjoy the game!
Cricket Game Finished!
```
***注意***
本文参考了书籍《大话设计模式》，和菜鸟教程设计模式相关部分资料
