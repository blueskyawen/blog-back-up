---
title: Javascript-面向对象
date: 2018-05-05 19:30:58
tags: js
categories: 前端
comments: true
---

javascript是一门面向对象的脚本语言，但又于传统的面向对象语言不同，它没有类的概念  然而，在js中，对象被声明为一种无序属性的集合，属性值可以是普通数据，子对象，和方法对象，于是通过一些技术手段，能够在js中实现面向对象的编程特性
<!--more-->

## 1 对象
在ECMAScript里，对象的概念在我理解来说，可以类似于传统面向对象编程语言的“类”，而实例化出的多个实体叫对象实例
构建对象的方法很多，介绍几种常用的方法以及使用场景

#### 1.1 原始方式
通过原始对象Object直接构造,比较粗暴,实际上得到的是一个对象实例

    var animal = new Object();
    animal.type = "dog";
    animal.name = "wangcai";
    animal.display = function() {
        alert("I am a " + this.type + ",my name is " + this.name);
    };

    animal.display();
    //I am a dog，my name is wangcai

#### 1.2 字面量对象
同上，实际上得到的是一个对象实例，可用于简单的临时对象使用或放回值等

    var animal = {
        type: "dog",
        name: "wangcai",
        display: function() {
            alert("I am a " + this.type + ",my name is " + this.name);
        }
    };

    animal.display();
    //I am a dog，my name is wangcai

#### 1.3 工厂方法
简单工厂，封装对象构造过程

    function getAnimal(type,name) {
        var animal = new Object()
        animal.type = "dog";
        animal.name = "wangcai";
        animal.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
        };
        return animal;
    }

    var dog = getAnimal("dog","wangcai");
    dog.display();
    //I am a dog，my name is wangcai

#### 1.4 构造函数方法
在js中由于函数本身其实是对象，对象是属性集合，故可以使用函数作为“构造函数”来模拟对象创建

    function Animal(type,name) {
        this.type = type;
        this.name = name;
        this.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
        };
    }

    var dog = new Animal('dog','wangcai');
    dog.display();
    //I am a dog，my name is wangcai

这种方式特别简单，使用起来也比较像传统的面向对象类的用法，但缺点就是每个对象实例都会创建各自的方法对象，比较消耗内存，这是和类不一样的，我们知道类的函数是保存在堆上的，又实例共享

#### 1.5 原型方法
为了解决构造函数的内存问题，引入原型方式
在js中，每创建一个函数，会自动创建一个prototype属性指针，指向同时引入的函数的原型对象，这个原型对象保存在堆中，存储公共的属性和方法
当使用这个函数创建实例的时候会复制对象的prototype属性，所以每个对象实例都指向同一个原型对象

    function Animal() {}

    Animal.prototype.type = 'dog';
    Animal.prototype.eysNum = 2;
    Animal.prototype.name = 'wangcai';
    Animal.prototype.display = function() {
        alert("I am a " + this.type + ",my name is " + this.name);
    };

    var animal1 = new Animal();
    var animal2 = new Animal();
    animal1.display();
    animal2.display();
    //I am a dog，my name is wangcai
    //I am a dog，my name is wangcai

可以看到对象属性被实例共用，可以使用方法判断是否是某实例的原型或得到实例原型

    Animal1.prototype.isPrototypeOf(animal1); // true
    Object.getPrototypeOf(Animal1); // Animal1.prototype

js使用原型链搜索需要的属性或者方法，搜索顺序：**实例 --> 原型对象**
在实例中添加或重写原型的属性，可覆盖原型的同名变量，同上

    animal1.color = 'black';
    animal1.name = 'wangwang';
    console.log(animal1.color);  // black
    console.log(animal1.name);   // wangwang
    delete animal1.name;
    console.log(animal1.name);   // wangcai

    var animal3 = new Animal();
    console.log(animal3.color);  // undefined
    console.log(animal3.name);   // wangcai

- key in object和for-in可以判断和遍历对象实例的属性，不论属性在存在于实例还是原型中
- obj.hasOwnProperty(key)可检测属性是否在对象实例中
- obj.propertyIsEnumerable()判断属性是否可提高for-in遍历
- Object.keys()或者可遍历属性名

原型方式的缺点很明显，既然属性是实例共享的，当某一实例改变了属性值会立刻体现到所有实例上，尤其是对引用型变量的改变，这往往是不被希望的

#### 1.6 构造函数-原型混合
结合上面两个方式的优点，使用构造函数保存多变属性，使用原型保存共享属性/方法，混合方式也是使用最多的

    function Animal(type,name) {
        this.type = type;
        this.name = name;
        this.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
        };
    }
    Animal.prototype = {
        constructor: Animal,
        eyeNum: 2,
        dump: function() {
            console.log('I am '+ this.name + ', i have ' + this.eysNum + 'eyes.');
        }
    };

    var dog = new Animal('dog','wangcai');
    dog.display();
    dog.dump();
    // I am a dog，my name is wangcai
    // I am wangcai , i have 2 eyes.

    var cat = new Animal('cat','jiafei');
    cat.display();
    cat.dump();
    // I am a cat，my name is jiafei
    // I am jiafei , i have 2 eyes.


#### 1.7 工厂-构造函数混合
使用工厂方式，和装饰器模式来构造

    function Animal(type,name) {
        var animal = new Object()
        animal.type = "dog";
        animal.name = "wangcai";
        animal.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
        };
        return animal;
    }

    var dog = new Animal('dog','wangcai');


### 2 继承
在js中没有真正意义上的继承，主要依靠原型链的替换来实现“继承”的特性，再加上对象组合增强等设计方法，来优化实现

##### 2.1 原型链替换
这种方式其实很简单，想想之前说的原型链，对象和对象实例都通过prototype指针指向堆上的原型对象，所以通过指针实现原型链搜索来访问原型的属性
***如果把prototype指针替换成“父对象”的实例，这个子对象实例便可以通过自身的prototype访问“父对象实例”的属性，同时通过“父对象实例”的prototype访问“父对象”的原型属性，间接实现继承***
可以通过不断替换prototype属性多级继承

    function Animal(type,name) {
        this.type = type;
        this.name = name;
    }
    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
    };

    function Dog(color) {
        this.color = color;
        this.legNum = 4;
    }
    Dog.prototype = new Animal('dog','wangcai');

    var dog = new Dog('black');
    dog.display();
    // I am a dog，my name is wangcai

可以重写父对象的属性或添加新的属性，但记住要在prototype替换以后

    Dog.prototype.display = function() {
        console.log('HAHAHA');
    }
    Dog.prototype.dump = function() {
        console.log('my name is'+this.name+',my color is '+this.color);
    }

    var dog = new Dog('black');
    dog.display();
    // I am a dog，my name is wangcai
    dog.dump();
    // my name is wangcai,my color is black

当然父对象的方法也可使用子对的属性，比如

    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my color is " + this.color);
    };

在调用时会根据原型链来查找属性变量，先子对象 --> 父对象 --> ...,当然这不是好的设计方式，在此只是为了说明而已
与原型方式的对象方式一样，有着原型属性共享的问题，当其他继承者修改了“父对象”的属性时，将会反映在所有子类，即使设计时不是这样的

##### 2.2 仿冒子对象
利用函数的call()或apply()方法，在子对象的构造函数中直接调用执行“父对象”函数，并传入子对象的指针，间接使子对象继承了父对象的属性，且实例之间相互独立，类似于“组合模式”

    function Animal(type,name) {
        this.type = type;
        this.name = name;
        this.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
        };
    }

    function Dog(type,name,color) {
        Animal.call(this,type,name);
        this.color = color;
        this.legNum = 4;
    }
    Dog.prototype.dump = function() {
    	alert("I am " + this.name + ",color is " + this.color);
    }

    var dog = new Dog('dog','wangcai','black');
    dog.display();
    // I am a dog，my name is wangcai
    dog.dump();
    // I am wangcai,color is black

这种继承方式实际上是在子对像中实例化父对象的属性，上例就相当于直接在子对象添加父对象的属性，只不过是通过调用函数来实例化而已，与下面类似：

    function Dog(type,name,color) {
        //Animal
        this.type = type;
        this.name = name;
        this.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
        };
        //Animal

        this.color = color;
        this.legNum = 4;
    }
    Dog.prototype.dump = function() {
    	alert("I am " + this.name + ",color is " + this.color);
    }

可以看到，这种方式实际上是在子对像内部实例化“父类”的一个副本，增加内存负担，没有什么共享可言

##### 2.3 原型链-仿冒混合
和对象构建类似，也可以通过混合使用原型链和仿冒对象来获取各自的优点

    function Animal(type,name) {
        this.type = type;
        this.name = name;
    }
    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
    };

    function Dog(type,name,color) {
        Animal.call(this,type,name);
        this.color = color;
        this.legNum = 4;
    }
    Dog.prototype = new Animal();
    Dog.prototype.dump = function() {
    	alert("I am " + this.name + ",color is " + this.color);
    }

    var dog = new Dog('dog','KA','black');
    dog.display();
    //I am a dog,my name is KA
    dog.dump();
    //I am KA,color is black

##### 2.4 原型式
这个其实就是将原型链的实现进行“工厂化”，传入父对象，隐藏继承的实现

    function Animal(type,name) {
        this.type = type;
        this.name = name;
    }
    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
    };

    function SubAnimal(super_o,color) {
        function sub(color) {
         this.color = color;
	};
        sub.prototype = super_o;
        sub.prototype.dump = function() {
            alert(this.name + '@' + this.color);
        };
        return new sub(color);
    }

    var dog = SubAnimal(new Animal('dog','wangwang'),'BLUE');
    dog.display();
    // I am a dog，my name is wangwang
    dog.dump();
    // wangwang@BLUE

ECMAScript5引入了Object.create()方法进行原型式继承，如下

    function Animal(type,name) {
        this.type = type;
        this.name = name;
    }
    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
    };

    var dog = Object.create(new Animal('dog','wangwan22g'));
    dog.display();

上例中传入父对象来构建子对象dog,当只有着一个参数时其实没啥用，主要是第二个参数来为子对象添加属性，上面的例子还可以这样写：

    function Animal(type,name) {
        this.type = type;
        this.name = name;
    }
    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
    };

    var dog = Object.create(new Animal('dog','wangwang'),
                        {
                            color: {value: 'BLUE'},
                            dump: {
                                value: function() {
                                   alert(this.name + '@' + this.color);
                                }
                            }
                        });
    dog.display();
    dog.dump();
    // I am a dog，my name is wangwang
    // wangwang@BLUE

##### 2.5 寄生式
这种方式我的理解其实就是在原型式基础上进一步进行了工厂包装

    function Animal(type,name) {
        this.type = type;
        this.name = name;
    }
    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
    };

    function GetAnimal(super_o,color) {
        var sub = Object.create(super_o);
        sub.color = color;
        sub.dump = function() {
            alert(this.name + '@' + this.color);
        };
        return sub;
    }

    var dog = GetAnimal(new Animal('dog','wangwang'),'BLUE');
    dog.display();
    dog.dump();

##### 2.6 寄生组合式
寄生组合方式也是原型/寄生/构造函数等的组合应用，只是create子对象不是使用父对象实例，而是使用父对象的原型作为入参

    function GetAnimal(super_o,color) {
        var sub = Object.create(super_o.prototype);
        sub.color = color;
        sub.dump = function() {
            alert(this.name + '@' + this.color);
        };
        return sub;
    }