---
title: PWA与angular的Service Worker
date: 2019-02-15 19:14:09
tags: angular
categories: 前端
comments: true
---

# PWA
### 是什么
PWA，全称Progressive Web Apps，是一种接近原生用户体验的渐进增强的 Web 应用，利用现代浏览器特性从安全、性能和体验三个方面，
优化我们的Web App  
PWA不是某一项技术,而是一系列Web技术与标准的集合与应用,由Google在2015年提出，旨在渐进增强，不是替代现有的移动App，
而是让web应用有媲美原生应用的体验，更加对移动环境友好  

### 应用场景
PWA主要应用于移动应用的开发，当前APP对设备内存等资源的需求量越来越大，PWA是颠覆者，旨解决移动设备的硬件限制，以及网络状况不佳下的服务访问
<!--more-->

### 技术特点
PWA之所以能够成为未来的趋势，被广大开发者推崇，还是因为其具有一些优秀的技术特点：

- 安全，通过 HTTPS 来提供服务来防止网络窥探，保证内容不被篡改
- 渐进式，适用于任何设备，无论用户使用什么浏览器
- 响应式，适应任何环境：桌面电脑、智能手机、平板电脑，或者其他设备
- 轻量，web应用更加轻量级，整个APP都在KB占用内，这是原生APP无法比及的
- 离线支持，支持资源缓存，可以在离线或者低质量网络下工作
- 类APP体验，有像原生应用般的体验，可将应用添加到主屏，甚至可以提供没有地址栏的全屏体验
- 可安装，设备浏览器可提示用户安装web应用或添加到主屏上，不需要通过应用商店下载
- 持续更新，受益于service worker的更新进程，应用能够始终保持更新
- 可发现，可识别为“应用程序”，让搜索引擎能够找到 web 应用
- 可再次访问，通过 URL 可以轻松分享应用，不用复杂的安装即可运行
- 可链接，通过 HTTPS 来提供服务来防止网络窥探，保证内容不被篡改

这些技术特点足以让PWA媲美任何一种原生技术开发的移动app，甚至体验更好。除了优秀的使用特点之外，由于PWA属于web应用，对于开发者也带来极大的好处，降低了开发门槛

### PWA的核心技术
PWA的几个关键技术的应用，保证了PWA能够提供期望的性能体验

- Manifest，定义APP图标、标题栏颜色和主题等信息
- Service Worker，管理缓存，离线开发，地理位置信息处理以及app更新等
- App Shell，先显示APP的主结构，再填充主数据，更快显示更好体验
- Push & Notification – 消息推送和通知

####  Manifest
Manifest是一个JSON文件，定义了此Web应用的基本配置、主题颜色等，甚至可以设置横/竖屏、全屏显示

    {
      "name": "PWA Website",
      "short_name": "PWA",
      "description": "An example PWA website",
      "start_url": "/",
      "display": "standalone",
      "orientation": "any",
      "background_color": "#fff",
      "theme_color" : "#fff",
      "icons": [
        {
          "src" : "/images/logo/logo072.png",
          "sizes" : "72x72",
          "type" : "image/png"
        },
        {
          "src" : "/images/logo/logo152.png",
          "sizes" : "152x152",
          "type" : "image/png"
        },
        {
          "src" : "/images/logo/logo192.png",
          "sizes" : "192x192",
          "type" : "image/png"
        },
        {
          "src" : "/images/logo/logo256.png",
          "sizes" : "256x256",
          "type" : "image/png"
        },
        {
          "src" : "/images/logo/logo512.png",
          "sizes" : "512x512",
          "type" : "image/png"
        }
      ]
    }

应该要在头中引用这个文件

    <link rel="manifest" href="/manifest.json">

其中，

- name，应用名称
- short_name，应用短名称。
- description，应用描述。
- start_url，应用起始路径，相对路径，默认为/
- scope，URL范围
- background_color，欢迎页面的背景颜色和浏览器的背景颜色（可选）
- theme_color，应用的主题颜色，一般都会和背景颜色一样
- orientation，优先旋转方向，可选的值有：any, natural, landscape等
- display，显示方式，fullscreen（无Chrome），standalone（和原生应用一样），minimal-ui（最小的一套UI控件集）或者  browser（最古老的使用浏览器标签显示）
- icons，所有图片的数组，每个元素包含了图片的URL，大小和类型

通过manifest.json文件配置，可以设置应用的启动画面、背景图、应用名称，还可以支持应用在主屏桌面上添加一个快捷方式，以方便用户快速访问，
用户访问web应用时，直接在界面中提示添加到主屏桌面，但是目前支持此功能的浏览器不多，安卓手机上须使用57版本以上的谷歌浏览器可以支持该功能，而移动端IOS系统的支持并不好
参考可访问[pwa-demo](https://github.com/blueskyawen/pwa-demo2)

####  Service Worker
Service Worker是PWA中必不可少的一个服务，主要提供资源缓存、离线访问等功能，并可动态进行数据更新  
简单来说，Service Worker 就是一段运行在 Web 浏览器中，并为应用管理缓存的脚本，会拦截所有由应用发出的 HTTP 请求，并选择如何给出响应  
下图展示普通Web App与添加了Service Worker的Web App的差异: 

![service worker2](/images/service-worker2.png) 

其中,Service Worker充当了一个“客户端代理”角色

Service Worker是可编程的，我们可以注册他

    window.addEventListener('DOMContentLoaded', function() {
        if('serviceWorker' in navigator) {
            navigator.serviceWorker.register('/service-worker.js')
                .then(registration => {
                    ...
                }).catch(error => {
                    ...
                });
        }
    });

注册了Service Worker后，它会经历生命周期的各个阶段，同时会触发相应的事件。
整个生命周期包括：  

    installing --> installed --> activating --> activated --> redundant。

用户可以介入周期过程，比如使用event.waitUntil()方法在install事件前处理一些事情，并返回Primise通知服务继续下一阶段；此外，SW还有事件fetch，使用它可以方便的拦截请求和访问缓存  
具体不详述，这里有一篇比较详细的文章介绍SW的[Service Worker全面进阶](https://www.villainhr.com/page/2017/01/08/Service%20Worker%20%E5%85%A8%E9%9D%A2%E8%BF%9B%E9%98%B6#%E7%BC%93%E5%AD%98%E6%8D%95%E8%8E%B7)  
参考可访问[pwa-demo](https://github.com/blueskyawen/pwa-demo2)

#### Push & Notification
Push 和 Notification 是两个不同的功能，涉及到两个 API， 不过将两者相结合是一种常见的使用模式。

- Push：服务器端将更新的信息传递给 SW 
- Notification： SW 将更新的信息推送给用户。

这个能力让我们可以从服务端向用户推送各类消息并引导用户触发相应交互

**1. Push**  
根据Web Push协议草案描述，整个Push流程是这样的

![push service2](/images/push-service2.png) 

其中，“UA”就是我们的用户客户端，也就是浏览器；“Application Server”是后端服务；“Push Service”作为中间代理商，扮演着核心角色。  
Push Service接收客户端的消息订阅，维护管理“客户端url-公钥”对的列表，并将订阅和私钥信息发送给服务器进行存储；此外，它后续还得接收服务端的推送消息，校验并发送给对象的客户端进行展示。
Push Service还有一个非常重要的功能：当用户离线时，可以帮我们保存消息队列，直到用户联网后再发送给他们。目前，不同的浏览器厂商使用了不同的Push Service，chrome使用了自家的FCM，firefox也是使用自家的服务，不同push服务遵循共同的Web Push协议，具有标准的调用方式。  
但是，目前chrome的FCM在国内不可访问，因此我是使用firefox来调试demo的，在 firefox开发者工具的“service workers”里可以看到对应的Push Service  

![push service1](/images/push-service3.png) 

点“start”启动，“调试”进去开发调试，可以试用这个[图书搜索demo](https://github.com/blueskyawen/book-pwa-demo),
通过restclient模拟后端发送推送消息，最后的效果是这样的  

![push msg1](/images/push-msg1.png)   

![push msg2](/images/push-msg2.png) 

通过消息推送API能实现跟原生APP一样消息的体验，但是，国内的PWA应用还不支持这种即时推送，在国外像Facebook、Twitter就做的非常好。  


#### APP Shell
App Shell架构是构建 PWA 应用的一种方式，它通常提供了一个最基本的 Web App 框架，包括应用的头部、底部、菜单栏等结构  
顾名思义，我们可以把它理解成应用的一个「空壳」，这个「空壳」仅包含页面框架所需的最基本的 HTML 片段，CSS 和 javaScript，这样一来，用户重复打开应用时就能迅速地看到 Web App 的基本界面，只需要从网络中请求、加载必要的内容。  
我们使用 Service Worker 对 App Shell 做离线缓存，以便它可以在离线时正常展现，达到类似 Native App 的体验。
开发一个 App Shell, 我们通常要注意以下几点：

- 将动态内容与 Shell 分离
- 尽可能使用较少的数据，实现快速加载
    - 使用本地缓存中的静态资源

### PWA的发展现状及挑战
PWA由谷歌和W3C强力推广，在国外已经是百花齐放，众多的公共应用和知名APP都提供了PWA的版本，比如：推特，Instagram，Flipboard等，并且据说用户体验与原生App相差不大
但是目前在国内的发展并不明显，主要有几个原因

- 作为谷歌的竞争者，苹果态度不明确，支持力度不够
- 国内互联网巨头BAT的挑战，比如：微信小程序，支付宝小程序，凭借强大的用户群体，推广起来非常顺畅
- 移动硬件设备的升级，网络的完善以及随处可见的wifi，使得用户对PWA的需求并没有那么的强烈

虽然在国内知名网站和APP已经陆续有了PWA版本，但是根据个人的使用来讲，真正的PWA应用很少，大部分也只是实现了部分的PWA技术，比如service worker,大家在访问一些网站是可以在开发者工具看到一些运行中的service worker  
  
![service workers](/images/service-workers.jpg)  

但是,只有service worker,没有支持Manifest,消息推送通知等其他技术;目前支持最好最全的应该是[新浪微博](https://m.weibo.cn/beta?pwa=1)  
总之，在国内的发展仍然挑战很大             

### 应用体验
    
    - 手机登录 https://pwa.rocks/，这个网址有很多pwa应用
    - 国内大厂的APP：阿里巴巴 http://m.alibaba.com，新浪微博 https://m.weibo.cn/beta?pwa=1

### PWA的实现
实现 PWA 所需要的特性，主要是围绕着 Service Workers 的基于事件的 cache 系统和消息推送的一套新的 API，此外还需要定义 manifest.json 来定义安装行为或是样式等

# Angular的Service Worker实现

Service Worker是PWA中必不可少的一部分，可提供资源缓存、离线访问等功能  
Service Worker会在首次访问时加载，之后可直接访问，即使关闭了浏览器标签 
使用Service Worker主要是为了改善应用对网络的依赖，提高用户体验  

### 应用
Angular的Service Worker会从服务器上下载一个manifest文件，这个 文件描述了要缓存的资源，并包含每个文件内容的哈希值。     当发布了应用的一个新版本时，manifest的内容就会改变，通知 Service Worker 应该下载并缓存应用的一个新版本了  
Angular使用文件ngsw-config.json来配置Service Worker，指定需呀缓存那些资源以及其它的行为设定，而本应该最终要缓存的资源会记录到生成的ngsw.json文件里

### 使用
在Angular中使用Service Worker像引入一个 NgModule 一样简单  

    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { AppComponent } from './app.component';
    import { ServiceWorkerModule } from '@angular/service-worker';
    import { environment } from '../environments/environment';
    
    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
        ServiceWorkerModule.register('/ngsw-worker.js', { enabled: environment.production })
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule { }

除此之外，ServiceWorkerModule还提供了一些其他的服务来与 Service Worker 进行通信交互， 比如SwUpdate 服务就可以用来要求Service Worker检查和激活更新，并接收响应的版本更新和激活成的通知  

    import { interval } from 'rxjs';
    
    @Injectable()
    export class LogUpdateService {
      constructor(updates: SwUpdate) {
        this.registerUpEvent();
        this.startUpdateCheck();
      }
      
      registerUpEvent() {
        this.updates.available.subscribe(event => {
          console.log('current version is', event.current);
          console.log('available version is', event.available);
          if (promptUser(event)) {
            this.updates.activateUpdate().then(() => document.location.reload());
          }
        });
        this.updates.activated.subscribe(event => {
          console.log('old version was', event.previous);
          console.log('new version is', event.current);
        });
      }
      
      startUpdateCheck() {
          interval(6 * 60 * 60).subscribe(() => updates.checkForUpdate());
      }
    }

### 更新
由于Service Worker使得应用是可以离线运行的，即使本地停止了服务，故应用的版本和清单文件可以随时更新变动。  
而当用户打开或刷新应用程序时，通过查看清单（manifest）文件 “ngsw.json” 的更新来检查该应用程序的更新。如果找到了更新，就会自动下载并缓存这个版本，并在下次加载应用程序时提供。  
此外，用户也手动可以新版本的检查和激活，或定时进行类似操作，确保更新的内容能及时获得，就像上面做的一样  

### ngsw-config配置
Angular使用文件ngsw-config.json来管理应用的缓存，可以配置它来处理相应的资源存储，详细参考官网[Service Worker 配置](https://www.angular.cn/guide/service-worker-config "Service Worker 配置")  
