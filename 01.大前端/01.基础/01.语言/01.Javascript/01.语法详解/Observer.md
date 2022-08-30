# JS 中的 Observer
## IntersectionObserver
当元素位于视口中触发回调
~~~javascript 
var observer = new IntersectionObserver(callback[, options])
observer.observe(target)
observer.unobserve(target)
observer.disconnect()
~~~
## MutationObserver 
当元素发生变化时触发回调
~~~javascript
var observer = new MutationObserver(callback)
observer.observe(target, config)
~~~
## ResizeObserver 
元素尺寸发生变化时进行通知
~~~javascript
var observer = new ResizeObserver(callback)
observer.observe(target)
~~~
## PerformanceObserver 
性能指标发生变化时，触发回调
~~~javascript
var observer = new PerformanceObserver(callback)
observer.observe({entryTypes: [entryTypes]})
~~~