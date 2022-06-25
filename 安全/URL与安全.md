# URL
#### 本文章大部分内容引用自《白帽子讲浏览器安全》一书。
## URL 的标准形式
### scheme://[user:password@]domain:port/path?query_string#fragments
## URL、字符集、URI
### 最初 URL 中允许出现的字符又 A-Za-z0-9 、半角的连字符。
### 后来引入了对 Unicode 字符进行特殊处理的通用编码/解码算法。这种 URI 成为国际化资源定位符（IRI，Internationalized Resource Identifier)。

## 字形欺骗钓鱼攻击（国际化字形欺骗攻击）
### 利用一些语系字母外形和英文字母外形几乎一样的特性，从而欺骗受害者的视觉判断的攻击模式。
### 解决方案：
* 当URL中出现 punycode 时，浏览器会提示用户域名中含有Unicode字符。
* 浏览器对部分字符做了特殊处理（自动纠错）。
* Unicode分解映射：调用Unicode国际化组件库的标准化函数来将特定 Unicode 分解为 ASCII 字符。
## 登录信息钓鱼攻击
### 利用 URL 中登录信息的部分伪装成正常 URL 来钓鱼，例如 http://www.qq.com:80@evil.com/ 的形式。
### 解决方案：
* 浏览器通过高亮显示，区分登录信息和域名。
* 隐藏登录信息。