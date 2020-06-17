> 笔记略长，阅久略乏，故一分为二。

# 5. 插槽

当子组件模板只有一个无属性插槽时，父组件传入的整个内容片段将插入到插槽所在的 DOM 位置，并替换掉插槽标签本身。 `<slot>` 标签中内容都被视为**备用内容**，在没有要插入的内容时才显示备用内容。



## 5.1 具名插槽

`<slot>` 元素可以用一个特殊的特性 `name` 来配置如何分发内容。与其相对的是**匿名插槽**，作为找不到匹配的内容片段的备用插槽。

假定`app-layout` 组件，其模板为：

```
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

父组件模板为：

```
<app-layout>
  <h1 slot="header">这里可能是一个页面标题</h1>
  <p>主要内容的一个段落。</p>
  <p>另一个主要段落。</p>
  <p slot="footer">这里有一些联系信息</p>
</app-layout>
```

渲染结果为：

```
<div class="container">
  <header>
    <h1>这里可能是一个页面标题</h1>
  </header>
  <main>
    <p>主要内容的一个段落。</p>
    <p>另一个主要段落。</p>
  </main>
  <footer>
    <p>这里有一些联系信息</p>
  </footer>
</div>
```



## 5.2 作用域插槽

作用域插槽用作一个 (能被传递数据的) 可重用模板，来代替已经渲染好的元素。

在子组件中，只需将数据传递到插槽，就像你将 prop 传递给组件一样：

```
<ul>
  <slot name="item"
    v-for="item in items"
    :text="item.text">
    <!-- 这里写入备用内容 -->
  </slot>
</ul>
```

在父级中，`slot-scope` 的值将被用作一个临时变量名，此变量接收从子组件传递过来的 prop 对象：

```
<my-awesome-list :items="items">
  <!-- 作用域插槽也可以是具名的 -->
  <li slot="item"
    slot-scope="props">
    {{ props.text }}
  </li>
</my-awesome-list>
```



# 6. 混合 (Mixins)

混合 (Mixins) 使Vue组件灵活地复用。

```
// 定义一个混合对象
var myMixin = {
  created: function () {
    console.log('hello from mixin!')
  }
}
// 定义一个使用混合对象的组件
var Component = Vue.extend({
  mixins: [myMixin]
})
var component = new Component() // => "hello from mixin!"
```



## 6.1 选项合并

- 混合对象的钩子与组件自身钩子同名，两者都被调用， 混合对象的钩将先执行 。

```
var mixin = {
  created: function () {
    console.log('混合对象的钩子被调用')
  }
}
new Vue({
  mixins: [mixin],
  created: function () {
    console.log('组件钩子被调用')
  }
})
// => "混合对象的钩子被调用"
// => "组件钩子被调用"
```

- 值为对象的选项，例如 `methods`, `components` 和 `directives`，将被混合为同一个对象。两个对象键名冲突时，取组件对象的键值对。

```
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}
var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})
vm.foo() // => "foo"
vm.bar() // => "bar"
vm.conflicting() // => "from self"
```



## 6.2 全局混合

一旦全局注册混合对象，将会影响到 所有之后创建的 Vue 实例。

```
Vue.mixin({})
```



# 7. 自定义指令

除默认核心指令 (`v-model` 和 `v-show`)外，Vue 允许注册自定义指令。

例如，当页面加载时，元素将获得焦点 (注意：autofocus 在移动版 Safari 上不工作)。

- 全局注册

```
// 注册一个全局自定义指令 v-focus
Vue.directive('focus', {
  // 当绑定元素插入到 DOM 中。
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
```

- 局部注册

```
directives: {
  focus: {
    // 指令的定义
    inserted: function (el) {
      el.focus()
    }
  }
}
```

- 使用指令

```
<input v-focus>
```



## 7.1 钩子函数

指令定义函数提供了几个钩子函数 ：

- `bind`：只调用一次，指令第一次绑定到元素时调用，可定义始化动作。
- `inserted`：被绑定元素插入父节点时调用 。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其孩子的 VNode 更新之前**。可通过比较更新前后的值来忽略不必要的模板更新 。
- `componentUpdated`：所在组件的 VNode **及其孩子的 VNode** 全部更新时调用。
- `unbind`：只调用一次，指令与元素解绑时调用。

## 7.2 钩子函数的参数

函数参数包括 `el`，`binding`，`vnode`，`oldVnode`。



# 8. 渲染函数 (render)

Vue 绝大多数情况下使用 template 来创建 HTML。但也能通过 JavaScript 来实现，这得用到 **render 函数**。

```
Vue.component('blog-header', {
	render: function (createElement) {
  		return createElement(
  		'h1',             //tag name 标签名称
  		this.blogTitle    //子组件中的阵列
  		)
	}
})
```

> render可以做很多事情，请务必仔细看文档。



# 9.  插件

- 定义插件

```
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或属性
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }
  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })
  // 3. 注入组件
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  })
  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
}
```

- 使用插件

```
Vue.use(MyPlugin, { someOption: true })
```

- [awesome-vue](https://github.com/vuejs/awesome-vue#components--libraries) 集合了来自社区贡献的数以千计的插件和库。

# 10. 过滤器

- 自定义过滤器

```
new Vue({
  // ...
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
})
```

- 使用

```
<!-- in mustaches -->
{{ message | capitalize }}
<!-- in v-bind -->
<div v-bind:id="message | capitalize"></div>
```



