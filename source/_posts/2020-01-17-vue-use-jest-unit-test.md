---
title: Vue-使用Jest进行组件单元测试
date: 2019-08-25 22:15:51
tags: Vue
categories: 前端
comments: true
toc: true
---

Vue应用项目一般都由许多的组件构建而成，而vue的单文件组件的这样一个组织方式，使得我们对组件的隔离测试更加方便和直接。
首先，对组件做单元测试由许多好处，比如：
<!--more-->
- 提供描述组件行为的文档
- 节省手动测试的时间
- 守护现有代码，减少研发新特性时产生的 bug
- 改进设计
- 促进重构

既然单元测试的好处这么多，那么组件的单元测试该怎么写呢，有原则？所有的单元测试都应该满足以下特点：

- 可以快速运行
- 易于理解
- 只测试一个独立单元的工作

而对于组件的测试来说，不推荐追求行级覆盖率，导致过分关注组件的内部实现细节。我们应该面向接口进行测试，即断言一些输入 (用户的交互或 prop 的改变) 提供给组件，然后判断是否导致预期的结果(渲染结果或触发自定义事件)，更像一个“黑盒”测试。
这样即使该组件的内部实现以后发生了改变，只要组件的公共接口不变，测试用例就能完成很好的测试。
示例源码：[vue-start](https://github.com/blueskyawen/vue-start)

# 添加Jest测试运行器
Jest 是功能最全的测试运行器，它所需的配置是最少的，默认安装了 JSDOM，而且内置断言且命令行的用户体验非常好。
在项目package.json同级目录安装Jest测试相关插件包，
```javascript
> npm i jest --dev-save
> npm i babel-jest --dev-save
> npm i vue-jest --dev-save //单文件组件预处理器
> npm i vue-template-compiler --dev-save
> npm i jest-serializer-vue --dev-save
```
**配置babel-jest部分**：.babelrc文件
```javascript
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }],
    "stage-2"
  ],
  "plugins": ["transform-vue-jsx", "transform-runtime"],
  "env": {
    "test": {
      "presets": ["env", "stage-2"],
      "plugins": ["transform-vue-jsx", "transform-es2015-modules-commonjs", "dynamic-import-node"]
    }
  }
}
```
**配置package.json的jest部分**，或使用单独配置文件，本文使用单独文件：test/unit/jest.conf.js
```javascript
const path = require('path')

module.exports = {
  rootDir: path.resolve(__dirname, '../../'),
  moduleFileExtensions: [
    'js',
    'json',
    'vue'
  ],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  transform: {
    '^.+\\.js$': '<rootDir>/node_modules/babel-jest',
    '.*\\.(vue)$': '<rootDir>/node_modules/vue-jest'
  },
  testPathIgnorePatterns: [
    '<rootDir>/test/e2e'
  ],
  snapshotSerializers: ['<rootDir>/node_modules/jest-serializer-vue'],
  setupFiles: ['<rootDir>/test/unit/setup'],
  mapCoverage: true,
  coverageDirectory: '<rootDir>/test/unit/coverage',
  testRegex: '(<rootDir>/test/unit/specs/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$',
  collectCoverageFrom: [
    'src/components/vc-cat/*.{js,vue}',
    '!src/components/vc-cat/index.js',
    //'!src/**/*.{js,vue}',
    '!src/main.js',
    '!src/router/index.js',
    '!**/node_modules/**'
  ],
  'testURL': 'http://localhost'
}
```
各个字段的含义请参考官网[Configuring Jest](https://jestjs.io/docs/zh-Hans/configuration)
**配置package.json的测试命令**
```javascript
{
    ...
    scripts: {
        "unit": "jest --config test/unit/jest.conf.js",
        // 生成覆盖率报告
        "unit-c": "jest --config test/unit/jest.conf.js --coverage", 
    }
}
```
**编写测试用例**，一个测试用例在：test/unit/specs/vcInput.spec.js
```javascript
import Vue from 'vue'
import vcinput from '@/components/vc-cat/vc-input'

export function getLoaderDom(component, propsData, selector, key) {
  const Constructor = Vue.extend(component)
  const vm = new Constructor({propsData:propsData}).$mount()
  return vm.$el.querySelector(`${selector}`)[key]
}

describe('test vc-input', () => {
  it('props should object', () => {
    expect(typeof vcinput.props).toBe('object')
  })
  it('data should function', () => {
    expect(typeof vcinput.data).toBe('function')
  })
  it('instance is correct when value props', () => {
    expect(getLoaderDom(vcinput,{value: 'Hello'},".nc-form-group-item > input","value"))
      .toBe('Hello')
    expect(getLoaderDom(vcinput,{value: 'World'},".nc-form-group-item > input","value"))
      .toBe('World')
  })
  it('update input value', (done) => {
    const Constructor = Vue.extend(vcinput)
    const vm = new Constructor({propsData:{value: 'Vuejs'}}).$mount()
    expect(vm.$el.querySelector('.nc-form-group-item > input').value).toBe('Vuejs')
    vm.value = 'taiping'
    Vue.nextTick(() => {
      expect(vm.$el.querySelector('.nc-form-group-item > input').value).toBe('taiping')
      done()
    })
  })
})
```
**测试执行**，npm run unit或npm run unit-c

![npm-run-unit](/images/npm-run-unit.jpg)

# 添加Vue Test Utils 
安装test-utils相关包
```javascript
> npm i @vue/test-utils --dev-save
> npm i @vue/server-test-utils --dev-save
```
**编写测试用例**，一个测试用例在：test/unit/specs/Foo.spec.js
```javascript
import { shallowMount } from '@vue/test-utils'
import Foo from '@/components/test-foo'

const factory = (values = {}) => {
  return shallowMount(Foo, {
    data () {
      return {
        ...values
      }
    }
  })
}

describe('Foo', () => {
  it('renders a welcome message', () => {
    const wrapper = factory()

    expect(wrapper.find('.message').text()).toEqual("Welcome to the Vue Start")
  })

  it('renders an error when username is less than 7 characters', () => {
    const wrapper = factory({ username: ''  })

    expect(wrapper.find('.error').exists()).toBeTruthy()
  })

  it('renders an error when username is whitespace', () => {
    const wrapper = factory({ username: ' '.repeat(7)  })

    expect(wrapper.find('.error').exists()).toBeTruthy()
  })

  it('does not render an error when username is 7 characters or more', () => {
    const wrapper = factory({ username: 'Lachlan'  })

    expect(wrapper.find('.error').exists()).toBeFalsy()
  })
})

```
**测试执行**，npm run unit或npm run unit-c

# 配置typescript使用
单元测试可以配合ts使用，配置步骤和上面类似，知识需要对ts及其文件做一些处理和配置，可参考官网[配合 TypeScript 使用](https://vue-test-utils.vuejs.org/zh/guides/#配合-typescript-使用)
一个vue-ts的demo项目：[vue-demo-for-ts](https://github.com/blueskyawen/vue-demo-for-ts)，可以做测试。

更多内容在官网[vue-test-utils.vuejs.org](https://vue-test-utils.vuejs.org/zh/)



