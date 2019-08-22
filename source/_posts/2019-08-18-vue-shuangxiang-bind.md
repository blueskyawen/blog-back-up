---
title: Vue-双向绑定的几种支持方式
date: 2019-07-08 12:09:53
tags: Vue
categories: 前端
comments: true
---

与angular一样，vue也可以进行数据的双向绑定并拥有多种实现形式，可用于基础组件的封装。  
下面以`Switch基础组件`为例介绍vuejs里面双向绑定的几种方式，组件实现的基本效果[像这样](http://blueskyawen.com/ng-shadow-cat/components/switch)，可点击查看 
<!--more-->

## 1. 使用value-input
vue的绑定指令v-model本质上的是value输入和input事假组成的语法糖，

- 绑定数据值到value的props上
- 通过input事件，触发发射新的值

比如：
```javascript
<input v-model="searchText">
```

等价于：

```javascript
<input v-bind:value="searchText" 
       v-on:input="searchText = $event.target.value">
```

类似地，可以讲其他适用到普通的元素和组件中，通过props和自定义事件来实现组件的双向绑定，如下

```javascript
<template>
  <div class="vc-switch">
    <div class="switch-container" :class="switchClasses" @click="changeSwitch">
      <div class="switch" :class="spinClasses"></div>
      <span v-if="isHasLabel">{{label}}</span>
    </div>
  </div>
</template>
<script>
export default {
  name: 'vc-switch',
  props: {
    type: {
      type: String,
      default: 'max'
    },
    tipLabels: {
      type: Array,
      default: function () {
        return []
      }
    },
    value: {
      type: Boolean,
      default: true
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      isHasLabel: false,
      label: ''
    }
  },
  created: function () {
    this.isHasLabel = this.tipLabels.length !== 0
    this.label = !this.isHasLabel ? '' : this.value ? this.tipLabels[0] : this.tipLabels[1]
  },
  computed: {
    switchClasses: function () {
      return {'switch-start': !this.isHasLabel && !this.value,
        'switch-end': !this.isHasLabel && this.value,
        'switch-row': this.isHasLabel && !this.value,
        'switch-reverse': this.isHasLabel && this.value,
        'container-max': this.type === 'max',
        'container-min': this.type === 'min',
        'disabled': this.disabled}
    },
    spinClasses: function () {
      return {'switch-max': this.type === 'max',
        'switch-min': this.type === 'min'}
    }
  },
  methods: {
    changeSwitch: function () {
      if (this.disabled) return
      if (!this.value) {
        this.label = this.tipLabels[0]
      } else {
        this.label = this.tipLabels[1]
      }
      this.$emit('input', !this.value)
    }
  }
}
</script>
<style scoped>
    ...
</style>
```
使用它：

```
<vc-switch v-model="isAggre"></vc-switch>
```

## 2. 自定义v-model
虽然可以通过value和input来实现双向绑定，但是这两个属性用在普通元素上仍然怪异，可以自定义v-model来实现新的语法糖，比如

- 绑定数据值到switch
- 通过change事件触发发射新的值

只修改了部分js

```javascript
<script>
export default {
  name: 'vc-switch',
  model: {
    prop: 'switch',
    event: 'change'
  },
  props: {
    ...
    switch: {
      type: Boolean,
      default: true
    },
    ...
  },
  ...
  created: function () {
    this.isHasLabel = this.tipLabels.length !== 0
    this.label = !this.isHasLabel ? '' : this.switch ? this.tipLabels[0] : this.tipLabels[1]
  },
  computed: {
    switchClasses: function () {
      return {'switch-start': !this.isHasLabel && !this.switch,
        'switch-end': !this.isHasLabel && this.switch,
        'switch-row': this.isHasLabel && !this.switch,
        'switch-reverse': this.isHasLabel && this.switch,
        'container-max': this.type === 'max',
        'container-min': this.type === 'min',
        'disabled': this.disabled}
    },
    spinClasses: function () {
      return {'switch-max': this.type === 'max',
        'switch-min': this.type === 'min'}
    }
  },
  methods: {
    changeSwitch: function () {
      if (this.disabled) return
      if (!this.switch) {
        this.label = this.tipLabels[0]
      } else {
        this.label = this.tipLabels[1]
      }
      this.$emit('change', !this.switch)
    }
  }
}
</script>
```

## 3. 使用.sync
通过新增的.sync修饰符和update:propName一样实现双向绑定的语法糖，虽然官方文档比较推荐这个，且陈述了理由，但真正用下来区别不大  
下面还是以switch组件为例，只修改js即可

```javascript
<script>
export default {
  name: 'vc-switch',
  props: {
    ...
    switch: {
      type: Boolean,
      default: true
    },
    ...
  },
  ...
  methods: {
    changeSwitch: function () {
      if (this.disabled) return
      if (!this.switch) {
        this.label = this.tipLabels[0]
      } else {
        this.label = this.tipLabels[1]
      }
      this.$emit('update:switch', !this.switch)
    }
  }
}
</script>
```
使用时

```
<vc-switch v-bind:switch.sync="isAggre"></vc-switch>
```

> 和v-bind可以绑定对像并且对象每个属性建立响应一样，使用.sync同样可以与对象进行双向绑定，并且给每个对象属性建立双向绑定，与给每个属性字段分别绑定效果一样
