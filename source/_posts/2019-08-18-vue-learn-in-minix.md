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




## 选项合并策略
