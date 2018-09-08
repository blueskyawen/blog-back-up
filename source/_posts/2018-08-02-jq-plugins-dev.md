---
title: JQuery-自写插件集一
date: 2018-08-02 23:04:55
tags: Jquery
categories: 前端
comments: true
---

在熟悉了jquery和插件写法之后，作者将之前使用原生js和jquery写的一些组件，封装成插件，权当兴趣
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

### table表格

[插件代码](https://runjs.cn/code/e3pqshbw)

使用方式：

    <table id="table1"></table>

    $(function() {
        var columnDefs = [{checkbox: true},
                        {field: 'name',title: '姓名'},
                        {field: 'sex',title: '性别'},
                        {field: 'address',title: '地址'}];
        var rowData = [{name:'张山',sex:'男',address:'上海'},
                    {name:'李四',sex:'男',address:'苏州'},
                    {name:'王五',sex:'女',address:'嘉兴'},
                    {name:'张麻子',sex:'男',address:'宁波'},
                    {name:'赵六',sex:'男',address:'长沙'}];
        $.fn.bootTable.defaultOptions = {odd: 'odd',
                                        even: 'even',
                                        selected:'checked'};
        $("#table1").bootTable({id: 'table1',
                                caption: '表格标题',
                                column: columnDefs,
                                data: rowData,
                                option: {odd: 'odd2',
                                        even: 'even2'}
                                });
         //获取已选记录
         var selection = $("#table1").bootTable('getSelection');

    });

效果：[bootTable表格](https://sandbox.runjs.cn/show/e3pqshbw)

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






