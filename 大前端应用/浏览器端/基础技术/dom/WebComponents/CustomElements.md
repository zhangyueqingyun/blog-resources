# Custom Elements
## 类型
- Autonomous custom elements 是独立的元素，它不继承其他内建的 HTML 元素。你可以直接把他们写成 HTML 标签的形式，在页面使用。
- Customized Built-in elements
## Autonomous custom elements 
独立元素，不继承其他内建的 HTML 元素。可以直接把它写成 HTML 标签的形式在页面上使用。
~~~JavaScript
// 定义 class 组件
class CustomizedElement extends HTMElement {
    constructor() {
        super()
    }
}
// 注册
CustomElements.define('customized-element', CustomizedElement)
// 使用
<customized-element></customized-element>
~~~
## Customized built-in elements 
继承自基本的 HTML 元素。在创建时，你必须指定所需扩展的元素，使用时，需要先写出基本的元素标签，并通过 is 属性指定 custom element 的名称。
~~~JavaScript
// 定义
class CustomizedElement extends HTMElement {
    constructor() {
        super()
    }
}
// 注册
customElements.defined('expanding-list', ExpandingList, {extends: 'ul'})
// 使用
<ul is="expanding-list">
</ul>
~~~
## 生命周期
### connectedCallback
当 custom element 首次被插入文档时调用
### disconnectedCallback
当 custom element 从文档中被删除时调用
### adoptedCallback
当 custom element 被移动到新的文档时调用
### attributeChangedCallback 
当 custom element 增加、删除、修改自身属性时调用