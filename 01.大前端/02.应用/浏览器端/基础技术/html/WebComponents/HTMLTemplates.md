# HTMLTemplateElement
html template 和它的名字一样，是一个模板标签，在它创建的上下文中，浏览器是不回去解析的，也不会去加载里面的任何资源。当使用 js 操作 dom 时可以利用 template.content 将其下子节点转换为真实的 dom 进行解析。  

template 元素除了所有元素的共有属性，只有一个私有属性：content，这个属性是只读的，返回 templates 内部的文档片段对象及其 DOM 结构。
~~~javascript
// html
<template id="tpl">
    <div>template</div>
</template>
<div id="container">
</div>
// js 
const tpl = document.getElementById('tpl')
const container = document.getElementById('container')
container.appendChild(tpl.content)
~~~