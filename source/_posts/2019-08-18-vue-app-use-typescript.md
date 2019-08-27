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
    

## 类成员使用

上面把组件选项放进装饰器函数的写法虽然方便和简单，但和使用js开发基本无异，不能体现ts的特点，使用vue-class-component是可以把一部分的选项“变成”类成员的，只是在使用上与一些限制，下面部分介绍是从vue-class-component的github上翻译扒过来的

> 当前版本只支持ECMAScript第1阶段装饰器，不支持第二阶段装饰器  
如果使用Babel，需要安装babel-plugin-transform-decorators-legacy。
如果使用TypeScript，需要在tsconfig.json设置experimentalDecorators为true

### 1）组件选项
可以把部分组件选项声明为雷的成员，并不是所有，目前支持情况如下：

- methods，可直接声明为类成员方法
- computed，可声明为类属性访问器
- 初始initdata，可声明为类属性
- data/render/生命周期钩子，可直接声明为类成员方法，但是不能在实例本身上调用它们

对于其他不支持的选项，将它们传递给装饰器函数即可  
下面看个例子，说明下使用限制和注意的地方：
```javascript
<template>
  <div class="aboutMe">
    <h2 @click="clickFunc">msg = {{ msg }}</h2>
    <h2 @click="makeH">name = {{ name }}</h2>
    <h2 @click="setNameNum">msg2 = {{msg2}}</h2>
    <h2 @click="makeClassVar">get msgNameNum = {{msgNameNum}}</h2>
    <h2>get nameNum = {{nameNum}}</h2>
    <p>getData: {{getData}}</p>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component，mixins } from 'vue-class-component'
import MyMixin from './mixin.js'

@Component({
  props: {
    msg: {
      type: String
    }
  },
  data: function () {
    return {
      name: 'name-in-component',
      num: 10
    }
  },
  created () {
    console.log('AboutMe -- created in-component')
  },
  methods: {
    makeH: function () {
      alert('click - makeH - 2')
    },
    makeClassVar: function () {
      alert('click - makeClassVar - ' + (this as any).msg2)
    }
  },
  computed: {
    getData () {
      return this.$data
    }
  }
})
export default class AboutMe extends Vue {
  name: string = 'name-in-class'
  msg2: string = (this as any).msg + '**' + this.name

  created () {
    console.log('AboutMe -- created')
  }

  clickFunc (): void {
    alert('clickFunc--' + this.name)
  }

  makeH () {
    alert('click - makeH - in - class')
  }

  get msgNameNum () {
    return (this as any).msg + '@' + this.name + '@' + (this as any).num             + '@' + this.msg2 + '@' + (this as any).me
  }

  get nameNum () {
    return this.name + '@@@' + (this as any).num
  }

  set nameNum (str: string) {
    let stes = str.split('@@@')
    this.name = stes[0];
    (this as any).num = +stes[1]
  }

  setNameNum () {
    this.nameNum = 'new Name set@@@20'
  }
}
</script>
```
结果输出，

    msg = hello about
    name = name-in-component
    msg2 = hello about**name-in-class
    get msgNameNum = hello about@name-in-component@10@hello about**name-in-class@undefined
    get nameNum = name-in-component@@@10
    getData: { "name": "name-in-component", "num": 10, "msg2": "hello about**name-in-class"}

可以看到，装饰器函数里声明的data和class里声明的initData数据成员将合并，此外，还有许多需要注意的问题：

- class里声明的data函数会覆盖装饰器函数里声明的data
- data函数里声明的同名属性优先级大于class里声明的initData，如：name
- class里声明的method，computed，声明钩子方法会覆盖装饰器函数里声明的同名选项，如:makeH()
- class的initData在初始化使用的this.xx优先使用class的initData成员xx，找不到再取data里的同名数据，如：msg2的name
- class里可以直接使用this访问class成员,但要访问装饰器函数里声明组件属性，比如：data属性，方法等，则需要使用(this as any)，否则编译器里会报错； 同理，装饰器函数里的方法等也需要使用(this as any)来访问class成员
- class只能有一个extends成员

### 2）mixins混入
插件提供mixins辅助函数，以类继承的方式使用mixins，TypeScript可以推断mixin类型并在组件类型上继承它们  
在上个例子加上混入，先声明混入：
```javascript
//mixin.js
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
export class MyMixin extends Vue {
  mixinValue = 'I am MyMixin'
  created () {
    console.log('MyMixin -- created')
  }
  makeH () {
    alert('click - makeH - in - YourMixin')
  }
  makeMyMixin () {
    alert('click - makeMyMixin - in - MyMixin')
  }
}

@Component
export class YourMixin extends Vue {
  mixinValue = 'I am YourMixin'
  mixinMsg = 'love your Mixin !'
  created () {
    console.log('YourMixin -- created')
  }
  makeH () {
    alert('click - makeH - in - YourMixin')
  }
  makeMyMixin () {
    alert('click - makeMyMixin - in - YourMixin')
  }
  makeYourMixin () {
    alert('click - makeYourMixin - in - YourMixin')
  }
}
```
在组件中使用
```javascript
<template>
  <div class="aboutMe">
    <h2 @click="clickFunc">msg = {{ msg }}</h2>
    <h2 @click="makeH">name = {{ name }}</h2>
    <h2 @click="setNameNum">msg2 = {{msg2}}</h2>
    <h2 @click="makeClassVar">get msgNameNum = {{msgNameNum}}</h2>
    <h2 @click="makeMyMixin">get nameNum = {{nameNum}}</h2>
    <p @click="makeYourMixin">getData: {{getData}}</p>
  </div>
</template>
<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import { mixins } from 'vue-class-component'
import { MyMixin, YourMixin } from './mixin.js'

@Component({
  ...
})
export default class AboutMe extends mixins(YourMixin, MyMixin) {
  name: string = 'name-in-class'
  msg2: string = (this as any).msg + '**' + this.name

  created () {
    console.log('AboutMe -- created')
  }
  ...
}
</script>
```
可以看到，mixin也可以通过component的形式书写，而通过mixins(YourMixin, MyMixin)可以将多个mixin混入到组件中，效果同在组件中直接这么写相同：

    mixins(MyMixin，YourMixin)

可以看到使用类继承形式的书写顺序刚好相反，即在选项合并的时候，MyMixin为parent，YourMixin为child，具体的合并策略可查看前面的文章[Vue-源码详解mixin混入和合并策略](http://blueskyawen.com/2019/07/16/vue-learn-in-minix/)

### 3）自定义装饰器
您可以通过创建自己的装饰器来扩展此库的功能，插件提供createDecorator帮助器来创建自定义装饰器。createDecorator期望将回调函数作为第一个参数，并且回调将接收以下参数：

- options：Vue组件选项对象，此对象的更改将影响提供的组件。
- key：应用装饰器的属性或方法键。
- parameterIndex：如果自定义装饰器用于参数，则为装饰参数的索引

```javascript
// decorators.js
import { createDecorator } from 'vue-class-component'

export const NoCache = createDecorator((options, key) => {
  // 组件options选项必须传入这个回调函数
  // 更新了配置，它可使用于任何component里
  options.computed[key].cache = false
})

// MyComp.js
import { NoCache } from './decorators'

@Component
class MyComp extends Vue {
  // 加上次装饰器后，该计算属性不再缓存
  @NoCache
  get random () {
    return Math.random()
  }
}
```

### 4）添加自定义钩子
如果您使用某些Vue插件，如VueRouter，您可能希望类组件解决它们提供的钩子。 对于这种情况，Component.registerHooks允许你这样注册和使用钩子

```javascript
// class-component-hooks.js
import Component from 'vue-class-component'
// 使用钩子名称来注册
Component.registerHooks([
  'beforeRouteEnter',
  'beforeRouteLeave',
  'beforeRouteUpdate' // for vue-router 2.2+
])

// MyComp.js
import Vue from 'vue'
import Component from 'vue-class-component'

@Component
class MyComp extends Vue {
  beforeRouteEnter (to, from, next) {
    console.log('beforeRouteEnter')
    next() 
  }

  beforeRouteLeave (to, from, next) {
    console.log('beforeRouteLeave')
    next() 
  }
}

// main.js
// 注意必须在使用之前注册他们
import './class-component-hooks'
import Vue from 'vue'
import MyComp from './MyComp'

new Vue({
  el: '#app',
  components: {
    MyComp
  }
})
```

### 5）注意事项
**1）箭头函数不能访问类实例里初始化的属性**  
由于插件是利用Vue的原始构造函数来将类属性实例化成vue实例数据的，因此不能在箭头函数中使用this访问类属性，因为在初始化时会将this替代为Vue实例对象，而原Vue不存在这些数据
```javascript
@Component
class MyComp extends Vue {
  foo = 123
  bar = () => {
    this.foo = 456
  }
}
```

**2）类属性初始化不要使用undefined**  
当声明类的属性为undefined，将得不到响应式数据特性，请务必初始化为null
```javascript
@Component
class MyComp extends Vue {
  // 非响应式
  foo = undefined
  // 响应式
  bar = null

  data () {
    return {
      // 响应式
      baz: undefined
    }
  }
}
```

