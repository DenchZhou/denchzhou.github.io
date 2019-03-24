---
layout: post
title: 设计模式之代理模式
categories: DesignPattern
description: Java设计模式总结之代理模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 代理模式（Proxy Pattern）
代理模式，为其他对象提供了一种代理以控制对这个对象的访问。
#### ***结构图***
![代理模式结构图](https://upload-images.jianshu.io/upload_images/14607771-53e91bb0a7a736fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ***应用场景***
当我们使用别人的代码的时候，可以通过代理，对方法增强，避免了修改别人的代码。Spring AOP。

#### ***优点***
职责清晰了；高扩展性；智能化

#### ***缺点***
由于多了一个代理对象，可能会使请求的处理速度变慢。有些代理的实现较为复杂。

#### ***代码***
代理类和真实类的公用接口
```
public interface Subject {
    void request();
}
```
真实类的请求
```
public class RealSubject implements Subject{
    @Override
    public void request() {
        System.out.println("真实请求");
    }
}
```
代理请求，引入了真实类对象，对方法进行了增强。
```
public class Proxy implements Subject{

    private RealSubject realSubject;

    @Override
    public void request() {
        if (realSubject == null) {
            realSubject = new RealSubject();
        }
        realSubject.request();
        System.out.println("代理请求");
    }
}
```
主要测试类
```
public class JavaDemo {

    public static void main(String[] args){
        Proxy proxy = new Proxy();
        proxy.request();
    }
}
```
#### ***注：***
本文参考了书籍《大话设计模式》和菜鸟教程设计模式相关部分资料
