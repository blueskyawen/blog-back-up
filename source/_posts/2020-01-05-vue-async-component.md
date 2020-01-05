---
title: Vue-双向绑定的几种支持方式
date: 2019-10-25 12:09:53
tags: Vue
categories: 前端
comments: true
toc: true
---
vue应用在大部分情况下不用关心模块加载问题，但是当程序规模变得越来越大的时候，就需要考虑性能优化问题了。在这个过程中，你可能使用了拆分代码和延迟加载这两种方法，它们通过将代码块的加截推迟到需要的时候加载，从而使应用程序的初始包变得更小。
在程序中模块的加载方式一般有两种：静态加载和动态加载，像下面这样，
```javascript
// 静态加载方式
import utils from './utils' 
// 动态加载方式
import('./utils').then(utils => { ... });
```
<!--more-->
对于现代的打包器，比如Webpack（从version2开始）会将这种语法理解为自动为该模块创建一个单独的文件，然后在需要时进行加载。在Vue里也提供了按需加载的功能，叫异步组件，下面将从使用方式、源码等几个角度来说明。
从组件的使用方式来将，可以有：**直接渲染异步组件和路由异步组件**；而直接渲染异步组件一般是通过component的 is 属性和v-if来动态切换实现。
# 1、直接渲染异步组件
顾名思义就是在模版直接使用的组件，一般公共组件和应用子组件大都是这种用法，只不是加载的方式是按需加载
## 1.1 使用方法
Vue文档里提供了3中使用异步组件的方法，也是核心代码里内置实现的，这个后面再说，这三种方式如下：
```javascript
import Vue from 'vue'
import asyncLoading from './componentss/async-loading-component.vue'
import asyncError from './componentss/async-error-component.vue'

Vue.component('async-webpack-example-c', function (resolve) {
  // 这个特殊的 `require` 语法将会告诉 webpack
  // 自动将你的构建代码切割成多个包，这些包
  // 会通过 Ajax 请求加载
  require(['./componentss/async-component-c.vue'], resolve)
})

Vue.component(
  'async-webpack-example-d',
  // 这个 `import` 函数会返回一个 `Promise` 对象。
  () => import('./componentss/async-component-d.vue')
)

const AsyncComponentA = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./componentss/async-component-a.js'),
  // 异步组件加载时使用的组件
  loading: asyncLoading,
  // 加载失败时使用的组件
  error: asyncError,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  // delay时间之后开开始加载并显示loading组件效果
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 15000
})
// 也可注册局部组件
{...
    components: {
        'aync-component-b': () => import('./componentss/async-component-b.js')
    }
...
}
```
可以使用v-if和component/is来切换使用，这里使用is特性为例：
```javascript
<template>
    <div class="demo-item-group">
      <ul>
        <li v-for="item of comptIsNames" :key="item" @click="curComponentName = item"
            :class="{active: curComponentName === item}">{{item}}</li>
      </ul>
      <div>
        <component :is="curComponentName"></component>
      </div>
    </div>
</template>
<script>
export default {
  name: 'basic-async-component',
  data: function () {
    return {
      showCcc: false,
      comptIsNames: ['async-webpack-example-c', 'async-webpack-example-d', 'aync-component-b','aync-component-a'],
      curComponentName: 'global-compt-1'
    }
  },
  components: {
    'aync-component-b': () => import('./componentss/async-component-b.js'),
    'aync-component-a': AsyncComponentA
  }
}
</script>
```
想查看loading效果可以打开devTools-network-presets改为**slow 3G**，查看加载错误可设置为**Offline**。
## 1.2 看源码
看源码能更好的理解异步组件的内部实现，从而更好的使用它们。
异步组件实现的本质是 2 次渲染，除了 0 delay 的高级异步组件第一次直接渲染成 loading 组件外，其它都是第一次渲染生成一个注释节点，当异步获取组件成功后，再通过 forceRender 强制重新渲染，这样就能正确渲染出异步加载的组件了。
异步组件也要create组件实例，但是由于其定义形式是工厂函数而不是对象，不会直接extend实例构造函数，而是走异步工厂那一套，如下，源文件在：vue/src/core/vdom/create-component.js,
```javascript
export function createComponent (
  Ctor: Class<Component> | Function | Object | void,
  data: ?VNodeData,
  context: Component,
  children: ?Array<VNode>,
  tag?: string
): VNode | Array<VNode> | void {
  if (isUndef(Ctor)) {
    return
  }

  const baseCtor = context.$options._base

  // plain options object: turn it into a constructor
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor)
  }

  // if at this stage it's not a constructor or an async component factory,
  // reject.
  if (typeof Ctor !== 'function') {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Invalid Component definition: ${String(Ctor)}`, context)
    }
    return
  }

  // async component 异步组件处理
  // Ctor不是对象，不会走上面那一套，Ctor.cid未定义
  let asyncFactory
  if (isUndef(Ctor.cid)) {
    asyncFactory = Ctor
    Ctor = resolveAsyncComponent(asyncFactory, baseCtor)
    if (Ctor === undefined) {
      // return a placeholder node for async component, which is rendered
      // as a comment node but preserves all the raw information for the node.
      // the information will be used for async server-rendering and hydration.
      return createAsyncPlaceholder(
        asyncFactory,
        data,
        context,
        children,
        tag
      )
    }
  }
  ...
  return vnode
}
```
这里resolveAsyncComponent是处理异步组件的主函数，createAsyncPlaceholder就是第一次渲染生成一个注释节点，在高级异步组件形式的delay=0的情况下直接渲染Ctor返回undefined不生成注释节点，其他情况下都会先生成诸事节点然后再执行resolveAsyncComponent根据加载情况不同进行组件的渲染。
来看一下resolveAsyncComponent，源码位置：vue/src/core/vdom/helpers/resolve-async-component.js,
```javascript
export function resolveAsyncComponent (
  factory: Function,
  baseCtor: Class<Component>
): Class<Component> | void {
  // 处理加载error，一般在再次执行次方法式会实际处理到
  if (isTrue(factory.error) && isDef(factory.errorComp)) {
    return factory.errorComp
  }
  // 处理加载成功resolved，一般在再次执行次方法式会实际处理到
  if (isDef(factory.resolved)) {
    return factory.resolved
  }
  // 处理多个vm实例加载一个异步组件的情况，只加载实例化一次
  const owner = currentRenderingInstance
  if (owner && isDef(factory.owners) && factory.owners.indexOf(owner) === -1) {
    // already pending
    factory.owners.push(owner)
  }
  // 处理加载loading，一般在再次执行次方法式会实际处理到
  if (isTrue(factory.loading) && isDef(factory.loadingComp)) {
    return factory.loadingComp
  }

  if (owner && !isDef(factory.owners)) {
    const owners = factory.owners = [owner]
    let sync = true
    let timerLoading = null
    let timerTimeout = null

    ;(owner: any).$on('hook:destroyed', () => remove(owners, owner))
    // 通知组件所属的实例更新渲染
    const forceRender = (renderCompleted: boolean) => {
      for (let i = 0, l = owners.length; i < l; i++) {
        (owners[i]: any).$forceUpdate()
      }

      if (renderCompleted) {
        owners.length = 0
        if (timerLoading !== null) {
          clearTimeout(timerLoading)
          timerLoading = null
        }
        if (timerTimeout !== null) {
          clearTimeout(timerTimeout)
          timerTimeout = null
        }
      }
    }
    // 包装异步组件加载成功resolve方法
    // 存储实例对象，下一次执行次函数便可获得
    const resolve = once((res: Object | Class<Component>) => {
      // cache resolved
      factory.resolved = ensureCtor(res, baseCtor)
      // invoke callbacks only if this is not a synchronous resolve
      // (async resolves are shimmed as synchronous during SSR)
      if (!sync) {
        forceRender(true)
      } else {
        owners.length = 0
      }
    })
    // 包装异步组件加载成功reject方法
    // 标记出错，下一次执行次函数便可处理error，如有error组件可获得实例
    const reject = once(reason => {
      process.env.NODE_ENV !== 'production' && warn(
        `Failed to resolve async component: ${String(factory)}` +
        (reason ? `\nReason: ${reason}` : '')
      )
      if (isDef(factory.errorComp)) {
        factory.error = true
        forceRender(true)
      }
    })
    // 执行异步组件工厂函数
    const res = factory(resolve, reject)

    if (isObject(res)) {
      if (isPromise(res)) {
        // () => Promise
        // 处理() => Promise形式异步组件
        if (isUndef(factory.resolved)) {
          res.then(resolve, reject)
        }
      } else if (isPromise(res.component)) {
        // 处理高级异步组件
        res.component.then(resolve, reject)
        // 处理高级异步组件的err组件
        if (isDef(res.error)) {
          factory.errorComp = ensureCtor(res.error, baseCtor)
        }
        // 处理高级异步组件的loading组件
        if (isDef(res.loading)) {
          factory.loadingComp = ensureCtor(res.loading, baseCtor)
           // 处理高级异步组件的加载延时delay
          if (res.delay === 0) {
            factory.loading = true
          } else {
            timerLoading = setTimeout(() => {
              timerLoading = null
              if (isUndef(factory.resolved) && isUndef(factory.error)) {
                factory.loading = true
                forceRender(false)
              }
            }, res.delay || 200)
          }
        }
        // 处理高级异步组件的timeout
        if (isDef(res.timeout)) {
          timerTimeout = setTimeout(() => {
            timerTimeout = null
            if (isUndef(factory.resolved)) {
              reject(
                process.env.NODE_ENV !== 'production'
                  ? `timeout (${res.timeout}ms)`
                  : null
              )
            }
          }, res.timeout)
        }
      }
    }

    sync = false
    // return in case resolved synchronously
    return factory.loading
      ? factory.loadingComp
      : factory.resolved
  }
}
```
这个方法比较复杂，它处理了上文说到的3 种异步组件的创建方式，具体直接看上面的代码中已标好的注释。
## 1.3 封装动态异步组件
封装一个可以根据配置动态加载任意组件的公共组件，这是在某些时候是有需要的。
在vue中动态组件的典型方式是通过该component和is属性来实现，而动态异步组件就是动态+异步，就是这个思路实现而来。
示例如下：
```javascript
// vc-async-component.vue
<template>
  <transition v-if="transition && keepAlive" mode="out-in"
              :enter-class="transitionClass.enter"
              :enter-active-class="transitionClass.enterActive"
              :enter-to-class="transitionClass.enterTo"
              :leave-class="transitionClass.leave"
              :leave-active-class="transitionClass.leaveActive"
              :leave-to-class="transitionClass.leaveTo">
    <keep-alive>
      <component :is="componentName" v-bind="$attrs" v-on="$listeners"></component>
    </keep-alive>
  </transition>
  <transition v-else-if="transition && !keepAlive" mode="out-in"
              :enter-class="transitionClass.enter"
              :enter-active-class="transitionClass.enterActive"
              :enter-to-class="transitionClass.enterTo"
              :leave-class="transitionClass.leave"
              :leave-active-class="transitionClass.leaveActive"
              :leave-to-class="transitionClass.leaveTo">
    <component :is="componentName" v-bind="$attrs" v-on="$listeners"></component>
  </transition>
  <keep-alive v-else-if="!transition && keepAlive">
    <component :is="componentName" v-bind="$attrs" v-on="$listeners"></component>
  </keep-alive>
  <component v-else :is="componentName" v-bind="$attrs" v-on="$listeners"></component>
</template>

<script>
import vcAsyncLoading from './vc-async-loading'
import vcAsyncError from './vc-async-error'

export default {
  inheritAttrs: false,
  name: 'vc-async-component',
  props: {
    path: {
      type: String,
      required: true
    },
    keepAlive: {
      type: Boolean,
      default: false
    },
    transition: {
      type: Boolean,
      default: false
    },
    delay: {
      type: Number,
      default: 200
    },
    timeout: {
      type: Number,
      default: 3000
    },
    transitionClass: {
      type: Object,
      default: function () {
        return {
          enter: 'v-enter',
          enterTo: 'v-enter-to',
          enterActive: 'v-enter-active',
          leave: 'v-leave',
          leaveTo: 'v-leave-to',
          leaveActive: 'v-leave-active'
        }
      }
    }
  },
  data: function () {
    return {
      componentName: () => ({
        component: import(`@/${this.path}`),
        loading: vcAsyncLoading,
        error: vcAsyncError,
        delay: this.delay,
        timeout: this.timeout
      })
    }
  },
  watch: {
    path: function () {
      this.componentName = () => ({
        component: import(`@/${this.path}`),
        loading: vcAsyncLoading,
        error: vcAsyncError,
        delay: this.delay,
        timeout: this.timeout
      })
    }
  }
}
</script>
<style scoped>
.v-enter,.v-leave-to {
  opacity: 0;
}
.v-enter-to,.v-leave {
  opacity: 1;
}
.v-enter-active,.v-leave-active {
  transition: opacity .5s;
}
</style>
```
使用方式：
```javascript
<template>
  <div>
    <vc-async-component :path="comptName" :timeout="timeoutLen" :keep-alive="isKeepAlive"
                        :transition="isTransition" :transition-class="asyncClass">
    </vc-async-component>
  </div>
</template>
<script>
import Vue from 'vue'
export default {
  name: 'basic-async-component',
  data: function () {
    return {
      comptName: 'components/basic/componentss/async-component-c.vue',
      isKeepAlive: false,
      isTransition: false,
      asyncClass: {
        enter: 'async-enter',
        enterActive: 'async-enter-active',
        enterTo: 'async-enter-to',
        leave: 'async-leave',
        leaveActive: 'async-leave-active',
        leaveTo: 'async-leave-to'
      }
    }
  },
  ...
}
</script>
```
# 2、路由异步组件
在vue-router中同样需要路由组件的懒加载，把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样更加高效。
结合上面说的Vue的异步组件和Webpack的代码分割功能，可以轻松实现路由组件的懒加载。
## 2.1 普通异步组件的路由懒加载
```javascript
const routes = [
  { path: '/', redirect: '/hello' },
  { 
    path: '/hello',
    component: () => import('@/components/hello')
  }
]
```
## 2.2 高级异步组件的路由懒加载
```javascript
import asyncLoading from '@/components/vc-cat/vc-async-loading.vue'
import asyncError from '@/components/vc-cat/vc-async-error.vue'

function lazyLoadView () {
  const asyncComponentForm = () => ({
    component: import('@/components/hello/hello-form'),
    loading: asyncLoading,
    error: asyncError,
    delay: 200,
    timeout: 8000
  })
  return Promise.resolve({
    functional: true,
    render (h, {data, children}) {
      return h(asyncComponentForm, data, children)
    }
  })
}

const routes = [
  { 
    path: '/form',
    component: () => lazyLoadView()
  }
]
```
参考文档：[https://blog.csdn.net/refreeom/article/details/90437394](https://blog.csdn.net/refreeom/article/details/90437394)
