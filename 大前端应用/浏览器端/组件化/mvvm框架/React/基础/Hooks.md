# Hooks 
## 分类
### 状态
- useState 
- useReducer
- useContext
- useRef
- useDefferedValue
- useMemo (其实该按优化分类，但也可这么用，react 框架会在一些情况下自动抛弃缓存数据，重新计算)
### 副作用
- useEffect
- useLayoutEffect
### 优化
- useCallback
- useMemo 
### 功能
- useRef
- useImpretiveHandle
### 调试
- useDebugValue
### 过渡
- useTransition
### 内部 hooks 
- useSyncExternalStore
- useInsertionEffect
## 详解
### 状态 hooks 
#### useState 
useState 和类组件的 setState 不同，不会自动合并更新对象。
~~~javascript
const [state, setState] = useState({});
setState((prevState) => ({
        ...prevState,
        ...updateState
}));
setState({});
~~~
初始化 state
~~~javascript
const [state, setState] = useState(() => {
    const initialState = someExpensiveComputation(props);
    return initialState;
})
~~~
#### useReducer 
状态管理,单向数据流
~~~jsx
const initialState = { count: 1 };

function initialFunc() {
    return initialState;
}

function reducer(state, action) {
    switch(action.type) {
        case 'increment':
            return {count: state.count + 1};
        case 'decrement':
            return {count: state.count - 1};
        defualt:
            throw new Error();
    }
}

function Counter() {
    const {state, dispatch} = useReducer(reducer, initialState, initialFunc);
    return (<>
        Count: {state.count}
        <button onClick={()=>{dispatch({type: "decrement"})}}>-</button>
    </>)
}
~~~
#### useContext 
实现跨组件状态传递
~~~jsx
const value = useContext(MyContext) 
~~~
例子
~~~jsx
const themes = {
    light: [
        foreground: '#000000',
        background: '#eeeeee'
    ],
    dark: {
        foreground: '#ffffff',
        background: '#222222'
    }
} 

const ThmeContext = React.createContext(themes.light);

function App() {
    return (
        <ThemeContext.Provider value={themes.dark}>
            <Toolbar />
        </ThemeContext.Provider>
    );
}

function Toolbar(props) {
    return (<div>
        <ThemeButton />
    </div>)
}

function ThemedButton() {
    const theme = useContext(ThemeContext);
    return (
        <button style={{background: theme.background, color: theme.foreground}}>
            I am styled by theme context!
        </button>
    )
}
~~~
#### useRef 
跨生命周期保存数据, 不引发渲染。
~~~jsx 
function Demo {
    const timerId = useRef()
    useEffect(()=>{
        timerId.current = setInterval(()=>{
        }, 1000);
    }, []);
    return (<></>);
}
~~~
#### useDefferedValue 
使用低优先级的更新状态，在有渲染任务时会延迟更新 value;
~~~jsx
function Typeahead() {
    const query = useSearchQuery('');
    const defferredQuery = useDeferredValue(query);

    const suggestions = useMemo(
        () => <SearchSuggestions query={defferedQuery} />,
        [deferredQuery]
    );
    
    return (
        <>
            <SearchInput query={query} />
            <Suspense fallback="Loading results...">
                {suggestions}
            </Suspense>
    )
}
~~~
#### useMemo 
使用计算状态，当依赖不改变时，使用缓存的值。如果不提供依赖数组，则每次都会计算。
~~~javascript
const memoIzedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
~~~
### 副作用
#### useEffect
副作用功能并发模式的实现 (mutation, subscription, timers, logging) 

当组件渲染到屏幕上后，会调用副作用功能。当组件被卸载时会执行函数返回的清理函数。
~~~javascript
useEffect(() => {
    const subscription = props.source.subscrive();
    // 清理函数
    return () => {
        subscription.unsubscribe();
    };
});
~~~
当没有依赖项时，副作用会每次调用。当有依赖项时，副作用会在依赖项改变时调用。
#### useLayoutEffect 
副作用同步模式的实现。在使用不能推迟执行的副作用时调用该 api。
### 优化
#### useCallback 
对函数性能的优化，以防每次更新重复生成函数。
~~~javascript
const memoIzedCallback = useCallback(
    () => {
        doSomething(a, b);
    },
    [a, b]
);
~~~
#### useMemo 
使用计算状态，当依赖不改变时，使用缓存的值。如果不提供依赖数组，则每次都会计算。
~~~javascript
const memoIzedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
~~~