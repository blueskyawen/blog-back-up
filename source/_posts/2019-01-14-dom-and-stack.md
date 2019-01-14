---
title: 元素的层叠
date: 2019-01-14 21:50:06
tags: CSS
categories: 前端
comments: true
---

多个元素在一起难免出现先后和堆叠呈现问题，最近编写相关控件时遇到了类似的问题，查阅了许多资料理解了一些，顺便记录下来。

### 层叠
说到元素的层叠，就不得不提到两个概念，叫堆叠上下文(Stacking Context)和堆叠水平，
规范一点的解释可以参见下面的文档：[z-index的工作原理](https://www.w3cplus.com/css/how-z-index-works.html)
我的理解，简单来讲：

- 堆叠上下文就是元素相对于用户的呈现平面，每个元素都有自己的堆叠平面
- 堆叠水平就是元素堆叠上下文的级别，每个元素都有上下文级别，可以和其他元素相同，也可以不同，这是决定元素是否“胜出”的关键，即哪些元素能够呈现在用户面前

<!--more-->

普通元素拥有一个零级的上下文，而通过某些CSS属性可以为元素重新创建上下文，在其他资料里也都提过，主要有：

- <html>元素
- 被定位了的元素，并且拥有一个z-index值（不为auto）
- 元素被设置了opacity，transform, transform3d,filter, css-regions, paged media,motion-path,mix-blend-mode,will-change,-webkit-overflow-scrolling等属性
- 父元素的display设置了flex或者inline-flex值的子元素
- 父元素的display设置了grid或者inline-grid值的子元素
- 元素被设置了isolation:isolate
- 元素的mix-blend-mode值不为normal
- 元素的overflow-scrolling值不为touch

一旦多个元素设置了这些属性，页面变回创建出多个新的堆叠上下文平面，每个元素的上下文是独立存在的，并影响其中子元素的呈现效果，包括子元素建的“堆叠竞争”
虽然页面里可以有许多的堆叠上下文元素，当由于呈现平面只有一个，最后只能有一个或者几个元素能够出现在用户面前，于是就有了规则顺序：

- 有明显的层叠水平标示的时候，比如由定位和z-index创建上下文的元素，则值越大越有机会胜出
- 对于层叠水平一样的几个元素，位置在后面的覆盖前面的，即所谓的“后来居上”
- 每个元素层叠上下文独立成型，子元素的层级显示手父元素的层级限制，子元素只会在父元素的层叠平面和水平上进行规则判决，即：如果父元素的层叠水平和其他元素一致，则子元素可和其他同级别元素的子元素竞争；否则，如果父元素创建了层叠上下文，子元素将只能在父元素层叠上下文平面内进行“内部竞争”，如果父元素层叠水平较低，子元素的z-index再高也是枉然
- 有层叠上下文的元素层叠水平要比普通元素高

这里有个例子，

    <div><span class="red">Red</span>
    </div><div> <span class="green">Green</span>
    </div><div> <span class="blue">Blue</span></div>

    div:nth-child(1) {
      background:red;
      position:absolute;
      //transform:scale(4);
      z-index:-1;
    }
    div:nth-child(2) {
      background:green;
      position:absolute;
    }
    div:nth-child(3) {
      background:blue;
      position:absolute;
    }
    div span {
      display:inline-block;
      height:60px;
      width:100px;
      background:#fff;
    }
    div {
      height:100px;
      display:block;
      width:200px;
    }
    .red {
      position:absolute;
      z-index:100;
    }

可以进去修改样式来查看效果：[例子](http://jsrun.net/gKXKp/edit)

关于层叠元素的显示顺序，张鑫旭的博客有个清晰的图片，可以直接查看
[深入理解CSS中的层叠上下文和层叠顺序](https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/)

### 层叠带来的问题
1. z-index和定位的滥用导致页面元素层级混乱，普通元素单独设置z-index是没有用的,建议使用translateZ() 或者translate3d()来替代z-index
2. transform变换的时候会让 z-index “临时失效”，其实并非 z-index 失效了，在变换的transform时创建了新的层叠上下，z-index生效在了一个新的不同的上下文平面上，当然等结束后又能回到正常的状态；至于怎么解决这个问题，张同学也给了参考意见：任意父级（非body级别）设置overflow:hidden 或者使用transform3d
3. 通过z-index配合伪元素::before或者::after时让其z轴在元素的底部，特别是碰到大的元素渲染(比如全屏背景图)，会直接影响性能，特别是在移动端，会造成客户端闪退，也就是大家所说的Crash
