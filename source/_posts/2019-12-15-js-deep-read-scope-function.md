---
title: Javascipt-深入理解闭包
date: 2019-09-06 19:49:05
tags: js
categories: 前端
comments: true
toc: true
---

网上关于闭包的文章很多，各种专业文献上的“闭包”定义非常抽象，很难看懂。官方对闭包的解释是：一个拥有许多变量和绑定了这些变量的环境的表达式，还是看不懂？
不着急，引用《JavaScript权威指南》英文原版对闭包的定义如下:
<!--more-->

***This combination of a function object and a scope (a set of variable bindings) in which the function’s variables are resolved is called a closure in the computer science literature.***

个人翻译为：**闭包指的是一个函数和该函数能访问和引用的所有变量对象（也叫作用域对象）的集合**。
我觉的也可以这么理解：一个闭包就是当一个函数被其包含作用域外部引用时，一个没有释放资源的栈区，栈区包含该函数本身和函数所引用的作用域链上的变量对象。
所以，在Javascript中，从不同角度看有两种形式：
1）从理论角度看，所有的函数都是闭包，包括全局函数，因为他们都有自己的变量对象和作用域链，访问全局变量就相当于访问最外层的作用域对象
2）从实践角度看，闭包函数一般有以下特点的：创建它们的上下文（比如父函数执行环境）已经销毁，它们仍然存在并被引用，而且还引用了其作用域链里其他作用域对象的变量。 
最简单的就像这样：
```javascript
function F_func() {
  var num = 42;
  var S_func = functio() {
	return ++num;
  }
  return S_func;
}
var add = F_func();
```
当一个或多个内部函数被包含函数返回，而内部函数引用了其他作用域的变量，当内部函数在包含函数之外被调用，我们就可以称它们构成了闭包。

# 一、闭包的原理
1、外部函数在调用时会创建自身的执行环境、变量对象和作用域链，调用结束后这些回收，但是函数的变量对象若有变量被闭包函数调用，则基于Js的垃圾回收机制，函数调用结束后其执行环境、作用域链将销毁，变量对象却不会立即回收，而是在闭包函数的调用链里；
2、每调用一次外部函数就会创建自身的执行环境、变量对象等操作，便产生一个新的闭包，以前的闭包依旧存在且互不影响；
3、同一个闭包会保留上一次的状态，当它被再次调用时会在同一作用域链上进行；
还是用上面的例子说明
```javascript
var name = 'scope';
function F_func() {
  var num = 42;
  function S_func() {
    var t = 1;
	num += t;
	return num;
  }
  return S_func;
}
var a = F_func();
a(); // 43
a(); // 44
var b = F_func();  // 重新调用F_func()，产生新的闭包
b(); // 43  
a = null;   // 通知回收
b(); // 44 
```
大概的执行过程(其中作用链以指针关联,vo-指代变量对象)：
> - 作用域链的各个变量对象以指针关联
> - vo-指代变量对象

1）进入全局上下文，创建全局执行环境、全局变量对象和作用域链，入栈
```javascript
   //对应作用域链
   global ----- global-vo {
                    name: 'scope',
                    F_func: Function,
                    a: undefined,
                    b: undefined
                }
```
2）第一次执行F_func函数，创建F_func执行环境、变量对象和作用域链，入栈
3）执行F_func函数，初始化上下文、变量对象和内部函数的作用域链
```javascript
   //对应作用域链
   F_func ----- F_func-vo { ----- global-vo {
                    num: 42,            name: 'scope',
                    S_func              F_func,
                }                       a: Function,
                    |                   b: undefined
                    |               }
                    |			 
   S_func ----- S_func-vo { 
                    t: 1      
                } 
```
4）F_func函数执行完毕，变量a引用返回的内部函数S_func，F_func的执行环境和作用域链出栈并销毁
```javascript
   //对应作用域链
                F_func-vo { ----- global-vo {
                    num: 42,        name: 'scope',
                    S_func          F_func,
                }                   a: Function,
                    |               b: undefined
                    |           }
                    |			 
   a=S_func ----- S_func-vo { 
                    t: 1 
                }
```
5）再次调用a函数，后续每次调用都在这个作用域链里进行，将会继承变量对象里变量的变化
6）第二次执行F_func函数，重新创建F_func执行环境...重复上述过程，b也将获得一个自己的执行环境、变量对象和作用域链，同a是一样的
```javascript
   //对应作用域链
                F_func-vo { ----- global-vo {
                    num: 42,            name: 'scope',
                    S_func              F_func,
                }                       a: Function,
                    |                   b: undefined
                    |               }
                    |			 
   b=S_func ----- S_func-vo { 
                    t: 1 
                }
```
7）最后设置a为null，解除了a和内部函数的引用，垃圾回收机制下一次检查时将回收存储并清除内部函数的作用域链和变量对象
8）b不受影响，依然可以调用 
4、当外部函数中存在多个内部函数时，这些函数构成闭包引用的同一份父函数的变量对象，适合构造对象；
举个例子：
```javascript
function F_func() {
  var num = 42;
  return {
      fun1: function() { console.log(num); },
      fun2: function() { num++; },
      fun3: function() { num--; } 
  };
}
var obj = F_func();
obj.fun2();
obj.fun2();
obj.fun3();
obj.fun1(); // 43
```
3个匿名内部函数引用的都是F_func执行完生产的同一份变量对象

# 二、闭包的用途
闭包可以用在许多地方，主要有：

- 读取函数内部的变量
- 让这些变量的值始终保持在内存中。
- 模拟块级作用域变量，构造变量临时性死区，同时也能创建私有作用域，避免向全局作用域添加过多变量，减少与其他开发者产生命名冲突
示例：
```javascript
(function() {
	var num= 10;
	var add = function(n) {return n + num; }
	...
})();
```
- Js里变量是函数级作用域，函数便是变量的**私有作用域**，函数外不能直接访问，只能通过调用域链，可以封装私有属性和私有方法，并开发共有接口，一般用与面向对象的构造
示例：
```javascript
function Person(name) {
	var age = 18;
	getAge = function() {
		return age;
	};
	this.getName = function() {
		return name + getAge;
	};
}
var lucy = new Person('Lucy');
lucy.getName();  // Lucy18
```

# 三、注意事项
1）由闭包会使得函数中的变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题和内存泄露。解决方法是，在退出函数之前，将不使用的局部变量全部设置为null
2）闭包增加的变量查找的作用域链的长度，会在一定程度上影响查找速度

# 四、实例分析
**（1）修改前**
```javascript
function buildArr(arr) {
   var result = [];
   for (var i = 0; i < arr.length; i++) {
     var item = 'item' + i;
     result.push(function() {console.log(item + ' ' + arr[i])});
   }
   return result;
}
var fnlist = buildArr([1,2,3]);
fnlist[0]();  //  item2 undefined
fnlist[1]();  //  item2 undefined
fnlist[2]();  //  item2 undefined
```
外部函数buildArr执行完后几个fnlist匿名函数的作用域链为：
```javascript
 fnlist[n] {  -----  buildArr-vo {           -----   global-vo {
 }                       result:[Function],              buildArr: Function,
                         i: 3,                           fnlist: undefined
                         item: item2                 }
                         arr: [1,2,3]
                      }	
```
**（2）修改后**
```javascript
function buildArr(arr) {
   var result = [];
   for (var i = 0; i < arr.length; i++) {
      result.push((function(n) {
	      var item = 'item' + n;
          return function() {
            console.log(item + ' ' + arr[n]);
         }
     })(i));
  }
  return result;
}
var fnlist = buildArr([1,2,3]);
fnlist[0]();  //  item0 1
fnlist[1]();  //  item1 2
fnlist[2]();  //  item2 3
```
外部函数buildArr执行完后几个fnlist匿名函数的作用域链为：
```javascript
fnlist[0] {  -----  匿名函数-vo {  -----  buildArr-vo {          ----- global-vo {
}                       n: 0                result: [Function],          buildArr: Function,
                        item: item0         i: 3,                        fnlist: undefined
                     }                      arr: [1,2,3]              }
                                          }	
                                                |
fnlist[1] {  -----  匿名函数-vo { ---------------- 
}                        n: 1                   |
                         item: item1            |
                     }                          |
fnlist[2] {  -----  匿名函数-vo {  ---------------
}                       n: 2                
                        item: item2         
                     }   						
````

# 五、实现闭包的方法
闭包主要依靠外部和内部函数来构造，而函数可以又有多种使用方式，比如：一般函数、对象方法等，所以可以很多实现闭包的方法，只要满足闭包的特点即可。
（1）作为普通函数使用
```javascript
1.1 最简单闭包：
var myFunc = (function() {
	var num = 10;
	return function() {
	  return num++;
	};
})();

1.2 返回对象或数组：也可叫模块模式或增强单例对象
var myArr = (function() {
	var num = 10;
	function add() {
	  return num++;
	};
	return [add];
})();
var myObj = (function() {
	var num = 10;
	function add() {
	  return num++;
	};
	return {
		add1: add,
		getNum: function() {return num;} // 公共接口
	};
})();

1.3 多级作用域链
var myObj = (function() {
	var num = 10;
	var arr = [];
	for (var i = 0;i < num;i++) {
		arr.push((function(n) {
			return function() { return n + 1;};
		})(i));
	}
	return {
		funs: arr
	};
})();

1.4 闭包函数以其他方式被引用
var $dom = document.querySelector("#id")
function() {
	var num = 10;
	$dom.onclick = function() {
		console.log(num);
	};
}
function() {
	var num = 10;
	setInterval(function() {
	  console.log(num);
	},2000);
}
function() {
	var num = 10;
	clickMe = function() { 
		console.log(num);
	};
}
```
（2）作为对象属性方法使用
```javascript
var circle = {  
   "PI":3.14159,  
   "area": function(r) {  
                 return this.PI * r * r;  
            }                
};  
console.log(circle.area(2));
```
（3）作为构造函数和面向对象
> 广泛用于面向对象编程方式中
```javascript
3.1 实例方法
function Person(name) {
	var age = 18;
	getAge = function() {
		return age;
	};
	this.getName = function() {
		return name + getAge;
	};
}
var lucy = new Person('Lucy');
lucy.getName();  // Lucy18

3.2 静态方法和变量，静态共享
(function() {
	var name = 'lilei';
	var age = 18;
	Person = function(name) {
		name = name;
	};
	Person.prototype.getName = function() {
	   return name + age;
	};
})();
var lucy = new Person('Lucy');
lucy.getName();  // Lucy18
var lilei = new Person('Lilei');
lucy.getName();   // Lilei18
lilei.getName();  // Lilei18

3.3 原始对象
var Circle = new Object();  
Circle.PI = 3.14159;  
Circle.Area = function( r ) {  
       return this.PI * r * r;  
}  
alert( Circle.Area( 1.0 ) );

3.3 工厂模式生产对象
var Circle = function() {  
   var obj = new Object();  
   obj.PI = 3.14159;  

   obj.area = function( r ) {  
       return this.PI * r * r;  
   }  
   return obj;  
}          
var c = new Circle();  
alert( c.area( 1.0 ) );

3.3 原型模式生产对象
function Circle(r) {  
      this.r = r;  
}  
Circle.PI = 3.14159;  
Circle.prototype.area = function() {  
  return Circle.PI * this.r * this.r;  
}  
var c = new Circle(1.0);     
alert(c.area());

其他...
```




