# PerformanceObserver
（摘录自 mdn 和其他博客）
## PerformanceObserver
PerformanceObserver() 构造函数使用给定的观察者生成一个新的 PerformanceObserver 对象。当通过 observe() 方法注册的条目类型的性能条目事件被记录下来时，调用观察者回调。
## 语法
~~~javascript 
var observer = new PerformanceObserver(callback)
~~~
callback 的第一个参数是性能观察条目列表，第二个参数是观察者对象。返回值是观察者对象。
~~~javascript
var observer = new PerformanceObserver(function(list obj) {
    var entries = let.getEntries()
    for(var i = 0; i < entries; i++){

    }
})
observer.observe({
    entryTypes: ["mark", "frame"]
})
function perf_observer(list, observer) {
    // Process the "measure" event
}
var observer2 = new PerformanceObserver(perf_observer)
observer2.observe({
    entryTypes: ["measure"]
})
~~~
## PerformanceObserverEntryList
~~~javascript 
const observe_all = new PerformanceObserver(function(list, obs){
    const perfEntries = list.getEntries()
    for(let i = 0; i < perfEntries; i++) {
        print_perf_entry(perfEntries[i])
    }
})
~~~