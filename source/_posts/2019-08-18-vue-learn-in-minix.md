---
title: Vue-源码详解mixin混入和合并策略
date: 2019-07-16 12:09:53
tags: Vue
categories: 前端
comments: true
---

<style>
.mixin-lizi {
    margin-top: -30px;
}
.mixin-lizi figure {
    max-height: 500px;
}
</style>

在vue里面，混入（mixin）是一种特殊的使用方式。一个混入对象可以包含任意的组件选项，可根据需求随意“封装”组件的可复用单元，并在使用时根据一定的策略合并到组件的选项当中，使用时与组件自身选项无异。  
官方文档对mixin介绍比较少，不能了解甚少，于是便想研究下源码对它混入做个研究和总结
> 本文基于Vue源码2.x版本  

<!--more-->

## 说在前面
在分析mixin之前，先看看两个方法，它们在混入的合并过程中扮演着重要的角色  

**（1） Vue.extend()**  
Vue.extend()是基础Vue构造器，参数是一个包含组件选项的对象，可用于显式的扩展组件和混入对象，如  
```javascript
var Component = Vue.extend({
  mixins: [myMixin]
})
```
其实，在组件内部的混入合并也是通过它来完成的  

**（2） extend()**  
extend()是对象合并方法，参数是源和目标两个对象，用于对象的合并操作  
```javascript
extend(target, source)
```
其中source对象将合并到target对象，如果source和target的key值相同，则直接覆盖，否则属性添加；作用类似于Object.assgin()和underscore的_.extend(destination, *sources)   

## 合并图示
通过对vue源码的研究，我发现混入对于选项的“合并”并不是一步到位的，而是两两合并，并通过合并策略和优先级向一定的方向逐步进行合并操作，最终才得到合并的结果，就像默认的选项合并策略：
```javascript
const defaultStrat = function (parentVal: any, childVal: any): any {
    return childVal === undefined
        ? parentVal
        : childVal
}
```
其中childVal和parentVal是每一次合并中的两个源和目标对象，比如：vm.options.xx和mixin.options.xx，下面用一个例子和图示进行说明  
***示例代码：***  
<div class="mixin-lizi">
```javascript
import Vue from 'vue'

Vue.mixin({
  data () {
    return {
      msg: '全局混入-msg',
      msg1: '全局混入1-msg-1',
      msg2: '全局混入1-msg-2',
      msg4: '全局混入1-msg4',
      msg5: '全局混入1-msg5',
      site: {
        name: '腾讯',
        url: 'www.tenant.com',
        hahaList: [3, 4],
        city: 'shenzhen'
      }
    }
  },
  created: function () {
    console.log('全局混入1 --- created')
  },
  methods: {
    startMix: function () {
      console.log('全局混入1 --- startMix')
    }
  },
  computed: {
    getMsg: function () {
      return 'getMsg 全局混入1!!'
    },
    getMsg2: function () {
      return 'getMsg2 全局混入1!!'
    },
    getMsg5: function () {
      return 'getMsg5 全局混入1!!'
    }
  }
})

Vue.mixin({
  data () {
    return {
      msg: '全局混入2-msg',
      msg1: '全局混入2-msg-1',
      msg4: '全局混入2-msg4',
      numList: [4, 5],
      site: {
        name: 'alibaba',
        hahaList: [3, 4],
        url: 'www.alibaba.com',
        people: 30000
      }
    }
  },
  created: function () {
    console.log('全局混入2 --- created')
  },
  methods: {
    startMix: function () {
      console.log('全局混入2 --- startMix')
    },
    hello: function () {
      console.log('全局混入2 --- hello')
    }
  },
  computed: {
    getMsg: function () {
      return 'getMsg 全局混入2!!'
    },
    getMsg2: function () {
      return 'getMsg2 全局混入2!!'
    }
  }
})

var localMix3 = {
  data () {
    return {
      msg: '实例混入3',
      msg1: '实例混入3-msg1',
      msg2: '实例混入3-msg2',
      msg3: '实例混入3-msg3',
      site: {
        name: '淘宝111',
        url: 'www',
        country: 'China'
      }
    }
  },
  created: function () {
    console.log('实例混入3 --- created')
  },
  methods: {
    startMix: function () {
      console.log('实例混入3 --- startMix')
    },
    hello: function () {
      console.log('实例混入3 --- hello')
    }
  },
  computed: {
    getMsg: function () {
      return 'getMsg 实例混入3!!'
    },
    getMsg2: function () {
      return 'getMsg2 实例混入3!!'
    },
    getMsg3: function () {
      return 'getMsg3 实例混入3!!'
    },
    getMsg4: function () {
      return 'getMsg4 实例混入3!!'
    }
  }
}
var localMix2 = {
  data () {
    return {
      msg: '实例混入2',
      msg1: '实例混入2-msg1',
      msg2: '实例混入2-msg2',
      site: {
        name: '淘宝',
        url: 'www.taobao.com',
        title: 'I am 淘宝'
      }
    }
  },
  created: function () {
    console.log('实例混入2 --- created')
  },
  methods: {
    startMix: function () {
      console.log('实例混入2 --- startMix')
    },
    hello: function () {
      console.log('实例混入2 --- hello')
    }
  },
  computed: {
    getMsg: function () {
      return 'getMsg 实例混入2!!'
    },
    getMsg2: function () {
      return 'getMsg2 实例混入2!!'
    },
    getMsg3: function () {
      return 'getMsg3 实例混入2!!'
    }
  }
}
var localMix1 = {
  data () {
    return {
      msg: '实例混入1',
      msg1: '实例混入1-msg1',
      numList: [2, 3],
      site: {
        name: '淘宝',
        url: 'www.taobao.com',
        hahaList: [3, 4]
      }
    }
  },
  mixins: [localMix2],
  created: function () {
    console.log('实例混入1 --- created')
  },
  methods: {
    startMix: function () {
      console.log('实例混入1 --- startMix')
    },
    hello: function () {
      console.log('实例混入1 --- hello')
    }
  },
  filters: {
    capitalize2: function (value) {
      if (!value) return ''
      return value + ' 实例混入1!!'
    }
  },
  computed: {
    getMsg: function () {
      return 'getMsg 实例混入1!!'
    },
    getMsg2: function () {
      return 'getMsg2 实例混入1!!'
    }
  }
}

export default {
  name: 'basicExtend',
  data () {
    return {
      msg: 'basicExtend',
      numList: [1, 2],
      site: {
        name: '百度',
        hahaList: [1, 2]
      }
    }
  },
  mixins: [localMix3, localMix1],
  created: function () {
    console.log('basicExtend --- created')
  },
  methods: {
    startMix: function () {
      console.log('basicExtend --- startMix')
    }
  },
  computed: {
    showData: function () {
      return JSON.stringify(this.$data)
    },
    getMsg: function () {
      return this.msg + ' !!!'
    }
  }
}
```
</div>
其中合并后的data和输出如下，
```javascript
vm.$data:
{
    "msg":"basicExtend",
    "numList":[1,2],
    "site":{
        "name":"百度",
        "hahaList":[1,2],
        "url":"www.taobao.com",
        "title":"I am 淘宝",
        "country":"China",
        "people":30000,
        "city":"shenzhen"
    },
    "msg1":"实例混入1-msg1",
    "msg2":"实例混入2-msg2",
    "msg3":"实例混入3-msg3",
    "msg4":"全局混入2-msg4",
    "msg5":"全局混入1-msg5"
}

computed计算属性:
getMsg: basicExtend !!!
getMsg2: getMsg2 实例混入1!!
getMsg3: getMsg3 实例混入2!!
getMsg4: getMsg4 实例混入3!!
getMsg5: getMsg5 全局混入1!!
```
下面根据这个例子整理的“合并父子图示”，根据后面的讲解再反过来看为什么合并的结果会是这样。  

![vue-mixins](/images/vue-mixins.png)  

图示描述了源代码中选项合并方法的入参"父子"关系，其中，

- 全局注册的混入最先完成混入，并按注册的顺序来逐个合并，先注册的先完成混入合并，依次类推
- 局部注册的混入次之，并按mixins数组里声明的顺序依次完成合并
- 每个混入也可以包含mixins局部混入数组，mixins先完成合并，本混入的options再进行合并
- 组件options最后完成混入合并
- 先合并的"优先级"低，后合并的"优先级"高，也就是组件的options合并优先级最高
- 不同的选项根据自身的混入策略合并方向不一样，这个在下面会有说

## 选项合并

混入对象的混入是每个对象选项和组件选项的“混入合并”，比如：options.data、options.props等，根据不同选项合并策略的不同，各类选项会以不同的方式和方向进行“合并”操作  
**（1） el，propsData**  
> 合并方向: parent --> child

源码：
```javascript
 src/core/util/options.js

const strats = config.optionMergeStrategies

if (process.env.NODE_ENV !== 'production') {
  strats.el = strats.propsData = function (parent, child, vm, key) {
    if (!vm) {
      warn(
        `option "${key}" can only be used during instance ` +
        'creation with the `new` keyword.'
      )
    }
    return defaultStrat(parent, child)
  }
}
/**
 * Default strategy.
 */
const defaultStrat = function (parentVal: any, childVal: any): any {
  return childVal === undefined
    ? parentVal
    : childVal
}
```
可见，el , propsData使用的是默认合并策略，默认策略比较简单干脆，以child选项为主，若无则使用parent选项

**（2） 生命周期钩子**  
> 合并方向: parent --> child

源码：
```javascript
export const LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured',
  'serverPrefetch'
]
LIFECYCLE_HOOKS.forEach(hook => {
  strats[hook] = mergeHook
})

function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  const res = childVal
    ? parentVal
      ? parentVal.concat(childVal) // 合并为一个数组
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
  return res
    ? dedupeHooks(res)
    : res
}
```
可见，LIFECYCLE_HOOKS会将每个hook合并成一个数组，按照图示从父到子开始一步步链接合并成数组，parent在前，child在后。在钩子触发时，按照数组从头顺序调用触发，所以我们看到调用顺序是这样的，与图示一致：

    全局混入hook --> 实例混入hook ... --> 组件实例hook

**（3） data**  
> 合并方向: parent --> child

源码：
```javascript
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )
      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}

export function mergeDataOrFn (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    // in a Vue.extend merge, both should be functions
    if (!childVal) {
      return parentVal
    }
    if (!parentVal) {
      return childVal
    }
    // when parentVal & childVal are both present,
    // we need to return a function that returns the
    // merged result of both functions... no need to
    // check if parentVal is a function here because
    // it has to be a function to pass previous merges.
    return function mergedDataFn () {
      // parentVal作为from，childVal作为to，parent->child方向合并
      return mergeData(
        typeof childVal === 'function' ? childVal.call(this, this) : childVal,
        typeof parentVal === 'function' ? parentVal.call(this, this) : parentVal
      )
    }
  } else {
    return function mergedInstanceDataFn () {
      // instance merge
      const instanceData = typeof childVal === 'function'
        ? childVal.call(vm, vm)
        : childVal
      const defaultData = typeof parentVal === 'function'
        ? parentVal.call(vm, vm)
        : parentVal
      if (instanceData) {
        // parentVal作为from，childVal作为to，parent->child方向合并
        return mergeData(instanceData, defaultData)
      } else {
        return defaultData
      }
    }
  }
}

function mergeData (to: Object, from: ?Object): Object {
  if (!from) return to
  let key, toVal, fromVal

  const keys = hasSymbol
    ? Reflect.ownKeys(from)
    : Object.keys(from)

  for (let i = 0; i < keys.length; i++) {
    key = keys[i]
    // in case the object is already observed...
    if (key === '__ob__') continue
    toVal = to[key]  // child
    fromVal = from[key] // parent
    if (!hasOwn(to, key)) {
      set(to, key, fromVal) //若to没有此key，添加它
    } else if (
      toVal !== fromVal &&
      isPlainObject(toVal) &&
      isPlainObject(fromVal)
    ) {
      //若to有此key，且不等
      //若to有此key，值非对象，否则进行深度合并
      mergeData(toVal, fromVal)
    }
  }
  return to
}
```
这里set（）方法操作添加key并创建响应属性值  
可以看到，data的合并比较复杂，按照图示从父到子开始递归合并，以child为主，比较key规则如下：

- 若child无此key，parent有，直接合并此key
- 若child和parent都有此key，且非object类型，忽略不作为
- 若child和parent都有此key，且为object类型，则递归合并对象

**（4） components，directives，filters**  
> 合并方向: parent <-- child

源码：
```javascript
export const ASSET_TYPES = [
  'component',
  'directive',
  'filter'
]
ASSET_TYPES.forEach(function (type) {
  strats[type + 's'] = mergeAssets
})

function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null) // 原型委托
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    return extend(res, childVal) // child合并到parent
  } else {
    return res
  }
}
```
可以看到，components, directives,filters的合并策略比较简单，使用extend方法合并为一个对象，按
照图示从子到父进行合并。  
其实这是采用原型链委托的方式在合并时把child的属性委托在parent上，这样在使用的时候，在child上查找，没有的再从parent上找，以此类推，所以child的优先级的更高的。

**（5） watch**  
> 合并方向: parent --> child

源码：
```javascript
strats.watch = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  // work around Firefox's Object.prototype.watch...
  if (parentVal === nativeWatch) parentVal = undefined
  if (childVal === nativeWatch) childVal = undefined
  /* istanbul ignore if */
  if (!childVal) return Object.create(parentVal || null)
  if (process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = {}
  extend(ret, parentVal) //获取parent选项
  for (const key in childVal) {
    let parent = ret[key] //获取parent选项值
    const child = childVal[key] //获取child选项值
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    //每个wather选项合并为数组
    ret[key] = parent
      ? parent.concat(child)
      : Array.isArray(child) ? child : [child]
  }
  return ret
}
```
可见，watch会将每个watcher合并成一个数组，按照图示从父到子顺序合并。在同名wather属性触发时，按照数组从头顺序调用触发，所以我们看到触发顺序是这样的，与图示一致：

    全局混入 --> 实例混入 ... --> 组件实例

**（6） props，methods，computed，inject**  
> 合并方向: parent <-- child

源码：
```javascript
strats.props =
strats.methods =
strats.inject =
strats.computed = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): ?Object {
  if (childVal && process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal
  const ret = Object.create(null)
  extend(ret, parentVal) //合并parent
  if (childVal) extend(ret, childVal) //合并child
  return ret
}
```
可以看到，props，methods，computed，inject的合并策略和components比较相似，都是使用extend方法合并为一个对象，按照图示从子到父进行合并，所以在调用查找时child优先级更高。  

**（7） provide**  
> 合并方向: parent --> child

源码：
```javascript
strats.provide = mergeDataOrFn
```
provide的合并策略和data类似  

## 选项合并策略

如上，对于mixin和组件的每个选项都有对应的合并策略，你可以像下面这样改变默认的合并策略
```javascript
const strats = Vue.config.optionMergeStrategies
strats.methods = strats.data
```
但是对于默认选项不建议这么做，会引起合并错误，虽然提供了手段  
对于新增的选项，比如vuex，myVOption，也需要什么合并策略，可以使用现有的策略，比如：
```javascript
Vue.config.optionMergeStrategies.myVOption = strats.methods
```
也可以自定义策略
```javascript
Vue.config.optionMergeStrategies.myVOption = function(
  parentVal: ?Object,
  childVal: ?Object,
): ?Object {
    if (!parentVal) return childVal
    if (!childVal) return parentVal
    return extend(parentVal, childVal)
}
```
> 注意：全局混入将对所有的vue实例有效，尽量不要使用；但可以用于插件的发布，比如发布一个画图插件，并提供若干绘画方法
