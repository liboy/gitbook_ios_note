# Charles —— 青花瓷

Charles其实是一款代理服务器，通过过将自己设置成系统（电脑或者浏览器）的网络访问代理服务器，然后截取请求和请求结果达到分析抓包的目的。该软件是用Java写的，能够在Windows，Mac，Linux上使用。安装Charles的时候要先装好Java环境。

## 主要功能：

- 截取Http 和 Https 网络封包。

- 支持重发网络请求，方便后端调试。

- 支持修改网络请求参数。

- 支持网络请求的截获并动态修改。

- 支持模拟慢速网络。

## 使用

* 手机&电脑在同一个局域网
* 确保电脑能够通过路由器访问互联网
* 电脑安装 `Charles`
* 启动 `Charles`，禁用 `MAC OS X Proxy` & `Mozilla FireFox Proxy`
* 设置手机的网络代理
    * `ip`: 电脑的 ip
    * `端口`: 8888

**注意**，如果让电脑通过手机的 `3G` 访问网络，无法拦截数据

iOS使用Charles（青花瓷）抓包并篡改返回数据图文详解
https://www.cnblogs.com/dsxniubility/p/4621314.html

Charles青花瓷 解锁https
https://blog.csdn.net/yuanliyin079/article/details/79493052