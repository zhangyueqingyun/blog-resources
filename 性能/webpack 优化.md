# Webpack 性能优化
## 减少 ES6 转为 es5 的冗余代码
使用 babel-blugin-transform-runtime 
## 提取公共代码
当项目没有将每个页面的公共模块提取出来的话，会存在
- 相同的资源被重复加载，浪费用户的流量和服务器的成本
- 每个页面需要加载的资源太大，导致网页首屏加载变得缓慢，影响用户体验
所以我们要将多个页面的公共代码抽离成为单独的文件，来优化以上问题。Webpack 内置了专门用于提取 Chunk 中的公共部分的插件 CommonsChunkPlugin，我们在项目中 CommonsChunkPlugin 的配置如下
~~~javascript
new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    minChunks: function(module, count) {
        return (
            module.resource &&
            /\.js/.test(module.resource) && 
            module.resource.indexOf(
                path.join(__dirname, '../node_modules')

            ) === 0
        )
    }
}) ,
new webpack.optimize.CommonChunkPlugin({
    name: 'manifest',
    chunks: ['vender']
})
~~~
## 模板预编译
当使用 DOM 内模板或 JavaScript 内的字符串模板时，模板会在运行时北边也为渲染函数。预编译模板最简单的方式就是使用单文件组件，相关的构建设置会自动把预编译处理好，所以构建好的代码已经包含了编译出来的渲染函数而不是原始的模板字符串。

如果你是用了webpack，喜欢分离Javascript模板文件，可以使用 vue-template-loader，它也可以在构建的过程中预编译模板。
## 提取组件 CSS 
当使用单文件组件时，组件内的 CSS 会以 style 标签的方式通过 JavaScript 动态注入。这有一些小小的开销。如果使用服务端渲染，会导致一段“无样式内容闪烁”。将所有组件的 CSS 提取到同一个文件可以避免这个问题。
- webpack + vue-loader
- Browserify + vueify
- rollup + rollup-plugin-vue
## 按需加载代码
通过 Vue 写单页面应用时，会有很多的路由引入。当打包构建的时候，JS 包会变得非常大，影响加载。如果我们能够把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应的组件，这样就更加高效了。
~~~javascript
const Foo = () => import('./Foo.vue')
const router = new Router({
    routes:[
        {path: '/foo', component: Foo}
    ]
}) 
~~~