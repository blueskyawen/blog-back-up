---
title: JQuery
date: 2018-07-27 22:23:04
tags: Jqury
categories: 前端
comments: true
---

JQurey是一个javascript库，用与简化开发人员的工作，感兴趣的同学可进一步去学习和使用
<!--more-->

### JQurey的特点

简化js操作文档的复杂度，写的更少，做的更多，这得益于jqurey的几大特点：

- 集合操作，大部分的元素操作都是集合操作，比如：$("p")就是p元素的集合
- 链式写法，对元素的操作无限叠加，典型的函数式编程
  $("p").addClass("active")...
- Dom操作和事件的混合，降低了代码复杂度
- 优秀的浏览器兼容性，js在操作时遇到了太多浏览器兼容性问题，一个操作有时候需要写一堆的代码，而jq很好的封装了这种差异性，提供了统一的接口

典型的Jquery语法：

    $("p").removeClass("BBB").filter(".AAA")
        .addClass("active")
        .click(function() {...});

### 关于$
$符号是JQurey的标志，所有的jq操作都已$开始，就连开发者对JQurey对象的命名也是以$开头，比如：

    var $volume = $("p");

$只是JQurey的代号，使用$的地方都可以使用JQurey代替，比如：

    $("p") 同 JQurey("p")
    $.ajax() 同 JQurey.ajax()

### JQurey对象和Dom对象
JQurey是对js操作的封装库，JQurey对象自然也是Dom对象的封装，两种可以相互转换
Dom对象 --> JQ对象：

    var cr=document.getElementByid("aa");
    var $cr = $(cr);

JQ对象 --> Dom对象：

    var $cr = $("#aa");
    var cr = $cr[0] 或 $cr.get(0)

JQ对象方法多样，可以完成大部分Dom操作，但是有时候也有判断不了的问题，比如:判断页面是否存在p元素，由于JQ总是会生成一个封装对象，故无法直接判断，这时候需要转化成dom对象

    var $cr = $("#aa");
    if($cr[0]) {
        alert('页面没有这个元素');
    }

### JQurey的核心操作
JQurey的核心操作，我觉得就是三个方面的内容：
- 元素获取
- 元素操作

###### 1 元素获取
主要通过选择器和各种遍历方法来获取元素的个体或集合，比如：

    $("p")
    $("div > span.classA")

    $("p").parent().find("span");
    $("div").children();

具体的选择器和DOM方法和javascript类似
**注意**

> Jquery对于父子标签组合的合理性有过检查，对于不合理的子标签查找不出

比如：我们div放进了table里

    <table>
        <div>
            <input type="text" />
        </div>
    </table>

使用$(”table div input“)无法匹配到input元素,但是$(”div input“)可以

###### 2 元素操作

（1）事件、动画的添加

    $("p").click(function() {
        alert("click it!!");
    });
    $("p").hover(function() {...},
                function() {...});

    $("p").fadeIn('slow',function() {...});
    $("p").animate({left: '200px'},'slow');

（2）Dom操作

**节点操作**,节点的get/del/add/update/copy

    $("div p")
    $("div p").remove();

    var newSpan = $("<span>newSpan</span>");
    $("div p").append(newSpan);

    $("div p").replacewith(newSpan);

    $("div p").clone();

**属性操作**，get/del/update/add

    $("div p").attr("title");
    $("div p").attr("title","I am title!!");
    $("div p").removeAttr("title");

**样式操作**，get/add/del

    $("div p").css("width");
    $("div p").css("font-weight","bolder");
    $("div p").css({
        "font-weight": "bolder",
        "color": "red"
    });

    $("div p").addClass("AAA");
    $("div p").removeClass("AAA");
    $("div p").toggleClass("AAA");
    $("div p").hasClass("AAA");

> 三元操作符很实用

    if(isActive) {
        $("p").addClass("AAA");
    } else {
        $("p").removeClass("AAA");
    }
    //使用三元操作符书写
    $("p")[isActive ? "addClass" : "removeClass"]("AAA");

**文本操作**，get/update

    $("div p").text();
    $("div p").text("javascript中的ajax书写复杂");
    $("div p").html();
    $("div p").html("<span>newSpan</span>");
    $("div input").val();
    $("div input#name").val("jack");

**标签包裹**，add

    $("p span").wrap("<div></div>");

### 3 Ajax
javascript中的ajax书写复杂，而且在不同浏览器之间有差异，开发者要详细考虑兼容各个浏览器的写法
在JQuery中，封装了这种差异，提供了统一的ajax方法，简单易用

**$().load()**
对象方法，load方法可根据url直接加载页面并添加到指定元素中，也可加载目标页面的部分元素

    $(selector).load(url,[data],[callback])

其中url为加载地址，data为可选入参数据，callback为可选加载完成的回调函数

    $("#target").load(url);
    $("#target").load("url selector");
    $("#target").load(url，{name: 'jack'},
        function(reponseText,textStatus,XMLHttpRequest) {
            ...
        });

使用load方法带与不带data，将对应调用get和post方法

**$.get()**
使用get方法可进行异步数据请求

    $.get(url,[data],[callback],[type])

其中type为请求的返回数据的规定格式，有：xml,html,script,json,text等，不填默认为xml

    $("button.send").click(function() {
        $.get(url,{name: 'jack'},function(data,textStatus) {
            ...
        });
     });
     //data响应数据，textStatus请求状态

    $("button.send").click(function() {
        $.get(url,{name: 'jack'},function(data,textStatus) {
            ...
        },"json");
     });

**$.post()**
使用post方法可向服务器发送数据

    $.post(url,[data],[callback])

使用post也能获取数据，且隐秘性更强，但一般用于数据写操作

    $("button.send").click(function() {
        $.post(url,{name: 'jack'},function(data,textStatus) {
            ...
        });
     });

**$.getScript()**
getScript方法用于动态加载js脚本，并且加载成功立即执行

    $.getScript("text.js");

**$.getJSON()**
getJSON方法用于动态JSON数据或文档，并且加载成功立通过回调函数处理

    $.getJSON("text.json",function(data) {
        ....
    });

**$.ajax()**
ajax()方法是jquery-ajax的核心方法，有很多的参数配置，可以设置错误处理，超时处理等，功能强大，前面几种方法都是基于此方法来实现的

    $.ajax({
        type: "GET",
        url: "temp.json"，
        dataType: "json",
        success: function(data) {
            ....
        }
    });

这是ajax()的一般用法，具体了解可参考jquery的官方文档

本文之作简单的导入介绍，了解更多可在网上资源学习，当然还需要一定的练习
附：使用jquery改造过的一些组件，之前有js书写
[jquery-组件改造](https://runjs.cn/code/igehlguh)