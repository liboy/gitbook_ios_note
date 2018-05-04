# Cocoapods

* `CocoaPods` 是 iOS 最常用最有名的类库管理工具
* 作为 iOS 程序员，掌握 CocoaPods 的使用是必不可少的基本技能

## 安装

```bash
# 添加源
$ sudo gem sources -a http://ruby.taobao.org/
# 删除源
$ sudo gem sources -r https://rubygems.org/
# 安装
$ sudo gem install cocoapods
# 设置
$ pod setup
```

## 使用

```bash
# 搜索
$ pod search AFNetworking
# 生成 Podfile
$ echo "pod 'AFNetworking'" > Podfile
# 安装
$ pod install
# 升级
$ pod update
```

### git 操作

```bash
# 将修改添加到暂存区
$ git add .

# 提交修改
$ git commit -m "添加 AFN框架程序"
```

## gem 常用命令

```bash
# 查看gem源
$ gem sources –l
# gem自身升级
$ sudo gem update --system
# 查看版本
$ gem --version
# 清除过期的gem
$ sudo gem cleanup
# 安装包
$ sudo gem install cocoapods
# 删除包
$ gem uninstall cocoapods
# 更新包
$ sudo gem update
# 列出本地安装的包
$ gem list
```

## 插件

```bash
curl -fsSL https://raw.github.com/supermarin/Alcatraz/master/Scripts/install.sh | sh
```

github 地址：https://github.com/supermarin/Alcatraz
