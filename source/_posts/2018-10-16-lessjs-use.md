---
title: less深入指南（译）
date: 2018-08-14 00:00:30
tags: Less
categories: 前端
comments: true
---

这是有关LESS语言功能的深入指南，有关Less的简单概念和语法，请参阅[less概述](https://less.bootcss.com/#)。
有关安装和设置Less环境的深入指南，以及有关Less开发的文档，可查看上一篇译文[less.js用法](http://blueskyawen.com/2018/08/10/less-common/)
<!--more-->

这是有关LESS语言功能的深入指南，有关Less的简单概念和语法，请参阅[less概述](https://less.bootcss.com/#)。
有关安装和设置Less环境的深入指南，以及有关Less开发的文档，请参阅上一篇译文[less.js用法](http://blueskyawen.com/2018/08/10/less-common/).

# 变量
使用单个变量控制常用值
### 概览
在样式表中看到相同的值重复数十甚至数百次，这种情况并不罕见，比如下面的颜色值：

    a,
    .link {
      color: #428bca;
    }
    .widget {
      color: #fff;
      background: #428bca;
    }

通过使用变量可以使您的代码更易于维护：

    // Variables
    @link-color:        #428bca; // sea blue
    @link-color-hover:  darken(@link-color, 10%);

    // Usage
    a,
    .link {
      color: @link-color;
    }
    a:hover {
      color: @link-color-hover;
    }
    .widget {
      color: #fff;
      background: @link-color;
    }

### 变量插值
上面的示例着重于使用变量来控制CSS规则中的值，但它们也可以在其他地方使用，例如选择器名称，属性名称，URL和@import语句，相当于其他语言里的一个**宏替代**。
**应用于选择器**


    // Variables
    @my-selector: banner;

    // Usage
    .@{my-selector} {
      font-weight: bold;
      line-height: 40px;
      margin: 0 auto;
    }

编译后：

    .banner {
      font-weight: bold;
      line-height: 40px;
      margin: 0 auto;
    }

**应用于属性**

    @property: color;

    .widget {
      @{property}: #0ee;
      background-@{property}: #999;
    }

编译后：

    .widget {
      color: #0ee;
      background-color: #999;
    }

**应用于URL**

    @images: "../img";

    // Usage
    body {
      color: #444;
      background: url("@{images}/white-sand.png");
    }

**应用于@import语句**
*请注意，在v2.0.0之前，只考虑在根或当前作用域中声明的变量，并且在查找变量时仅考虑当前文件和调用文件。*

    @themes: "../../src/themes";

    // Usage
    @import "@{themes}/tidal-wave.less";

### 变量的变量
在Less中，您可以使用变量来定义另一个变量的名称，进行间接替代

    @primary:  green;
    @secondary: blue;

    .section {
      @color: primary;

      .element {
        color: @@color;
      }
    }

编译后：

    .section .element {
      color: green;
    }

### 懒评估
在less中变量可以后向声明，即变量可以先使用，后声明

    .lazy-eval {
      width: @var;
    }

    @var: @a;
    @a: 9%;

编译后：

    .lazy-eval {
      width: 9%;
    }

当定义相同变量两次时，在一个作用域内使用变量的最后一个定义，这与css本身类似，其中定义中的最后一个属性用于确定最后的替代值，比如：

    @var: 0;
    .class {
      @var: 1;
      .brass {
        @var: 2;
        three: @var;
        @var: 3;
      }
      one: @var;
    }

编译后：

    .class {
      one: 1;
    }
    .class .brass {
      three: 3;
    }

实质上，每个作用域范围都有一个“最终”值，类似于浏览器中的属性，就像使用自定义属性的示例：

    .header {
      --color: white;
      color: var(--color);  // the color is black
      --color: black;
    }

这意味着，与其他CSS预处理语言不同，Less变量的行为与CSS非常相似。

### 属性即变量（新特性）
在最新的V3.0.0版本中，你可以直接把css属性名当作“变量”来使用，只要利用$符号，这样有时候可以使你的代码更加轻量化，比使用@事先定义变量要方便，例如：

    .widget {
      color: #efefef;
      background-color: $color;
    }

编译后：

    .widget {
      color: #efefef;
      background-color: #efefef;
    }

请注意，与变量一样，Less将选择当前/父范围内的最后一个属性作为“最终”值。

    .block {
      color: red;
      .inner {
        background-color: $color; //使用的blue
      }
      color: blue;
    }

### 默认变量
我们有时会收到默认变量的请求 - 只有在尚未设置变量时才能设置变量。 此功能不是必需的，因为您可以通过后面的定义轻松覆盖变量。

    // library
    @base-color: green;
    @dark-color: darken(@base-color, 10%);

    // use of library
    @import "library.less";
    @base-color: red;

其中，@base-color被重写了，最后生效的是red

# 父选择器
使用＆引用父选择器
＆运算符表示嵌套规则的父选择器，在将链接类或伪类应用于现有选择器时最常用：

    a {
      color: blue;
      &:hover {
        color: green;
      }
      &.b {
        color: black;
      }
    }

编译后：

    a {
      color: blue;
    }

    a:hover {
      color: green;
    }
    a.b {
        color: black;
    }

可以看到，＆就是代表父选择器a,编译后作了替换链接
“父选择器”运算符具有多种用途，本上，只要您需要嵌套规则的选择器以默认的其他方式组合。 例如，＆的另一个典型用法是产生重复的类名：

    .button {
      &-ok {
        background-image: url("ok.png");
      }
      &-cancel {
        background-image: url("cancel.png");
      }

      &-custom {
        background-image: url("custom.png");
      }
    }

编译后：

    .button-ok {
      background-image: url("ok.png");
    }
    .button-cancel {
      background-image: url("cancel.png");
    }
    .button-custom {
      background-image: url("custom.png");
    }

### 多次使用&
＆可能会在选择器中出现多次。，这使得可以重复引用父选择器而不重复其名称。

    .link {
      & + & {
        color: red;
      }
      & & {
        color: green;
      }
      && {
        color: blue;
      }
      &, &ish {
        color: cyan;
      }
    }

编译后：

    .link + .link {
      color: red;
    }
    .link .link {
      color: green;
    }
    .link.link {
      color: blue;
    }
    .link, .linkish {
      color: cyan;
    }

请注意，＆表示的多层嵌套的父选择器（不仅仅是最近的祖先），因此以下示例：

    .grand {
      .parent {
        & > & {
          color: red;
        }
        & & {
          color: green;
        }
        && {
          color: blue;
        }
        &, &ish {
          color: cyan;
        }
      }
    }

输出：

    .grand .parent > .grand .parent {
      color: red;
    }
    .grand .parent .grand .parent {
      color: green;
    }
    .grand .parent.grand .parent {
      color: blue;
    }
    .grand .parent,
    .grand .parentish {
      color: cyan;
    }

### 更改选择器顺序
将选择器添加到继承的（父）选择器可能很有用。 这可以通过放置＆after当前选择器来完成。 例如，使用Modernizr时，您可能希望根据支持的功能指定不同的规则：

    .header {
      .menu {
        border-radius: 5px;
        .no-borderradius & {
          background-image: url('images/button-background.png');
        }
      }
    }

编译输出：

    .header .menu {
      border-radius: 5px;
    }
    .no-borderradius .header .menu {
      background-image: url('images/button-background.png');
    }

### 组合选择器
＆还可用于生成逗号分隔列表中每个可能的选择器排列：

    p, a, ul, li {
      & + & {
        border-top: 0;
      }
    }

编译输出：

    p + p,
    p + a,
    p + ul,
    p + li,
    a + p,
    a + a,
    a + ul,
    a + li,
    ul + p,
    ul + a,
    ul + ul,
    ul + li,
    li + p,
    li + a,
    li + ul,
    li + li {
      border-top: 0;
    }

# Extend伪类
> 发布v1.4.0

Extend是一个Less伪类，它将放置的选择器与它引用的选择器相匹配。

    nav ul {
      &:extend(.inline);
      background: blue;
    }
    .inline {
      color: red;
    }

在上面的规则集中，:extend选择器将“扩展选择器”（nav ul）应用到.inline类，即将.inline的css属性扩展到父选择器“nav ul”。
编译输出：

    nav ul {
      background: blue;
    }
    .inline,
    nav ul {
      color: red;
    }

注意，nav ul:extend(.inline）选择器如何作为nav ul输出 - 在输出之前删除extend并且选择器块保持原样。 如果没有属性放入该块，则它将从输出中删除（但扩展仍可能影响其他选择器）

### 扩展语法
extend可以附加到选择器，也可以放在规则集中。

    .a:extend(.b) {}

    .a {
      &:extend(.b);
    }
    // the above blocks does the same thing

它看起来像一个带有selector参数的伪类，后面可以跟可选的关键字all：

    .c:extend(.d all) {
      // extends all instances of ".d" e.g. ".x.d" or ".d.x"
    }
    .d { color: red; }
    .d.classX { font-size: 12px; }
    .classY.d { font-weight: 800; }

    //输出
    .c,.d { color: red; }
    .c,.d.classX { font-size: 12px; }
    .c,.classY.d { font-weight: 800; }
    //从而合成的.c
    .c {
        color: red;
        font-size: 12px;
        font-weight: 800;
    }

它可以包含一个或多个要扩展的类，用逗号分隔,下面的例子实现的效果相同

    .e:extend(.f) {}
    .e:extend(.g) {}

    // the above an the below do the same thing
    .e:extend(.f, .g) {}

### 扩展附加到选择器



