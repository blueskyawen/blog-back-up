---
title: Vue-应用的typescript支持
date: 2019-07-28 12:09:53
tags: vue
categories: 前端
comments: true
toc: true
---

<style>
.ts-lizi {
    margin-top: -30px;
}
.ts-lizi figure {
    max-height: 600px;
}
</style>

vue默认使用js作为开发语言，但通过配置和插件也在vue中支持typescript，通过ts的强类型系统可以防止许多的错误，提高开发效率。  
vue-cli已经在创建应用提供了是否支持ts的选项，

<!--more-->

    vue create my-app 

选择自定义选项

    ? Please pick a preset: 
      default (babel, eslint) 
    ❯ Manually select features 
    
在选项中选择支持Typescript

    ? Please pick a preset: Manually select features
    ? Check the features needed for your project: 
     ◉ Babel
    ❯◉ TypeScript
     ◯ Progressive Web App (PWA) Support
     ◉ Router
     ◯ Vuex
     ◉ CSS Pre-processors
     ◉ Linter / Formatter
     ◯ Unit Testing
     ◯ E2E Testing

悬着是否使用class类的component语法

    ? Check the features needed for your project: Babel, TS
    ? Use class-style component syntax? (Y/n) Y
    
按照选择的配置，创建的应用便可支持ts

# 引入vue-class-component 

[vue-class-component](https://github.com/vuejs/vue-class-component)是官方维护的装饰器，可基于类的形式来声明和开发组件，使用angular的开发人员是比较熟悉的  
引入装饰器之后，便可像下面这样基于类的API来书写组件了。
```javascript
import Vue from 'vue'
import Component from 'vue-class-component'

// @Component 修饰符注明了此类为一个 Vue 组件
@Component({
  // 所有的组件选项都可以放在这里
})
export default class MyComponent extends Vue {
  // class component提供的相关成员
}
```

## 基本使用方式
从上面可以看到，使用component开发组件最简单的方式就是把组件的所有选项放在装饰器函数里，这和js的写法类似，但是也有一些坑  
下面看个例子 

<div class="ts-lizi">
```javascript
<template>
  <div class="hello">
    <h2 @click="clickMsg" v-local-direc>msg: {{ msg }}</h2>
    <h3>name: {{myName}} {{myYear | transY}}</h3>
    <div class="opers">
      <span @click="addYear">Add Year</span>
    </div>
    <h4>For-List</h4>
    <ul>
      <li v-for="num in numList" :key="num">{{num}}</li>
    </ul>
    <h4>Component-insert</h4>
    <div>
      <local-compt :num="500"></local-compt>
      <global-compt v-bind:num="1000"></global-compt>
      <local-compt02 :num="100"></local-compt02>
      <vue-compt :num="66"></vue-compt>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Component from 'vue-class-component'
import vueCompt from './compt.vue'

let localDirec = {
  inserted: function (el : any, binding : any) {
    el.style.backgroundColor = '#ccffff'
  }
}

Vue.component('global-compt', {
  template: '<div><h4>{{name}}: {{num}}</h4></div>',
  props: ['num'],
  data () {
    return {
      name: '全局组件'
    }
  }
})

@Component({
  template: '<div><h4>{{name}}: {{num}}</h4></div>',
  props: ['num'],
  data () {
    return {
      name: '局部组件'
    }
  }
})
class localCompt extends Vue {}

let localCompt02 = {
  props: ['num'],
  data () {
    return {
      name: '局部组件02'
    }
  },
  template: '<div><h3>{{name}}: ##{{num}}</h3></div>'
}

// 全局混入
Vue.mixin({
  data () {
    return {
      msg2: '全局混入',
      numList: [2, 3],
      site: {
        name: '腾讯'
      }
    }
  },
  created: function () {
    console.log('全局混入 --- created')
  },
  methods: {
    startMix2: function () {
      console.log('全局混入 --- startMix')
    }
  }
})
var localMix2 = {
  data () {
    return {
      msg2: 'localMix2',
      msg3: 'localMix2-msg3',
      msg4: 'localMix2-msg4',
      msg5: 'localMix2-msg5',
      numList: [4, 5, 9],
      site: {
        name: '阿里巴巴',
        url: 'www.alibaba.com',
        title: '我是阿里巴巴',
        haha: 'haha localMix2'
      }
    }
  },
  created: function () {
    console.log('实例混入: localMix2 --- created')
  },
  methods: {
    startMix: function () {
      console.log('实例混入: localMix2 --- startMix')
    },
    hello: function () {
      console.log('实例混入: localMix2 --- hello')
    }
  }
}

@Component({
  props: {
    msg: {
      type: String
    }
  },
  directives: { localDirec },
  components: { 'local-compt': localCompt, localCompt02, vueCompt },
  mixins: [localMix2],
  data: function () {
    return {
      name: 'vue-ts-HelloWorld',
      city: 'shanghai',
      year: 50,
      date: new Date(),
      numList: [1, 2, 3],
      myYear: 1000
    }
  },
  methods: {
    clickMsg: function () {
      alert('click - msg - ' + (this as any).msg)
    },
    addYear: function () {
      (this as any).year = (this as any).year + 5
      this.$emit('yearChange', (this as any).year);
      (this as any).startMix();
      (this as any).startMix2()
    }
  },
  computed: {
    myName: {
      get: function () {
        return this.name + 'haha!!'
      }
    }
  },
  watch: {
    year: function () {
      this.myYear += this.year
    }
  },
  filters: {
    transY: function (value: string) {
      return value + ' Coco'
    }
  }
})
export default class HelloWorld extends Vue {
}
</script>
```
</div>

可以看到，组件选项的写法和使用js基本一致，但是在methods/computed里访问data数据的时候，使用this可能会导致编译报错，在tsconfig.json加下面字段可消除

    "noImplicitThis": false,

如果发现还有地方报此错误消息，可以使用(this as any)代替了this，可消除从物告警  
在ts里也可以注册指令，全局组件和局部组件，并且可以基于component来声明局部组件  
```javascript
@Component({
  template: '<div><h4>{{name}}: {{num}}</h4></div>',
  props: ['num'],
  data () {
    return {
      name: '局部组件'
    }
  }
})
class localCompt extends Vue {}

@Component({
  ...
  directives: { localDirec },
  components: { 'local-compt': localCompt, localCompt02}
})
```
但是你会发现注册的组件并不能成功渲染，这是因为子组件的模版不可用，这时要将应用从“运行模式”修改为“运行编译模式”，修改vue.config.js即可  

    runtimeCompiler: true,
    
