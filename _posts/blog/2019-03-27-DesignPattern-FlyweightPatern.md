---
layout: post
title: 设计模式之享元模式
categories: DesignPattern
description: Java设计模式总结之享元模式
keywords: DesignPattern
---


## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 享元模式（Flyweight Pattern）
运用共享技术有效的支持大量细粒度的对象。

#### ***结构图***
![享元模式结构图](https://upload-images.jianshu.io/upload_images/14607771-f11d4bac711bf999.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### ***应用场景***
有大量相似对象存在的时候，通过享元模式共享。需要缓冲池的场景。String的设计。

#### ***优点***
减少了对象的创建，降低了系统的内存。

#### ***缺点***
提高了系统的复杂度，要分离出内部状态及外部状态，而且外部状态具有固有化的性质，不应该随着内部状态的变化而变化，否则会造成系统的混乱。

#### ***代码***
所有享元类的超类或者接口Flyweight 以及两个实现类ConcreteFlyweight和不需要共享的Flyweight的子类UnsharedConcreteFlyweight
```
public abstract class Flyweight {
    public abstract void Operation(int extrinsicstate);

}
public class ConcreteFlyweight extends Flyweight{
    @Override
    public void Operation(int extrinsicstate) {
        System.out.println("具体的Flyweight: " + extrinsicstate);
    }
}
public class UnsharedConcreteFlyweight extends Flyweight{
    @Override
    public void Operation(int extrinsicstate) {
        System.out.println("不共享的具体Flyweight: " + extrinsicstate);
    }
}
```
享元工厂，用来创建并管理Flyweight对象。
```
public class FlyweightFactory {
    private HashMap<String, Flyweight> flyweights = new HashMap<>();
    public FlyweightFactory(){
        flyweights.put("x", new ConcreteFlyweight());
        flyweights.put("y", new ConcreteFlyweight());
        flyweights.put("z", new ConcreteFlyweight());
    }

    public Flyweight getFlyweight(String key){
        return flyweights.get(key);
    }
}
```
主类及结果
```
public class JavaDemo {
    public static void main(String[] args){
        int extrinsicstate = 22; // 代码的外部状态
        FlyweightFactory flyweightFactory = new FlyweightFactory();

        Flyweight fx = flyweightFactory.getFlyweight("x");
        fx.Operation(--extrinsicstate);

        Flyweight fy = flyweightFactory.getFlyweight("y");
        fy.Operation(--extrinsicstate);

        Flyweight fz = flyweightFactory.getFlyweight("z");
        fz.Operation(--extrinsicstate);

        UnsharedConcreteFlyweight uf = new UnsharedConcreteFlyweight();
        uf.Operation(--extrinsicstate);
    }
}

具体的Flyweight: 21
具体的Flyweight: 20
具体的Flyweight: 19
不共享的具体Flyweight: 18
```
享元模式主要是避免创建大量相似类的开销，通过共享解决。程序设计中可能会存在大量细粒度的类实例来表示数据，如果能够发现这些实例中除了几个参数外基本都是相同的，就可以通过享元模式大幅度减少需要实例化类的数量。而参数的不同通过方法调用的时候将他们传进来。

#### ***参考资料***
本文参考了书籍《大话设计模式》和菜鸟教程设计模式相关部分资料