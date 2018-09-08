---
title: JSON-序列化和反序列化
date: 2018-07-28 16:35:13
tags: js
categories: 前端
comments: true
---

JSON,由于其语法简单，表达清晰等特点，在数据载体方面扮演这重要的角色。尤其在是javascript中，由于语法上与js对象相似，可方便的进行相互的转换
js提供了全局对象JSON进行数据结构的序列化和反序列化
<!--more-->
### 对象序列化
JSON.stringify()用于将对像转化成JSON格式的字符串

JSON.stringify(object,[filter],[space])

其中第一个参数为必选项的待转换对象；第二个参数是过滤器，可以是简单的key值数组，也可以是过滤处理的函数；第三个参数为制定缩进和换行，可以是缩进字符数目(最多10个)，也可以是填充字符串

    var book = {
        name: 'jack',
        age: 18,
        skills: ['talk','eat']
    }

    JSON.stringify(book);
    -->
    {name: 'jack',age: 18,skills: ['talk','eat']}

    JSON.stringify(book,['name','age']);
    -->
    {name: 'jack',age: 18}

    JSON.stringify(book,['name','age'],4);
    -->
    {
        name: 'jack',
        age: 18
    }

    JSON.stringify(book,['name','age'],'*');
    -->
    {
    ****name: 'jack',
    ****age: 18
    }

    JSON.stringify(book,function(key,value) {
        if(key === 'skills') {
            return value.join(&);
        } else {
            return value + '_1';
        }
    });
    -->
    {name: 'jack_1',age: 18_1,skills: 'talk&eat'}

> 注意，当对象里存在toJSON()实现方法是，调用stringify将会优先调用toJSON来返回结果；否则结果同上

### 对象反序列化
JSON.parse()用于将JSON格式的字符串转化成对像

JSON.parse(jsonstr,[filter])

其中第一个参数为必选项的待转换json字符串；第二个参数是过滤器，可以是处理的函数，当对于的key的处理返回是undefined，则在转换后对象中删除此字段

    var book = JSON.stringify({
        name: 'jack',
        age: 18,
        skills: ['talk','eat']
    });

    JSON.parse(book);
    -->得到一个和转换前结构一样的对象

    JSON.parse(book，function(key,value) {
        if(key === 'skills') {
            return undefined;
        } else {
            return value + '2';
        }
    });
    -->
    {
        name: 'jack2',
        age: 182,
    }

### 对象拷贝
利用对象的序列和反序列化的特点，可以简单的实现js对象的深拷贝

    var book = JSON.stringify({
        name: 'jack',
        age: 18,
        skills: ['talk','eat']
    });

    var bookclone = JSON.parse(JSON.stringify(book));