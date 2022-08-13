# FSWatcher 
文件系统观察者，由 fs.watch() 创建，所有的文件系统观察者当观察到文件被改动时都会触发 "change" 事件。
## watcher.close()
停止观察
## watcher.ref()
当被调用时，wather 不再需要 Nodejs 的事件循环保持活跃。如果没有其他的活动保持事件循环运行，当 wathcer 的 callback 执行完后，
进程会种植。多次调用 watcher.unref 没有副作用 