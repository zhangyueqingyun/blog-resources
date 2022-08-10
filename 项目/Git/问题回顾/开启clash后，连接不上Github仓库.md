# 开启 Clash 后连接不上Git仓库

解决方案：配置 Git 的 http 代理

~~~
/**
    端口号改为自己的端口号
 */
git config --global http.proxy http://localhost:1080
~~~