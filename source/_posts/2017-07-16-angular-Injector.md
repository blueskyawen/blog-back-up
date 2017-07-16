---
title: Angular-注入器
date: 2017-07-16 17:16:49
tags: Augular
categories: 前端
comments: true
---

依赖注入是一种程序设计模式，指的是在应用构建里将所需的服务或者部件委托给第三方厂商提供，自身只关注应用逻辑，需要什么服务只需从厂商请求即可，而不用关心服务具体的创建过程和升级
相对于以前大而全的设计方式，这种思路的转变，可以使应用本身和所需的服务相对独立的开发和升级，而不用存在太大的耦合，下面是简单的示意图：
<!--more-->
![inector-design](/images/inector-design.jpg)
在angular里，自带的“依赖注入框架”和注入器就扮演着这样的一个角色，为构建angular应用的服务需求提供了可靠的提供机制，简便了应用的搭建

## 一. 注入器树
注入器树时angular基本树之一，在官方的[angular高级文档-多级注入器](https://www.angular.cn/docs/ts/latest/guide/hierarchical-dependency-injection.html "angular-多级注入器")一章里已经作了相对清晰的讲解，描述的是和另一基本树-组件树平级对应的注入器，组件树里的每一个组件节点都可以拥有自己的注入器，于是便有了这个注入器树，这个很好理解
这里不再赘述，这里要描述的是由angular中多个不同类型注入器组成的多级注入器树
在angular中，根据注入器的启动时间和使用范围的不同，可划分为以下几种:
- 根注入器
在main.ts里引导启动应用时，即创建了根注入器，整个应用范围内有效；除了根模块的privodes里的供应商直接注册到根注入器之外，直接导入到根模块的其他模块里的供应商也是注册根注入器
- 模块子注入器
指懒加载模块内的注入器，在加载后启动，使用范围限于本模块，供应商注册在子注入器中，包括被导入模块的供应商
- 组件注入器
指组件内的注入器，供应商在组件的privode数组内声明，只在组件和子组件范围内有效，且每个组件实例都有一份属于自己的注入器。在组件级提供服务可以确保组件的每个实例都得到一个自己的、私有的服务实例

多级注入器树由这些不同的注入器构成，树的结构与模块/组件的组织有着相似的层级和对应关系，下图是一个简单的注入器树示意：
![injector-tree](/images/injector-tree.jpg)


## 二.服务的注入
angular应用中配置使用注入服务需要定义可注入服务，注册供应商，服务请求等几个步骤，下面描述一下：
#### 1. 声明可注入服务
使用@Injectable()声明可注入服务，Injectable标识一个类可以被注入器实例化，示例如下：

    import { Injectable } from '@angular/core';
    import { HEROES }     from './mock-heroes';
    import { Logger }     from '../logger.service';
    @Injectable()
    export class HeroService {
      constructor(private logger: Logger) {  }
      getHeroes() {
        this.logger.log('Getting heroes ...');
        return HEROES;
      }
    }
@Injectable()装饰器把一个类标识为注入器实例化的目标，试图实例化没有被标识Injectable的类时，注入器会报错。

#### 2. 注册提供商

**什么是提供商**
提供商即服务供应商，提供了依赖值的一个具体的版本，指导注入器如何创建服务。注入器依靠提供商提供的版本来创建服务实例，然后将服务的实例注入组件或其它服务

**提供商的种类**
提供商没有严格的定义，可以有很多种，只要它们能交付一个行为类似的对象即可，比如：类、对象、工厂函数、变量值等

**提供商声明**
(1) 类供应商
providers：[{ provide: key令牌, useClass: 对象实例 }]  比如：

    providers：[{ provide: Logger, useClass: Logger }]
    providers：[Logger] //简化版

(2) 备选服务供应商
使用超类作key令牌，子类作为实例化类，例如：

    @Injectable()
    class EvenBetterLogger extends Logger {
      constructor(private userService: UserService) { super(); }

      log(message: string) {
        let name = this.userService.user.name;
        super.log(`Message to ${name}: ${message}`);
      }
    }

    providers：[{ provide: Logger, useClass: EvenBetterLogger}]

(3) 别名服务供应商
使用已定义的类作为实例化类，其他类名作key令牌，例如：

    providers：[NewLogger，
               { provide: Logger, useExisting: NewLogger}]

(3) 值依赖供应商
使用一个对象或者变量作为供应商对像，这在一些全局参数配置中有重要的用处，例如：

    const silentLogger = {
      logs: ['Silent logger says "Shhhhh!". Provided via "useValue"'],
      log: () => {}
    };

    providers：[{ provide: Logger, useValue: silentLogger}]

注意：供应商在一个注入器中不能重复注册，否则会出现多个相同的实例，这是不被期望的

#### 3. 服务注入请求
作为重要的一步是用户如何进行注入请求，因为服务最终是要用于应用中，否则前面的配置都没有什么意义，服务注入分为隐式注入和显式注入两种
##### （1）隐式注入
最常见的注入配置时在我们组件构造函数里通过供应商令牌进行服务申请，比如：

    constructor(private logger: Logger) {}

constructor声明 + @component + provide供应商数组 共同指示注入器要注入服务实例，以及怎么进行注入
注意，我们在请求时是使用的供应商令牌来向注入器请求的，即上一节中的provide字段，而不管对应的实际供应商类或对象时哪个，注入器会自动根据令牌匹配一个服务实例来给组件使用，如果匹配不到，则创建一个
实际上，在angular实现中，向注入器注册提供商时，会把这个提供商和一个DI令牌关联起来了，注入器自身会维护一个内部的令牌-提供商映射表，在请求依赖会从这个映射表查找提供商实例，而令牌就是这个表的键值，这就是为什么请求时使用的是令牌

##### （2）显式注入
有两种显示注入方式，不过不提倡这么使用
方式一：直接创建注入器

    injector : any;
    constructor() {
      this.injector =
           ReflectiveInjector.resolveAndCreate([Logger, HeroService]);
      let logger = injector.get(Logger);
    }

方式二：直接使用注入器

    heroService : any;
    constructor(private injector: Injector) {}

    ngOnInit() {
        this.heroService = this.injector.get(HeroService);
    }


#### 4. 服务注入配置示意图
服务注入的几个方面可以在下图得到体现
![injector-config](/images/angular-inject-config.jpg)


## 三. 多级注入系统
点击[这里](https://www.angular.cn/docs/ts/latest/guide/hierarchical-dependency-injection.html "angular-多级注入器")查看原文，这里只是作一些总结
##### 1) 嵌套式注入器
Angular多级依赖注入系统支持与组件树并行的嵌套式注入器，应用程序中有一个与组件树平行的注入器树，我们可以在组件树中的任何级别上重新配置注入器
每个组件实例都有自己的注入器，也许说不对，组件可以没有注入器，但这时候说的组件的注入器可能是一个组件树中更高级的祖先注入器的代理，可以理解为组件存了上级注入器的一个指针，,这只是提升效率的实现细节，只要想象成每个组件都有自己的注入器就可以了。

##### 2) 注入冒泡
angular自下而上冒泡的方式匹配使用的服务依赖，具体来讲，当一个组件请求一个服务时，Angular 先尝试用本组件自己的注入器来满足它，如果在该组件的注入器没有找到对应的提供商，它就把这个请求转给它父组件的注入器来处理；如果父组件注入器也无法满足这个申请，则继续往上传递，一直到找到了一个能处理此请求的注入器，或者超出了组件树中的祖先位置还未找到，抛出一个错误

##### 3) 不同层级再次提供同一个服务
在应用实现中，可以在注入器树中的多个层次上为指定的依赖令牌重新注册提供商，即同一个服务可以在多个层级上重新配置，多个子注入器都具有该服务的实例，根据注入的冒泡匹配原则，遇到的第一个提供商实例会胜出
因此，注入器树中间层注入器上的提供商，可以拦截来自底层的对特定服务的请求，导致它可以“重新配置”和者说“遮蔽”高层的注入器
**带来的好处：**
- 服务隔离,服务范围限制在组件内，别的组件不可访问
- 组件特殊的提供商,下级组件可重新定义与上级同名的服务，定义自己特殊的实现
- 多重编辑会话，每个组件实例拥有自己的服务实例，可以存放处理自己的数据


#### 最后，一些好东西：
1 . 在一个注入器的范围内，依赖都是单例的,比如在应用范围内的根注入器，各个依赖值都是单例的

2 . 注入器同时会实例化Component这样的组件,为什么不标记他们为@Injectable()呢？因为没有必要，因为组件已经有@Component装饰器， 而@Component/@Directive/@Pipe等装饰器都是 @Injectable的子类

3 . 可选依赖，使用@Optional()定义可选依赖，这样即使在注入器里没有这个服务，像这样：

    import { Optional } from '@angular/core';
    constructor(@Optional() private logger: Logger) {
      if (this.logger) {
        this.logger.log(some_message);
      }
    }
这样的话在providers数组里可以不声明Logger，不会出错。不过使用的时候需要处理空值，因为当注入找不到这个服务时会返回一个null

4 . 非类依赖服务
当注入对象是一个字符串，函数或者接口对象时，没有对应的合适令牌可供使用，可采用InjectionToken作为提供商令牌，像这样：

    //注入对像
    export interface AppConfig {
      apiEndpoint: string;
      title: string;
    }
    export const HERO_DI_CONFIG: AppConfig = {
      apiEndpoint: 'api.heroes.com',
      title: 'Dependency Injection'
    };

    //令牌
    import { InjectionToken } from '@angular/core';
    export let APP_CONFIG = new InjectionToken<AppConfig>('app.config');

    //提供商
    providers: [{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }]

    //使用@Inject装饰器帮忙注入使用
    constructor(@Inject(APP_CONFIG) config: AppConfig) {
      this.title = config.title;
    }


