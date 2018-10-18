---
title: less深入指南一（译）
date: 2018-08-14 00:00:30
tags: Less
categories: 前端
comments: true
---

这是有关LESS语言功能的深入指南，有关Less的简单概念和语法，请参阅[less概述](https://less.bootcss.com/#)。
有关安装和设置Less环境的深入指南，以及有关Less开发的文档，可查看上一篇译文[less.js用法](http://blueskyawen.com/2018/08/10/less-common/)
<!--more-->

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
    .c.classX,.d.classX { font-size: 12px; }
    .classY.c,.classY.d { font-weight: 800; }

它可以包含一个或多个要扩展的类，用逗号分隔,下面的例子实现的效果相同

    .e:extend(.f) {}
    .e:extend(.g) {}

    // the above an the below do the same thing
    .e:extend(.f, .g) {}

### 扩展附加到选择器
附加到选择器的扩展看起来像普通的伪类，选择器作为参数。 选择器可以包含多个extend子句，但所有extends都必须位于选择器的末尾。

    pre:extend(div pre) { ... }
    同
    pre {
        ...
        &:extend(div pre)；
    }

- 在伪类后扩展,比如 pre:hover:extend(div pre)
- 允许选择器和扩展之间的空间,比如pre:hover :extend(div pre)
- 允许多个扩展,pre:hover:extend(div pre):extend(.bucket tr) - 注意这与下面表示效果相同: pre:hover:extend（div pre，.bucket tr）
- 下面这样是不允许的：pre：hover：extend（div pre）.nth-child（odd）,因为extend必须在最后。

如果规则集包含多个选择器，则其中任何一个都可以包含extend关键字,比如：

    .big-division,
    .big-bag:extend(.bag),
    .big-bucket:extend(.bucket) {
      // body
    }

### 扩展内部规则集
可以使用＆:extend（selector）语法将Extend放入规则集的正文中。 将extend放置到body中是将其放入该规则集的每个选择器的快捷方式。比如，下面两个例子的效果是一样的

    pre:hover, .some-class {
      &:extend(div pre);
    }

    pre:hover:extend(div pre),
    .some-class:extend(div pre) {}

### 扩展嵌套选择器
Extend能够匹配嵌套选择器,例如：

    .bucket {
      tr { // nested ruleset with target selector
        color: blue;
      }
    }
    .some-class:extend(.bucket tr) {} // nested ruleset is recognized

输出

    .bucket tr, .some-class {
      color: blue;
    }

本质上，扩展会查看已编译的css，而不是原始的less

    .bucket {
      tr & { // nested ruleset with target selector
        color: blue;
      }
    }
    .some-class:extend(tr .bucket) {} // nested ruleset is recognized

输出

    tr .bucket,
    .some-class {
      color: blue;
    }

### 与Extend完全匹配

默认情况下，扩展会查找选择器之间的完全匹配。选择器是否使用前导星比较重要。 两个第n个表达式具有相同的含义并不重要，它们需要具有相同的形式才能匹配。 唯一的例外是属性选择器中的引号，较少知道它们具有相同的含义并匹配它们。

    .a.class,
    .class.a,
    .class > .a {
      color: blue;
    }
    .test:extend(.class) {} // this will NOT match the any selectors above

    *.class {
      color: blue;
    }
    .noStar:extend(.class) {} // this will NOT match the *.class selector

伪类的顺序很重要。 选择器链接：hover：visited和link：visited：hover匹配相同的元素集，但extend将它们视为不同：

    link:hover:visited {
      color: blue;
    }
    .selector:extend(link:visited:hover) {} //NOT match link:hover:visite

### nth表达式
表示第N个表达形式很重要。 第N个表达式1n + 3和n+3是等价的，但是extend不匹配它们

    :nth-child(1n+3) {
      color: blue;
    }
    .child:extend(:nth-child(n+3)) {} //NOT match

属性选择器中的引用类型无关紧要，以下所有内容都是等效的。

    [title=identifier] {
      color: blue;
    }
    [title='identifier'] {
      color: blue;
    }
    [title="identifier"] {
      color: blue;
    }

    .noQuote:extend([title=identifier]) {}
    .singleQuote:extend([title='identifier']) {}
    .doubleQuote:extend([title="identifier"]) {}

输出

    [title=identifier],
    .noQuote,
    .singleQuote,
    .doubleQuote {
      color: blue;
    }

    [title='identifier'],
    .noQuote,
    .singleQuote,
    .doubleQuote {
      color: blue;
    }

    [title="identifier"],
    .noQuote,
    .singleQuote,
    .doubleQuote {
      color: blue;
    }

### 扩展所有all
当你在extend参数中指定all关键字时，它会告诉Less将该选择器与另一个选择器的一部分相匹配。 将复制选择器，然后仅使用extend替换选择器的匹配部分，从而生成新的选择器。

    .a.b.test,
    .test.c {
      color: orange;
    }
    .test {
      &:hover {
        color: green;
      }
    }

    .replacement:extend(.test all) {}

输出

    .a.b.test,
    .test.c,
    .a.b.replacement,
    .replacement.c {
      color: orange;
    }
    .test:hover,
    .replacement:hover {
      color: green;
    }

您可以将此操作模式视为基本上进行非破坏性搜索和替换。

### 具有扩展的选择器插值
Extend无法将选择器不能匹配变量，也不能匹配被变量代表的选择器。 比如下面的情况将不生效：

    @variable: .bucket;
    @{variable} { // interpolated selector
      color: blue;
    }
    .some-class:extend(.bucket) {} // does nothing, no match is found

    .bucket {
      color: blue;
    }
    .some-class:extend(@{variable}) {} // interpolated selector matches nothing
    @variable: .bucket;

但是，extend可以附加到插值选择器上，即插值选择器可以作为extend的父，不能作为extend的子,比如：

    .bucket {
      color: blue;
    }
    @{variable}:extend(.bucket) {}
    @variable: .selector;

    //编译后
    .bucket, .selector {
      color: blue;
    }

### 在@media媒体查询内扩展
目前，在@media声明中扩展只会匹配同一媒体声明中的选择器：

    @media print {
      .screenClass:extend(.selector) {} // extend inside media
      .selector { // this will be matched - it is in the same media
        color: black;
      }
    }
    .selector { // ruleset on top of style sheet - extend ignores it
      color: red;
    }
    @media screen {
      .selector {  // ruleset inside another media - extend ignores it
        color: blue;
      }
    }

编译后：

    @media print {
      .selector,
      .screenClass { /*  ruleset inside the same media was extended */
        color: black;
      }
    }
    .selector { /* ruleset on top of style sheet was ignored */
      color: red;
    }
    @media screen {
      .selector { /* ruleset inside another media was ignored */
        color: blue;
      }
    }


> 注意：扩展与嵌套的@media声明中的选择器不匹配：

    @media screen {
      .screenClass:extend(.selector) {} // extend inside media
      @media (min-width: 1023px) {
        .selector {  // ruleset inside nested media - extend ignores it
          color: blue;
        }
      }
    }
    //extend不生效

最外层顶级扩展,可以匹配嵌套媒体内的选择器在内的所有内容：

    @media screen {
      .selector {  /* ruleset inside nested media - top level extend works */
        color: blue;
      }
      @media (min-width: 1023px) {
        .selector {  /* ruleset inside nested media - top level extend works */
          color: blue;
        }
      }
    }

    .topLevel:extend(.selector) {} /* top level extend matches everything */

编译后：

    @media screen {
      .selector,
      .topLevel { /* ruleset inside media was extended */
        color: blue;
      }
    }
    @media screen and (min-width: 1023px) {
      .selector,
      .topLevel { /* ruleset inside nested media was extended */
        color: blue;
      }
    }

### 复制检测
目前没有重复检测。

    .alert-info,
    .widget {
      /* declarations */
    }

    .alert:extend(.alert-info, .widget) {}

输出

    .alert-info,
    .widget,
    .alert,
    .alert {
      /* declarations */
    }

### 用例扩展
**经典用例**
经典用例是避免添加基类，比如：

    .animal {
      background-color: black;
      color: white;
    }

并且您希望拥有一个覆盖背景颜色的动物子类型，那么您有两个选项，首先更改您的HTML

    <a class="animal bear">Bear</a>

    .animal {
      background-color: black;
      color: white;
    }
    .bear {
      background-color: brown;
    }

或者简化了html并使用了更少的扩展。 例如

    <a class="bear">Bear</a>

    .animal {
      background-color: black;
      color: white;
    }
    .bear {
      &:extend(.animal);
      background-color: brown;
    }

**减少CSS大小**
Mixins将所有属性复制到选择器中，这可能导致不必要的重复。 因此，您可以使用extends而不是mixins将选择器移动到您希望使用的属性，从而减少生成的CSS。

    .my-inline-block() {
      display: inline-block;
      font-size: 0;
    }
    .thing1 {
      .my-inline-block;
    }
    .thing2 {
      .my-inline-block;
    }

输出

    .thing1 {
      display: inline-block;
      font-size: 0;
    }
    .thing2 {
      display: inline-block;
      font-size: 0;
    }

又例如

    .my-inline-block {
      display: inline-block;
      font-size: 0;
    }
    .thing1 {
      &:extend(.my-inline-block);
    }
    .thing2 {
      &:extend(.my-inline-block);
    }

输出

    .my-inline-block,
    .thing1,
    .thing2 {
      display: inline-block;
      font-size: 0;
    }

**结合样式/更高级的Mixin**
另一个用例是mixin的替代方案 - 因为mixins只能用于简单的选择器，如果你有两个不同的html块，但是需要将相同的样式应用于两者，你可以使用extends来关联两个区域。

    li.list > a {
      // list styles
    }
    button.list-style {
      &:extend(li.list > a); // use the same list styles
    }





