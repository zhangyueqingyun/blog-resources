# Vue3 项目结构概述

## 源码仓库地址（Vue2 迁移到 Vue3）
### Vue3 的源码仓库地址为 https://github.com/vuejs/core
### Vue2 的源码仓库地址为 https://github.com/vuejs/vue
#### (大家注意不要搞错)

## Vue3 项目结构
### Vue3 使用的包管理工具为 pnpm （monorepo）, 使用的构建工具为 rollup, 源码目录里的 packages 目录下面包含着各个库的源码。
### 下面为 Vue3 依赖的所有子包：
* compiler-core
* compiler-dom
* compiler-sfc
* compiler-ssr
* reactivity
* reactivity-transform
* runtime-core
* runtime-dom
* runtime-test
* server-renderer
* sfc-playground
* shared
* size-check
* template-explorer
* vue
* vue-compat
### 从 Vue3 的包构成来看，Vue3 主要分为编译时、运行时两个大的模块，还有处理响应式逻辑的模块、服务端渲染的模块等。
## 核心包介绍
### 编译阶段
* compiler-core 内有模板解析、生成 AST 等逻辑。
* compiler-dom 在模板编译过程中，和 dom 相关的逻辑。
### 运行阶段
* runtime-core 虚拟 Dom 、patch、diff 等逻辑。
* runtime-dom 运行时和 HTML5 相关逻辑。
### 响应式
* reactivity 响应式相关逻辑
