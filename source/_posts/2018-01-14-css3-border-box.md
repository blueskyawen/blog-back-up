---
title: CSS3-边框和文本
date: 2018-01-14 15:27:46
tags: CSS
categories: 前端
comments: true
---

### 边框
###### border-radius
顺序设置元素每个角的圆角边框
<!--more-->

    border-radius: 1-4 length|% / 1-4 length|%;
    -webkit-border-radius: 1-4 length|% / 1-4 length|%;
    -moz-border-radius: 1-4 length|% / 1-4 length|%;
    -o-border-radius: 1-4 length|% / 1-4 length|%;

其中1-4就是四个角处的半径，单位可以是：N或%，若省略了部分值，则以对角的设置为准，如下图：
![border-radius](/images/2018-01-08_borde-radius.jpg)

以下都是合法的写法：

    border-radius:1px 2px 3px 4px / 2px 3px 4px 5px;

    border-radius:1px 2px 3px 4px / 2px 3px 4px;
    同 border-radius:1px 2px 3px 4px / 2px 3px 4px 3px;

    border-radius:1px 2px 3px 4px / 2px 3px;
    同 border-radius:1px 2px 3px 4px / 2px 3px 2px 3px;

    border-radius:1px 2px 3px 4px / 2px;
    同 border-radius:1px 2px 3px 4px / 2px 2px 2px 2px;

    border-radius:1px 2px / 2px 3px;
    同 border-radius:1px 2px 1px 2px / 2px 3px 2px 3px;

    border-radius:1px 2px;
    同 border-radius:1px 2px / 1px 2px;

    border-radius:2px;
    同 border-radius:2px 2px 2px 2px / 2px 2px 2px 2px;

以下分开写的属性：

    border-top-left-radius:a b; //length或%，定义了左上角的弧度
    border-top-right-radius   //定义了右上角的弧度
    border-bottom-right-radius   //定义了右下角的弧度
    border-bottom-left-radius   //定义了左下角的弧度
    

具体Demo请点击此处：[border-radius-demo](http://blueskyawen.com/angular-work-cook/main/other/css3/text-box/border-radius)

###### box-shadow
向框添加一个或多个阴影

    box-shadow: x-shadow y-shadow blur spread color inset;

- x-shadow,水平阴影的位置。为正值时，阴影在右边框往右;负值时，阴影在左边框往左。但当inset内阴影时，正值时阴影在左边框往右，负值时阴影在右边框往左。
- y-shadow,垂直阴影的位置。为正值时，阴影在下边框往下;负值时，阴影在上边框往上。但当inset内阴影时，正值时阴影在上边框往下，负值时阴影在下边框往上。
- blur,模糊距离
- spread，阴影的尺寸
- color，阴影的颜色
- 将外部阴影 (outset) 改为内部阴影

具体Demo请点击此处：[box-shadow-demo](http://blueskyawen.com/angular-work-cook/main/other/boxShadow/lizi)

###### border-image
图片边框,几种属性的简写

    border-image：border-image-source border-image-slice / border-image-width / border-image-outset fill border-image-repeat

其中：
- border-image-source，用在边框的图片的路径
- border-image-slice，4个边框向内切割的偏移量
- border-image-width，边框宽度
- border-image-outset，图像边框和内容框的间距偏移距离
- border-image-repeat，切割后的图片在边框拉伸中的实现方式，平铺(repeated)、铺满(rounded)或拉伸(stretched)
- fill，指示中间部分是否显示，fill则显示，否则不显示

简单写来：

    border-image：image-source slice1 slice2 slice3 slice / width1 width2 width3 width4 / outset1 outset2 outset3 outset4 fill repeat;

其中：
**width1-4**是“上右下左”顺序四个图片边框的宽度，省略时以对面为准，同border其他属性的设置方式
**slice1-4**是“上右下左”顺序四个切割的宽度，切割出来的图片被用来显示成边框，如下图，类似于9宫格的切片方式
![border-image](/images/2018-01-08_border-images.jpg)

四个角用作图边框的角，四边切出来的图片用作image边框4边的填充图

具体Demo请点击此处：[border-image-demo](http://blueskyawen.com/angular-work-cook/main/other/css3/text-box/border-image)

### 背景
##### 背景色

    background-color: color;
    background-clip: border-box;
    或
    background: color border-box;

其中clip为背景绘制区域，有border-box/padding-box/content-box几种类型值

##### 背景图片

    background-image: url(***.jpg);
    background-position: left top;
    background-size: contain;
    background-repeat: no-repeat;
    background-origin: border-box;
    background-attach: scroll;
    或
    background: image position / size repeat attach;

其中:

- background-origin，是背景图片的定位方式，有border-box/padding-box/content-box取值，类似与背景的clip
- background-attach，定义背景图是否随着页面内容滚动而滚动，取值scroll/fixed，但是这个属性是针对于整个页面生效的，对单个盒子是不生效的

具体Demo请点击此处：[background-demo](http://blueskyawen.com/angular-work-cook/main/other/css3/box-back/background)


### 文本
###### white-space
设置如何处理元素内的空白，值
- normal，默认，空白会被浏览器忽略。
- pre，空白会被浏览器保留。
- nowrap，文本不会换行，文本会在在同一行上继续，直到遇到br
- pre-wrap，保留空白符序列，但是正常地进行换行
- pre-line，合并空白符序列，但是保留换行符
- inherit，规定应该从父元素继承 white-space 属性的值

###### text-shadow
向文本添加一个或多个阴影

    text-shadow: h-shadow v-shadow blur color;
其中：
- h-shadow，水平阴影的位置。正值阴影在右边，负值阴影在左边
- v-shadow，垂直阴影的位置。正值阴影在下边，负值阴影在上边
- blur，可选，模糊的距离
- color，阴影的颜色

具体Demo请点击此处：[text-shadow](http://blueskyawen.com/angular-work-cook/main/other/css3/text-box/text-shadow)

###### text-overflow
规定当文本溢出包含元素时发生的事情
- clip，修剪文本
- ellipsis，显示省略符号来代表被修剪的文本
- string，使用给定的字符串来代表被修剪的文本

具体Demo请点击此处：[text-overflow](http://sandbox.runjs.cn/show/zapn5yh5)

###### word-wrap
属性允许长单词或 URL 地址换行到下一行
- normal，只在允许的断字点换行（浏览器保持默认处理）。
- break-word，在长单词或 URL 地址内部进行换行，使用时整个长单词都换行到下一行

###### word-break
属性规定自动换行的处理方法
- normal，使用浏览器默认的换行规则。
- break-all，允许在单词内换行。
- keep-all，只能在半角空格或连字符处换行。

具体Demo请点击此处：[word-break](http://sandbox.runjs.cn/show/zapn5yh5)
