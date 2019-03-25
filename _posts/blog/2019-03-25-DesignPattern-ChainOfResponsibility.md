---
layout: post
title: 设计模式之职责链模式
categories: DesignPattern
description: Java设计模式总结之职责链模式
keywords: DesignPattern
---


## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 职责链模式（Chain of Responsibility）
使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

#### ***结构图***
![职责链模式结构图](https://upload-images.jianshu.io/upload_images/14607771-4c49141c08b02f02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ***应用场景***
可以让多个对象处理同一个请求，实现职责的分离，实现单一职能的原则；可以动态指定一组对象处理请求；可以在不明确指定接收者的情况下，向多个对象中的一个提交一个请求

#### ***优点***
降低耦合度，请求者和发送者解耦；简化了对象，单一对象只需要负责自己的职责；增强了给对象指派职责的灵活性，可以灵活地对职责链进行增加或者删除。

#### ***缺点***
不能保证请求一定被接收；系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用；可能不容易观察运行时的特征，有碍于除错。

#### ***代码***
处理请示的接口
```
public abstract class Handler {

    protected Handler successor;

    public void setSuccessor(Handler successor){
        this.successor = successor;
    }

    public abstract void HandlerRequest(int request);
}
```
当请求在0到10处理该请求，否则转到下一个
```
public class ConcreteHandler1 extends Handler{
    @Override
    public void HandlerRequest(int request) {
        if (request >= 0 && request < 10){ // 0-10处理此请求
            System.out.println("ConcreteHandler1 处理请求" + request);
        }else if (successor != null){
            successor.HandlerRequest(request); // 转移到下一位
        }
    }
}
```
当请求在10到20处理该请求，否则转到下一个
```
public class ConcreteHandler2 extends Handler{
    @Override
    public void HandlerRequest(int request) {
        if (request >= 10 && request < 20){ // 10-20处理此请求
            System.out.println("ConcreteHandler2 处理请求" + request);
        }else if (successor != null){
            successor.HandlerRequest(request); // 转移到下一位
        }
    }
}
```
当请求在20以上处理该请求，否则转到下一个
```
public class ConcreteHandler3 extends Handler{

    @Override
    public void HandlerRequest(int request) {
        if (request >= 20){ // 20以上处理此请求
            System.out.println("ConcreteHandler3 处理请求" + request);
        }else if (successor != null){
            successor.HandlerRequest(request); // 转移到下一位
        }
    }
}
```
客户端方法，向链上的具体处理者对象提交请求
```
public class JavaDemo {
    public static void main(String[] args){
        Handler handler1 = new ConcreteHandler1();
        Handler handler2 = new ConcreteHandler2();
        Handler handler3 = new ConcreteHandler3();
        handler1.setSuccessor(handler2);  // 设置职责链的上下家
        handler2.setSuccessor(handler3);
        List<Integer> list = new ArrayList<>(Arrays.asList(2, 5, 14, 22, 18, 3, 27, 20));
        for (int i : list){
            handler1.HandlerRequest(i);
        }
    }
}
```
运行结果如下
```
ConcreteHandler1 处理请求2
ConcreteHandler1 处理请求5
ConcreteHandler2 处理请求14
ConcreteHandler3 处理请求22
ConcreteHandler2 处理请求18
ConcreteHandler1 处理请求3
ConcreteHandler3 处理请求27
ConcreteHandler3 处理请求20
```
#### ***参考资料***
本文参考了书籍《大话设计模式》和菜鸟教程设计模式相关部分资料