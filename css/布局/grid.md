# grid布局
grid 是一个二维布局系统。由行和列组合包围的网格线叫做 grid item, 由最外围的航和列的边框叫做 grid container。还有行分界线和列分界线。
## 概览
用 display: grid 将容器元素定义为一个 grid 网格布局。使用 grid-template-column 和 grid-template-rows 设置列和行的尺寸大小，然后通过 grid-column 和 grid-row 将其子元素放入网格中。
### 网格容器
应用 display: grid 的元素。这是所有网格项的直接父级元素。
### 网格项
网格容器的子元素。
### 网格线
构成网格结构的分界线。他们可以是垂直的，也可以是水平的，并且位于行和列的任意一侧。
### 网格轨道
两条相邻网格线之间的空间
### 网格单元格
两个相邻的行和两个相邻的列网格线之间的空间，是网格系统的一个单元。
### 网格区域
四条网格线包围的总空间。一个网格区域可以由任意数量的网格单元格组成。
## 网格容器属性
- display 
- grid-template-columns 
- grid-template-rows 
- grid-template-areas
- grid-template 
- grid-column-gap
- grid-row-gap 
- grid-gap 
- justify-items 
- align-items 
- place-items 
- justify-content 
- align-content 
- place-content 
- grid-auto-columns 
- grid-auto-rows 
- grid-auto-flow 
- grid
## 网格项名称
- grid-column-start 
- grid-column-end 
- grid-row-start 
- grid-row-end 
- grid-column
- grid-row 
- grid-area 
- justify-self 
- align-self 
- place-self 