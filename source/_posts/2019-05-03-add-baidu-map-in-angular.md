---
title: Angular中引入百度地图
date: 2019-05-03 01:37:37
tags: angular
categories: 前端
comments: true
---

最近做项目需要在页面嵌入地图类，好多网站都类似的需求，有使用谷歌地图的，也有使用高德地图的。由于在国内，于是考虑引入百度地图，研究了一下使用方式，记录下来。   
<!--more-->

百度地图有开放的开放平台，只要获取到对应的应用AK即可方便使用，几个步骤可完成：

 - 首先，进入[百度地图开放平台](http://lbsyun.baidu.com/)，注册并登录百度帐号
 - 进入控制台，选择“创建应用”，获取AK，这是使用百度地图的关键；创建应用时可以选择应用类型，只在浏览器端使用的话请选择“浏览器端”，白名单一定要设置，不限制访问域名的话可填“**”
 - 查看创建的应用，应用的AK码串就是之后要使用的

有了AK，我们便可以在在我们的应用中使用百度地图了，根据需求可以参考平台的开发文档进行开发使用，本次我们使用的javascript api在web端引入百度地图。  
但是在angular项目中引如百度地图却有几个不同的使用方式，有原生API调用和模块化使用方式。

## 原生API引入地图
首先在index.html引入百度地图API，在head里添加

    <script type="text/javascript" src="http://api.map.baidu.com/api?v=3.0&ak=your_ak"></script>

在组建中使用

    //map.html
    <div id="map-container" class="map-content"></div>

    //map.ts
    import { Component, OnInit } from '@angular/core';
    declare var BMap: any;
    declare var BMAP_ANIMATION_BOUNCE: any;
    declare var BMAP_NAVIGATION_CONTROL_SMALL: any;
    declare var BMAP_MAPTYPE_CONTROL_MAP: any;
    declare var BMAP_ANCHOR_TOP_LEFT: any;
    declare var BMapLib: any;
    
    @Component({
      selector: 'app-baidu-map-for-js',
      templateUrl: './baidu-map-for-js.component.html',
      styleUrls: ['./baidu-map-for-js.component.less']
    })
    export class BaiduMapForJsComponent implements OnInit {
        map: any;
        point: any;
        scrollZoom: boolean = false;
        bDragg: boolean = true;
        overviewMap: any;
        openOverview: boolean = false;
        cityControl: any;
        showCityControl: boolean = false;
        marker: any;
        circle: any;
        showMarker: boolean = true;
        myDis: any;
        isCeju: boolean = false;
        constructor() { }
      
        ngOnInit() {
            // 创建地图实例
            this.map = new BMap.Map("map-container");
            // 初始化地图，设置中心点坐标和地图级别
            this.point = new BMap.Point(121.5097199, 31.2425584);
            this.map.centerAndZoom(this.point, 15);
        
            //添加地图控件
            this.addNavigationControl();
            this.addOverviewMap();
            this.map.addControl(new BMap.MapTypeControl({type: BMAP_MAPTYPE_CONTROL_MAP}));
    
            //添加标记和信息框
            this.addMaker();
            this.addInfoWindow();
        }
      
        addNavigationControl() {
            let opts = {type: BMAP_NAVIGATION_CONTROL_SMALL,enableGeolocation: true};
            this.map.addControl(new BMap.NavigationControl(opts));
        }
        
        addOverviewMap() {
            this.overviewMap = new BMap.OverviewMapControl({isOpen:this.openOverview });
            this.map.addControl(this.overviewMap);
            this.overviewMap.addEventListener("viewchanged", (e) => {
                this.openOverview = e.isOpen;
            });
        }
        
        addMaker() {
            this.circle = new BMap.Circle(this.point,500,{strokeColor:"blue", strokeWeight:2, strokeOpacity:0.3}); //创建圆
            this.map.addOverlay(this.circle);
            this.marker = new BMap.Marker(this.point);        // 创建标注
            this.map.addOverlay(this.marker);                     // 将标注添加到地图中
            this.marker.setAnimation(BMAP_ANIMATION_BOUNCE); //跳动的动画
        }

        addInfoWindow() {
            var opts = {
                width : 150,     // 信息窗口宽度
                height: 80,     // 信息窗口高度
                title : "陆家嘴"  // 信息窗口标题
                //enableMessage:true,//设置允许信息窗发送短息
                //message:"亲耐滴，晚上一起吃个饭吧？戳下面的链接看下地址喔~"
            };
            var infoWindow = new BMap.InfoWindow("上海金融中心！！", opts);  // 创建信息窗口对象
            this.marker.addEventListener("click", () => {
                this.map.openInfoWindow(infoWindow,this.point); //开启信息窗口
            });
        }
    }

效果如： [百度地图-使用原生API](http://blueskyawen.com/angular-work-cook/main/other/baiduMap/usejs)

需要更多地图API可参考: [百度地图JavaScript API](http://lbsyun.baidu.com/index.php?title=jspopular3.0/guide)

## 模块化组件引入地图
angular有个已封装的百度地图使用模块angular2-baidu-map，首先安装模块包，

    npm install angular2-baidu-map --save-dev
    
在根模块中引入module，

    import { BrowserModule } from '@angular/platform-browser'
    import { NgModule } from '@angular/core'
    import { AppComponent } from './app.component'
     
    import { BaiduMapModule } from 'angular2-baidu-map'
     
    @NgModule({
      declarations: [AppComponent],
      imports: [BrowserModule, 
        BaiduMapModule.forRoot({ ak: 'your_ak' })],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule {}
    
在组件画地图，

    //map.component.html
    <baidu-map [options]="options" (loaded)="mapLoaded($event)"></baidu-map>
    
    //map.component.ts
    import { Component, OnInit, ElementRef, AfterViewInit } from '@angular/core';
    import {ControlAnchor, MapOptions, MapTypeControlOptions, MapTypeControlType,NavigationControlOptions, NavigationControlType, OverviewMapControlOptions,ScaleControlOptions, BNavigationControl, BOverviewMapControl, MarkerOptions, Animation,MapTypeEnum, GeolocationControlOptions, CircleOptions, Point, BCircle, BMarker, BMapInstance} from 'angular2-baidu-map'

    @Component({
        selector: 'app-baidu-map-for-module',
        templateUrl: './baidu-map-for-module.component.html',
        styleUrls: ['./baidu-map-for-module.component.less']
    })
    export class BaiduMapForModuleComponent implements OnInit {
        options: MapOptions;
        constructor() { }
        
        ngOnInit() {
            this.options = {
                centerAndZoom: {
                    lat: 30.241628,
                    lng: 120.138121,
                    zoom: 13
                },
                enableKeyboard: true
            };
        }
    }
    
添加控件和标记，

    //map.component.html
    <baidu-map [options]="options" (loaded)="mapLoaded($event)">
        <control type="navigation" [options]="controlOpts"></control>
        <control type="maptype" [options]="mapTypeOpts"></control>
        <control type="scale" [options]="scaleOpts"></control>
        <control type="overviewmap" [options]="overviewmapOpts" (loaded)="overviewLoaded($event)"></control>
        <circle [center]="center" radius="1200" [options]="circleOptions" (loaded)="circleLoaded($event)"></circle>
        <marker *ngFor="let marker of markers" [point]="marker.point" [options]="marker.options" (clicked)="showWindow($event)" (loaded)="markerLoaded($event)" ></marker>
    </baidu-map>
  
    //map.component.ts
    public controlOpts: NavigationControlOptions;
    public overviewmapOpts: OverviewMapControlOptions;
    public scaleOpts: ScaleControlOptions;
    public mapTypeOpts: MapTypeControlOptions;
    public geolacOpts: GeolocationControlOptions;
    private BMAP_NAVIGATION_CONTROL_SMALL: any;

    public center: Point;
    public circleOptions: CircleOptions;
    public markers: Array<{ point: Point; options?: MarkerOptions }>;
  
    ngOnInit() {
        //添加控件
        this.addNavigationControl();
        this.addMapTypes();
        this.addOverviewMap();
        //添加标记
        this.addCircle();
        this.addMarker();
    }
    
    addNavigationControl() {
        this.controlOpts = {
            anchor: ControlAnchor.BMAP_ANCHOR_TOP_LEFT,
            type: NavigationControlType.BMAP_NAVIGATION_CONTROL_SMALL,
            enableGeolocation: true
        };
    }
    
    addMapTypes() {
        this.mapTypeOpts = {
            type: MapTypeControlType.BMAP_MAPTYPE_CONTROL_MAP,
            mapTypes: [MapTypeEnum.BMAP_NORMAL_MAP, MapTypeEnum.BMAP_SATELLITE_MAP]
        }
    }
    
    addOverviewMap() {
        this.overviewmapOpts = {
            anchor: ControlAnchor.BMAP_ANCHOR_BOTTOM_RIGHT,
            isOpen: this.showFlagss.overview
        }
    }
    
    mapLoaded(emap: any) {
        this.mapInstance = emap;
    }
    
    addCircle() {
        this.center = {
            lat: 30.241628,
            lng: 120.138121
        };
        
        this.circleOptions = {
            strokeColor: 'blue',
            strokeWeight: 2,
            strokeOpacity:0.3
        };
    }

    addMarker() {
        this.markers = [
            {
                point: {
                    lat: 30.241628,
                    lng: 120.138121
                }
            },
            {
                options: {
                    icon: {
                        imageUrl: '***.jpg',
                        size: {
                            height: 80,
                            width: 50
                        },
                        imageSize: {
                            height: 80,
                            width: 50
                        },
                        imageOffset: {
                            height: 0,
                            width: 0
                        }
                    }
                },
                point: {
                    lat: 30.3006,
                    lng: 120.200
                }
            }
        ]
    }

    circleLoaded(circle: BCircle) {
        this.circleIntance = circle;
        if(!this.showFlagss.maker) {
            this.circleIntance.hide();
        }
    }
    
效果如： [百度地图-使用模块包](http://blueskyawen.com/angular-work-cook/main/other/baiduMap/usemodule)

需要更多地图效果可参考: [angular2-baidu-map ](https://leftstick.github.io/angular2-baidu-map/)

> angular2-baidu-map封装了常用的百度地图的接口，但与百度原生API相比，少了很多的使用api，比如城市列表，热力图等，可根据需求选择使用

