---
title: JQuery-自写插件集二
date: 2018-08-02 23:04:55
tags: Jquery
categories: 前端
comments: true
---
jquery封装成插件
<!--more-->

### Collapse折叠面板

[插件代码](https://runjs.cn/code/lmfma6gm)

使用方式：

    <div class="collapseset" id="collapset1">
        <div class="collapse">
            <header><span></span>
            杨过</header>
            <article>杨过，名过，字改之，是金庸武侠小说《神雕侠侣》[1]  中的主人公，前作《射雕英雄传》中杨康与穆念慈之子，西毒欧阳锋的义子。名字为郭靖、黄蓉所取，取“有过则改之”之意。</article>
        </div>
        <div class="collapse">
            <header><span></span>
            小龙女</header>
            <article>小龙女，是金庸小说《神雕侠侣》的女主角，金庸笔下广受读者喜爱的女角之一。容貌秀美若仙、清丽绝俗。性格宽容恬淡、天真可爱。武功轻灵飘逸，于婀娜妩媚中击敌制胜。</article>
        </div>
        <div class="collapse">
            <header><span></span>
            黄药师</header>
            <article>黄药师，外号“东邪”，天下“五绝”之一，是金庸武侠小说《射雕英雄传》和《神雕侠侣》中的角色。是黄蓉之父，对其妻冯氏（小字阿衡）一往情深。。  </article>
        </div>
    </div>

    $(function() {
	    $("#collapset1").bootCollapse({type: 'normal'});
	    $("#collapset2").bootCollapse({type: 'eachOne'});
    });

效果：[折叠面板](https://sandbox.runjs.cn/show/lmfma6gm)

### Model模态框

[插件代码](https://runjs.cn/code/hmmqzjc9)

使用方式：

    <div class="dialog-container" id="model1">
        <div class="shade"></div>
        <div class="myDialog">
            <header>
                <span>操作名称</span>
                <span class="cancel">×</span>
            </header>
        <article>
            <div class="description">
                <span class="grade">!</span>
                <span class="text">HTML 4 的新特性之一是可以使 HTML 事件触发浏览器中的行为，比方说当用户点击某个 HTML 元素时启动一段 JavaScript.</span>
            </div>
            <form>
                <div class="form-group">
                    <label>姓名</label>
                    <div class="group-right">
                        <input type="text" name="_name" value="Jack" maxlength="8"
                        autocomplete="on"  required />
                    </div>
                </div>
                <div class="form-group">
                    <label>技能</label>
                    <div class="group-right">
                        <input type="text" name="_skill" value="" autocomplete="on" />
                    </div>
                </div>
            </form>
        </article>
        <footer>
            <div class="buttons">
                <button class="confirm">确定</button>
                <button class="cancel">取消</button>
            </div>
        </footer>
        </div>
    </div>

    $(function() {
	    $("#model1").bootModel();

	    $("#model1").bootModel('show');
        $("#model1").bootModel('hide');
    });

效果：[Model模态框](https://sandbox.runjs.cn/show/hmmqzjc9)

### 轮播图

[插件代码](https://runjs.cn/code/tbziv3gx)

使用方式：

    <div id="carousel-fade"></div>      //渐变
    <div id="carousel-slide"></div>     //滑动
	<div id="carousel-slide-data-api"></div>   //使用data-api

    $(function() {
	var optionss = [
            {name: 'fuzhi',
             url: 'http://dimg04.c-ctrip.com/images/t1/vacations/203000/202469/ef1ae815bbef4a0698d4fa2ac15614e0.jpg'},
            {name: 'qingshuisi',
             url: 'http://upload.shunwang.com/2014/0625/1403670070972.JPG'},
            {name: 'xiaodao',
             url: 'http://digitalphoto.cocolog-nifty.com/photos/uncategorized/2008/08/02/dsc_d300_0006950_2.jpg'},
            {name: 'xiaodao2',
             url: 'http://f4.topitme.com/4/b3/e5/1118294546054e5b34o.jpg'},
            {name: 'xiaodao3',
             url: 'http://img17.3lian.com/d/file/201701/19/fd92ea2409b6b157c247b78ce4eda95a.jpg'}
	];

	$("#carousel-fade").bootCarousel({options: optionss,type: 'fade'});
	$("#carousel-slide").bootCarousel({options: optionss,type: 'slide'，
	                                    interval:5000,pause:true});
    $("#carousel-slide-data-api").bootCarousel({options: optionss,
                                                type: 'fade',
                                                dataApi:true});

    //手动切换项目
    $("#carousel-fade").bootCarousel('prev');
    $("#carousel-fade").bootCarousel('next');
    $("#carousel-fade").bootCarousel(Num);

    //手动启停自动循环播放
    $("#carousel-slide").bootCarousel('cycle'); //启动
    $("#carousel-slide").bootCarousel('pause'); //停止

    //自定义事件
    $("#carousel-slide").on("change.bc.carousel",function() {
        count++;
    });
    $("#carousel-slide").on("changed.bc.carousel",function() {
        count++;
    });

效果：[轮播图](https://sandbox.runjs.cn/show/tbziv3gx)

### dropdown下拉菜单

[插件代码](https://runjs.cn/code/j5y0shd5)

使用方式：

    <div class="dropdown" id="dropdown1">
        <button class="dropbtn dropdown-toggle">下拉菜单
            <span class="caret"></span>
        </button>
        <ul class="dropdown-content menu-width">
            <li class="dropdown-item">
            <a tabindex="-1" href="javascript:void(0);">新浪</a>
            </li>
            <li class="dropdown-item">
            <a tabindex="-1" href="javascript:void(0);">搜狐</a>
            </li>
            <li  class="divider"></li>
            <li class="dropdown-item">
            <a tabindex="-1" href="javascript:void(0);">小米</a>
            </li>
        </ul>
    </div>

    $(function() {
	$("#dropdown1").bootDropdown();
	$("#dropdown2").bootDropdown({place:'right'});
	$("#dropdown3").bootDropdown({place:'up'});
	$("#dropdown4").bootDropdown({type: 'click',place:'left'});
	$("#dropdown5").bootDropdown({type: 'hover',place:'down'});
	$("#dropdown6").bootDropdown({type: 'hover',place:'right'});
	$("#dropdown7").bootDropdown({type: 'hover',place:'up'});
	$("#dropdown8").bootDropdown({type: 'hover',place:'left'});

       //手动切换菜单
	$("#dropdown1").bootDropdown('toggle');
    });

效果：[dropdown下拉菜单](https://sandbox.runjs.cn/show/j5y0shd5)

### TAB选项卡

[插件代码](https://runjs.cn/code/0gamemrt)

使用方式：

    <div class="nav-tabs" id="tab1">
        <ul class="tab-bars">
            <li  class="disabled" name="html">html</li>
            <li name="css">css</li>
            <li name="dom">dom</li>
            <li name="javascript">javascript</li>
        </ul>
        <div class="tab-content" >
            <div name="html">Html</div>
            <div name="css">Css</div>
            <div name="dom">Dom</div>
            <div name="javascript">Javascript</div>
        </div>
    </div>

    $(function() {
        $("#tab1").bootTab();
        $("#tab2").bootTab({defaultActive: 0});
        $("#tab3").bootTab({defaultActive: ‘css’});

        //手动激活选项卡
        $("#tab3").bootTab(0);
        $("#tab3").bootTab('javascript');
    });

效果：[TAB选项卡](https://sandbox.runjs.cn/show/0gamemrt)

### scrollspy滚动监听

[插件代码](https://runjs.cn/code/g88xu3yo)

使用方式：

    <nav class="navbar" id="navbar-example">
    ...
    </nav>
    <div class="mavcontent" data-target="#navbar-example" id="mymavcontent">
    ...
    </div>

    $(function() {
        $("#mymavcontent").bootScrollspy({id:'navbar-example',offset:10});
    });

效果：[滚动监听](https://sandbox.runjs.cn/show/g88xu3yo)

### Affix附加导航

[插件代码](https://runjs.cn/code/bzsqbghc)

使用方式：

    <div class="container">
       <div class="jumbotron">
            <h1>Bootstrap Affix</h1>
        </div>
        <div class="row">
            <div class="scrollspy" id="myScrollspy">
                <ul class="nav" id="myAffix">
                    <li><a href="#section-1">第一部分</a></li>
                    <li><a href="#section-2">第二部分</a></li>
                    <li><a href="#section-3">第三部分</a></li>
                    <li><a href="#section-4">第四部分</a></li>
                    <li><a href="#section-5">第五部分</a></li>
                </ul>
            </div>
            <div class="content" data-target="#myScrollspy">
                <h2 id="section-1">第一部分</h2>
                <p>...</p>
                <hr>
                <h2 id="section-2">第二部分</h2>
                <p>...</p>
                <hr>
                <h2 id="section-3">第三部分</h2>
                <p>... </p>
                <hr>
                <h2 id="section-4">第四部分</h2>
                <p>...</p>
                <hr>
                <h2 id="section-5">第五部分</h2>
                <p>...</p>
            </div>
        </div>

        <div class="footer">
         ...
        </div>
    </div>

    $(function() {
        $("#myAffix").bootAffix({id:'myAffix',offset:20});
    });

效果：[Affix附加导航](https://sandbox.runjs.cn/show/bzsqbghc)