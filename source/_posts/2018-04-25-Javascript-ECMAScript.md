---
title: Javascript-ECMAScript基础
date: 2018-04-25 00:08:35
tags: js
categories: 前端
comments: true
---

ECMAscript是Javascript的一部分，主要规定脚本的语法相关集合，和其他编程语言一样具有类型,运算符等组成部分
<!--more-->

### 1 数据类型
js的变量是动态的，没有其他语言严格的变量类型规定，也就是变量可以被赋予任何类型的变量对象
js有6大基本类型：

- String
- Boolean
- Number
- Null
- Undefined
- Function

可以使用typeOf来测试变量的基本类型，比如：

    typeOf "I love"  --> String
    typeOf 123  --> Number
    typeOf () => {}  --> Function

### 运算符
js也有其他语言的类似运算符，也有特殊的情况，比如：

- == 和 !=,比较时会进行自动类型转换
- === 和 !==，全等比较，比较时不会转换类型，换言之就是值和类型一起参与比较

### 2 语句
js的语句借鉴C语言特点，大部分语句规则和C类似，也有特殊的语言标签，比如：for-in，label标签等

### 3 函数
js中的函数是对象，函数名是对象实例名，定义方式有好几个：

    function fun(x,y) { return x + y; }

    var fun = function(x,y) { return x + y; };

    var fun = (x,y) => { return x + y; };

    var fun = new Function('x','y','return x + y;');

既然函数是对象，则函数可以作为参数，变量进行赋值，甚至作为返回值，比如：

    var fun2 = function fun(x,y) {
        return (x,y) => { return 5 * x + y; };
    }

    fun2(2,3);
    或
    （(x,y) => {
        return (x,y) => { return 5 * x + y; };
     })(4,5);
     或
     （function() => {
         return 123;
      })();

### 4 对象
在js中，一切皆是对象，包括我们常用的数值，函数等，主要关系如下图：

![image-2018-03-02_160947](/images/2018-03-02_160947.jpg)

1.所有的对象都继承于object，所以其他子对象都具有object的公共方法，包括：toString(),valueOf()等
2.每种基本类型都对应一种引用型对象，比如：

    var a = 'hello world!'; //string - String

在定义变量表达式是js引擎自动在内存创建基本类型对应的引用类型变量，所以变量自然拥有对象的方法，因为当调用方法时js引擎会自动查找对象是否有可用的方法，当变量使用完后该对象自动销毁
3.基本类型对应的内置对象可以当作是通过“构造函数方式”利用Function/原型构建的，所以这些对象的原型constructor都是构造函数，比如:

    Array => function Array() { ... }
    Number => function Number() { ... }

4.基本类型和引用类型对象的检测方式不同，基本类型使用typeOf，引用类型使用instanceOf，比如：

    typeOf 123 => Number
    typeOf '123' => String
    typeOf [1,2,3] => object
    typeOf null => object
    typeOf undefined => Undefined
    typeOf ()=> {} => Function

    [1,2,3] instanceOf Array => true
    /[0-9a-z]*/gi instanceOf RegExp => true

5.既然对象都是继承于object,则可以使用object直接构造内置对象，比如：

    var str = new Object("123")
    str instanceOf String = true

    var num = new Object(123)
    num instanceOf Number = true

