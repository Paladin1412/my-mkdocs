> 看过 Vue.js 的 Cookbook 之后才明白其强大之处，多说无益。

# 1. 前言

## 1.1 概念

Vue.js 是一套构建用户界面的**渐进式框架**，采用自底向上增量开发的设计。一个轻巧，高性能，可组件化的MVVM库。



## 1.2 兼容性

Vue.js 支持所有[兼容 ECMAScript 5 的浏览器](http://caniuse.com/#feat=es5)。



## 1.3 较之其它框架

### 与 React 比较

React 和 Vue 有许多相似之处，它们都有：

- 使用 Virtual DOM
- 提供了响应式 (Reactive) 和组件化 (Composable) 的视图组件。
- 将注意力集中保持在核心库，而将其他功能如路由和全局状态管理交给相关的库

#### # 框架性能

通常 Vue 会有少量优势，因为 Vue 的 Virtual DOM 实现相对更为轻量一些。

#### # 状态管理

React 社区在状态管理方面非常有创新精神 (比如 Flux、Redux)，而这些状态管理模式甚至 [Redux 本身](https://github.com/egoist/revue)也可以非常容易的集成在 Vue 应用中。实际上，Vue 更进一步地采用了这种模式 ([Vuex](https://github.com/vuejs/vuex))，更加深入集成 Vue 的状态管理解决方案。

#### # 原生渲染

React Native 能使你用相同的组件模型编写有本地渲染能力的 APP (iOS 和 Android)。能同时跨多平台开发，对开发者是非常棒的。相应地，Vue 和 [Weex](https://alibaba.github.io/weex/) 会进行官方合作，Weex 是阿里的跨平台用户界面开发框架，Weex 的 JavaScript 框架运行时用的就是 Vue。这意味着在 Weex 的帮助下，你使用 Vue 语法开发的组件不仅仅可以运行在浏览器端，还能被用于开发 iOS 和 Android 上的原生应用。

但目前，Weex 还在积极发展，成熟度也不能和 React Native 相抗衡。



### 与 Angular2 比较

Vue 的一些语法和 Angular1的很相似 (例如 `v-if` vs `ng-if`)，因为 Angular1是 Vue 早期开发的灵感来源。

这里只与Angular2做一下比较。

#### # TypeScript

Angular 事实上必须用 TypeScript 来开发，因为它的文档和学习资源几乎全部是面向 TS 的。TS 有很多好处——静态类型检查在大规模的应用中非常有用，同时对于 Java 和 C# 背景的开发者也是非常提升开发效率的，但在不用 TS 的情况下使用 Angular 会很有挑战性。对于Vue，大量用户也在生产环境中使用 Vue + TS 的组合。

#### # 大小

一个包含了 Vuex + Vue Router 的 Vue 项目 (30kB gzipped) 相比优化后的 `angular-cli` 生成的默认项目尺寸 (~130kB) 还是要小的多。

#### # 灵活性

Vue 相比于 Angular 更加灵活，Vue 官方提供了构建工具来协助你构建项目，但它并不限制你去如何组织你的应用代码。



## 1.4 安装

[官方命令行工具](https://github.com/vuejs/vue-cli)安装，只需几分钟即可创建并启动一个带热重载、保存时静态检查以及可用于生产环境的构建配置的项目：

```
# 全局安装 vue-cli
$ npm install --global vue-cli
# 创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
# 安装依赖，走你
$ cd my-project
$ npm install
$ npm run dev
```



# 2.  Vue 实例

```
var data = { a: 1 } 
//创建一个实例
var vm = new Vue({
  el: '#example',
  data: data
})
```



## 2.1 实例的属性

当一个 Vue 实例被创建时，它向 Vue 的**响应式系统**中加入了其 `data` 对象中能找到的所有的属性。

```
// 对于上述实例，vm 和 data 引用相同的对像
vm.a === data.a // => true
```



## 2.2 实例的方法

除了 data 属性，Vue 实例暴露了一些有用的实例属性与方法。它们都有前缀 `$`，以便与用户定义的属性区分开来。例如：

```
// 针对上述实例
vm.$data === data // => true
vm.$el === document.getElementById('example') // => true
// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
```



## 2.3 实例生命周期

 Vue 实例在被创建之前都要经过一系列的初始化过程，例如设置数据监听、编译模板、挂载实例到 DOM、在数据变化时更新 DOM 等。

同时会运行一些叫做**生命周期钩子**的函数，例如`created`、`mounted`、`updated`、`destroyed`等，钩子的 `this` 指向调用它的 Vue 实例。

```
new Vue({
  data: { a: 1 },
  created: function () {
    console.log('a is: ' + this.a); // `this` 指向 vm 实例
  }
})
```

官方提供的实例的生命周期图，如下所示。

![lifecycle](https://cn.vuejs.org/images/lifecycle.png)	

# 3. 计算属性

在模板中放入太多的逻辑会让模板过重且难以维护。例如：

```
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

所以，对于任何复杂逻辑，都应当使用**计算属性**。

```
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

`vm.reversedMessage` 的值始终取决于 `vm.message` 的值。当 `vm.message` 发生改变时，所有依赖 `vm.reversedMessage` 的绑定也会更新。

```
console.log(vm.reversedMessage) // => 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```



## 3.1 计算属性 vs 方法

可将函数定义为一个方法，实现相同结果。

```
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}

// 在模板中调用
<p>{{ reverseMessage() }}</p>
```



不同的是**计算属性是基于它们的依赖进行缓存的**。计算属性只有在它的相关依赖发生改变时才会重新求值。这就意味着只要 `message` 还没有发生改变，多次访问 `reversedMessage` 计算属性会立即返回之前的计算结果，而不必再次执行函数。

相反，计算属性`vm.now`将不会再更新，因为`Date.now()`不是响应式依赖：

```
computed: {
  now: function () {
    return Date.now()
  }
}
```

**若不希望有缓存，请用方法来替代。**



## 3.2 计算属性的setter

计算属性默认只有 getter ，不过也可以提供一个 setter ：

```
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// 运行 vm.fullName = 'John Doe' 时，setter 会被调用。
```



## 3.3 计算属性 vs 侦听属性

当有一些数据需要随着其它数据变动而变动时，你很容易滥用 `watch`中的**侦听属性**。然而，通常更好的做法是使用计算属性而非命令式的 `watch` 回调。

细想如下例子：

```
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

```
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```



## 3.4 何时使用侦听器 watch

**当需要在数据变化时执行异步或开销较大的操作时，使用侦听器**。使用 `watch` 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。



# 4. 组件



## 4.1 全局注册

要注册一个全局组件，可以使用 `Vue.component(tagName, options)`：

```
Vue.component('my-component', {
  // 选项
})
```



## 4.2 局部注册

也可以通过某个 Vue 实例/组件的实例选项 `components` 注册仅在其作用域中可用的组件：

```
var Child = {
  template: '<div>A custom component!</div>'
}
new Vue({
  // ...
  components: {
    // <my-component> 将只在父组件模板中可用
    'my-component': Child
  }
})
```



## 4.3 父子组件通信

在 Vue 中，父子组件的关系可以总结为 **prop 向下传递，事件向上传递**。父组件通过 **prop** 给子组件下发数据，子组件通过**事件**给父组件发送消息。

![lifecycle](https://cn.vuejs.org/images/props-events.png) 





## 4.4 Prop 单项数据流

Prop 是单向绑定的：当父组件的属性变化时，将传导给子组件，但是反过来不会。这是为了防止子组件无意间修改了父组件的状态，来避免应用的数据流变得难以理解。

> 如果 prop 是一个对象或数组，在子组件内部改变它**会影响**父组件的状态。



## 4.5 自定义事件

每个 Vue 实例都实现了[事件接口](https://cn.vuejs.org/v2/api/#实例方法-事件)，即：

- 使用 `$on(eventName)` 监听事件
- 使用 `$emit(eventName)` 触发事件

```
vm.$on('test', function (msg) {
  console.log(msg)
})
vm.$emit('test', 'hi')   // => "hi"
```



## 4.6 非父子组件通信

非父子关系的两个组件之间也需要通信。简单场景下，可以使用一个空的 Vue 实例作为**事件总线**：

```
var bus = new Vue()
```

```
// 触发组件 A 中的事件
bus.$emit('id-selected', 1)
```

```
// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
})
```

复杂场景下，应该使用专门的**状态管理模式**，如vuex。



## 4.7 动态组件

通过使用**保留的 `<component>` 元素**，动态地绑定到它的 `is` 特性，我们让多个组件可以使用同一个挂载点，并动态切换：

```
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

```
<component v-bind:is="currentView">
  <!-- 组件在 vm.currentview 变化时改变！ -->
</component>
```



## 4.8 子组的引用

```
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```

```
var parent = new Vue({ el: '#parent' })
// 访问子组件实例
var child = parent.$refs.profile
```

> `$refs` 只在组件渲染完成后才填充，它是非响应式的，应避免在模板或计算属性中使用 `$refs`



## 4.9 异步组件

Vue.js 允许将组件定义为一个工厂函数，异步地解析组件的定义。Vue.js 只在组件需要渲染时触发工厂函数，并且把结果缓存起来，用于后面的再次渲染。例如：

```
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // 将组件定义传入 resolve 回调函数
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```



## 4.10 高级异步组件

```
const AsyncComp = () => ({
  // 需要加载的组件。应当是一个 Promise
  component: import('./MyComp.vue'),
  // 加载中应当渲染的组件
  loading: LoadingComp,
  // 出错时渲染的组件
  error: ErrorComp,
  // 渲染加载中组件前的等待时间。默认：200ms。
  delay: 200,
  // 最长等待时间。超出此时间则渲染错误组件。默认：Infinity
  timeout: 3000
})
```

