# Vue 性能优化
## 合理使用 v-if 和 v-show
## 合理使用 watch 和 computed 
## v-for 优化
- 添加 key
- 避免同时使用 v-if 
## 长列表性能优化
Vue 会通过 Object.defineProperty 对数据进行劫持，来实现视图响应数据的变化，然而有些时候我们的组件就是纯粹的数据展示，不会有任何改变，我们就不需要 Vue 来劫持我们的数据，在大量数据展示的情况下，为了减少组件的初始化事件，我们可以通过 Object.freeze 来冻结对象
## 事件的销毁
手动移除事件以免造成内存泄露
## 图片资源懒加载
对于图片过多的页面，为了加速页面的加载速度，所以很多时候我们需要将页面内未出现在可是区域内的图片先不做加载，等到滚动到可视区域后再加载。这样对于页面加载性能上会有很大提升，也提高了用户体验。
## 路由懒加载
Vue 是单页面应用，可进行代码分割。
## 第三方插件的按需引入
- babel-plugin-component 
~~~javascript
// npm install babel-plugin-component -D 

/* .babelrc */
{
    "presets": [["es2015", {"modules": false}]],
    "plugins": [
        [
            "component",
            {
                "libraryName": "element-ui",
                "styleLibraryName": "theme-chalk"
            }
        ]
    ]
}
~~~
## 优化无限列表性能
如果你的应用存在非常长或者无限滚动的列表，那么需要采用窗口化的技术来优化性能，只需要渲染少部分区域的内容，减少重新渲染组件和创建 dom 节点的事件。可以参考 vue-virtual-scroll-list 和 vue-virtual-scroller 来优化这种无限列表的场景。
## 服务端渲染 SSR or 预渲染
如果你的项目需要服务端渲染来帮助你实现最佳的初始加载性能和 SEO，那么你的项目就需要服务端渲染来帮助你实现最佳的初始加载性能和 SEO 。

如果你的 Vue 项目只需要改善少数的营销页面的 SEO， 那么你可能需要预渲染，在构建时简单地商城对特定路由的静态 HTML 文件。有点事设置预渲染更简单，并可以将你的前段作为一个完全静态的站点。可以使用 prerender-spa-plugin。