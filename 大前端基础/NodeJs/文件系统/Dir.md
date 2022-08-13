# Dir 
一个表达文件夹流的类
## dir.close 
关闭文件夹下的资源句柄，如果再次使用关闭掉的文件句柄会引发错误。
## dir.read 
读取下一个文件夹入口
## dir[Symbol.asyncIterator]
文件夹迭代器，迭代所有文件夹入口。