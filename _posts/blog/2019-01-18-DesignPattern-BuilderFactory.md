---
layout: post
title: 设计模式之建造者模式
categories: DesignPattern
description: Java设计模式总结之建造者模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 创建型
### 建造者模式（Builder Pattern）
将一个复杂对象的创建与它的表示相分离，使同样的构造过程可以创建不同的表示。用户仅需指定需要建造的类型就可以得到，并不需要了解具体建造的过程和细节

***优点***
建造者易独立，易扩展。

***缺点***
产品需要有共同点；如果内部变化复杂，可能有很多建造类

***使用场景***
需要生成的对象具有复杂的内部结构。 需要生成的对象内部属性本身相互依赖。

***代码***
创建两接口，一个是商品品目item，一个是包装

```
//商品条目和包装的接口
public interface Item {
	public String name();
	public Packing packing();
	public float price();
}
//包装接口
public interface Packing {
	public String pack();
}
```
包装的两个实现类bottle和wrapper
item的两个抽象类burger和cloddrink
```
//包装实现类 botle
public class Bottle implements Packing{
	@Override
	public String pack() {
		return "bottle";
	}
}
//包装实现类 wrapper
public class Wrapper implements Packing{
	@Override
	public String pack() {
		return "Wrapper";
	}
}
//item接口的抽象类 burger
public abstract class Burger implements Item{
	public Packing packing() {
		return new Wrapper();
	}
	@Override
	public abstract float price();
}
//冷饮
public abstract class ColdDrink implements Item{
	@Override
	public Packing packing() {
		return new Bottle();
	}
	@Override
	public abstract float price();

}
```
冷饮和汉堡的具体实现类 Coke Pepsi ChickenBurger VegBurger
```
//冷饮实现类 可口可乐
public class Coke extends ColdDrink{
	@Override
	public String name() {
		return "Coke";
	}
	@Override
	public float price() {
		return 30.0f;
	}
}
//冷饮实现类 百事可乐
public class Pepsi extends ColdDrink{
	@Override
	public String name() {
		return "Pepsi";
	}
	@Override
	public float price() {
		return 35.0f;
	}
}
//鸡腿汉堡
public class ChickenBurger extends Burger{

	@Override
	public String name() {
		return "Chicken Burger";
	}

	@Override
	public float price() {
		return 50.0f;
	}
	
}
//蔬菜汉堡
public class VegBurger extends Burger{
	@Override
	public String name() {
		return "Veg Burger";
	}
	@Override
	public float price() {
		return 25.0f;
	}	
}
```
meal类组合具体的item
```
//meal类
public class Meal {
	
	private List<Item> items = new ArrayList<Item>();
	
	public void addItem(Item item) {
		items.add(item);
	}
	
	public float getCost() {
		float cost = 0.0f;
		for(Item item : items) {
			cost += item.price();
		}
		return cost;
	}
	
	public void showItems() {
		for(Item item : items) {
			System.out.println(item.name());
			System.out.println(item.packing().pack());
			System.out.println(item.price());
		}
	}
}
```

mealbuilder类，负责创建meal

```
public class MealBuilder {
	
	public Meal prepareVegMeal() {
		Meal meal = new Meal();
		meal.addItem(new VegBurger());
		meal.addItem(new Coke());
		return meal;
	}
	public Meal prepareNonVegMeal (){
       Meal meal = new Meal();
       meal.addItem(new ChickenBurger());
       meal.addItem(new Pepsi());
       return meal;
    }
}
```
测试demo
```
public class BuilderPatternDemo {
	
	 public static void main(String[] args) {
       MealBuilder mealBuilder = new MealBuilder();
 
       Meal vegMeal = mealBuilder.prepareVegMeal();
       System.out.println("Veg Meal");
       vegMeal.showItems();
       System.out.println("Total Cost: " +vegMeal.getCost());
 
       Meal nonVegMeal = mealBuilder.prepareNonVegMeal();
       System.out.println("\n\nNon-Veg Meal");
       nonVegMeal.showItems();
       System.out.println("Total Cost: " +nonVegMeal.getCost());
    }
}
```
程序输出
```
Veg Meal
Veg Burger
Wrapper
25.0
Coke
bottle
30.0
Total Cost: 55.0


Non-Veg Meal
Chicken Burger
Wrapper
50.0
Pepsi
bottle
35.0
Total Cost: 85.0
```

***注意***
建造者模式倾向于通过组件的组合构造出出一个对象，而工厂方法则通过整体构建。