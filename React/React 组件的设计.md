# React 中的组件设计
这边文章根据 React 官方文档整理，简明概述了官方推荐的几种 React 组件的设计方法。

React 官方文档对组件和组件之间的包含关系与特例关系推荐了实现方式，并且提出了 render-props 解决横切关注点的问题。
## 包含关系
（类似于 vue 的插槽）

使用组合的设计
~~~jsx
function SplitPane({children, left, right}) {
    <div className="SplitPane">
        <div className="SplitPane-left">
            {left}
        </div>
        <div className="SplitPane-content">
            {children}
        </div>
        <div className="SplitPane-right">
            {right}
        </div>
    </div>
}

function App() {
    return (
        <SplitPane 
            left={<Left />}
            right={<Riglt />}
        >
            <Content />           
        </SplitPane>
    )
}
~~~ 
## 特例关系
~~~javascript
function Dialog({title, message}) {
    return (
        <FancyBorder color="blue">
            <h1 className="Dialog-title">
                {title}    
            </h1>
            <p>
                {message}
            </p>
        </FancyBorder>
    )
}

function WelcomeDiialog() {
    return (
        <Dialog 
            title="Welcome"
            message="Thank you for visiting our spacecraft" 
        />
    )
}
~~~
## 横切关注点
render prop 是一个用于告知组件需要渲染什么内容的函数 prop。

组件是包含关系，内部组件需要的属性通过函数调用的方式获取到外部组件的值。
~~~jsx
class Cat extends React.Component {
    render() {
        const {mouse} = this.props
        return (
            <img
                class="cat"   
                src="/cat.jpg"
                style={{
                    left: mouse.x,
                    top: mouse.y
                }}
        )
    }
}

class Moutse extends React.Component {
    constructor(props){
        super(props)
        this.state = {
            x: 0,
            y: 0
        }
    }
    handleMouseMove = (event) => {
        this.setState({
            x: event.clientX,
            y: event.clientY
        })
    }

    render() {
        const {render} = this.props
        return (
            return <div>
                {render(this.state)}
            </div>
        )
    }
}
class MouseTracker extends react.Component {
    render() {
        return (<div>
            <h1>移动鼠标！</h1>
            <Mouse render={mouse => (
                <Cat mouse={mouse} />
            )}
        </div>)
    }
}
// 使用高阶组件
function withMouse(Component) {
    return class extends React.Component {
        render() {
            return (
                <Mouse render={
                    mouse => (<Component {...this.props} mouse={mouse} />)
                }
        }
    }
}
~~~
## 高阶组件
使用高阶组件解决横切关注点的问题
~~~jsx 
function logProps(WrappedComponent) {
    return class extends React.Component {
        componentDidUpdate(prevProps) {
            console.log('Current porps: ', this.props)
            console.log('Previous props: ', prevProps)
        }

        render() {
            return <WrappedComponent {...this.props} />
        }
    }
}
~~~

