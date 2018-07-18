# GitBook

## 安装
下面介绍在本地如何安装 GitBook，如果不需要本地调试 & 不需要获得生成的 html 文件，可以直接使用 [官网](https://www.gitbook.com/) 提供的服务。
<!-- toc -->

## 环境要求

* NodeJS(v4.0.0及以上)

## 通过NPM安装
运行下面的命令进行安装
```bash
npm install gitbook-cli -g
```
其中`gitbook-cli`是gitbook的一个命令行工具, 通过它可以在电脑上安装和管理gitbook的多个版本.



## 运行
* 安装 GitBook
```bash
npm install gitbook-cli -g
```
* Clone 代码到本地并运行
```bash
git clone git@github.com:zhangjikai/gitbook-use.git
cd gitbook-use
gitbook install
gitbook serve
```
* 在浏览器中打开 `http://localhost:4000/` 进行访问


## GitBook 资源

* [GitBook主页](https://www.gitbook.com/)
* [Github地址](https://github.com/GitbookIO/)
* [GitBook编辑器](https://www.gitbook.com/editor/osx)
* [GitBook Toolchain Documentation](http://toolchain.gitbook.com/)
* [GitBook Documentation](http://help.gitbook.com/)
