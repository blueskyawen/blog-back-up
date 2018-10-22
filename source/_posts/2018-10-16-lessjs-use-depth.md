---
title: less深入指南二（译）
date: 2018-08-15 00:00:30
tags: Less
categories: 前端
comments: true
---

# 合并
合并功能允许将多个属性中的值聚合到单个属性下的逗号或空格分隔列表中。 merge对于背景和变换等属性很有用。
<!--more-->

### 使用逗号附加属性值
> Released v1.5.0

    .mixin() {
      box-shadow+: inset 0 0 10px #555;
    }
    .myclass {
      .mixin();
      box-shadow+: 0 0 20px black;
    }

输出

    .myclass {
      box-shadow: inset 0 0 10px #555, 0 0 20px black;
    }

### 使用空格附加属性值
> Released v1.7.0

    .mixin() {
      transform+_: scale(2);
    }
    .myclass {
      .mixin();
      transform+_: rotate(15deg);
    }

输出

    .myclass {
      transform: scale(2) rotate(15deg);
    }

> 为了避免任何无意的连接，merge需要在每个连接挂起声明上使用显式的+或+ _标志。

# Mixins混合
您可以混合使用类选择器和id选择器，例如：

    .a, #b {
      color: red;
    }
    .mixin-class {
      .a();
    }
    .mixin-id {
      #b();
    }

输出

    .a, #b {
      color: red;
    }
    .mixin-class {
      color: red;
    }
    .mixin-id {
      color: red;
    }

目前和历史上，mixin调用中的括号是可选的，但是不推荐使用可选括号，并且在将来的版本中将需要这些括号。

    .a();
    .a;  // currently works, but deprecated; don't use

### 不输出Mixin
如果你想创建一个mixin但你不希望mixin以代码形式出现在你的CSS输出中，请在mixin定义之后添加括号。

    .my-mixin {
      color: black;
    }
    .my-other-mixin() {
      background: white;
    }
    .class {
      .my-mixin();
      .my-other-mixin();
    }

输出

    .my-mixin {
      color: black;
    }
    .class {
      color: black;
      background: white;
    }

### Mixins中的选择器
Mixins可以包含的不仅仅是属性，它们也可以包含选择器。

    .my-hover-mixin() {
      &:hover {
        border: 1px solid red;
      }
    }
    button {
      .my-hover-mixin();
    }

输出

    button:hover {
      border: 1px solid red;
    }

可以看到编译将my-hover-mixin()的内容仅仅进行了替换，如下：

    button {
      &:hover {
        border: 1px solid red;
      }
    }

### 命名空间
如果要在更复杂的选择器中混合属性，可以堆叠多个id或类选择器

    #outer() {
      .inner {
        color: red;
      }
    }
    .c {
      #outer > .inner();
    }

>和空格都是可选的,下面几种形式的效果相同

    #outer > .inner();
    #outer .inner();
    #outer.inner();

命名空间减少了你的mixins与其他库mixin或用户mixin的冲突，但它也可以是一种“组织”mixins组的方法，比如：

    #my-library {
      .my-mixin() {
        color: black;
      }
    }
    // which can be used like this
    .class {
      #my-library.my-mixin();
    }

### 受保护的命名空间
如果命名空间是受保护的，则仅在保护条件返回true时才使用由其定义的mixins。 命名空间保护的计算方式与mixin的保护完全相同，因此以下两个mixin的工作方式相同：

    #namespace when (@mode = huge) {
      .mixin() { /* */ }
    }
    #namespace {
      .mixin() when (@mode = huge) { /* */ }
    }

所有嵌套命名空间和mixin的默认函数具有相同的值。以下mixin从未被评估过，其中一名守卫肯定是假的：

    #sp_1 when (default()) {
      #sp_2 when (default()) {
        .mixin() when not(default()) { /* */ }
      }
    }

### !important关键字
在mixin调用之后使用！important关键字将其继承的所有属性标记为！important：

    .foo (@bg: #f5f5f5, @color: #900) {
      background: @bg;
      color: @color;
    }
    .unimportant {
      .foo();
    }
    .important {
      .foo() !important;
    }

输出

    .unimportant {
      background: #f5f5f5;
      color: #900;
    }
    .important {
      background: #f5f5f5 !important;
      color: #900 !important;
    }

### 参数混合Mixin
**如何将参数传递给mixins**
Mixins也可以接受参数，这些参数是混合在一起时传递给选择器块的变量

    .border-radius(@radius) {
      -webkit-border-radius: @radius;
         -moz-border-radius: @radius;
              border-radius: @radius;
    }

以下是我们如何将其混合到各种规则集中：

    #header {
      .border-radius(4px);
    }
    .button {
      .border-radius(6px);
    }

参数mixin也可以为其参数设置默认值：

    .border-radius(@radius: 5px) {
      -webkit-border-radius: @radius;
         -moz-border-radius: @radius;
              border-radius: @radius;
    }

我们现在可以像这样调用它：

    #header {
      .border-radius();
    }

它将包括5px边界半径。
您还可以使用不带参数的参数化mixins。 如果要从CSS输出中隐藏规则集，但希望在其他规则集中包含其属性，这非常有用：

    .wrap() {
      text-wrap: wrap;
      white-space: -moz-pre-wrap;
      white-space: pre-wrap;
      word-wrap: break-word;
    }

    pre { .wrap() }

输出

    pre {
      text-wrap: wrap;
      white-space: -moz-pre-wrap;
      white-space: pre-wrap;
      word-wrap: break-word;
    }

**具有多个参数的混合**
参数可以是分号或逗号分隔，建议使用分号。逗号具有双重含义：它可以解释为mixin参数分隔符或css列表分隔符。
使用逗号作为mixin分隔符使得无法将逗号分隔列表创建为参数。另一方面，如果编译器在mixin调用或声明中看到至少一个分号，则它假定参数由分号分隔，并且所有逗号都属于css列表：

- 两个参数，每个参数都包含以逗号分隔的列表：.name（1,2,3; some，else），
- 三个参数，每个包含一个数字：.name（1,2,3），
- 使用dummy分号创建mixin调用，其中一个参数包含逗号分隔的css列表：.name（1,2,3;），
- 逗号分隔的默认值：.name（@ param1：red，blue;）。

定义具有相同名称和参数数量的多个mixin是合法的。 Less将使用所有可应用的属性。如果你使用带有一个参数的mixin，例如.mixin（绿色）;然后将使用具有一个必需参数的所有mixins的属性：

    .mixin(@color) {
      color-1: @color;
    }
    .mixin(@color; @padding: 2) {
      color-2: @color;
      padding-2: @padding;
    }
    .mixin(@color; @padding; @margin: 2) {
      color-3: @color;
      padding-3: @padding;
      margin: @margin @margin @margin @margin;
    }
    .some .selector div {
      .mixin(#008000);
    }

输出：

    .some .selector div {
      color-1: #008000;
      color-2: #008000;
      padding-2: 2;
    }

**命名参数**
mixin参考可以通过其名称而不仅仅是位置来提供参数值。 任何参数都可以通过其名称引用，并且它们不必具有任何特殊顺序：

    .mixin(@color: black; @margin: 10px; @padding: 20px) {
      color: @color;
      margin: @margin;
      padding: @padding;
    }
    .class1 {
      .mixin(@margin: 20px; @color: #33acfe);
    }
    .class2 {
      .mixin(#efca44; @padding: 40px);
    }

输出

    .class1 {
      color: #33acfe;
      margin: 20px;
      padding: 20px;
    }
    .class2 {
      color: #efca44;
      margin: 10px;
      padding: 40px;
    }

**@arguments变量**
@arguments在mixins中有一个特殊含义，它包含调用mixin时传递的所有参数。 如果您不想处理单个参数，这非常有用：

    .mixin(...) {        // matches 0-N arguments
    .mixin() {           // matches exactly 0 arguments
    .mixin(@a: 1) {      // matches 0-1 arguments
    .mixin(@a: 1; ...) { // matches 0-N arguments
    .mixin(@a; ...) {    // matches 1-N arguments

    .mixin(@a; @rest...) {
       // @rest is bound to arguments after @a
       // @arguments is bound to all arguments
    }

### 模式匹配
有时候，您可能希望根据传递给它的参数来更改mixin的行为，比如：

    .mixin(dark; @color) {
      color: darken(@color, 10%);
    }
    .mixin(light; @color) {
      color: lighten(@color, 10%);
    }
    .mixin(@_; @color) {
      display: block;
    }

下面我们使用它

    @switch: light;

    .class {
      .mixin(@switch; #888);
    }

输出

    .class {
      color: #a2a2a2;
      display: block;
    }

可以看到，传递给.mixin的颜色减轻了，如果@switch的值是dark，结果将是一个较暗的颜色，这里发生了参数匹配：

- 第一个mixin定义不匹配，因为它期望dark作为第一个参数。
- 第二个mixin定义匹配，因为它预期light。
- 第三个mixin定义匹配，因为它可以是任何值。

仅使用匹配的mixin定义来进行编译输出，变量匹配并绑定到任何值，除变量之外的任何内容仅与值等于其自身的值匹配。
我们也可以匹配arity，这是一个例子：

    .mixin(@a) {
      color: @a;
    }
    .mixin(@a; @b) {
      color: fade(@a; @b);
    }

现在如果我们用一个参数调用.mixin，我们将得到第一个定义的输出，但如果我们用两个参数调用它，我们将得到第二个定义，即@a淡化为@b。

### 从mixin返回值
从mixin返回变量或mixins,从Less 3.5开始，您可以使用属性/变量访问器从mixin获取“返回值”，基本上像函数一样使用它。

    .average(@x, @y) {
      @result: ((@x + @y) / 2);
    }

    div {
      padding: .average(16px, 50px)[@result];  // call a mixin and look up its "@return" value
    }

输出

    div {
      padding: 33px;
    }

**调用范围内的变量**

> DEPRECATED - 使用属性/值访问器

mixin中定义的变量和mixin是可见的，可以在调用者的范围内使用。 只有一个例外：如果调用者包含一个具有相同名称的变量（包括由另一个mixin调用定义的变量），则不会复制变量。 只有受调用者本地范围中存在的变量才受到保护,从父作用域继承的变量将被重写。

    .mixin() {
      @width:  100%;
      @height: 200px;
    }

    .caller {
      .mixin();
      width:  @width;
      height: @height;
    }

输出

    .caller {
      width:  100%;
      height: 200px;
    }

直接在调用者范围中定义的变量不能被覆盖。 但是，调用者父作用域中定义的变量不受保护，将被覆盖：

    .mixin() {
      @size: in-mixin;
      @definedOnlyInMixin: in-mixin;
    }

    .class {
      margin: @size @definedOnlyInMixin;
      .mixin();
    }

    @size: globaly-defined-value; // callers parent scope - no protection

输出


    .class {
      margin: in-mixin in-mixin;
    }

最后，mixin中定义的mixin也可作为返回值使用，例子：

    .unlock(@value) { // outer mixin
      .doSomething() { // nested mixin
        declaration: @value;
      }
    }

    #namespace {
      .unlock(5); // unlock doSomething mixin
      .doSomething(); //nested mixin was copied here and is usable
    }

    Results in:

    #namespace {
      declaration: 5;
    }

### 递归混合
在Less中，mixin可以自调用。 当与Guard表达式和模式匹配结合使用时，这种递归mixin可用于创建各种迭代/循环结构。

    .loop(@counter) when (@counter > 0) {
      .loop((@counter - 1));    // next iteration
      width: (10px * @counter); // code for each iteration
    }

    div {
      .loop(5); // launch the loop
    }

输出

    div {
      width: 10px;
      width: 20px;
      width: 30px;
      width: 40px;
      width: 50px;
    }

使用递归循环生成CSS网格类的一般示例：

    .generate-columns(4);

    .generate-columns(@n, @i: 1) when (@i =< @n) {
      .column-@{i} {
        width: (@i * 100% / @n);
      }
      .generate-columns(@n, (@i + 1));
    }

    Output:

    .column-1 {
      width: 25%;
    }
    .column-2 {
      width: 50%;
    }
    .column-3 {
      width: 75%;
    }
    .column-4 {
      width: 100%;
    }

### Mixin Guards
当您想要匹配表达式而不是简单值或arity时，防护很有用。 如果您熟悉函数式编程，则可能已经遇到过它们。
为了尽可能地保持CSS的声明性，Less选择通过受保护的mixins而不是if / else语句实现条件执行，这是@media查询功能规范的一部分。举个例子：

    .mixin(@a) when (lightness(@a) >= 50%) {
      background-color: black;
    }
    .mixin(@a) when (lightness(@a) < 50%) {
      background-color: white;
    }
    .mixin(@a) {
      color: @a;
    }

关键是when关键字，它引入了一个保护序列（这里只有一个保护）。 现在，如果我们运行以下代码：

    .class1 { .mixin(#ddd) }
    .class2 { .mixin(#555) }

将输出如下：

    .class1 {
      background-color: black;
      color: #ddd;
    }
    .class2 {
      background-color: white;
      color: #555;
    }

**比较运算符**
守卫中可用的比较运算符的完整列表是：>，> =，=，= <，<。 此外，关键字true是唯一的truthy值，使这两个mixin等效：

    .truth(@a) when (@a) { ... }
    .truth(@a) when (@a = true) { ... }

除关键字true之外的任何值都认为是false：

    .class {
      .truth(40); // Will not match any of the above definitions.
    }

请注意，您还可以相互比较参数，或使用非参数：

    @media: mobile;

    .mixin(@a) when (@media = mobile) { ... }
    .mixin(@a) when (@media = desktop) { ... }

    .max(@a; @b) when (@a > @b) { width: @a }
    .max(@a; @b) when (@a < @b) { width: @b }


**逻辑运算符**
您可以将逻辑运算符与守卫一起使用，语法基于CSS媒体查询。
使用and关键字组合守卫：

    .mixin(@a) when (isnumber(@a)) and (@a > 0) { ... }

您可以通过用逗号分隔守卫来模拟或运算符：

    .mixin(@a) when (@a > 10), (@a < -10) { ... }

使用not关键字否定条件：

    .mixin(@b) when not (@b > 0) { ... }

**类型检查方法**
最后，如果要根据值类型匹配mixins，可以使用is函数：

    .mixin(@a; @b: 0) when (isnumber(@b)) { ... }
    .mixin(@a; @b: black) when (iscolor(@b)) { ... }

常用的方法：

- iscolor
- isnumber
- isstring
- iskeyword
- isurl

如果您想检查某个值是否在某个特定单位中而不是数字，您可以使用以下方法之一：

- ispixel
- ispercentage
- isem
- isunit

### Mixins别名
> Released v3.5.0-beta.4

**将mixin调用分配给变量**
可以将Mixins分配给变量以作为变量调用来调用，或者可以将其用于映射查找。

    #theme.dark.navbar {
      .colors(light) {
        primary: purple;
      }
      .colors(dark) {
        primary: black;
        secondary: grey;
      }
    }

    .navbar {
      @colors: #theme.dark.navbar.colors(dark);
      background: @colors[primary];
      border: 1px solid @colors[secondary];
    }

    output:

    .navbar {
      background: black;
      border: 1px solid grey;
    }

**可变调用**
整个mixin调用可以是别名并称为变量调用。 如：

    #library() {
      .rules() {
        background: green;
      }
    }
    .box {
      @alias: #library.rules();
      @alias();
    }

输出

    .box {
      background: green;
    }

注意，与root中使用的mixin不同，mixin调用分配给变量并且不带参数调用**总是需要括号**。 以下内容无效。

    #library() {
      .rules() {
        background: green;
      }
    }
    .box {
      @alias: #library.colors;
      @alias;   // ERROR: Could not evaluate variable call @alias
    }

这是因为如果变量被分配了选择器列表或mixin调用，则它是不明确的。 例如，在Less 3.5+中，此变量可以这种方式使用。

    .box {
      @alias: #library.colors;
      @{alias} {
        a: b;
      }
    }

输出结果：

    .box #library.colors {
      a: b;
    }

# CSS守卫
> released v1.5.0

像Mixin Guards一样，守卫也可以应用于css选择器，这是用于声明mixin然后立即调用它的语法糖。
举个例子，在1.5.0之前你必须这样做：

    .my-optional-style() when (@my-option = true) {
      button {
        color: white;
      }
    }
    .my-optional-style();

现在，你可以直接加在选择器上：

    button when (@my-option = true) {
      color: white;
    }

您还可以通过将此功能与＆功能相结合来实现if类型语句，从而允许您对多个守卫进行分组。

    & when (@my-option = true) {
      button {
        color: white;
      }
      a {
        color: blue;
      }
    }

# 分离的规则集
将整个css规则集当作变量使用
> 发布v1.7.0

分离的规则集是一组css属性/嵌套规则集/媒体声明或存储在变量中的任何其他内容。 您可以将其包含在规则集或其他结构中，并将其所有属性复制到那里。 您也可以将它用作mixin参数变量进行传递。

    @detached-ruleset: { background: red; }; // semi-colon is optional in 3.5.0+

    // use detached ruleset
    .top {
        @detached-ruleset();
    }

输出

    .top {
      background: red;
    }

分离的规则集调用后的括号是必需的，调用@ detached-ruleset; 是不行的。

当您想要定义一个在媒体查询中或者从不支持的浏览器类名中抽象出来的mixin时，它非常有用，例如，

    .desktop-and-old-ie(@rules) {
      @media screen and (min-width: 1200px) { @rules(); }
      html.lt-ie9 &                         { @rules(); }
    }

    header {
      background-color: blue;

      .desktop-and-old-ie({
        background-color: red;
      });
    }

这里的desktop-and-old-ie定义了媒体查询和根类，以便您可以使用mixin来包装一段代码。 这将输出

    header {
      background-color: blue;
    }
    @media screen and (min-width: 1200px) {
      header {
        background-color: red;
      }
    }
    html.lt-ie9 header {
      background-color: red;
    }

此外，还可以带代表完整的Less规则集，例如，

    @my-ruleset: {
        .my-selector {
          background-color: black;
        }
    };

甚至可以利用媒体查询冒泡

    @my-ruleset: {
        .my-selector {
          @media tv {
            background-color: black;
          }
        }
    };
    @media (orientation:portrait) {
        @my-ruleset();
    }

    //output
    @media (orientation: portrait) and tv {
      .my-selector {
        background-color: black;
      }
    }

分离规则集的调用以与mixin调用相同的方式解锁（返回）其所有mixin到调用者，但是，它不返回变量。

    // detached ruleset with a mixin
    @detached-ruleset: {
        .mixin() {
            color:blue;
        }
    };
    // call detached ruleset
    .caller {
        @detached-ruleset();
        .mixin();
    }

输出

    .caller {
      color: blue;
    }

私有变量：

    @detached-ruleset: {
        @color:blue; // this variable is private
    };
    .caller {
        color: @color; // syntax error
    }

### 作用域
分离的规则集可访问变量和mixin的范围，包括定义它的位子以及调用它的位置。如果两个范围包含相同的变量或mixin，则声明范围值优先。
声明范围是定义分离规则集主体的范围。 规则集仅通过引用访问，将分离的规则集从一个变量复制到另一个变量而不能修改其范围。
最后，分离的规则集可以通过解锁（导入）到范围来获得对范围的访问。

**定义范围可见性**
分离的规则集可以看到调用者的变量和mixins：

    @detached-ruleset: {
      caller-variable: @caller-variable; // variable is undefined here
      .caller-mixin(); // mixin is undefined here
    };

    selector {
      // use detached ruleset
      @detached-ruleset();

      // define variable and mixin needed inside the detached ruleset
      @caller-variable: value;
      .caller-mixin() {
        variable: declaration;
      }
    }

编译输出：

    selector {
      caller-variable: value;
      variable: declaration;
    }

变量和mixins可访问的同名变量定义胜过调用者中可用的那些：

    @variable: global;
    @detached-ruleset: {
      variable: @variable;
    };

    selector {
      @detached-ruleset();
      @variable: value;
    }

编译输出

    selector {
      variable: global;
    }


**引用不会修改分离的规则集范围**
规则集仅通过在那里引用而无法访问新范围：

    @detached-1: { scope-detached: @one @two; };
    .one {
      @one: visible;
      .two {
        @detached-2: @detached-1; // copying/renaming ruleset
        @two: visible; // ruleset can not see this variable
      }
    }

    .use-place {
      .one > .two();
      @detached-2();
    }

throws an error:
*ERROR 1:32 The variable "@one" was not declared.*

**解锁将修改分离的规则集范围**
分离的规则集通过在作用域内解锁（导入）来获得访问权限：

    #space {
      .importer-1() {
        @detached: { scope-detached: @variable; }; // define detached ruleset
      }
    }

    .importer-2() {
      @variable: value; // unlocked detached ruleset CAN see this variable
      #space > .importer-1(); // unlock/import detached ruleset
    }

    .use-place {
      .importer-2(); // unlock/import detached ruleset second time
       @detached();
    }

    compiles into:

    .use-place {
      scope-detached: value;
    }

