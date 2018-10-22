---
title: 详解javascript和jquery里的各种尺寸和偏移
date: 2018-08-22 00:00:40
tags: js
categories: 前端
comments: true
---

<head>
<script src="https://cdn.bootcss.com/jquery/1.10.2/jquery.min.js">
</script>
</head>

最近在写jquery的滚动监听和附加导航的插件的时候，把元素的尺寸，定位和偏移等各种参数和计算捋了一遍，包括js原生的和jquery的计算方法，以及
之间的区别，想总结下来，怕忘记，方便以后翻阅
<!--more-->

# 1 Dom元素的位置呈现
一个Dom元素在页面的位置涉及到多个参数：视口，父包含容器以及元素本身，一般的呈现形式是下面这样的：

<div class="container22 container-screen ">
		<div class="head">
			文档滚动到视口外的部分
		</div>
  <div class="viewScreen">
    <span>视口</span>
		<div class="body">
			<div class="content">
				容器
				<div class="elemz" id="target">
					目标元素
				</div>
			</div>
    </div></div>
		<div class="foot">
			文档未进入视口的部分
		</div>
</div>

可以看到，Dom元素的呈现分为好几个部分，文档的尺寸是任意的，在视口或父包含容器尺寸一定的时候也能正常显示
当容器缺失的时候，整个视口就元素的容器，是一般位置的特殊形式

<div class="container22">
		<div class="head">
			文档滚动到视口外的部分
		</div>
		<div class="body">
			<div class="content">
				视口
				<div class="elemz" id="target">
					目标元素
				</div>
			</div>
		</div>
		<div class="foot">
			文档未进入视口的部分
		</div>
</div>

# 2 Dom元素的尺寸和偏移
从上一节的元素位置图可以看到，要描述一个元素以及其位置需要许多的参数：height,width,offsetTop,offsetLeft...等，
以及包含它的每一层级的容器元素，而每一层元素都拥有自己的属性和特性

## jquery表示
#### 尺寸
**$('targetSelector').height()/width()**
元素内容区的宽和高，不包括padding

**$('targetSelector').innnerHeight()/innnerWidth()**
元素内边距区的宽和高，包括padding,但不包括border

**$('targetSelector').outerHeight(false)/outerWidth(false)**
元素边界区的宽和高，包括padding/border,但不包括margin

**$('targetSelector').outerHeight(true)/outerWidth(true)**
元素外边距区的宽和高，包括padding/border/margin

#### 滚动位置
**$('targetSelector').scrollTop()/scrollLeft()**
元素相对于视口或者父容器的滚动条的位置的滚动距离，也是父容器的滚动条的位置，即隐藏在视口之外的部分

#### 偏移
**$('targetSelector').offset()**
元素相对于视口的偏移，从视口的边界到元素border的偏移距离,距离包含了原书的margin

- offset().top
- offset().left

**$('targetSelector').position()**
元素相对于上一级position:relative的父元素的偏移，从视口的边界到元素margin的偏移距离,距离不包含了原书的margin
当父元素是视口的时候，其数值和offset()相同

- position().top
- position().left

> 注意，这两个偏移计算值是随着文档的滚动而变化;当文档向上滚动是，偏移的距离是需要减去滚动距离的

比如，在初始没有滚动的时候，offset偏移为：

    var offset.maxTop = $('targetSelector').offset().top;
    var offset.maxLeft = $('targetSelector').offset().left;

当文档进行滚动的时候

    $('targetSelector').offset().top = offset.maxTop - $('targetSelector').scrollTop();
    $('targetSelector').offset().left = offset.maxLeft - $('targetSelector').scrollLeft();

position()和offset()类似

## js-dom元素表示

    var element = $('targetSelector').get(0);

#### 尺寸
**element.offsetHeight/offsetwidth**
元素边界区的宽和高，包括padding/border和滚动条,但不包括margin

**element.clientHeight/clientwidth**
元素内边距区的宽和高，包括padding,但不包括border/margin和滚动条

#### 滚动位置
**element.scrollTop/scrollLeft**
元素相对于视口或者父容器的滚动条的位置的滚动距离，也是父容器的滚动条的位置,即隐藏在视口之外的部分

**element.scrollHeight/scrollWidth**
文档元素的滚动尺寸，实际大小，而不论视口有多大，多的部分滚动隐藏在视口之外

#### 偏移
**element.offsetTop/offsetLeft**
元素相对于上一级position:relative的父元素的偏移，偏移计算值是不随着文档的滚动而变化,是一定的

## jquery-dom元素尺寸的相同参数

    var $elem = $('targetSelector');
    var element = $elem.get(0);

- $elem.innerHeight()/innerWidth() 同 element.clientHeight/clientwidth
- $elem.outerHeight(false)/outerWidth(false) 同 element.offsetHeight/offsetwidth
- $elem.scrollTop(100)/scrollLeft(100) 同 element.scrollTop/scrollLeft = 100


# 3 尺寸和偏移观察测试
观察的元素属性，大概是这个样子

<div class="container-screen2">
  <span class="shihou">视口</span>
  <span class="shihou2">20</span>
  <span class="shihou3">10</span>
  <div class="container3">
    <span class="shihou4">5</span>
    <span class="shihou5">50</span>
    <span class="shihou6">容器</span>
    <span class="shihou7">content</span>
    <div class="content3">
      <div class="etemz3">
        <div class="etemz3-content">元素</div>
      </div>
      <div class="etemz3">
        <div class="etemz3-content">元素</div>
      </div>
    </div>
  </div>
</div>

各个尺寸css如下

    .container {
        box-sizing: border-box;
        width: 420px;
        height: 160px;
        margin: 20px 10px;
        padding: 5px 50px;
        border: solid 1px;
    }

    .elem {
        box-sizing: border-box;
        height: 40px;
        width: 150px;
        margin: 10px;
        padding: 5px;
        border: solid 5px blue;
    }

点击链接可以操作查看各种尺寸和偏移：

[尺寸和偏移观察](http://jsrun.net/hfhKp)

<style>
.container22 {
	display:flex;
	flex-direction:column;
	width:420px;
	align-items:center;
	margin-top:20px;
}
.container22 .head {
  background:#ff8566;
  width:98%;
  height:100px;
}
.container-screen .head {
  width:50%;
}
.container-screen .viewScreen {
  	width:100%;
		border:solid 1px blue;
		opacity：0;
		height:160px;
		display:flex;
		justify-content:center;
		align-items:center;
    position:relative;
}
.container22 .body {
		width:100%;
		border:solid 1px;
		opacity：0;
		height:160px;
		display:flex;
		flex-direction:column;
		align-items:center;
	}
.container-screen .body {
  width:50%;
  height:120px;
}
	.container22 .body .content {
		width:98%;
		height:100%;
		background: #ccc;
		position:relative;
		overflow-y:scroll;
		//overflow:visible;
		//overflow:visible;
	z-index:100p
	}
 .elemz {
		height:40px;
		width:150px;
		background:#006600;
		position:absolute;
		top:30px;
		left:50px;
		color:#fff;
		padding:5px;
		margin:10px 5px;
		border:solid 5px #999;
	}
	.container22 .foot {
		background:#4da6ff;
		width:98%;
		height:60px;
	}
.container-screen .foot {
  width:50%;
}

.container-screen span{
  position:absolute;
  top:10px;
  left:10px;
}
.container-screen .elemz {
  	height:30px;
		width:100px;
}


.container-screen2 {
  border:solid 1px;
  background:#f2f2f2;
  position:relative;
}
.container-screen2 .shihou {
  position:absolute;
}
.container-screen2 .shihou2 {
  position:absolute;
  left:100px;
  top:10px;
}
.container-screen2 .shihou3 {
  position:absolute;
  top:100px;
}
.container3 {
  display:flex;
	flex-direction:column;
	width:420px;
  height:160px;
	align-items:center;
	margin:40px 20px;
  border:solid 1px blue;
  box-sizing:border-box;
  background:#ffe5b3;
  padding:8px 60px;
  position:relative;
  left:20px;
  top:-80px;
}
.container3 .shihou4 {
  position:absolute;
  top:0;
  left:40px;
}
.container3 .shihou5 {
  position:absolute;
  left:0;
  top:60px;
}
.container3 .shihou6 {
  position:absolute;
  left:10px;
  bottom:10px;
}
.container3 .shihou7 {
  position:absolute;
  right:80px;
  bottom:30px;
}
.container3 .content3 {
  width:100%;
  height:100%;
  background:#bbff99;
}
.container3 .content3 .etemz3 {
  width:150px;
  height:40px;
  background:#ff80ff;
  box-sizing:border-box;
  border:solid 5px #3366ff;
  padding:5px;
  margin:10px;
  position:relative;
  top:-30px;
}
.container3 .content3 .etemz3 .etemz3-content {
  width:90%;
  height:70%;
  background:#e5ffff;
  position:absolute;
  top:5px;
  left:6px;
}
</style>

