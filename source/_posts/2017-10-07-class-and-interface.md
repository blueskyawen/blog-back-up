---
title: 类与接口
date: 2017-10-07 01:09:48
tags: Typescript
categories: 前端
comments: true
---

作为javascript的超集，typescript遵循了ES6标准，同时扩展了多方面的能力，而类和接口时Typescript两个十分重要的特性
<!--more-->

### 一. 类
Typescript支持基于类的面向对象编程，类作为面向对象具有三大特性：封装性，继承性，多态

1) 封装

类将属性和方法封装成独立的个体，作为模板供实例化使用

    class Car {

        public engine : string;
        private distance : number;
        protected price : number;

        constructor(engine : string,price : number) {
            this.distance = 0;
            this.engine = engine;
            this.price = price;
        }

        drive(distance :number) {
            this.distance = distance;
        }
    }
    let mycar = new Car('Aodi',200);
    mycar.drive(10);

2) 继承

基类可派生出多个子类，子类可继承基类的基本属性

    class Moto extends Car {

        constructor(engine : string,price : number) {
            super(engine,price)
        }
    }
    let myMoto = new Moto('Aodi',30);
    myMoto.drive(10);

3) 多态

    class Moto extends Car {

        constructor(engine : string,price : number) {
            super(engine,price)
        }

        drive(distance :number) {
            this.distance = distance;
        }
    }
    let myMoto ： Car = new Moto('Aodi',30);
    myMoto.drive(10); //调用子类方法

4) 抽象类继承

抽象类不能实例化，只提供子类继承，用于提炼声明一些公共的属性和方法

    abstract class basicCar {
        color : string;
        abstract drive(distance :number) : void;
    }
    class Car extends basicCar {
        drive() : void {
            ///...//
        }
    }

 ### 二. 接口

 接口在面向对象设计中用来声明可复用的模板单位，比如：属性接口，类接口等，同时接口也只是用来严格限制类型使用的，是设计模式的一种设计方式，比如使用属性接口定义的变量需要严格进行参数检测的，而继承类接口的class必须实现接口里的方法，否则将会检测报错

1) 属性接口

属性型接口可将一些字段集合在一些以供声明，类似于结构体，但使用属性接口定义的变量在赋值时需要进行接口检查器的检测，所赋予的值里必须存在接口声明中存在的字段且类型一致，否则报错，十分严格

    interface Name {
        firstName : string;
        secondName ?: string;
    }

    let name : Name = {firstName: 'Mical',secondName: 'Jordon'};
    let name2 : Name = {firstName: 'Mical'};
    let name3 : Name = {firstName: 'Mical',secondName: 'Jordon',age: 33};
    //ok

2) 函数接口

函数型接口主要声明严格类型的方法原型，包括入参的个数/类型以及返回值的类型，使用此接口定义的变量可接受与声明方式一致的函数，不可接受其他类型的函数赋值

    interface Function {
        (aa : string,bb: number): string
    }

    let myfun1 : Function = function(a : string,b : number) {
        return a + b;
    }
    myfun1('mybaby', 9); //ok

    let myfun1 : Function = function(a : string,b : string) {
        return a + b;
    } //No ok!!

3) 索引接口

索引型接口主要声明可又索引值来访问其中value值的结构性变量模板，包括数组和一般的接口

    //Array
    interface aRRAY {
        [aa : number]: string
    }
    let aa : aRRAY = ['11','ddd'];

     //interface
    interface aRRAY {
        [aa : string]: string
    }
    let bbb : aRRAY = ['name':'ssas'];

3) 类接口

类接口主要为了给引用的class来提供一些共有的属性和方法，本身并不实现这些方法，而class引用了它就必须实现这些，相当严格；相关类似于抽象类

    interface basic {
        name: string;
        eat() : void;
    }

    interface basic2 extends basic {
        walk() : void;
    }

    class IT implements basic2 {
        name : string;
        constructor(name : string) {
            this.name = name;
        }

        eat() : void {
         //eat
        }

        walk() : void {
         //walk
        }
    }

### 类和接口的区别

- 接口是为了数据声明而设计，拥有严格接口检测，而类是为了对象和数据封装
- 类需要实例化且调用构造函数，接口不需要，类接口也不需要
- 接口在使用上和类是本质不同的
- 类的继承使用extends，而引用接口使用implements
- 类可以实现自己的方法，类接口不行