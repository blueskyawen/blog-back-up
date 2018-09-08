---
title: javascipt-闭包/作用域和this
date: 2018-05-06 20:00:23
tags: js
categories: 前端
comments: true
---

在javascript中，有一些特有且容易出错的特性，包括闭包，作用域，和this,这里将个人理解记录一波
<!--more-->

### 1 闭包
当函数在执行时，会自动创建自身的执行环境，包括：作用域链和活动对象，然后执行时每遇到一个子函数就会形成新的执行环境，并推进执行环境栈里...，于是形成了执行环境链

- 作用域链，函数所能访问的作用范围，包含执行环境所能访问的所有活动对象，从外层环境到当前环境，在访问变量时将从当前执行环境的活动对象开始往外搜索变量
- 活动对象，保存当前执行环境的变量和方法

创建闭包的常用方法是在一个函数创建子函数，子函数可访问外层函数的变量，形成闭包

    function ageAdd(defalutAge) {
        var age = defalutAge;
        return function() {
            if(age < 10) {
                age += 5;
                return age;
            } else if(age < 20) {
                age += 2;
                return ++age;
            } else {
                return defalutAge++;
            }
        }
    }
    var funAdd = ageAdd(4)
    funAdd();  // 9
    funAdd();  // 14
    funAdd();  // 16
    funAdd();  // 18
    funAdd();  // 20
    funAdd();  // 4

当外层函数执行完以后，其执行环境和作用域链将会销毁，但是由于它的活动对象被子函数引用，故不会被销毁直到子函数执行完毕，于是形成了闭包
*常见的普通函数对全局变量的引用也算是一种特殊的闭包*
js没有块作用域，只有全局作用域和函数级作用域，但是，利用闭包可模拟块作用域，如下：

    (function() {
        块作用域...
    })()

例子

    var Age = (function() {
            var age = 10;
            return ++age;
    })();

    alert('age is : ' + Age);  // age is : 11
    alert('age is : ' + age);  // 出错

### 2 作用域
在传统的面向对象里，对象属性有公开/受保护/私有之分，但是在js中没有这些限制，js是动态脚本语言，对象额所以属性都是公开的，换句话说，在对象定义以外，可以随意访问对象属性和方法

    function Animal(type,name) {
        this.type = type;
        this.name = name;
    }
    Animal.prototype.display = function() {
            alert("I am a " + this.type + ",my name is " + this.name);
    };

    var dog = new Dog('black');
    //直接访问
    dog.display();
    alert(dog.name);

设计所致，没有办法，但是在设计中为了区分去私有还是共有属性，开发者之间形成了一个约定，对于私有属性的命名，两边加下划线或下划线开头，比如这样

    function Animal(type,name) {
        this._type_ = type;
        this._name = name;
    }

写法不能改变属性的特征，只是起到提示效果

> 当然可以使用闭包实对象的私有变量效果，但是没有实际意义

### 3 this指针
在javaScript中，在创建函数或对象(其实也是函数)时，会自动生成两个参数：arguments和this指针，由于this指针总是指向调用者，即函数执行时的“上下文”，所以this是动态的，需要具体分析
##### 3.1 作为普通函数调用

    function test() {
      alert(this);
    }
    test(); // Window

由于所有全局函数都属于window对象，所以上面的调用其实是:

    window.test();
    // 调用者是window

同理如下

    (function test() {
      alert(this);
    })();
    // Window

##### 3.2 作为对象方法调用
当函数作为对象方法定义时，在对象实例化时会自动创建作用域并把this指针指向该对象

    var animal = {
        name: 'wangcai',
        display: function() {
            alert(this.name);
        }
    };
    animal.display(); // wangcai

即使先创建的对象，后添加给对象或原型的方法也一样会绑定this给对象，比如：

    function test() {
      alert(this.name);
    }
    var animal= {
        name: 'wangcai'
    };
    animal.display = test;
    animal.display(); // wangcai

**对于构造函数方式创建的对象来说**，在实例化对象的时候在除了携带入参数组初始化内部属性以外，还将绑定this指针，即使后面再添加方法也一样

    function Animal(name,age) {
            this.name = name;
            this.age = age;
            this.getName = function() {
                alert(this.name);
            }
    }
    var jenny = new Person('wangwang',24);
    Person.prototype.getAge = function() {
        alert(this.age);
    };

    jenny.getName(); // wangwang
    jenny.getAge();  // 24

**对于多重对象来说，**this将匹配直接包含本方法的对象，比如：

    function Animal(name,age) {
        this.name = name;
        this.age = age;
        this.display = function() {
            alert(this.name);
        }
        this.dog = {
            name: '汪汪',
            display: function() {
                alert(this.name);
            }
        }

    }
    var ani = new Animal('wangcai',24);
    ani.display();  // wangcai
    ani.dog.display(); // 汪汪

但是有对象的方法里再调用外部函数时又有所不同，

    function FF() {
            alert(this);
    }

    function Animal(name,age) {
            this.name = name;
            this.age = age;
            this.display = function() {
                    alert(this.name);
                    FF();

            }
    }
    var ani = new Animal('wangcai',24);
    ani.display();
    // wangcai
    // Window

由于FF()是全局函数，执行的时候其实在使用的全局对象，即使是在对象方法调用的
**同理，在对象方法定义并执行内部函数意一样，比如：**

    function Animal(name,age) {
            this.name = name;
            this.age = age;
            this.display = function() {
                alert(this.name);
                (function() {
                    alert(this);
                })();

            }
    }
    var ani = new Animal('wangcai',24);
    ani.display();
    // wangcai
    // Window

和调用全局函数一样，这里实际关联的是全局对象上下文，但是这个普遍认为是脚本设计问题，可以通过保存this规避

    function Animal(name,age) {
            this.name = name;
            this.age = age;
            this.display = function() {
                    alert(this.name);
                    var that = this; //临时保存
                    (function() {
                            alert(that.name); //使用保存的this
                     })()

            }
    }
    var ani = new Animal('wangcai',24);
    ani.display();
    // wangcai
    // wangcai

**然而当把上面的对象方法赋值给全局变量来执行，又是完全不一样的结果：**

    function Animal(name,age) {
            this.name = name;
            this.age = age;
            this.display = function() {
                    alert(this);
                    var that = this; //临时保存
                    (function() {
                            alert(that); //使用保存的this
                     })()

            }
    }
    var ani = new Animal('wangcai',24);
    var dump = ani.display;
    dump();
    // Window
    // Window

这里其实就验证了一句话，this关联的调用者的上下文，虽然display方法是对象内部方法，但是方法是对象，对象名仅仅只是个指针，这里只是让全局变量指向函数代码而已，最终调用的是全局对象，实际上和下面没有任何区分

    var dump = function() {
                    alert(this);
                    var that = this; //临时保存
                    (function() {
                        alert(that); //使用保存的this
                    })()

                };

##### 3.3 对象/函数作为返回值的场景
当把对象或者函数作为函数返回时，有一些有趣的事情

    function Animal(name) {
       this.name = name;
       return {};
    }
    var ani = new Animal('汪汪');
    alert(ani.name); // undefined

    function Animal(name) {
       this.name = name;
       return function() {};
    }
    var ani = new Animal('汪汪');
    alert(ani.name); // undefined

    function Animal(name) {
       this.name = name;
       return 12;
    }
    var ani = new Animal('汪汪');
    alert(ani.name); // 汪汪

    function Animal(name) {
       this.name = name;
       return null; // 或undefined;
    }
    var ani = new Animal('汪汪');
    alert(ani.name); // 汪汪

可以看到，当函数返回空对象或空函数时，this指针发生了变化，指向了返回的对象/函数;但是当返回是变量，常量，甚至是null/undefined时，this无法进行指向切换，故还是指向的原来的对象，即new对象时创建的对象

##### 3.4 call,apply改变this指向
我们知道函数有两个特殊的方法：call(this,..),apply(this,[..]),函数恶意通过它们来执行
这两个方法的第一个参数都是this,即方法内部变量的指向，可以通过她们来人为改变函数执行的this指向
以全局函数为里例

    function test() {
      alert(this.name);
    }
    test(); // undefined

使用call改变

    var ani = {
        name: 'wangwang',
        age: 24,
        display: function() {alert(this.name);}
    }
    test.call(ani);
    // wangwang

> 在严格版中的默认的this不再是window，而是undefined