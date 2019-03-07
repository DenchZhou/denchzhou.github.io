---
layout: post
title: 设计模式之组合模式
categories: DesignPattern
description: Java设计模式总结之组合模式
keywords: DesignPattern
---

## 概述
设计模式（Design Pattern）是一套被反复使用、多数人知晓的、经过分类的、代码设计经验的总结。
使用设计模式的目的：为了代码可重用性、让代码更容易被他人理解、保证代码可靠性。 设计模式使代码编写真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。
设计模式可以分为三大类，分别是创建型、结构型和行为型。
## 结构型
### 组合模式（composite）
将对象组合成树形结构以表示“部分-整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。

#### ***结构图***
![image.png](https://upload-images.jianshu.io/upload_images/14607771-59267d3ae2d8e2f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### ***应用场景***
拥有部分和整体的场景，例如，文件和文件夹，树形菜单。

#### ***优点***
高层代码调用简单；节点可以自由增加

#### ***缺点***
在使用组合模式时，其叶子和树枝的声明都是实现类，而不是接口，违反了依赖倒置原则。

#### ***代码***
顶层接口
```
public abstract class Component {
    protected  String name;
    public Component(String name){
        this.name = name;
    }
    public abstract void add(Component c);
    public abstract void remove(Component c);
    public abstract void display(int depth);

}
```
叶子节点
```
public class Leaf extends Component{

    public Leaf(String name){
        super(name);
    }

    @Override
    public void add(Component c) {
        System.out.println("Cannot add a leaf");
    }

    @Override
    public void remove(Component c) {
        System.out.println("Cannot remove from a leaf");
    }

    @Override
    public void display(int depth) {
        String str = "";
        for (int i=0; i<depth; i++){
            str = str + "-";
        }
        System.out.println(str + name);
    }
}
```
节点 相当于枝干，可以存储更多节点
```
public class Composite extends Component{

    public Composite(String name){
        super(name);
    }
    private List<Component> children = new ArrayList<>();

    @Override
    public void add(Component c) {
        children.add(c);
    }

    @Override
    public void remove(Component c) {
        children.remove(c);
    }

    @Override
    public void display(int depth) {
        String str = "";
        for (int i=0; i<depth; i++){
            str = str + "-";
        }
        System.out.println(str + name);
        for (Component component : children){
            component.display(depth + 2);
        }
    }

```
主类测试
```
public class JavaDemo {
    public static void main(String[] args){
        Composite root = new Composite("root");
        root.add(new Leaf("Leaf A"));
        root.add(new Leaf("Leaf B"));

        Composite cmp = new Composite("Composite X");
        cmp.add(new Leaf("Leaf XA"));
        cmp.add(new Leaf("Leaf XB"));

        root.add(cmp);
        root.display(1);
    }
}
```
#### ***注：***
本文参考了书籍《大话设计模式》和菜鸟教程设计模式相关部分资料