# Shadow DOM 
Shadow Dom 接口可以将一个隐藏的、独立的 dom 附加到一个元素上。

Shadow Dom 允许将隐藏的 DOM 树附加到常规的 DOM 树中，它以 shadow-root 节点为起始根节点，在这个根节点的下方可以是任意元素，和普通的  DOM 元素相同。
## 基本概念
### shadow host
常规的 dom 节点，Shadow Dom 会被附加到这个节点上。
### shadow tree 
Shadow dom 内部的 dom 树。
### shadow boundary
Shadow DOM 结束的地方，也是常规 DOM 开始的地方。
### shadow root 
Shadow tree 的根节点
## 用法
### attachShadow 
创建 shadow root
~~~javascript
// 可以通过页面的方法来获取 Shadow DOM  
let shadow = elementRef.attachShadow({mode: 'open'})
// 不可以通过页面的方法来获取 Shadow DOM 
let shadow = elementRef.attachShadow({mode: 'closed'})
// 为 shadow-dom 添加元素
const element = document.createElement('div')
shadow.appendChild(element)
~~~
### 结合 CustomElements 使用 ShadowDom
~~~javascript
class PopupInfoElement extends HTMLElement {
    constructor() {
        super()
        this.init()
        this.appendDiv()
    }
    init() {
        this.shadow = this.attachShadow({mode: open})
    }
    appendDiv() {
        const divElement = document.createElement('div')
        this.shadow.appendChild(divElement)
    }
} 
~~~