# 功能概览
cnpmjs 是基于 koa 搭建的 nodejs 包镜像服务器。后台数据库支持 mysql, sqlite, postgres, mariadb。cnpmjs 镜像服务器提供了大致 80 个路由提供包管理客户端使用。分别为 Web 端和包管理端。
## 接口概览
### Web 端主要接口
- 根据包名和版本号查询包
- 获取所有私有包
- 查找包
- 获取用户信息
- 获取日志
- 获取范围内（scope）的包（例如 @babel/）
### 包管理主要接口
- 获取所有包 （npm search）
- 获取所有包名
- 登录注册，用户管理，token 相关
- 获取 scope 中的包
- 添加包
- 添加 tag
- need limit by ip (暂时每看这部分是干什么的)
- 删除 tarball 并移除一个版本号
- 更新一个包
- 删除所有版本
- 获取用户下所有包
- 获取下载次数
- 获取包的依赖
- dist-tags 相关功能 
