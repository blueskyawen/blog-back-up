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




