# StatWatcher 
当调用 fs.watchFile() 时被返回。
## watcher.ref()
当被调用后，watcher 必须和 Node eventLoop 一起活跃
## watcher.unref()
被调用后，Node eventLoop 可以不活跃。