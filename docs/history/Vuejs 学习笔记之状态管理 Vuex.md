> Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**，对整个应用程序的状态进行集中式管理。



# 1. 前言



## 1.1 应用场景

当我们的应用遇到**多个组件共享状态**时，我么可能需要用到状态管理机制：

- 多个视图依赖于同一状态。
- 来自不同视图的行为需要变更同一状态。

在简单场景下，一个简单的 **global event bus** 就足够所需。

```
var bus = new Vue()

// 触发组件 A 中的事件
bus.$emit('id-selected', 1)

// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
	// ...
})
```

但是，如果需要构建一个中大型单页应用，您会考虑如何更好地在**组件外部管理状态**，此时应当选择 Vuex 。



## 1.2 Vuex 状态转移图  

Vuex 的基本思想借鉴了 [Flux](https://facebook.github.io/flux/docs/overview.html)、[Redux](http://redux.js.org/)，同时较之数据响应的粒度更细致，更契合Vue.js。

 ![vuex](https://vuex.vuejs.org/zh-cn/images/vuex.png)	



## 1.3 基本思想

Vuex 应用的核心是 store。“store” 是一个仓库，包含应用中大部分的**状态 (state)**：

1. Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，相应的组件会更新。
2. 改变 store 中的状态的唯一途径是显式地**提交 (commit) mutation**。



# 2. 核心概念



## 2.1 State

Vuex 使用**单一状态树**，作为“唯一数据源 ([SSOT](https://en.wikipedia.org/wiki/Single_source_of_truth))”，用一个对象包含全部的应用层级状态。

#### **`mapState` 辅助函数**

```
import { mapState } from 'vuex'

computed: mapState({
    count: state => state.count
    // ...
})
```

#### 对象展开运算符

`mapState` 函数返回的是一个对象。ECMASCript提供的对象展开运算符，有助于`mapState` 函数返回值与局部计算属性混合使用。

```
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```



## 2.2 Getter

Vuex 允许我们在 store 中定义“getter”（可以认为是 store 的计算属性）。就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。



#### 定义 getter

```
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
  	// 第一个参数为 state，第二个参数为其他 getter 
    doneTodos: (state, getters) => {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

Getter 会暴露为 `store.getters` 对象：

```
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```



#### `mapGetters` 辅助函数

`mapGetters` 辅助函数仅仅是将 store 中的 getter 映射到局部计算属性:

```
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```



## 2.3 Mutations

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 类似于事件：每个 mutation 都有一个字符串的 **事件类型 (type)** 和 一个 **回调函数 (handler)**。

```
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state, payload) {
      state.count += payload.amount
    }
  }
})
```

“当触发一个类型为 increment 的 mutation 时，调用函数 mutation handler。”

```
 store.commit('increment', {
  amount: 10
})
```



#### 对象风格的提交方式

```
store.commit({
  type: 'increment',
  amount: 10
})
```



#### 使用常量替代 Mutation 事件类型

```
const SOME_MUTATION = 'SOME_MUTATION'
mutations: {
    [SOME_MUTATION] (state) { // ES2015 风格的计算属性命名
      // mutate state
    }
  }
```



#### `mapMutations` 辅助函数

可在组件中使用 `this.$store.commit() `提交 mutation，或使用 `mapMutations` 辅助函数将组件中的 methods 映射为 `store.commit` 调用

```
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
      'incrementBy' // `this.incrementBy(amount)` 映射为`this.$store.commit('incrementBy', amount)`
    ])
  }
}
```

> 一条重要的原则就是要记住 **mutation 必须是同步函数**。



## 2.4 Action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

```
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    incrementAsync (context) {
    	setTimeout(() => {    // mutation 必须同步执行, action 就不受约束
      		context.commit('increment')
    	}, 1000)
    }
  }
})
```



#### 分发 Action

```
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})
// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```



#### `mapActions` 辅助函数

在组件中使用 `this.$store.dispatch()` 分发 action，或者使用 `mapActions` 辅助函数将组件的 methods 映射为 `store.dispatch` 调用。

```
import { mapActions } from 'vuex'

export default {
  methods: {
    ...mapActions([
      'increment', // 将 `this.increment()` 映射为 `this.$store.dispatch('increment')`
      'incrementBy' //`this.incrementBy(amount)` 映射为 `this.$store.dispatch('incrementBy', amount)`
    ])
  }
}
```



#### 组合 Action

假设利用 [async / await](https://tc39.github.io/ecmascript-asyncawait/) 这个 JavaScript 新特性：

```
// 假设 getData() 和 getOtherData() 返回的是 Promise
actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // 等待 actionA 完成
    commit('gotOtherData', await getOtherData())
  }
}
```



## 2.4 Module

Vuex 可将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：

```
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  // ...
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```



#### 命名空间

默认情况下，模块内部的 action、mutation 和 getter 是注册在**全局命名空间**的。若想模块具有更高的封装度和复用性，可添加 `namespaced: true` 使其成为命名空间模块。



#### 在命名空间模块内访问全局内容（Global Assets）

如果向使用全局 state 和 getter，`rootState` 和 `rootGetter` 会作为第三和第四参数传入 getter，也会通过 `context` 对象的属性传入 action。



#### 带命名空间的绑定函数

 ```
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,  // ===  a: state => state.some.nested.module.a,
    b: state => state.b   // ===  b: state => state.some.nested.module.b
  })
},
methods: {
  ...mapActions('some/nested/module', [
    'foo',   // === 'some/nested/module/foo',
    'bar'    // === 'some/nested/module/bar',
  ])
}
 ```

或者，

```
import { createNamespacedHelpers } from 'vuex'
const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
export default {
  computed: {
    // 在 `some/nested/module` 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // 在 `some/nested/module` 中查找
    ...mapActions([
      'foo',
      'bar'
    ])
  }
}
```



#### 模块动态注册

在 store 创建**之后**，你可以使用 `store.registerModule` 方法注册模块：

```
// 注册模块 `myModule`
store.registerModule('myModule', {
  // ...
})
// 注册嵌套模块 `nested/myModule`
store.registerModule(['nested', 'myModule'], {
  // ...
})
```

可以使用 `store.unregisterModule(moduleName)` 来动态卸载模块。



# 3. 项目结构

Vuex 需遵守以下规则：

1. 应用层级的状态应该集中到单个 store 对象中。
2. 提交 **mutation** 是更改状态的唯一方法，并且这个过程是同步的。
3. 异步逻辑都应该封装到 **action** 里面。

只要遵守以上规则，如何组织代码随便。

```
├── index.html
├── main.js
├── api
│   └── ... # 抽取出API请求
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # 我们组装模块并导出 store 的地方
    ├── actions.js        # 根级别的 action
    ├── mutations.js      # 根级别的 mutation
    └── modules
        ├── cart.js       # 购物车模块
        └── products.js   # 产品模块
```



# 4. 插件

Vuex 的 store 接受 `plugins` 选项，这个选项暴露出每次 mutation 的钩子。Vuex 插件就是一个函数，store 作为其唯一参数：

```
const myPlugin = store => {
  // 当 store 初始化后调用
  store.subscribe((mutation, state) => {
    // 每次 mutation 之后调用
    // mutation 的格式为 { type, payload }
  })
}
```

使用：

```
const store = new Vuex.Store({
  state,
  mutations,
  plugins: [myPlugin]
})
```



