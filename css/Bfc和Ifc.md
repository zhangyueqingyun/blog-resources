# Bfc 和 Ifc
## FC （Formatting Context）
页面中的一个渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。

- bfc (Blocking Formatting Context)
- ifc (Inline Formatting Context)

## BFC 
BFC，即块级格式化上下文，它既不是 css 的一个属性，也不是一段代码，而是一个规范，就是页面上的一个隔离区域，容器里面的子元素不会在布局上影响到外面的元素。
### BFC 规则
- 内部盒子在垂直方向防止，内部元素不会受到外部元素的影响。
- 盒子垂直方向的距离由 margin 决定，属于同一个 BFC 的两个相邻 Box 的上下 margin 之间会发生重叠
- 每个元素的左边，与包含的盒子的左边相接触，即使存在浮动也是如此
- BFC 的区域不会与外面的浮动元素重叠
- 计算 BFC 高度时包含浮动子元素的高度
### 如何产生 BFC 
- 根元素
- float 的属性不为 none
- position: absolute 或 fixed
- display: inline-block table-cell table-caption flex
- overflow 不为 visible 的块级元素
### BFC 用途
- 清除内部浮动
- 解决外边距重叠
## IFC 
内联格式化上下文。IFC 的 line box (线框)高度由其包含行内元素中最高的实际高度计算而来（不受到竖直方向的 padding/margin 影响）

IFC 中的 line box 一般左右都紧贴整个 IFC, 但会因为 float 扰乱。float 元素会位于 IFC 与 line box 之间，使得 line box 宽度缩短。同个 ifc 下的多个 line box 高度会不同。 IFC 中时不可能有块级元素的，当插入块级元素时，会产生两个匿名块与 div 分隔开，即产生两个 IFC，每个 IFC 对外表现为块级元素，与 div 垂直排列。

### IFC 的作用
- 水平居中： 当一个块要在环境中水平居中时，设置其为 inline-block 则会在外层产生 IFC，通过 text-align 则可以使其水平居中。
- 垂直居中：当创建一个 IFC, 用其中一个元素撑开父元素的高度，然后设置其 vertical-align: middle，其他行内元素则可以在此父元素下垂直居中。