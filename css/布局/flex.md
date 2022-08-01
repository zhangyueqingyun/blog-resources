
# flex布局
采用 Flex 布局的元素，称为 Flex 容器，它的所有子元素自动称为容器成员，称为 Flex 项目。

容器默认存在两根轴线，主轴和交叉轴。主轴的开始位置称为 main start，结束为止称为 main-end。 交叉轴开始的位置称为 cross start, 结束为止称为 cross-end。

## 容器的属性
- flex-direction: row|row-reverse|column|column-reverse 主轴方向 
- flex-wrap: nowrap|wrap|wrap-reverse 如何换行
- flex-flow: row nowrap 前两个简写
- justify-content: flex-start|flex-end|center|space-between|space-around 主轴对齐方式
- align-items: flex-start|flex-end|center|stretch|baseline 交叉轴的对齐方式
- align-content: flex-start|flex-end|center|stretch|space-between|space-around 多根轴线的对齐方式

## 项目的属性
- order 项目的排列顺序
- flex-grow 放大比例，默认0 不放大
- flex-shrink 缩小比例，默认1 缩小 
- flex-basis 定义了在分配多于空间之前，项目占据的株洲空间。浏览器根据这个属性，计算主轴是否有多余的空间。默认值为 auto。
- flex: 前三个属性的简写 默认为 0 1 auto 
- align-self 允许单个项目与其他项目有不同的对齐方式，可以覆盖 align-items 属性，默认值为 auto，表示继承父元素的 align-items 属性。