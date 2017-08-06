---
title: Angular/cli-命令集
date: 2017-07-23 21:52:01
tags: Augular
categories: 前端工具
comments: true
---

Angular CLI是一个命令行界面工具，它可以创建项目、添加文件以及执行一大堆开发任务，比如测试、打包和发布等
使用Angular-CLI工具可以快速创建代码框架，减少重复性开发的工作量，而且创建的应用和文件符合风格指南的推荐风格，开发者可以从中获益
<!--more-->

## 基本命令
查询版本,获取帮助

    ng help
    ng -v
新建项目，启动应用

    ng new projectName
    cd projectName
    ng serve --open
指定主机和端口号启动

    ng serve --host 0.0.0.0 --port 4201 --open

工具支持相对目录形式的创建命令，比如,当前所处目录/home/app：

    ng g component login           //在/home/app目录下创建
    ng g component login  ../my    //在/home/my目录下创建
    ng g component login  AA       //在/home/app/AA目录下创建


## 核心命令
#### ng new
ng new my-app
默认生成同名目录，并初始化一个angular应用框架
Option选项：
+ `--`directory (简写:-dir) ，指定新建项目的目录名，比如：ng new my-app -dir AAA;不指定则默认以应用名命名目录
+ --dry-run (简写:-d) ，默认值: false，生成项目后立即run起来，列出所有生成的项目文件
- --inline-style (简写: -is) ，默认不加时为false，，指定生成应用的组件为行内样式，即样式表位于元数据的styles[]内
- --inline-template (简写: -it) ，默认不加时为 false，指定生成应用的组件为行内模板，即模板位于元数据的templates[]内
- --minimal，默认值: false，创建最小化APP
- --prefix (简写: -p) 默认: app，指定生成文件的选择器selector的前缀，也可在.angular-cli.json文件中修改
- --routing， 默认不加时为 false，指定生成相应的路由模块
- --skip-commit (简写: -sc) 默认不加时为 false，指定忽略掉应用提到到库
- --skip-git (简写: -sg)  默认不加时为 false，指定忽略掉初始化git仓库
- --skip-install (简写: -si) 默认不加时为: false，指定忽略掉安装packages
- --skip-tests (简写: -st) 默认不加时为: false，指定忽略掉安装测试文件spec/e2e
- --source-dir (简写: -sd) 默认值: src，指定生成项目的源文件目录名，也可在.angular-cli.json (apps[0].root)中修改
- --style，默认值: css，指定css文件类型（css/scss/less/sass/styl (stylus)），也可在文件.angular-cli.json (defaults.styleExt)中修改
- --verbose (简写: -v)， 默认不加时为: false，生成时输出log日志

#### ng serve
ng serve
编译应用并启动web服务
Option选项：
- `--`host (简写: -H)， 不加时的默认值: localhost，指定启动的主机
- --hmr，不加时的默认值: false，是否启动热模块替换
- --live-reload (简写: -lr)，不加时的默认值: true，是否在页面变更时重新载入
- --public-host (简写: --live-reload-client)，说明浏览器使用的URL
- --disable-host-check，默认值: false，不检测已连接的host
- --open (简写: -o)， 不加时默认值: false，指定启动后自动打开浏览器呈现
- --port (简写: -p)， 默认值 4200，指定服务的监听端口
- --ssl，使用HTTPS
- --ssl-cert (简写: -)，默认值SSL:
- --ssl-key，指定使用 serving HTTPS的SSL key
- --aot，指定进行预编译
- --base-href (简写: -bh)，指定应用项目的基准路径,指的是其他应用文件现对于index.html的相对路径，在.angular-cli.json文件可设置根目录和index的路径，如root/src/index,可设-bh ../
- --deploy-url (简写: -d)，指定文件部署的URL
- --environment (简写: -e)，定义编译环境
- --extract-css (简写: -ec)，指定从全局样式表抽取样式
- --i18n-file，使用i18n国际化
- --i18n-format，指定--i18n-file国际化文件的格式
- --locale，Locale to use for i18n.
- --output-hashing (简写: -oh) ，定义the output filename cache-busting hashing mode. Possible values: none, all, media, bundles
- --output-path (简写: -op) ，指定输出文件的路径
- --poll，Enable and define the file watching poll time period (milliseconds) .
- --progress (简写: -pr)， 默认值: true，是否显示构建过程
- --sourcemap (简写: -sm, sourcemaps)，Output sourcemaps.
- --target (简写: -t, -dev, -prod) 默认值: development，指定编译方式
- --vendor-chunk (简写: -vc)，默认值: true，Use a separate bundle containing only vendor libraries.
- --verbose (简写: -v)， 默认值: false，生成时输出log日志
- --watch (aliases: -w)，添加后文件变更会触发重新构建

#### ng generate
ng generate 类型 文件名 或 ng g 类型 文件名
帮助开发者生成开发文件，类和相应代码骨架，比如：模块，组件，指令等，方便快捷开发，下面一行命令将自动生成组件的目录，和相关的css/ts/html文件

    ng g component my-comp

** （1） module**
ng g module module_name
生成特性模块
Option选项：
- `--`app (简写: -a)， 默认值: 1st app
- --flat，指定是否新建同名目录存放模块文件
ng g module 或 ng g module --flat=false ： 新建同名目录
ng g module --flat或 ng g module --flat=true : 不新建同名目录
- --module (简写: -m)，说明模块在哪导入
- --spec，指定是否连带创建spec测试文件
ng g module 或 ng g module --spec=false： 同步创建spec
ng g module --spec 或 ng g module --spec=非false: 不创建同名spec
- --routing，指定是否连带创建路由模块，forChild()
ng g module 或 ng g module --routing=false： 不同步创建路由模块
ng g module --routing 或  ng g module --routing=非false : 创建路由模块

** （2） component**
ng g component
生成组件
Option选项：
- `--`app (简写: -a) 默认值: 1st app
- --change-detection (简写: -cd)，不知道什么用
- --flat， 默认值: false，指定是否新建组件目录
ng g component  或 ng g component --flat=false: 新建目录
ng g component  --flat 或 ng g component --flat=true: 不新建目录
- --export，默认值: false，指定是否公开该组件
ng g component  或 ng g component --export=false: 不公开
ng g component  --export 或 ng g component --export=true: 公开
- --inline-style (简写: -is)，默认值: false，指定组件是否行内样式
ng g component  或 ng g component -is=false: 不是行内样式
ng g component  -is 或 ng g component -is=true: 是行内样式
- --inline-template (简写: -it)， 默认值: false，指定组件是否行内模板
ng g component  或 ng g component -it=false: 不是行内模板
ng g component  -it 或 ng g component -it=true: 是行内模板
- --module (简写: -m)，指定是否更新模块，默认更新
- --prefix，指定是否去掉组件selector选择器的前缀
ng g component  或 ng g component --prefix=false: 不加前缀
ng g component  --prefix 或 ng g component --prefix=true: 默认加前缀
- --skip-import，默认值: false，Allows for skipping the module import.
- --spec，指定是否连带创建spec测试文件
ng g component --spec=false: 不创建spec
ng g component --spec=true 或 ng g component --spec 或 ng g component: 创建spec
- --view-encapsulation (简写: -ve)，Specifies the view encapsulation strategy

> 命令生成的组件默认更新于距离最近的module模块的declares数组里，比如当前所处目录为：/app/main/login/,生成组件后自动注册到本目录的模块
中，如果本目录下无模块，则查找上一层目录的模块进行注册，指导根模块

**（3） directive**
ng g directive name
用于生成属性指令相关文件和类 ，选项使用方式基本和component类似
Option选项：
- `--`app (简写: -a) 默认值: 1st app
- --export default value: false
- --flat，ng g component --flat=false: 新建目录，其他情况均不新建
- --module (aliases: -m)
- --prefix
- --skip-import
- --spec

生成的指令类默认是这样的

    mport { Directive } from '@angular/core';
    @Directive({selector: '[appA]'})
    export class ADirective {
      constructor() { }
    }

**（4） pipe**
ng g pipe name
用于生成管道相关文件和类 ，选项使用方式和directive类似
Option选项：
- `--`app
- --export
- --flat
- --module (aliases: -m)
- --skip-import
- --spec

生成的管道类默认是这样的

    import { Pipe, PipeTransform } from '@angular/core';
    @Pipe({  name: 'my'})
    export class MyPipe implements PipeTransform {
      transform(value: any, args?: any): any {
        return null;
      }
    }

**（5） service**
ng g service name
用于生成服务相关文件和类 ，选项使用方式和directive类似
Option选项：
- `--`app (aliases: -a)
- --flat
- --module (aliases: -m)
- --spec

生成的可注入服务默认是这样的

    import { Injectable } from '@angular/core';
    @Injectable()
    export class LoginService {
      constructor() { }
    }

**（6） guard**
ng g guard name
生成路由守卫类和相关文件
Option选项：
- `--`app
- --flat
- --module (aliases: -m)
- --spec

生成的守卫类默认是这样的

    import { Injectable } from '@angular/core';
    import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
    import { Observable } from 'rxjs/Observable';

    @Injectable()
    export class AdminGuard implements CanActivate {
      canActivate(
        next: ActivatedRouteSnapshot,
        state: RouterStateSnapshot): Observable<boolean> | Promise<boolean> | boolean {
        return true;
      }
    }

**（7） class**
ng g class name
生成类
Option选项：
- `--`app
- --spec

生成的类默认是这样的

    export class Hero {
    }

**（8） interface**
ng g interface name type
Option选项：
- `--`app
- type

生成的接口默认是这样的

    export interface Name {
    }

** （） enum**
ng g enum name
Option选项：
- `--`app

生成的enum默认是这样的

    export enum HERO {
    }


#### ng lint
ng lint
命令使用应用配置的tslint规则文件进行lint检查
ng lint
命令使用应用配置的tslint规则文件进行lint检查
Option选项：
- `--`fix, 默认值: false，指定检查时是否同步修正lint错误
- --force，默认值 false，是否强制检查失败加上后就说有lint错误也成功
- --type-check，默认值: false，是否类型检查
- --format (简写: -t), 默认值: prose,指定lint输出文件格式，有下列选择：prose, json, stylish, verbose, pmd, msbuild, checkstyle, vso, fileslist

#### ng test
ng test
编译应用后生成输入文件，并运行测试UT

#### ng e2e
ng e2e
命令部署应用并运行e2e测试用例

#### ng build
ng build
编译应用并生成目标文件，一般存放于dist/ directory
下面命令是等效的

    ng build --target=production --environment=prod
    ng build --prod --env=prod
    ng build --prod
下面也是等效的

    ng build --target=development --environment=dev
    ng build --dev --e=dev
    ng build --dev
    ng build

而已编译时修改index.html配置的基准路径

    ng build --base-href /myUrl/
    ng build --bh /myUrl/

下面时两种模式编译默认携带的选项
--dev模式： --aot false；--environment dev；--output-hashing media；--sourcemaps true；--extract-css 	false；
--prod模式：--aot true；--environment prod；--output-hashing all；--sourcemaps false；--extract-css 	true；
Options选项：
- aot，
- app
- base-href
- deploy-url
- environment
- extract-css
- i18n-file
- i18n-format
- locale
- output-hashing
- output-path
- delete-output-path
- poll
- progress
- sourcemap
- stats-json
- target
- vendor-chunk
- verbose
- watch
- show-circular-dependencies

#### ng get/ng set
ng get
获取配置文件的内容
ng set
设置配置文件的内容
比如：

    ng get angular-cli.json
    //获取整个文件内容

    ng get app.name
    //获取文件内容中app的name属性值

    ng set app.name myApp
    //修改文件内容中app的name属性值

#### ng doc
ng doc [search term]
命令将自动在浏览器打开官方文档angular.io，并调到指定字段处

#### ng eject
ng eject
ejects your app and output the proper webpack configuration and scripts

#### ng xi18n
ng xi18n
从应用的默版中收集i18n国际化的信息