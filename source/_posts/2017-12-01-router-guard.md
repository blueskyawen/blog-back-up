---
title: Angular-路由器守卫
date: 2017-12-01 22:11:34
tags: Augular
categories: 前端
comments: true
---

路由守卫，顾名思义，是守护路由的导航激活，在路由激活之前进行逻辑判断和数据处理;比如：路由权限的判断，表单填写丢弃的询问，以及路由激活前数据额预加载..等等，都是守卫干的工作，下面会根据使用场景不同介绍几种守卫的实现
<!--more-->

### 路由激活守卫

CanActivate守卫，用于对某个页面导航的守卫，在导航前可以作逻辑判断，比如：是否有访问权限，来判断是否激活这个路由来查看页面
主要步骤有：

- 封装服务，实现CanActivate接口，返回true，则允许激活;否则，不允许激活
- 注入服务到对应模块，在路由配置中添加对于服务

定义守卫服务：实现CanActivate守卫接口，判断是否登录;若已登录,则返回true,激活路由;否则，则返回flase，无法激活路由并返回登录首页

    import { Injectable }       from '@angular/core';
    import {CanActivate, Router,ActivatedRouteSnapshot,RouterStateSnapshot}                           from '@angular/router';
    import { AuthService }      from './auth.service';

    @Injectable()
    export class AuthGuard implements CanActivate {
      constructor(private authService: AuthService, private router: Router) {}

      canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
        let url: string = state.url;

        if (this.authService.isLoggedIn) {
            return true;
        } else {
            this.authService.redirectUrl = url;
            this.router.navigate(['/login']);
            return false;
        }
      }
    }

    const adminRoutes: Routes = [
      {
        path: 'admin',
        component: AdminComponent,
        canActivate: [AuthGuard],
        children: [
          {
              { path: 'crises', component: ManageCrisesComponent },
              { path: 'heroes', component: ManageHeroesComponent }
          }
        ]
      }
    ];

### 子路由激活守卫

linkCanAcitvateChild守卫，用于对某个子页面导航的守卫，在导航前可以作逻辑判断，比如：是否有访问权限，来判断是否激活这个路由来查看页面，实现与上面类似

    import { Injectable }       from '@angular/core';
    import {CanActivate, Router,ActivatedRouteSnapshot,
      RouterStateSnapshot,CanActivateChild} from '@angular/router';
    import { AuthService }      from './auth.service';

    @Injectable()
    export class AuthGuard implements CanActivate, CanActivateChild {
      constructor(private authService: AuthService, private router: Router) {}

      canActivateChild(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
        let url: string = state.url;

        if (this.authService.isLoggedIn) {
            return true;
        } else {
            this.authService.redirectUrl = url;
            this.router.navigate(['/login']);
            return false;
        }
      }

    /* . . . */
    }

    const adminRoutes: Routes = [
      {
        path: 'admin',
        component: AdminComponent,
        children: [
          {
            path: '',
            canActivateChild: [AuthGuard],
            children: [
              { path: 'crises', component: ManageCrisesComponent },
              { path: 'heroes', component: ManageHeroesComponent },
              { path: '', component: AdminDashboardComponent }
            ]
          }
        ]
      }
    ];

### 表单去激活守卫

CanDeactivate守卫，用于在填写表单时路由离开时使用，可进行判断是否离开，并进行一些数据处理

- 封装服务，实现CanDeactivate接口，实现逻辑判断，返回true，则允许去激活，直接导航出去;否则，不允许激活，一般会以弹框询求确定
- 注入服务到对应模块，在路由配置中添加对于服务

去激活守卫有两种实现方式：公共方式和特有组件方式

**1）公共方式**

封装公共服务实现CanDeactivate接口，接受具体组件类的方法调用;各个需要使用的组件实现CanDeactivate方法即可

dialog.ts

    import 'rxjs/add/observable/of';
    import { Injectable } from '@angular/core';
    import { Observable } from 'rxjs/Observable';

    /**
     * Async modal dialog service
     * DialogService makes this app easier to test by faking this service.
     * TODO: better modal implementation that doesn't use window.confirm
     */
    @Injectable()
    export class DialogService {
      /**
       * Ask user to confirm an action. `message` explains the action and choices.
       * Returns observable resolving to `true`=confirm or `false`=cancel
       */
      confirm(message?: string): Observable<boolean> {
        const confirmation = window.confirm(message || 'Is it OK?');

        return Observable.of(confirmation);
      };
    }

deactive-guard.service

    import { Injectable }    from '@angular/core';
    import { CanDeactivate } from '@angular/router';
    import { Observable }    from 'rxjs/Observable';

    export interface CanComponentDeactivate {
      canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
    }

    @Injectable()
    export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {
      canDeactivate(component: CanComponentDeactivate) {
        return component.canDeactivate ? component.canDeactivate() : true;
      }
    }

具体组件内实现canDeactivate方法：

    export class TemplateFormComponent {
      name : any;
      password : any;

      constructor(public dialogService : DialogService) {

      }

      canDeactivate(): Observable<boolean> | boolean {
        if (!this.name && !this.password) {
          return true;
        }

        return this.dialogService.confirm('已有修改未保存，确定离开吗?');
      }
    }

路由配置

    const appRoutes: Routes = [
      { path: 'templateForm',
        component: TemplateFormComponent,
        canDeactivate: [CanDeactivateGuard]}
    ];

**2）特定方式**

即为具体组件实现对于的服务，注入特定组件的实例来进行逻辑判断

    import { Injectable }           from '@angular/core';
    import { Observable }           from 'rxjs/Observable';
    import { CanDeactivate,
      ActivatedRouteSnapshot,
      RouterStateSnapshot }  from '@angular/router';

    import { TemplateFormComponent } from './template-form/template-form.component';

    @Injectable()
    export class CanDeactivateGuard2 implements CanDeactivate<TemplateFormComponent> {

      canDeactivate(
        component: TemplateFormComponent,
        route: ActivatedRouteSnapshot,
        state: RouterStateSnapshot
      ): Observable<boolean> | boolean {
        console.log(state.url);

        if (!component.name && !component.password) {
          return true;
        }

        return component.dialogService.confirm('已有修改未保存，确定离开吗?');
      }
    }

    //路由
    const appRoutes: Routes = [
      { path: 'templateForm',
        component: TemplateFormComponent,
        canDeactivate: [CanDeactivateGuard2]}
    ];



### 数据预加载

预加载，顾名思义，就是在路由激活前对路由模板组件所需要的数据进行预先的获取，这样在路由激活展现时数据就已经有了，提高体验;而且，对于加载失败的情况也能作相应处理

- 封装服务，实现Resolve接口，进行数据获取，获取成功，返回结果Data，继续激活路由;否则，不允许激活，一般会导航到首页
- 注入服务到对应模块，在路由配置中添加对应服务
- 组件使用时，直接使用路由保存的data

封装服务：

    import { Injectable }             from '@angular/core';
    import { Observable }             from 'rxjs/Observable';
    import { Router, Resolve, RouterStateSnapshot,
             ActivatedRouteSnapshot } from '@angular/router';
    import { Crisis, CrisisService }  from './crisis.service';

    @Injectable()
    export class DataResolver implements Resolve<User> {
      constructor(private userService: UserService, private router: Router) {}

      resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<Crisis> {
        return this.userService.getUser(id).map(res => {
          if (res) {
            return res;
          } else {
            this.router.navigate(['/login']);
            return null;
          }
        });
      }
    }

配置路由：

    const appRoutes: Routes = [
      { path: 'templateForm',
        component: TemplateFormComponent,
        canDeactivate: [CanDeactivateGuard]，
        resolve： {
            user: DataResolver
        }
    ];

组件使用：

    export class TemplateFormComponent {
      name : any;
      user : any;

      constructor(public dialogService : DialogService,private route: ActivatedRoute) {
        this.route.data
          .subscribe((data: { user: any }) => {
            this.user = data.user;
          });
      }
    }


### 模块访问守卫

CanLoad守卫，对懒加载模块的加载限制，加载前进行逻辑处理，满足条件才加载该模块

- 封装服务，实现CanLoad接口，进行逻辑判断，若返回true，则加载模块并路由;否则，不再加载
- 注入服务到对应模块，在路由配置中添加对应服务

封装服务：

    import { Injectable }       from '@angular/core';
    import {
       Router,
      ActivatedRouteSnapshot,
      RouterStateSnapshot,
      CanLoad,Route
    }                           from '@angular/router';
    import { appService }      from './app.service';

    @Injectable()
    export class AuthGuard implements CanLoad {
      constructor(private appS: appService, private router: Router) {}

      canLoad(route: Route): boolean {
        let url = `/${route.path}`;

        if (this.appS.isLogin) { return true; }

        this.appS.redirectUrl = url;
        this.router.navigate(['/controls']);
        return false;
      }
    }

配置路由：

    const appRoutes: Routes = [
      { path: 'controls', component: ControlComponent },
      { path: 'reactiveForm',
        loadChildren: 'app/reactive-form/reactive-form.module#ReactiveModule',
        canLoad: [AuthGuard]
      }
    ];


### 模块预加载

在每次成功的导航后，路由器会在自己的配置中查找尚未加载并且可以预加载的模块。 是否加载某个模块，以及要加载哪些模块，取决于预加载策略。
Router内置了两种预加载策略：

- 完全不预加载，这是默认值。惰性加载的特性区仍然会按需加载。
- 预加载所有惰性加载的特性区。

默认情况下，路由器或者完全不预加载或者预加载每个惰性加载模块。 路由器还支持自定义预加载策略，以便完全控制要预加载哪些模块以及何时加载。

**1) 预加载所有模块**

只需在根路由模块的forRoot方法传入附加参数即可，将加载策略设置preloadingStrategy设置为PreloadAllModules

    import { NgModule }              from '@angular/core';
    import { RouterModule, Routes, PreloadAllModules }  from '@angular/router';
    import { ControlComponent } from './control/control.component';
    import { TemplateFormComponent } from './template-form/template-form.component';

    import { CanDeactivateGuard } from './can-deactivate-guard.service';
    import { CanDeactivateGuard2 } from './can-deactivate-guard2.service';
    import { AuthGuard } from './auth-guard.service';

    const appRoutes: Routes = [
      { path: 'controls', component: ControlComponent },
      { path: 'templateForm',
        component: TemplateFormComponent,
        canDeactivate: [CanDeactivateGuard2]},
      { path: 'reactiveForm',
        loadChildren: 'app/reactive-form/reactive-form.module#ReactiveModule',
        canLoad: [AuthGuard]
      },
      { path: 'login',
        loadChildren: 'app/login/login.module#LoginModule'
      },
      { path: '',   redirectTo: '/controls', pathMatch: 'full' }
    ];

    @NgModule({
      imports: [
        RouterModule.forRoot(appRoutes,
          {
            enableTracing: true,
            preloadingStrategy: PreloadAllModules
          })
      ],
      exports: [
        RouterModule
      ]
    })

> - 加载策略只能在根路由模块里设置，因为只有forRoot()可以配置参数，forChild()不行
> - CanLoad守卫级别高于策略，会阻塞模块加载

**2） 自定义加载策略**

使用自定义预加载策略，可以控制路由器预加载哪些路由以及如何加载，作为例子，我们将只预加载那些data.preload标志为true的路由

- 设置路由标识，标识哪些模块被预先加载
- 定义策略服务，根据标识来判断并加载模块
- 配置策略服务到路由模块

策略服务

    import 'rxjs/add/observable/of';
    import { Injectable } from '@angular/core';
    import { PreloadingStrategy, Route } from '@angular/router';
    import { Observable } from 'rxjs/Observable';

    @Injectable()
    export class SelectivePreloadingStrategy implements PreloadingStrategy {
      preloadedModules: string[] = [];

      preload(route: Route, load: () => Observable<any>): Observable<any> {
        if (route.data && route.data['preload']) {
          // add the route path to the preloaded module array
          this.preloadedModules.push(route.path);

          // log the route path to the console
          console.log('Preloaded: ' + route.path);

          return load();
        } else {
          return Observable.of(null);
        }
      }
    }

策略配置

    const appRoutes: Routes = [
      { path: 'controls', component: ControlComponent },
      { path: 'templateForm',
        component: TemplateFormComponent,
        canDeactivate: [CanDeactivateGuard2]},
      { path: 'reactiveForm',
        loadChildren: 'app/reactive-form/reactive-form.module#ReactiveModule',
        data: {Preloaded: false}
      },
      { path: 'login',
        loadChildren: 'app/login/login.module#LoginModule',
        data: {Preloaded: true}
      },
      { path: '',   redirectTo: '/controls', pathMatch: 'full' }
    ];

    @NgModule({
      imports: [
        RouterModule.forRoot(appRoutes,
          {
            enableTracing: true,
            preloadingStrategy: SelectivePreloadingStrategy
          })
      ],
      exports: [
        RouterModule
      ]，
      providers: [SelectivePreloadingStrategy]
    })
    export class AppRoutingModule {}

> 在分层路由的每个级别上，你都可以设置多个守卫。 路由器会先按照从最深的子路由由下往上检查的顺序来检查 CanDeactivate() 和 CanActivateChild() 守卫。 然后它会按照从上到下的顺序检查 CanActivate() 守卫。   
如果特性模块是异步加载的，在加载它之前还会检查 CanLoad() 守卫。
