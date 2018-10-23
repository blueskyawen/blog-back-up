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

##### 编写你的第一个插件
使用@plugin的方式类似于使用@import导入.less文件。




