---
title: Angular-路由器
date: 2017-07-17 00:56:10
tags: Augular
categories: 前端
comments: true
---

路由与导航是组织多页面应用的基本需求，也是angular最重要的几个特性之一。Angular提供了强大的路由机制来保证页面之间的跳转，根据配置的地址导航对应视图页面，并将地址值更新到浏览器地址栏中，将应用的所以页面有机的组织在了一起
<!--more-->

## 路由树
路由树是angular三大基本树结构之一，有着自己的配置方式，一般以单独的路由模块提供，伴随着模块的导入和导出，形成树状路由，如下图时一个典型的伴随着模块的组织而形成的路由树状结构：
![route-tree](/images/route-tree.jpg)

## 路由路径导航
模块直接导入之后，相应的路由模块的配置与准入模块的路由进行了合并，成为树的同级枝丫，下面是根据上一节路由树所描绘处理的导航示意图，以及形成的URL与组件的对应关系：
![route-url-tree](/images/route-url-tree.jpg)
如图所示，每一个url路径对应一个组件，路由器会根据在浏览器地址栏输入的地址或者代码里命令形式导航的地址去匹配相应的url，渲染对应组件的视图

## 路由配置
关于angular路由请点击[这里](https://www.angular.cn/docs/ts/latest/guide/hierarchical-dependency-injection.html "angular-多级注入器")查看官方指导文档
基本的路由配置就几个步骤：
#### 1. 基准地址配置
angular使用pushState方式构造url，必须设置基准地址，pushState才能正常工作
**设置方式：**

1 ). 如果app目录是应用的根目录，可以在index.html设置：`<base href="/">`
2 ). 在根模块中使用适当的APP_BASE_HREF值配置provide路由器，不过这种配置方式，应用中图片、CSS文件等的引用要使用绝对路径，比如:

        providers: [{provide: APP_BASE_HREF, useValue: '/my/app'}]

> 当两种方式同时都有设置的时候，provide注入的优先级更高

**配置作用：**
- 告诉路由器该如何合成导航用的url
- 方便引用CSS文件、脚本和图片，浏览器会用基准地址的值作为相对URL的前缀


#### 2. 路由列表配置
路由列表以路由模块的方式来提供，下面是一个典型的简单路由列表：

    import { NgModule }              from '@angular/core';
    import { RouterModule, Routes }  from '@angular/router';
    import { CrisisListComponent }   from './crisis-list.component';
    import { HeroListComponent }     from './hero-list.component';
    import { PageNotFoundComponent } from './not-found.component';

    const appRoutes: Routes = [
      { path: 'crisis-center', component: CrisisListComponent },
      { path: 'heroe/:id',        component: HeroComponent },
      { path: '',   redirectTo: '/heroes', pathMatch: 'full' },
      { path: '**', component: PageNotFoundComponent }
    ];

    @NgModule({
      imports: [
        RouterModule.forRoot(appRoutes)
      ],
      exports: [
        RouterModule
      ]
    })
    export class AppRoutingModule {}

（1）路由项以{path:,component:}为基本定义格式，保存路由的注册映射表，当浏览器的URL变化或代码中直接导航到某一url路径时，路由器就会翻出映射表,激活或生成对应组件的实例，用来显示视图，更新url到地址栏
（2）path不需要以斜杠（/）开头，路由器会为解析和构建最终的URL
（3）路由匹配是第一次最先匹配即停止原则，所以声明顺序很重要，建议顺序：具体路由->空路由->通配路由，通配path(`**`)可放置在最后作为错误url的默认页面提示
（4）注册路由方式：RouterModule.forRoot只能用于根模块；RouterModule.forChild用于特性模块

#### 3. 路由导航
导航的声明方式有链接式和命令式导航两种：
- 链接式，`routerLink="{{'/hero/' + crisis.id}}"`
- 命令式，`this.router.navigate(['/hero', hero.id])`

假如要导航到一个英雄的详情页面，下面3中方式是等效的

    <button type="button" routerLink="{{'/hero/' + crisis.id}}" routerLinkActive="Active">
    </button>

    <button type="button" [routerLink]="['/hero', hero.id]"
    [routerLinkActive]="['Active','classA']"></button>

    <button type="button" (click)="gotoDetail()" routerLinkActive="Active"></button>
    gotoDetail() {this.router.navigate(['/hero', hero.id]);}

（1） routerLinkActive:RouterLinkActive指令属性绑定，用于在路由激活时把CSS类添加到该元素上,反之则移除
（2）RouterLinkActive可以绑定到一个CSS类组成的数组，绑定方式有[routerLinkActive]="['A','B']" 或 routerLinkActive=“A B”

> 注意：当使用button标签进行路由时，务必制定type="button",如果不指定默认是“submit”类型，会默认出发表单提交

（3）RouterLinkActive还可以绑定到多个链接组里，当其中一个匹配到就添加样式类，适合用于菜单项的激活显示

    <div outerLinkActiver="Active">
       <button type="button" routerLink="{{'/hero/'}}" >
        </button>
       <button type="button" routerLink="{{'/power/'}}">
        </button>
    </div>

#### 4. 路由参数提取

##### (1) 参数提取对象
ActivatedRoute，可以通过注入此路由服务来获取路由的路径和参数，它有很多有用的信息：

    url //路由路径的Observable对象，它的值是一个由路径中各个部件组成的字符串数组
    data //该路由提供的data对象的一个Observable对象，还包含从resolve守卫中解析出来的值
    params //包含该路由的必选参数和可选参数的Observable对象
    queryParams //一个包含对所有路由都有效的查询参数的Observable对象。
    fragment //一个包含对所有路由都有效的片段值的Observable对象。
    outlet //RouterOutlet的名字，用于指示渲染该路由的位置,对于未命名的RouterOutlet，这个名字是primary
    routeConfig //与该路由的原始路径对应的配置信息。
    parent //当使用子路由时，它是一个包含父路由信息的ActivatedRoute对象
    firstChild //包含子路由列表中的第一个ActivatedRoute对象。
    children //包含当前路由下激活的全部子路由。

> 注意：当在组件中订阅一个可观察对象时，我们通常总是要在组件销毁时取消这个订阅,但是也有少数例外情况不需要取消订阅，而ActivateRoute中的各种可观察对象就是属于这种情况。ActivateRoute及其可观察对象都是由Router本身负责管理的。 Router会在不再需要时销毁这个路由组件，而注入进去的ActivateRoute也随之销毁了

**注意：**

> 新版本使用paramsMap替代params，queryParamsMap替代queryParams，它们拥有如下方法:
> - has(key)-> 判断是否有此参数，true/false
> - get(key)-> 获取参数值，value/null
> - getAll(key) -> 获取所以参数值数组，[value,...] / []
> - key --> 获取所以键值数组，[key,...] / []
> 比如：router.snapshot.params.get('id');

##### （2）提取参数的方式
使用可观察对象订阅的方式，下面两种方式效果相同
**switchMap**
switchMap允许你在Observable的当前值上执行一个动作，并将它映射一个新的Observable,然后再使用subscribe方法解析Observable对象获取值
    import { ActivateRoute }  from '@angular/router';

    constructor(private route: ActivateRoute) {
        this.route.params
            .switchMap((params: Params) => this.service.getHero(+params['id']))
            .subscribe((hero: Hero) => this.hero = hero);
    }

**subscribe**
使用subscribe方法直接解析params参数信息，获取数据值
    import { ActivateRoute }  from '@angular/router';

    constructor(private route: ActivateRoute) {
      this.route.params
        .subscribe((params: Params) => {
            this.hero = this.service.getHero(+params['id']);
        });
    }

**快照**
如果组件的实例不会被复用，可以使用一次性的快照route.snapshot来简化实现，它提供了路由参数的初始值，可以通过它来直接访问参数，而不用订阅或者添加
    import { ActivateRoute }  from '@angular/router';

    constructor(private route: ActivateRoute) {
        this.hero = this.service.getHero(+this.route.snapshot.params['id']);
    }

> 注意事项：
1）像许多其它rxjs操作符一样，switchMap既可以处理Observable也可以处理Promise发射的值。并且，如果用户重新导航到该路由，并且它正在获取一个英雄时，switchMap操作符还会取消任何正在执行的请求
2）默认情况，如果没有访问过其它组件就导航到了同一个组件实例，路由器倾向于复用组件实例,比如：搜索功能，这就需要Observable对象的方法来动态处理

**可选参数的定义和提取**
可选参数是导航期间传送任意复杂信息的理想载体，可选参数不涉及到模式匹配，在表达上提供了巨大的灵活性
1）定义方式
只要在必要参数之后，通过一个独立的对象来定义可选参数，比如：

    this.router.navigate(['/heroes', heroId, { foo: '123' }]);

合成的URL为：/heroes/123456;foo=foo1 ，可选的路由参数在url中没有使用“？”和“&”符号分隔，而是以;和主url相隔
2）提取方式

    this.heroes = this.route.params
      .switchMap((params: Params) => {
        this.foo = +params['foo'];
        return this.service.getHeroes();
      });

## 路由视图目标出口的选择
理由器在为组件选择显示出口的时候，是以距离最近为原则，就是选择距离组件最近的上层outlet目标进行呈现，比如：

    {
        path: 'crisis-center',
        component: CrisisCenterComponent,
        children: [
            {
                path: 'compose',
                component: ComposeMessageComponent
            },
            {
                path: '',
                component: CrisisListComponent,
                children: [
                    {
                        path: ':id',
                        component: CrisisDetailComponent
                    },
                    {
                        path: '',
                        component: CrisisCenterHomeComponent
                    }
                ]
            }
        ]
    }

本例中，组件CrisisCenterComponent和CrisisListComponent的模板中都有outlet出口定义，所以
- ComposeMessageComponent将在CrisisCenterComponent的outlet渲染呈现
- CrisisCenterHomeComponent和CrisisDetailComponent在CrisisListComponent的outlet呈现；

若距离最近的父组件没有outlet，则往上匹配，在更上一级父组件的outlet；举个例子，如果把path: 'compose'放在于crisis-center同级，将直接匹配appcomponent的outlet位置；同理，如果放在于CrisisDetail同级作为list的子路由，则将优先匹配
CrisisListComponent的outlet位置，不行再往上匹配

## 相对路由
特性模块内部应该优先考试使用相对路由，以减少对上层路由的依赖
**“目录式”语法**
- ./或无前导斜线形式是相对于当前级别的。
- ../会回到当前路由路径的上一级。
- ../<sibling>导航到一个兄弟路由，先回到上一级，然后进入兄弟路由路径

使用例子：

    this.router.navigate([crisis.id], { relativeTo: this.route })；
    this.router.navigate(['../', { id: crisisId, foo: 'foo' }], { relativeTo: this.route });

使用RouterLink来进行导航，使用相同的链接参数数组，不过不需要提供relativeTo属性，因为ActivatedRoute已经隐含在了RouterLink指令中，下面定义与上面等效：

    <a [routerLink]="[crisis.id]">


##### 最后，一些好东西：
1 ) RouterLinkActive指令会基于当前的RouterState对象来为激活的RouterLink切换CSS类,这会一直沿着路由树往下进行级联处理，所以父路由链接和子路由链接可能会同时激活。使用routerLinkActiveOptions属性可以改变这种行为，比如：

    <a routerLink="/contact" routerLinkActive="Active"
          [routerLinkActiveOptions]="{exact: true}">
    </a>
如上设置可以限制只有在其URL与当前URL精确匹配时才会激活指定的RouterLink

2 ) 路由器中的一些关键词
- Router，路由器，为激活的URL显示应用组件，并管理从组件之间的导航
- RouterModule，路由器模块，一个独立的Angular模块，用于提供所需的服务提供商，以及用来在应用视图之间进行导航的指令
- Routes，路由数组，数组里每个成员都会把一个URL路径映射到一个组件。
- Route，路由成员，定义路由器该如何根据URL模式（pattern）来导航到组件，大多数路由都由路径和组件类构成
- RouterOutlet，路由出口<router-outlet>，用来标记出路由器该在哪里显示视图
- RouterLink，路由链接routerLink，该指令用来把一个可点击的HTML元素绑定到路由，点击带有绑定到字符串或链接参数数组的routerLink指令的元素就会触发一次导航
- RouterLinkActiv，活动路由链接routerLinkActive，当HTML元素上或元素内的routerLink变为激活或非激活状态时，该指令为这个HTML元素添加或移除CSS类
- ActivatedRoute，激活的路由，为每个路由组件提供提供的一个服务，它包含特定于路由的信息，比如路由参数、静态数据、解析数据、全局查询参数和全局碎片（fragment）
- RouterState，路由器状态，路由器的当前状态包含了一棵由程序中激活的路由构成的树。它包含一些用于遍历路由树的快捷方法
- 链接参数数组，这个数组会被路由器解释成一个路由操作指南。我们可以把一个RouterLink绑定到该数组，或者把它作为参数传给Router.navigate方法
- 路由组件，一个带有RouterOutlet的Angular组件，它根据路由器的导航来显示相应的视图

3 ) 特性领域的路由特征：
- 每个特性都有自己的路由模块。
- 每个特性区都有自己的根组件。
- 每个特性区的根组件中都有自己的路由出口及其子路由。
- 特性区的路由很少（或完全不）与其它特性区的路由交叉

4 ) 多路由出口
在模板中，路由器只能支持一个无名主路由出口,但可以有多个命名的路由出口
每个命名出口都自己有一组带组件的路由。多重出口可以在同一时间根据不同的路由来显示不同的内容

**出口定义: **

    <router-outlet></router-outlet>
    <router-outlet name="popup1"></router-outlet>
    <router-outlet name="popup2"></router-outlet>
**配置路由对象：**

    {
      path: 'compose',
      component: ComposeMessageComponent,
      outlet: 'popup'
    }

**使用方式：**

    <a [routerLink]="[{ outlets: { popup: ['compose'] } }]">Contact</a>

链接参数数组包含一个只有一个outlets属性的对象，它的值是另一个对象，这个对象以一个或多个路由的出口名作为属性名。 在这里，它只有一个出口名“popup”，它的值则是另一个链接参数数组，用于指定compose路由
>注意：
当有且只有一个无名出口时，外部对象中的这个outlets对象不是必须的，路由器假设这个路由指向了无名的主出口，并为我们创建这些对象。
当路由到一个命名出口时，我们就会发现这个隐藏的真相

**关闭命名出口：**

    this.router.navigate([{ outlets: { popup: null }}]);

**各个outlet出口独立导航：**
路由器在导航树中可以对多个独立的分支保持追踪，并在URL中对这棵树进行表达。
我们可以添加更多出口和更多路由（无论是在顶层还是在嵌套的子层）来创建一个带有多个分支的导航树， 路由器将会生成相应的URL。
通过像前面那样填充outlets对象，我们可以告诉路由器立即导航到一棵完整的树。 然后把这个对象通过一个链接参数数组传给router.navigate方法

5 ) 各类路由参数的定义提取的区别

- path参数：url/:id
  使用方式：router.navigate([url,123]); [routerLink]="[url, 123]"
  表现形式：url/123
  提取方式：
  this.activateRoute.params.subscribe((params: Params)) => { let id = params['id']; });

- 可选参数
  使用方式：router.navigate([url,{id: 123}]); [routerLink]="[url, {id: 123}]"
  表现形式：url;id=123
  提取方式：
  this.activateRoute.params.subscribe((params: Params)) => { let id = params['id']; });

- 查询参数
  使用方式：router.navigate([url], {queryParams: {id: 123}}); [routerLink]="[url]"  [queryParams]="{id: 123}"
  表现形式：url?id=123
  提取方式：
  this.activateRoute.queryParams.subscribe((params: queryParams)) => { let id = params['id']; });

6 ) navigate和navigateByUrl
相同点：底层都是调用的相同的方法router.scheduleNavigation()进行处理和导航
不同点：
a. 使用方式不同：navigate([]) ; navigateByUrl(url)
  比如下面两个效果是一样:
  navigate([url, 123]);navigateByUrl('url/123');
b. navigate可以实现相对路由，而navigateByUrl不行


