# Vue.js知识点梳理

## 0. 目录

[TOC]

## 1. Vue.js学习资料

### 第一步
Html+css+js+基础游览器原理（dom-tree操作，渲染流程）： **达成-开发mint模板**

相关资料：

- 基础：[W3CSchool](http://www.w3school.com.cn/) ：边学边练，每章之后还有小测试。
- 进阶1： **《CSS权威指南（第3版）》**属性细节不必记忆，以后用到肯定要再查的。着重点放在大局上，比如盒模型，浮动和定位这些，抓住重点快速过一遍。
- 进阶2：**《JavaScript高级程序设计（第3版）》**前七章是重中之重，必须反复阅读，直至完全理解。DOM，事件流，表单，JSON，Ajax与最后几章也相当重要。其余章节可以略读或跳过。
- 进阶3：**《你不知道的JS》**将JavaScript的坑一网打尽。比如闭包，this之类的都可以在这里找到答案。[可选]
- 进阶4：**《JavaScript设计模式与开发实践》**讲JavaScript的设计模式。[可选]
- 进阶5：**《ES6 标准入门》** ES6需要了解，这本书以API居多，留个印象就好。

### 第二步
初步了解，Node.js，npm，webpack，构建前端工程的基础

相关资料：

- Node.js与npm网上资料很多，专注前端的话了解即可
- webpack：前端模块化全靠它，参考[入门Webpack，看这篇就够了](http://www.jianshu.com/p/42e11515c10f)

### 第三步
选一种前端框架：vue / react / angular2.0，建议vue，入门简单，组件化强大，社区还行，中文文档多：**达成-开发前端项目**

相关资料：

- vue作者推荐的vue新手入门：[新手向：Vue 2.0 的建议学习顺序](https://zhuanlan.zhihu.com/p/23134551)
- vue中文文档：[vue中文文档](https://vuefe.cn/v2/guide/)
- 一个开源的vue后台管理系统：[vue-admin](https://github.com/vue-bulma/vue-admin)

## 2. Vue.js 知识点梳理

### 2.1. Vue.js介绍

是一套构建用户界面的渐进式框架。与其他重量级框架不同的是，Vue 采用自底向上增量开发的设计。Vue 的核心库只关注视图层，它不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与单文件组件和 Vue 生态系统支持的库结合使用时，Vue 也完全能够为复杂的单页应用程序提供驱动。

#### Hello world

```
<script src="https://unpkg.com/vue"></script>

<div id="app">
  <p>{{ message }}</p>
</div>

<script>
new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue.js!'
  }
})
</script>
```

### 2.2. 指令与数据绑定

Vue.js 的核心是，可以采用简洁的模板语法来声明式的将数据渲染为DOM（通过`{{}}`），数据和 DOM 被关联在一起，并且所有的数据和 DOM 都是响应式的。例如helloworld中的`message`变量，修改它的值可以同步更新UI。

除了文本插值(text interpolation)，我们还可以采用这样的方式绑定 DOM 元素属性：

```
<div id="app-2">
  <span v-bind:title="message">
    鼠标悬停此处几秒，
    可以看到此处动态绑定的 title！
  </span>
</div>
```

```
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: '页面加载于 ' + new Date()
  }
})
```

这里我们遇到一些新内容。`v-bind`属性被称为指令。指令带有前缀 `v-`，表示是由 Vue 提供的专用属性。可能你已经猜到了，它们会在渲染的 DOM 上产生专门的响应式行为。简而言之，这里该指令的作用就是：“将此元素的 `title` 属性与 Vue 实例的 `message` 属性保持关联更新”。

类似的属性还有`v-if`，`v-for`，`v-on`，`v-model`等等。


### 2.3. 组件

组件系统是 Vue 的另一个重要概念，因为它是一种抽象，可以让我们使用小型、自包含和通常可复用的组件，把这些组合来构建大型应用程序。仔细想想，几乎任意类型的应用程序界面，都可以抽象为一个组件树：

![](https://vuefe.cn/images/components.png)

在 Vue 中，一个组件本质上是一个被预先定义选项的 Vue 实例，在 Vue 中注册组件很简单：

```
Vue.component('todo-item', {
  // todo-item 组件现在接受一个 "prop"，
  // 类似于一个自定义属性。
  // 此 prop 名为 todo。
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```

现在你可以在另一个组件模板中组合使用它：

```
<div id="app-7">
  <ol>
    <!--
      现在我们为每个 todo-item 提供了 todo 对象，
      其中它的内容是动态的。
      我们还需要为每个组件提供一个 "key"，
      这将在之后详细解释。
    -->
    <todo-item
      v-for="item in groceryList"
      v-bind:todo="item"
      v-bind:key="item.id">
    </todo-item>
  </ol>
</div>
```

```
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { id: 0, text: '蔬菜' },
      { id: 1, text: '奶酪' },
      { id: 2, text: '其他人类食物' }
    ]
  }
})
```

大型应用中的一个理想化的组件应用模板：

```
<div id="app">
  <app-nav></app-nav>
  <app-view>
    <app-sidebar></app-sidebar>
    <app-content></app-content>
  </app-view>
</div>
```

### 2.4. Vue实例

每个 Vue.js 应用都是通过构造函数 Vue 创建一个 Vue 的根实例 启动的：

```
var vm = new Vue({
  // 选项
})
```

虽然没有完全遵循 MVVM 模式， Vue 的设计无疑受到了它的启发。因此在文档中经常会使用 vm (ViewModel 的简称) 这个变量名表示 Vue 实例。
在实例化 Vue 时，需要传入一个选项对象，它可以包含数据、模板、挂载元素、方法、生命周期钩子等选项。全部的选项可以在 [API](https://cn.vuejs.org/v2/api/#选项-数据) 文档中查看。

#### 计算属性

```
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // a computed getter
    reversedMessage: function () {
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    }
  }
})
```

### 2.5. 事件机制 `v-on`

```
<div id="example-2">
  <!-- `greet` 是在下面定义的方法名 -->
  <button v-on:click="greet">Greet</button>
</div>
```

```
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // 在 `methods` 对象中定义方法
  methods: {
    greet: function (event) {
      // `this` 在方法里指当前 Vue 实例
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM 事件
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
// 也可以用 JavaScript 直接调用方法
example2.greet() // -> 'Hello Vue.js!'
```

### 2.6. 自定义指令

除了 Vue 核心携带着的一些默认指令（v-model 和 v-show）之外，Vue 还允许你注册自己的自定义指令。请注意，在 Vue 2.0 中，代码重用和抽象(reuse and abstraction)的主要是以组件的形式 - 但是，可能某些情况下，还是需要对普通元素进行一些底层 DOM 访问，这也是自定义指令仍然有其使用场景之处。接着来看一个在输入元素获取焦点的示例，如下：

```
// 注册一个名为 v-focus 的全局自定义指令
Vue.directive('focus', {
  // 当绑定的元素插入到 DOM 时调用此函数……
  inserted: function (el) {
    // 元素调用 focus 获取焦点
    el.focus()
  }
})
```

### 2.7. 单文件组件

在很多Vue项目中，我们使用 Vue.component 来定义全局组件，紧接着用 new Vue({ el: '#container '}) 在每个页面内指定一个容器元素。

- 全局定义(Global definitions) 强制要求每个 component 中的命名不得重复
- 字符串模板(String templates) 缺乏语法高亮，在 HTML 有多行的时候，需要用到丑陋的 \
- 不支持CSS(No CSS support) 意味着当 HTML 和 JavaScript 组件化时，CSS 明显被遗漏
- 没有构建步骤(No build step) 限制只能使用 HTML 和 ES5 JavaScript, 而不能使用预处理器，如 Pug (formerly Jade) 和 Babel

文件扩展名为 .vue 的 single-file components(单文件组件) 为以上所有问题提供了解决方法，并且还可以使用 webpack 或 Browserify 等构建工具。

[](https://vuefe.cn/images/vue-component.png)


作为 promise，我们可以使用预处理器来构建简洁和功能更丰富的组件，比如 Pug，Babel (with ES2015 modules)，和 Stylus。

### 2.8. 路由

如果你只是需要非常基本的路由，而不希望涉及到功能全面路由库，那么你可以通过如下方式动态渲染页面级别组件：

```
const NotFound = { template: '<p>Page not found</p>' }
const Home = { template: '<p>home page</p>' }
const About = { template: '<p>about page</p>' }
const routes = {
  '/': Home,
  '/about': About
}
new Vue({
  el: '#app',
  data: {
    currentRoute: window.location.pathname
  },
  computed: {
    ViewComponent () {
      return routes[this.currentRoute] || NotFound
    }
  },
  render (h) { return h(this.ViewComponent) }
})
```

官方路由库 **Vue-router**：

```
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```

```
// 0. 如果使用模块化机制编程，導入Vue和VueRouter，要调用 Vue.use(VueRouter)

// 1. 定义（路由）组件。
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }

// 2. 定义路由
// 每个路由应该映射一个组件。 其中"component" 可以是
// 通过 Vue.extend() 创建的组件构造器，
// 或者，只是一个组件配置对象。
// 我们晚点再讨论嵌套路由。
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]

// 3. 创建 router 实例，然后传 `routes` 配置
// 你还可以传别的配置参数, 不过先这么简单着吧。
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
})

// 4. 创建和挂载根实例。
// 记得要通过 router 配置参数注入路由，
// 从而让整个应用都有路由功能
const app = new Vue({
  router
}).$mount('#app')

// 现在，应用已经启动了！
```

### 2.8. 状态管理模式：VueX

由于多个状态分散的跨越在许多组件和交互间各个角落，大型应用复杂度也经常逐渐增长。为了解决这个问题，Vue 提供 vuex： 我们有受到 Elm 启发的状态管理库。vuex 甚至集成到 vue-devtools，无需配置即可访问时光旅行。

**状态管理模式设计的目标是：记录变更 (mutation) 、保存状态快照、历史回滚/时光旅行
**

一个简单的VueX例子：

```
// 如果在模块化构建系统中，请确保在开头调用了 Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

[更多VueX信息](https://vuex.vuejs.org/zh-cn/intro.html)

### 2.9. webpack与vue-cli: *生产环境 vs. 发布环境*

#### 什么是webpack

WebPack可以看做是模块打包机：它做的事情是，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Scss，TypeScript等），并将其打包为合适的格式以供浏览器使用。

Webpack的工作方式是：把你的项目当做一个整体，通过一个给定的主文件（如：index.js），Webpack将从这个文件开始找到你的项目的所有依赖文件，使用loaders处理它们，最后打包为一个浏览器可识别的JavaScript文件。

![](http://upload-images.jianshu.io/upload_images/1031000-160bc667d3b6093a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 什么是 Vue-cli

vue-cli 是一个官方发布 vue.js 项目脚手架，使用 vue-cli 可以快速创建 vue 项目。

通过一行命令就可以生成一个Vue应用工程：

```
vue init webpack Vue-Project
```

Vue-cli与webpack是搭建**生产环境与发布环境分离**项目的基础工具。可以提供预编译、压缩、缓存、错误跟踪等等功能。

## 3. Node.js + Express 实现后端

### 3.1. Node.js & Express 简介
### 3.2. MVC模式与架构
### 3.3. MongoDB


