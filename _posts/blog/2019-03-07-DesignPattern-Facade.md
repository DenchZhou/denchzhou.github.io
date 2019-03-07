---
layout: post
title: 设计模式之外观模式
categories: DesignPattern
description: Java设计模式总结之外观模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 外观模式（Facade Pattern）
外观模式又称为门面模式，为子系统中的一组接口提供一个一致的界面，此模式定义了一个高层接口，这个接口使得这个子系统更加容易使用。
#### ***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-0762c9f3c93c7080.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ***应用场景***
为复杂的模块或者子系统提供外界访问的模块。子系统相对独立的场景。预防低水平人员带来的风险。Java中的日志框架slf4j就使用了外观模式，利于维护和各个类的日志处理系统的统一。

#### ***优点***
减少系统相互依赖；提高灵活性；提高了安全性

#### ***缺点***
不符合开放封闭原则，改东西很麻烦，继承重写都不合适。

#### ***代码***
子系统类
```
public class SubSystemOne {
    public void methodOne(){
        System.out.println("子系统方法一");
    }
}
public class SubSystemTwo {
    public void methodTwo(){
        System.out.println("子系统方法二");
    }
}
public class SubSystemThree {
    public void methodThree(){
        System.out.println("子系统方法三");
    }
}
```
外观类
```
public class Facade {
    SubSystemOne subSystemOne;
    SubSystemTwo subSystemTwo;
    SubSystemThree subSystemThree;

    public Facade(){
        subSystemOne = new SubSystemOne();
        subSystemTwo = new SubSystemTwo();
        subSystemThree = new SubSystemThree();
    }

    public void methodA(){
        System.out.println("方法组 A()---");
        subSystemOne.methodOne();
        subSystemTwo.methodTwo();
        subSystemThree.methodThree();
    }

    public void methodB(){
        System.out.println("方法组 B()---");
        subSystemOne.methodOne();
        subSystemTwo.methodTwo();
    }

}
```
测试类
```
public class JavaDemo {
    public static void main(String[] args){
        Facade facade = new Facade();
        facade.methodA();
        facade.methodB();
    }
}
```
运行结果如下
```
方法组 A()---
子系统方法一
子系统方法二
子系统方法三
方法组 B()---
子系统方法一
子系统方法二
```
#### ***注：***
本文参考了书籍《大话设计模式》和菜鸟教程设计模式相关部分资料