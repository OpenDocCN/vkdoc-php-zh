# 20. Vue.js

`Vue.js` 因其灵活性和被 Laravel 等社区采用，成为近年来最受赞誉的 JavaScript 项目之一。它的愿景首先聚焦于*渐进式可适配性*这一理念，即 `Vue.js` 既可以作为一个库，仅仅是用户界面的装饰器，也可以作为高度定制化架构的成熟框架。这一使命使 `Vue.js` 区别于我们在这些章节中讨论的其他一些项目。

`Vue.js` 文档中有如下陈述：

> *Vue（读音 /vjuː/，类似 view）是一个用于构建用户界面的渐进式框架。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。*

`Vue.js` 强调其追求的三大特性：

*   *易学易用*：`Vue.js` 旨在让编写 HTML、CSS 和 JavaScript 的开发者易于理解。

*   *灵活多变*：`Vue.js` 的目标是在范围有限的库和功能完善的框架之间保持灵活性。

*   *性能卓越*：`Vue.js` 拥有 20KB（min+gzip）的运行时和性能极高的虚拟 DOM。

几个显著的特征将 `Vue.js` 与用于创建 JavaScript 应用程序的其他常用工具区分开来。例如，与 React 类似，`Vue.js` 使用了虚拟 DOM，并提供了响应式和可组合的视图组件。此外，尽管 `Vue.js` 支持 JSX（React 那种类似 XML 的声明式语法，参见第 17 章），但 `Vue.js` 默认也提供了基于 HTML 的模板，这类似于 AngularJS 的方法（比较 AngularJS 指令 `ng-if` 和 Vue.js 指令 `v-if`）。

得益于 `Vue.js` 的渐进式可适配性，它可以在不同的范围内使用，包括作为一个 `<script>` 元素嵌入，而不是需要一个完整的 Node.js 构建。你可以从 `Vue.js` 网站下载开发版或生产就绪版并进行嵌入，或者你也可以像我们在这里做的那样，从一个内容分发网络（CDN）引入某个版本。

这一特性也突显了与 Angular 和 Ember 相比，`Vue.js` 的轻约束性。此外，PHP 社区中的其他人，尤其是 Laravel，已经探索并将 `Vue.js` 作为前端开发的标准基础。

**注意：** 关于 `Vue.js` 的完整文档，请参阅 `Vue.js` 网站 [`http://vuejs.org`](http://vuejs.org)。关于直接嵌入 `Vue.js` 的信息，请参阅 [`https://vuejs.org/v2/guide/installation.html#Direct-lt-script-gt-Include`](https://vuejs.org/v2/guide/installation.html%2523Direct-lt-script-gt-Include)。

## Vue.js 核心概念

`Vue.js` 的一些核心概念包括声明式渲染、指令、条件与循环以及组件。在本节中，我们还将探讨不断成熟的 `Vue.js` 生态系统中的元素，例如 Vue CLI、`Vue.js` 插件和 `Vue.js` 预设。不过，首先，有必要审视一下模型-视图-视图模型（MVVM）架构模式，`Vue.js` 从中汲取了大量灵感。

## Vue.js 的 MVVM 启发式模式

MVVM 架构模式与 MVC 架构类似，不同之处在于*视图模型*，它是一个值转换器，负责转换数据模型中存在的数据对象，以便这些数据能够被轻松渲染和维护。因此，视图模型可以被视为更像模型而非视图，它处理视图的大部分（即使不是全部）显示行为。

在 `Vue.js` 中，*视图模型* 是视图层之上的一个抽象层，它在一个 `options` 对象中提供属性，该对象包含 `data` 和 `methods`（用于定义行为）。视图模型也可以被视为模型内数据的当前状态，或者是一个*绑定器*，负责管理视图中处理和应用程序中数据绑定逻辑之间的所有通信。从这个意义上说，视图模型类似于 MVC 架构或 MVP 架构中的控制器。

例如，考虑以下示例，它定义了一个视图模型，并构成了我们的 `Vue.js` 应用程序的基础。

```javascript
// vm 是 ViewModel 的缩写。
var vm = new Vue({
    // options 对象
});
```

当我们创建一个 `Vue.js` 实例时，我们需要提供一个 `options` 参数作为一个对象，该对象包含应用程序中要使用的 `data` 和 `methods`。一个 `Vue.js` 应用程序通常由一个根 `Vue.js` 实例组成，该实例也可以有选择地组织成一个嵌套和可复用组件的树。因此，所有组件本身也是 `Vue.js` 实例，并接受 `options` 对象。

当我们像前面的例子那样实例化一个 `Vue.js` 实例时，它会将 `data` 对象中的所有属性提供给 `Vue.js` *响应式系统*，这样每当数据更新时，视图就会相应地做出反应。由于属性仅在实例化 `Vue.js` 实例时在 `data` 对象中才是响应式的，因此，如果一个属性不是立即使用，则必须提供初始值。^(⁸³)

## 声明式渲染与指令

`Vue.js` 使用存储为 `.vue` 文件（如果是 `<script>` 嵌入，则为 HTML 文件）的 HTML 模板，以及用 ES5 或 ES6 编写的、定义 `Vue.js` 应用程序行为的 JavaScript 文件。考虑以下示例，该示例在 `.app` 类上初始化一个 Vue 应用程序，并传入要在 `{{ greeting }}` 元素内渲染的数据。

将以下内容插入新创建的 `index.html` 文件和一个 `index.js` 文件中，我们也已将后者嵌入到我们的 HTML 中。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Vue.js app</title>
</head>
<body>
    <div class="app">
        {{ greeting }}
    </div>
    <script src="https://unpkg.com/vue"></script>
    <script src="index.js"></script>
</body>
</html>
```

```javascript
// index.js
var app = new Vue({
    el: '.app',
    data: {
        greeting: 'Hello world!'
    }
});
```

与 AngularJS 类似，`Vue.js` 指令以 `v-` 为前缀，并为渲染后的 DOM 提供特定的响应式行为。例如，考虑以下示例，它确保链接的 `href` 属性与在 `Vue.js` 中对 `url` 属性所做的任何修改保持同步。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Vue.js app</title>
</head>
<body>
    <div class="app">
        <a v-bind:href="url">What year is this?</a>
    </div>
    <script src="https://unpkg.com/vue"></script>
    <script src="index.js"></script>
</body>
</html>
```

```javascript
// index.js
var app = new Vue({
    el: '.app',
    data: {
        url: 'https://en.wikipedia.org/wiki/'
            + new Date().getFullYear().toString()
    }
});
```

我们还可以借助 `v-on` 指令处理动态用户输入，该指令包含我们可能希望在 `Vue.js` 中处理的某些用户操作。考虑以下示例，它允许用户点击一个按钮来更新应用程序初始化时生成的年份。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Vue.js app</title>
</head>
<body>
    <div class="app">
        {{ year }}
        <button v-on:click="updateYear">Update year</button>
    </div>
    <script src="https://unpkg.com/vue"></script>
    <script src="index.js"></script>
</body>
</html>
```

```markdown
考虑一个示例实现，其中我们通过 JSON API 在 `Vue.js` 中检索了一个节点集合。使用 `v-for` 指令，我们可以构建一个遍历该集合的 `for` 循环，并渲染 JSON API 响应中提供的某些属性。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Vue.js app</title>
</head>
<body>
    <div class="app">
        <div v-for="node in nodes">
            <h2>{{ node.attributes.title }}</h2>
            <p>{{ node.attributes.created }}</p>
            <p>{{ node.attributes.body.value }}</p>
        </div>
    </div>
    <script src="https://unpkg.com/vue"></script>
    <script src="index.js"></script>
</body>
</html>
```

```javascript
// index.js
var app = new Vue({
    el: '.app',
    data: {
        nodes: [
            {
                attributes: {
                    title: 'Capto',
                    created: 1526387013,
                    body: {
                        value: 'Camur'
                    }
                }
            }
        ]
    }
});
```

## Vue.js 组件

考虑以下示例，其中我们定义了一个 `Vue.js` 组件，并为它提供了一些虚拟数据，然后通过指令将其渲染为 HTML。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Vue.js app</title>
</head>
<body>
    <div class="app">
        <node-item v-for="item in nodeList" v-bind:node="item"></node-item>
    </div>
    <script src="https://unpkg.com/vue"></script>
    <script src="index.js"></script>
</body>
</html>
```

```javascript
// index.js
Vue.component('node-item', {
    props: ['node'],
    template: `<div>
        <h2>{{ node.attributes.title }}</h2>
        <p>{{ node.attributes.created }}</p>
        <p>{{ node.attributes.body.value }}</p>
    </div>`
});

var app = new Vue({
    el: '.app',
    data: {
        nodeList: [
            {
                id: '3ca469da-b905-4a77-8d97-954abcdc4cf6',
                attributes: {
                    title: 'Capto',
                    created: 1526387013,
                    body: {
                        value: 'Camur'
                    }
                }
            }
        ]
    }
});
```

正如你所见，在这个代码示例中，我们定义了一个名为 `node-item` 的组件，它带有某些属性，这些属性是从表示虚拟 API 响应的数据中传入的。

**注意：** 在 JavaScript 中，反引号用于表示多行字符串。

## Vue.js 生态系统

在我们转向开发一个功能完备的、由 Drupal 驱动的 `Vue.js` 应用程序之前，我们首先应该考虑 `Vue.js` 生态系统中一些有助于开发者快速构建应用程序的重要元素。

例如，对于大型项目以及为了快速启动开发，使用 `Vue.js` 的官方命令行界面 `Vue CLI` 可能会很有用，它提供了许多项目模板可供使用。要全局安装 `Vue CLI`，请执行以下 `npm` 或 `yarn` 命令。

```
npm install -g @vue/cli
# 或
yarn global add @vue/cli
```