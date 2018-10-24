---
title: less深入指南三-@import导入（译）
date: 2018-08-23 00:00:30
tags: Less
categories: 前端
comments: true
---

# @import导入
在标准CSS中，@import语句必须在所有其他类型的规则之前声明，但Less并不关心你放置@import语句的位置

    .foo {
      background: #900;
    }
    @import "this-is-valid.less";

<!--more-->

### 识别文件扩展名
根据文件扩展名的不同，Less在处理@import导入的时候的方式也有所不同

- 如果文件具有.css扩展名，则将其视为CSS文件，并将@import语句保留为原样（请参阅下面的内联选项）。
- 如果它有任何其他扩展名，它将被视为less文件并导入。
- 如果它没有扩展名，则会附加.less，它将作为导入的Less文件包含在内。
例子：

    @import "foo";      // foo.less is imported
    @import "foo.less"; // foo.less is imported
    @import "foo.php";  // foo.php imported as a Less file
    @import "foo.css";  // statement left in place, as-is

下面要讲到的选项可以覆盖此行为。

### 导入选项
Less为CSS的@import提供了几个扩展，比使用外部文件具有更多的灵活性。
语法：

    @import（keyword）“filename”;

以下是支持的导入选项：

- reference: 使用Less文件但不输出
- inline: 在输出中包含源文件但不处理它
- less: 无论文件扩展名是什么，都将文件视为Less文件
- css: 无论文件扩展名是什么，都将文件视为CSS文件
- once: 只包含文件一次（默认行为）
- multiple: 多次包含该文件
- optional: 在找不到文件时继续编译

每个@import允许使用多个关键字，关键字之间使用逗号分隔，比如：

    @import (optional, reference) "foo.less";

##### reference
> Released v1.5.0

    @import（reference）“foo.less”;

导入外部文件，但不将引入的样式添加到已编译的输出，只是引用它们。

想象一下，在导入的文件中，reference使用一个引用来标记每一个at-rule和selector，就像正常导入一样。但是当生成CSS时，却不输出“reference”选择器，甚至是任何仅仅包含引用选择器的媒体查询。
除非参考样式被用作mixins或extend，否则引用样式不会显示在生成的CSS中。根据使用的方法（mixin或extend）不同，会产生不同的结果。

- extend：当使用扩展选择器时，只有新选择器被标记为未引用，并且它将被引入到@import引用语句的位置。
- mixins：当参考样式被用作隐式mixin时，其规则是混合的，标记为“非引用”，并且正常显示在引用的位置。

例子：
允许您通过执行以下操作从Bootstrap等库中仅提取特定的，有针对性的样式：

    .navbar：extend（.navbar all）{}

并且您将仅从Bootstrap中提取.navbar相关样式。

##### inline
> Released v1.5.0

    @import (inline) "not-less-compatible.css";

在输出中包含源文件但不处理它
当CSS文件可能不兼容时，您将使用此选项; 这是因为虽然Less支持大多数已知标准CSS，但它不支持某些注释，并且不支持所有已知的CSS补丁，它是不修改CSS的。

您可以使用它将文件包含在输出中，以便所有CSS都在一个文件中。

##### less
> Released v1.4.0

    @import (less) "foo.css";

将导入的文件视为Less，无论文件扩展名如何。

##### css
> Released v1.4.0

    @import (css) "foo.less";

    //outputs

    @import "foo.less";

将导入的文件视为常规CSS，无论文件扩展名如何。 这意味着import语句将保持原样。

##### once
> Released v1.4.0

@import语句的默认行为。 这意味着文件仅导入一次，并且将忽略该文件的后续导入语句。

    @import (once) "foo.less";
    @import (once) "foo.less"; // this statement will be ignored

##### multiple
> Released v1.4.0

使用@import（multiple）可以导入多个相同名称的文件，这是和once相反的行为。

    // file: foo.less
    .a {
      color: green;
    }
    // file: main.less
    @import (multiple) "foo.less";
    @import (multiple) "foo.less";

输出

    .a {
      color: green;
    }
    .a {
      color: green;
    }

##### optional
> Released v2.3.0

    @import (optional) "foo.less";

使用@import（optional）仅允许在文件存在时导入文件，在没有optional关键字时，Less会抛出FileError并在导入无法找到的文件时停止编译。

### @plugin插件
> Released v2.5.0

可以通过导入JavaScript插件以添加Less.js函数和功能

### 编写你的第一个插件
使用@plugin的方式类似于使用@import导入.less文件。

    @plugin "my-plugin";  // 字段添加.js后缀，如果不带扩展名

由于Less插件是在Less作用域范围内评估的，因此插件定义可以非常简单，比如：

    registerPlugin({
        install: function(less, pluginManager, functions) {
            functions.add('pi', function() {
                return Math.PI;
            });
        }
    })

或者你可以使用module.exports（在浏览器和Node.js中工作）。

    module.exports = {
        install: function(less, pluginManager, functions) {
            functions.add('pi', function() {
                return Math.PI;
            });
        }
    };

> 请注意，浏览器中不支持其他Node.js库比如CommonJS的使用方式，例如require（）。 编写跨平台插件时请记住这一点。

你能用插件做什么？ 很多，但让我们从基础开始。我们将首先关注您可能放在插件中放置的内容：

    // my-plugin.js
    install: function(less, pluginManager, functions) {
        functions.add('pi', function() {
            return Math.PI;
        });
    }

恭喜！ 你写了一个Less插件！您可以在样式表中这样使用它：

    @plugin "my-plugin";
    .show-me-pi {
      value: pi();
    }

输出：

    .show-me-pi {
      value: 3.141592653589793;
    }

但是，如果您希望将其与其他值相乘或执行其他Less操作，则需要返回正确的Less节点。 否则，在样式表中的输出是纯文本。

    functions.add('pi', function() {
        return less.dimension(Math.PI);
    });

> 注意：dimension的参数可以带单位，也可以不带单位，比如：dimension(10) 或 dimension(10,"px")。

现在您可以在运算操作中使用您的插件：

    @plugin "my-plugin";
    .show-me-pi {
      value: pi() * 2;
    }

您可能已经注意到插件文件有可用的全局变量，即函数注册表（函数对象）和较少的对象。 这些都是为了方便起见。

### 插件作用域范围
在使用@plugin添加函数的时候，是遵循Less作用域规则的。 这对于想要在不引入命名冲突的情况下添加函数的Less库作者来说非常有用。
例如，假设您有两个来自两个第三方库的插件，这两个插件都有一个名为“foo”的函数。

    // lib1.js
    // ...
        functions.add('foo', function() {
            return "foo";
        });
    // ...

    // lib2.js
    // ...
        functions.add('foo', function() {
            return "bar";
        });
    // ...

没关系！ 您可以选择哪个库的功能创建哪个输出。

    .el-1 {
        @plugin "lib1";
        value: foo();
    }
    .el-2 {
        @plugin "lib2";
        value: foo();
    }

输出：

    .el-1 {
        value: foo;
    }
    .el-2 {
        value: bar;
    }

对于共享插件的插件作者，这意味着您还可以通过将私有函数放在特定范围内来有效地创建私有函数。 所以，如下的使用方式是错误的：

    .el {
        @plugin "lib1";
    }
    @value: foo();

从Less 3.0开始，函数可以返回任何类型的Node类型，并且可以在任何级别调用。
但是，这会在2.x中引发错误，因为在2.x中函数必须是属性或变量赋值的一部分：

    .block {
        color: blue;
        my-function-rules();
    }

在3.x中，不再是这种情况，函数可以返回At-Rules，Rulesets，任何其他Less节点，字符串和数字（后两个转换为匿名节点）。

### 空函数
有时您可能想要调用一个函数，但是您不需要任何输出（例如存储值以供以后使用）。 在这种情况下，您只需要从函数返回false。

    var collection = [];
    functions.add('store', function(val) {
        collection.push(val);  // imma store this for later
        return false;
    });
    //输出
    @plugin "collections";
    @var: 32;
    store(@var);

或者可以这么写

    functions.add('retrieve', function(val) {
        return less.value(collection);
    });
    //输出
    .get-my-values {
        @plugin "collections";
        values: retrieve();
    }

### Less.js插件对象
Less.js插件应该导出具有一个或多个属性的对象。

    {
        /* Called immediately after the plugin is
         * first imported, only once. */
        install: function(less, pluginManager, functions) { },

        /* Called for each instance of your @plugin. */
        use: function(context) { },

        /* Called for each instance of your @plugin,
         * when rules are being evaluated.
         * It's just later in the evaluation lifecycle */
        eval: function(context) { },

        /* Passes an arbitrary string to your plugin
         * e.g. @plugin (args) "file";
         * This string is not parsed for you,
         * so it can contain (almost) anything */
        setOptions: function(argumentString) { },

        /* Set a minimum Less compatibility string
         * You can also use an array, as in [3, 0] */
        minVersion: ['3.0'],

        /* Used for lessc only, to explain
         * options in a Terminal */
        printUsage: function() { },

    }

PluginManager为 install() function作实例化，并且提供了添加访问者，文件管理器和后处理器的方法。
以下是一些显示不同插件类型的示例，可分别链接查看。

- [后处理器](https://github.com/less/less-plugin-clean-css)
- [访问者](https://github.com/less/less-plugin-inline-urls)
- [文件管理器](https://github.com/less/less-plugin-npm-import)

### 预加载插件
虽然@plugin调用适用于大多数情况，但有时您可能希望在解析开始之前加载插件。
请参阅：“使用Less.js”部分中的预加载插件，了解如何执行此操作。

# Maps (新)
> Released v3.5.0-beta.4

使用规则集和mixins作为值的映射
通过将命名空间与[]语法相结合，您可以将规则集/ mixin转换为映射，比如：

    @sizes: {
      mobile: 320px;
      tablet: 768px;
      desktop: 1024px;
    }

    .navbar {
      display: block;

      @media (min-width: @sizes[tablet]) {
        display: inline-block;
      }
    }

输出：

    .navbar {
      display: block;
    }
    @media (min-width: 768px) {
      .navbar {
        display: inline-block;
      }
    }

由于命名空间和重载混合的能力，Mixins比Map更具通用性。

    #library() {
      .colors() {
        primary: green;
        secondary: blue;
      }
    }

    #library() {
      .colors() { primary: grey; }
    }

    .button {
      color: #library.colors[primary];
      border-color: #library.colors[secondary];
    }

输出：

    .button {
      color: grey;
      border-color: blue;
    }

此外，你还可以通过别名访问mixins，那是个更容易的方式。

    .button {
      @colors: #library.colors();
      color: @colors[primary];
      border-color: @colors[secondary];
    }

请注意，当存在多个嵌套的规则集的时候，可以使用多个[]来进行查找访问，如下所示：

    @config: {
      @options: {
        library-on: true
      }
    }

    & when (@config[@options][library-on] = true) {
      .produce-ruleset {
        prop: val;
      }
    }

通过这种方式，规则集和变量调用可以模拟一种类型的“命名空间”，类似于mixins

至于是否将分配给变量的mixins或规则集用作映射，这取决于您。 您可能希望通过重新声明分配给rulset的变量来替换整个Map来使用； 或者您可能想要“合并”单个键/值对，在这种情况下，把mixins作为Map可能更合适。

### 在[]中使用变量的变量
有一件重要的事情需要注意，查找表达式[@lookup]中的@lookup是查找的主键，一般只接受常量，不能使用变量来访问，但是可以使用变量的变量来作为索引键值
例子：

    .foods() {
      @dessert: ice cream;
    }

    @key-to-lookup: dessert;

    .lunch {
      treat: .foods[@@key-to-lookup];
    }

编译输出:

    .lunch {
      treat: ice cream;
    }






