# Hooks 
## 分类
### 状态
- useState 
- useReducer
- useContext
- useRef
- useDefferedValue
- useMemo
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