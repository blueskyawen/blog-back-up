---
title: JQuery-自写插件集
date: 2018-09-12 23:04:55
tags: Jquery
categories: 前端
comments: true
---

在熟悉了jquery和插件写法之后，作者将之前使用原生js和jquery写的一些组件，封装成插件
<!--more-->

### switch开关

[插件代码](https://runjs.cn/code/0eao8cw2)

使用方式：

    <div id="switch1"></div>
    <div id="switch2"></div>

    $(function() {
        $("#switch1").bootSwitch({titles: [],
                                defalutCheck: true});
        $("#switch2").bootSwitch({titles:['开','关'],
                                defalutCheck: true});

        $(".tips button").click(function() {
        $(".tips span").text($("#switch1").bootSwitch('getChecked'));
        });

        //获取开关状态
        $("#switch1").bootSwitch('getChecked');
        //添加自定义事件
        $("#switch1").on("change.bc.switch",function() {
            console.log('switch1 change');
        });
    });

效果：[switch开关](https://sandbox.runjs.cn/show/0eao8cw2)

### select下拉选择

[插件代码](https://runjs.cn/code/3gpjwzds)

使用方式：

    <div id="select-normal"></div>   //正常下拉选择
    <div id="select-small"></div>    //正常下拉选择--小尺寸
    <div id="select-disable"></div>  //禁止下拉选择

    $(function() {
        var options1 = [
            {value: 'paletxUI',name: 'paletxUI'},
            {value: 'Iaas',name: 'Iaas'},
            {value: 'Paas',name: 'Paas'},
            {value: 'Saas',name: 'Saas'},
            {value: 'CMS',name: 'CMS'}
        ];
        $("#select-normal").bootSelect({id: 'select-normal',
                                    options: options1,
                                    defalutChecked: '',
                                    placeholder: '请选择'});
        $("#select-small").bootSelect({id: 'select-normal',
                                    options: options1,
                                    defalutChecked: '',
                                    placeholder: '请选择',
                                    width: 220});
        $("#select-disable").bootSelect({id: 'select-disable',
                                    options: options1,
                                    defalutChecked: '',
                                    placeholder: '请选择',
                                    disable: true});
    });

效果：[下拉选择](https://sandbox.runjs.cn/show/3gpjwzds)

### multilSelect多选下拉选择

[插件代码](https://runjs.cn/code/byn8l5d7)

使用方式：

    <div id="mutil-select01"></div>    //多选下拉选择框
    <div id="mutil-select02"></div>    //禁止多选下拉选择
    <div id="mutil-select03"></div>    //带复选框的下拉选择
    <div id="mutil-select04"></div>    //带全选的下拉选择
    <div id="mutil-select05"></div>    //显示选择个数的-带复选框
    <div id="mutil-select06"></div>    //显示选择个数的-普通
    <div id="mutil-small"></div>    //多选下拉选择框-小尺寸
    <div id="mutil-big"></div>    //多选下拉选择框-大尺寸

    $(function() {
	var optionsss = [
            {value: 'paletxUI-paletxUI-paletxUI-paletxUI-paletxUI-paletxUI-paletxUI-paletxUI',
             name: 'paletxUI-paletxUI-paletxUI-paletxUI-paletxUI-paletxUI-paletxUI-paletxUI'},
            {value: 'Paas-Paas',name: 'Paas-Paas'},
            {value: 'Iaas-Iaas',name: 'Iaas-Iaas'},
            {value: 'Saas-Saas',name: 'Saas-Saas'},
            {value: 'CMS-CMS',name: 'CMS-CMS'}
	];
        var defaultSelects = [
            {value: 'Iaas-Iaas',name: 'Iaas-Iaas'},
            {value: 'CMS-CMS',name: 'CMS-CMS'}
	];
	var disableOptions = [
            {value: 'CMS-CMS',name: 'CMS-CMS'}
	];
	$("#mutil-select01").bootMultilSelect({options: optionsss,												defalutSelected: defaultSelects,
                                        disable: disableOptions});
	$("#mutil-select02").bootMultilSelect({options: optionsss,
					defalutSelected: defaultSelects,
					disable: disableOptions,
					disabled:true});
	$("#mutil-select03").bootMultilSelect({options: optionsss,
					defalutSelected: defaultSelects,
					disable: disableOptions,
					checkbox:true});
	$("#mutil-select04").bootMultilSelect({options: optionsss,
					defalutSelected: defaultSelects,
					disable: disableOptions,
					checkbox:true,
					allcheck:true});
	$("#mutil-select05").bootMultilSelect({options: optionsss,
					defalutSelected: defaultSelects,
					disable: [],
					checkbox:true,
					allcheck:true,
					showNum:true});
	$("#mutil-select06").bootMultilSelect({options: optionsss,
					defalutSelected: defaultSelects,
					disable: [],
					showNum:true});
	$("#mutil-small").bootMultilSelect({options: optionsss,
					defalutSelected: defaultSelects,
					disable: [],
					width: 220});
	$("#mutil-big").bootMultilSelect({options: optionsss,
					defalutSelected: defaultSelects,
					disable: [],
					checkbox:true,
					width: 620});

        //添加选择变更事件
        $("#mutil-select01").on("change.bc.multilselect",function() {
            console.log('multilselect change');
        });
        //获取已选记录
        var selection = $("#mutil-select01").bootMultilSelect('getSelection');
    });

效果：[multil多选下拉框](https://sandbox.runjs.cn/show/byn8l5d7)

### transfer穿梭框

[插件代码](https://runjs.cn/code/szon4uz2)

使用方式：

    <div id="myTranfer"></div>

    $(function() {
        var sourceItems = [{value:'content01',name:'content01'},
                            {value:'content02',name:'content02'},
                            {value:'content03',name:'content03'},
                            {value:'content04',name:'content04'},
                            {value:'content05',name:'content05'},
                            {value:'content06',name:'content06'},
                            {value:'content07',name:'content07'},
                            {value:'content08',name:'content08'}];
        var targetItems = [{value:'content09',name:'content09'},
                            {value:'content10',name:'content10'}];

        $("#myTranfer").bootTransfer({sourOptions: sourceItems,
                                    targOptions: targetItems,
                                    disableItems: [],
                                    itemText:'项目',
                                    sourceText:'源',
                                    targetText:'目标'});

        //获取源记录
        $("#myTranfer").bootTransfer('getSource');
        //获取目标记录
        $("#myTranfer").bootTransfer('getTarget');
    });

效果：[transfer穿梭框](https://sandbox.runjs.cn/show/szon4uz2)

### Info通知

[插件代码](https://runjs.cn/code/hckrhfgi)

使用方式：

    <div id="success"></div>
    <div id="error"></div>
    <div id="warn"></div>

    $(function() {
        var $successInfo = $("#success").bootInfo({type: 'success',
                                             title: '操作成功,2s后自动消失',
                                             timelen: 2000});
        var $warnInfo = $("#warn").bootInfo({type: 'warn',
                                            title: '我是警告消息!'});
        var $errorInfo = $("#error").bootInfo({type: 'error',
                                            title: '我是错误消息!'});

        //展现通知
        $successInfo.bootInfo('show');
        $warnInfo.bootInfo('show');
        $errorInfo.bootInfo('show');
        //隐藏通知
        $warnInfo.bootInfo('hide');
        $errorInfo.bootInfo('hide');
    });

效果：[Info通知](https://sandbox.runjs.cn/show/hckrhfgi)

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




