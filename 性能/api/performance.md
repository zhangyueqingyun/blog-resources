# 浏览器性能 Api
Web Performance API 令开发者们可以精确地测量 Web 应用程序的性能。
## 浏览器自动测量的各种指标
window.performance 上的属性暴露了网页自动测量的各种指标。
- memory 网页内存使用情况
- navigation 打开网页的方式、重定向的次数
- timing 网页加载过程中的性能信息（建立连接、DNS 查询等）
- eventCounts 记录分发过事件的数量
- timeOrigin 性能测量开始时间高精度时间戳
- onresourcetimingbufferfull 事件回调，当触发 resourcetimingbufferfull 事件的时候会被调用
## 利用 Performance API 手动测量项目的性能
在时间轴上标记某个时刻
~~~javascript
performance.mark('start')
performance.mark('end')
~~~
计算两个标记之间的性能指标
~~~javascript
performance.measure('measure1','start', 'end')
~~~
获取性能指标
~~~javascript
// 获取所有指标
performance.getEntries()
// 根据名称获取指标
performance.getEntriesByName('start')
performance.getEntriesByName('end')
performance.getEntriesByName('measure1')
// 根据类型获取指标
performance.getEntriesByType('mark')
performance.getEntriesByType('measure')
performance.getEntriesByType('resource')
~~~ 
获取该时刻精确时间
~~~javascript
performance.now()
~~~
(明日追更)