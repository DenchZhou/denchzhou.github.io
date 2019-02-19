---
layout: post
title: 设计模式之观察者模式
categories: DesignPattern
description: Java设计模式总结之观察者模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 行为型
### 观察者模式（observer）

观察者模式又称作发布-订阅模式。观察者模式定义了一种多对多的依赖关系，让多个观察者对象同时监听某一个主题对象。当这个主体对象在状态发生变化时，会通知所有的观察者对象，使他们能够自动更新。

***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-3790a77aac12cf2c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### ***应用场景***
一个对象改变将导致其他一个或者多个对象发生改变，而不知道具体有多少个对象发生改变，降低耦合度；
一个对象必须通知其他对象，而并不知道这些对象是谁；

#### ***优点***
观察者和被观察者之间抽象耦合；建立了一套触发机制

#### ***缺点***
观察者和被观察者之间若是存在循环依赖，可能导致系统崩溃；如果一个被观察的对象同时也是一个观察者，这种情况下，如果直接或者间接关联过多可能会需要很多时间来通知。

#### ***代码***

被观察者的抽象类，其中用List保存所有观察该类的对象

```
public abstract class Subject {
    private List<Observer> observers = new ArrayList<Observer>();
    //增加观察者
    public void Attach(Observer observer){
        observers.add(observer);
    }
    //移除观察者
    public void Detach(Observer observer){
        observers.remove(observer);
    }
    //通知所有观察者更新状态
    public void notifyObserver(){
        for(Observer observer : observers){
            observer.update();
        }
    }
}
```
观察者抽象类，update方法即使更新最新状态
```
public abstract class Observer {

    public abstract void update();
}
```
被观察类的具体实例，有一个状态信息，当状态改变则通知所有观察者更新
```
public class ConcreteSubject extends Subject{
    private int state;
    public void setState(int state){
        this.state = state;
        super.notifyObserver();
    }
    public int getState(){
        return state;
    }

}
```
观察者的具体实现类，包含了自己所观察的一个对象，当update方法执行的时候，需要及时获取最新的state信息
```
public class ConcreteObserver extends Observer{
    private String name;
    private int state;
    private ConcreteSubject subject;

    public ConcreteObserver(ConcreteSubject subject, String name){
        this.name = name;
        this.subject = subject;
    }
    public void update() {
        state = subject.getState();
        System.out.println("观察者 "+name+" 的新状态是"+state);
    }
}
```
测试类
```
public class JavaDemo {
    public static void main(String[] args){
        ConcreteSubject s = new ConcreteSubject();
        s.Attach(new ConcreteObserver(s, "X"));
        s.Attach(new ConcreteObserver(s, "Y"));
        s.Attach(new ConcreteObserver(s, "Z"));
        //更改状态
        s.setState(2);
        s.setState(3);
    }
}
```
运行结果如下
```
观察者 X 的新状态是2
观察者 Y 的新状态是2
观察者 Z 的新状态是2
观察者 X 的新状态是3
观察者 Y 的新状态是3
观察者 Z 的新状态是3
```
#### ***注：***
本文参考了书籍《大话设计模式》和菜鸟教程设计模式相关部分资料
