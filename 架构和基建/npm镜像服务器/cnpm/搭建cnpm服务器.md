# 搭建 cnpm 服务器
## 克隆 cnpm 项目
~~~
git clone https://github.com/cnpm/cnpmjs.org.git
cd cnpmjs.org
npm install
~~~
## 安装 mysql 数据库
## 修改配置文件
~~~javascript
{
    bindingHost: '0.0.0.0',
    database: {
        db: 'dbname',
        host: 'dbhost',
        port: 3306,
        username: 'root',
        password: 'xxx'
    },
    enablePrivate: false,
    scopes: ['@xx'],
    syncMode: 'exist',
    enableAbbreviatedMetadata: true
} 
~~~
## 执行项目
~~~
npm run start
~~~
## 发布
### 添加用户
~~~
npm adduser --registry=http://xxx.xxx.xxx.xxx:7001
Username:
Password:
Email:
~~~
### 登录用户
~~~
npm login --registry=http://xxx.xxx.xxx.xxx:7001
Username:
Password:
Email:
~~~
### 配置映射
~~~
npm config set registry http://xxx.xxx.xxx.xxx:7001
~~~
### 上传代码
~~~
npm publish
~~~
