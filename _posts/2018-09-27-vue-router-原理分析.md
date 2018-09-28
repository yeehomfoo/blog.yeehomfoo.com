> Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。

那么是路由，什么是单页面应用呢？以下将探讨这两个问题。

# 路由

## 1、服务端路由

路由这个概念最早在后端出现，比如: Java 通过监听某个端口的所有请求，然后根据请求的 URL 定位到具体的资源（或业务逻辑函数），最终返回对应的资源（或函数处理结果）。根据 URL 定位到具体资源这个过程就叫做路由。简单来说，路由是一个动作，是根据路径定位到资源的动作，而能够提供这种功能的被称为路由器。

## 2、前端路由

随着异步交互的流行，单页面应用 (SPA) 出现了。在单页面应用中，页面是无跳转无刷新的，URL 变化后不再会重新请求服务端获取资源，而是在前端直接路由到对应的资源，所以就有了前端路由。Vue Router 就是为了构建但页面应用而生的前端路由。

# 单页面应用

在 Ajax 出现以后，人们尝到了页面无刷新和异步交互的好处后，慢慢地将这种交互体验升级成为了单页面应用。

单页面应用不仅仅在页面交互上是无刷新的，连页面跳转也是无刷新的。这种技术让用户使用产品时不再有“被打断”的感觉，仿佛是在浏览器中运行了一个桌面应用。

>单页Web应用，就是只有一张Web页面的应用。浏览器一开始会加载必需的HTML、CSS和JavaScript，之后所有的操作都在这张页面完成，这一切都由JavaScript来控制。因此，单页Web应用会包含大量的JS代码，模块化开发和架构设计的重要性不言而喻

在单页应用中，界面上的各种功能区块是动态生成的。因此，我们需要把产品功能划分为若干状态，每个状态映射到相应的路由，然后通过根据不同的 URL 动态路由到对应的功能界面。这就是前端路由存在的意义，也是 Vue Router 诞生的意义。

# Vue Router 的原理

Vue Router 通过 hash 和 [HTM5 Hisotry API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) 实现路由。它一共支持三种模式:

1. hash: 默认
2. history: 浏览器支持 History API 时使用
3. abstract: 非浏览器环境中使用

下面介绍 hash 和 history 模式。

## hash 模式

类似于服务端路由，前端路由实现起来其实也很简单，就是匹配不同的 URL 路径，进行解析，然后动态的渲染出区域 HTML 内容。但是这样存在一个问题，就是 URL 每次变化的时候，都会造成页面的刷新。那解决问题的思路便是在改变 URL 的情况下，保证页面的不刷新。hash 值的变化，并不会导致浏览器向服务器发出请求，浏览器不发出请求，也就不会刷新页面。另外每次 hash 值的变化，还会触发 `hashchange` 这个事件，通过这个事件我们就可以知道 hash 值发生了哪些变化。然后我们便可以监听 `hashchange` 来实现更新页面部分内容的操作：

```js
function matchAndUpdate () {
   // todo 匹配 hash 做 dom 更新操作
}

window.addEventListener('hashchange', matchAndUpdate)
```

## history 模式

2014 年以后，因为 HTML5 标准发布。多了两个 API，`pushState` 和 `replaceState`，通过这两个 API 可以改变 URL 地址且不会发送请求。同时还有`popstate` 事件。通过这些就能用另一种方式来实现前端路由了，但原理都是跟 hash 实现相同的。用了 HTML5 的实现，单页路由的 URL 就不会多出一个#，变得更加美观。但因为没有 # 号，所以当用户刷新页面之类的操作时，浏览器还是会给服务器发送请求。为了避免出现这种情况，所以这个实现需要服务器的支持，需要把所有路由都重定向到根页面。

```js
function matchAndUpdate () {
   // todo 匹配路径 做 dom 更新操作
}

window.addEventListener('popstate', matchAndUpdate)
```

# Vue Router 注意事项

## 1、模式降级

Vue Router 的模式默认为 hash。如果设置为 history，在不支持 HTML5 History API 的浏览器 (如：IE9) 中或本地直接打开页面时会自动降级成 hash 模式。

```js
const router = new VueRouter({
  mode: 'history', // 在 IE9 中会自动降级为 hash
  routes: [...]
})
```

## 2、history 模式

vue-router 默认 hash 模式 —— 使用 URL 的 hash 来模拟一个完整的 URL，于是当 URL 改变时，页面不会重新加载。

如果不想要很丑的 hash，我们可以用路由的 history 模式，这种模式充分利用 `history.pushState` API 来完成 URL 跳转而无须重新加载页面。

需要注意的是: __history 模式需要后台配置的支持。__否则浏览器直接访问时，可能会出现 404 错误。

## 3、beforeRouteUpdate 守卫

只是参数或查询的改变时，Vue 会重用当前组件，不会触发进入／离开的导航守卫。这时，可以通过观察 `$router` 或者使用 beforeRouterUpdate 组件内守卫来执行初始化逻辑 。

```js
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
  }
}
```

## 4、beforeRouterEnter 守卫

beforeRouterEnter 守卫不能访问组件实例 `this`，不过可以通过传入一个回调函数给 `next` 来访问。

```js
beforeRouteEnter (to, from, next) {
  next(vm => {
    // 通过 `vm` 访问组件实例
  })
}
```

# Vue Router 最佳实践

最佳实践为官方推荐的使用方式，以下内容主要来源于官方。

## 1、路由组件传参

在组件中使用 `$route` 会使之与其对应路由形成高度耦合，从而使组件只能在某些特定的 URL 上使用，限制了其灵活性。

可以使用 `props` 将组件和路由解耦，解除与 `$route` 的耦合。

```js
// 未解耦
const User = {
  template: '<div>User \{\{ $route.params.id \}\}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

``` js
// 通过 props 解耦
const User = {
  props: ['id'],
  template: '<div>User \{\{ id \}\}</div>'
}
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```

这样你便可以在任何地方使用该组件，使得该组件更易于重用和测试。

## 2、路由懒加载

当打包构建应用时，JavaScript 包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。

结合 Vue 的异步组件和 Webpack 的代码分割功能，轻松实现路由组件的懒加载。

```js
const Foo = () => import(/* webpackChunkName: "group-foo" */ './Foo.vue')
const Bar = () => import(/* webpackChunkName: "group-foo" */ './Bar.vue')
const Baz = () => import(/* webpackChunkName: "group-foo" */ './Baz.vue')
```

Webpack 会将任何一个异步模块与相同的块名称组合到相同的异步块中。

这种方式实际上还是需要先下载所有组件的 JavaScript，仅仅只是推迟了客户端渲染。如果想要更近一步，还是需要用到 [SSR](https://ssr.vuejs.org/)。

# 总结

如果不理解单页面应用将难以理解 Vue Router 的意义。前端生态关联紧密，想要了解其中一环，往往需要先去了解相关联的各个环节，有一种环环相扣的感觉。

(done)

## 参考资料

[1] [Vue Router Guide](https://router.vuejs.org/guide/)

[2] [HTML5 History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API)

[3] [web 单页应用](https://www.jianshu.com/p/bae4f6604549)