# GitBook

## 安装
先行安装`Node.js`、`GitBook`、`GitBook Editor`、`Git`版本控制器
- 环境要求`NodeJS(v4.0.0及以上)`
```
//查看 nodejs 版本
$ node -v
v8.10.0
$ npm -v
5.7.1
```
- 安装gitbook
```bash
npm install gitbook-cli -g
```
- 查看 GitBook 版本
```bash
$ gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
```

## 常用的命令

```bash
gitbook init //初始化目录文件
gitbook help //列出gitbook所有的命令
gitbook --help //输出gitbook-cli的帮助信息
gitbook build //生成静态网页
gitbook serve //生成静态网页并运行服务器
gitbook build --gitbook=2.0.1 //生成时指定gitbook的版本, 本地没有会先下载
gitbook ls //列出本地所有的gitbook版本
gitbook ls-remote //列出远程可用的gitbook版本
gitbook fetch 标签/版本号 //安装对应的gitbook版本
gitbook update //更新到gitbook的最新版本
gitbook uninstall 2.0.1 //卸载对应的gitbook版本
gitbook build --log=debug //指定log的级别
gitbook builid --debug //输出错误信息
```

## 运行
* Clone 代码到本地并运行
```bash
git clone git@github.com:liboy/gitbook_ios_note.git
cd ios_study_notes
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
* [插件官网](https://plugins.gitbook.com/)