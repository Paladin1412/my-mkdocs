> 让设计看起来炫酷，也是我喜欢前端的原因之一，话说，谁又不爱呢？



# 0. 概述

Vue 在插入、更新或者移除 DOM 时，提供多种不同方式的应用过渡效果。



# 1. 过渡组件 transition 

Vue 提供了 `transition` 的封装组件，可以给任何元素和组件添加 entering/leaving 过渡。



## 1.1 过渡的过程

当插入或删除包含在 `transition` 组件中的元素时，Vue 将会做以下处理：

1. 自动嗅探目标元素是否应用了 **CSS 过渡或动画**，如果是，在恰当的时机添加/删除 CSS 类名。
2. 如果过渡组件提供了**JavaScript 钩子函数**，这些钩子函数将在恰当的时机被调用。
3. 如果没有找到 JavaScript 钩子并且也没有检测到 CSS 过渡/动画，DOM 操作 (插入/删除) 在下一帧中立即执行。(注意：此指浏览器逐帧动画机制，和 Vue 的 `nextTick` 概念不同)



## 1.2 过渡CSS 类名 

在进入/离开的过渡中，会有 6 个 class 切换。

1. `v-enter`：定义进入过渡的**开始状态**。在元素被插入时生效，在下一个帧移除。
2. `v-enter-active`：定义过渡的状态。在元素**整个过渡过程**中作用，在元素被插入时生效，在 `transition/animation` 完成之后移除。这个类可以被用来定义过渡的过程时间，延迟和曲线函数。
3. `v-enter-to`:  定义进入过渡的**结束状态**。在元素被插入一帧后生效 (于此同时 `v-enter` 被删除)，在 `transition/animation` 完成之后移除。
4. `v-leave`: 定义离开过渡的**开始状态**。在离开过渡被触发时生效，在下一个帧移除。
5. `v-leave-active`：定义过渡的状态。在元素**整个过渡过程**中作用，在离开过渡被触发后立即生效，在 `transition/animation` 完成之后移除。这个类可以被用来定义过渡的过程时间，延迟和曲线函数。
6. `v-leave-to`:  定义离开过渡的**结束状态**。在离开过渡被触发一帧后生效 (于此同时 `v-leave` 被删除)，在 `transition/animation` 完成之后移除。

* 此外可以定制一个显性的**过渡持续时间**：

  ```
  <transition :duration="{ enter: 500, leave: 800 }">...</transition>
  ```

  ​

## 1.3 CSS过渡动画

CSS 动画用法同 CSS 过渡，区别是在动画中 `v-enter` 类名在节点插入 DOM 后不会立即删除，而是在 `animationend` 事件触发时删除。

示例：

```
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```



## 1.4 自定义过渡类名

可通过以下特性来自定义过渡类名：

- `enter-class`
- `enter-active-class`
- `enter-to-class` (2.1.8+)
- `leave-class`
- `leave-active-class`
- `leave-to-class` (2.1.8+)

示例，与第三方 CSS 动画库 [Animate.css](https://daneden.github.io/animate.css/) 结合使用：

```
 <transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight"
  >
    <p v-if="show">hello</p>
  </transition>
```



## 1.5 Javascript 钩子

```
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

> 当只用 JavaScript 过渡的时候，**在 enter 和 leave 中，回调函数 done 是必须的**。否则，它们会被同步调用，过渡会立即完成。



# 2. 初始渲染过渡

可以通过 `appear` 特性设置节点的在初始渲染的过渡。

* 自定义CSS类名

```
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class" (2.1.8+)
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```

* 自定义Javascript钩子

```
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
>
  <!-- ... -->
</transition>
```



# 3. 多元素过渡

## 3.1 简单标签

> 当有**相同标签名**的元素切换时，需要通过 `key` 特性设置唯一的值来标记以让 Vue 区分它们。

```
<transition>
  <button v-if="isEditing" key="save">
    Save
  </button>
  <button v-else key="edit">
    Edit
  </button>
</transition>
```



## 3.2 动态组件

不需要使用 `key` 特性，只需要使用[动态组件](https://cn.vuejs.org/v2/guide/components.html#动态组件)：

```
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```



## 3.3 过渡模式

- `in-out`：新元素先进行过渡，完成之后当前元素过渡离开。
- `out-in`：当前元素先进行过渡，完成之后新元素过渡进入。



# 4. 列表过渡

在 `v-for` 场景中，可使用 `<transition-group>`组件同时渲染整个列表。

- 不同于 `<transition>`，它会以一个真实元素呈现：默认为一个 `<span>`。你也可以通过 `tag` 特性更换为其他元素。
- 内部元素 **总是需要** 提供唯一的 `key` 属性值

```
 <transition-group name="list" tag="ul">
    <li v-for="item in items" v-bind:key="item" class="list-item">
      {{ item }}
    </li>
  </transition-group>
```



## 4.1 进入/离开过渡

```
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to{
  opacity: 0;
  transform: translateY(30px);
}
```



## 4.2 排序过渡

```
.list-move {
  transition: transform 1s;
}
```



## 4.3 交错过渡

通过 data 属性与 JavaScript 通信 ，就可以实现列表的交错过渡。

```
<transition-group
    name="list"
    tag="ul"
    v-bind:css="false"
    v-on:before-enter="beforeEnter"
    v-on:enter="enter"
    v-on:leave="leave"
  >
    <li
      v-for="(item, index) in computedList"
      v-bind:key="item.msg"
      v-bind:data-index="index"
    >{{ item.msg }}</li>
  </transition-group>
```

>  可以配合使用第三方 JavaScript 动画库使用，如 Velocity.js



# 5. 过渡的复用

创建一个可复用过渡组件：

```
Vue.component('my-special-transition', {
  template: '\
    <transition\
      name="very-special-transition"\
      mode="out-in"\
      v-on:before-enter="beforeEnter"\
      v-on:after-enter="afterEnter"\
    >\
      <slot></slot>\
    </transition>\
  ',
  methods: {
    beforeEnter: function (el) {
      // ...
    },
    afterEnter: function (el) {
      // ...
    }
  }
})
```

> 当然，使用**render函数**更适合创建这类组建。



# 6. 状态过渡

对于数据元素本身的动态效果，比如：

- 数字和运算
- 颜色的显示
- SVG 节点的位置
- 元素的大小和其他的属性

使用第三方库来实现切换元素的过渡状态，如Tween.js。

> **状态过渡**大有可为，能让设计富有生趣，好生研究之。





