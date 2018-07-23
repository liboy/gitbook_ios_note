# GitBook 插件
  


## Search Plus
支持中文搜索, 需要将默认的 `search` 和 `lunr` 插件去掉。  

[插件地址](https://plugins.gitbook.com/plugin/search-plus)

```json
{
    "plugins": ["-lunr", "-search", "search-plus"]
}
```

## Prism
使用 `Prism.js` 为语法添加高亮显示，需要将 `highlight` 插件去掉。该插件自带的主题样式较少，可以再安装 `prism-themes` 插件，里面多提供了几种样式，具体的样式可以参考 [这里](https://github.com/PrismJS/prism-themes)，在设置样式时要注意设置 css 文件名，而不是样式名。

[Prism 插件地址](https://plugins.gitbook.com/plugin/prism) &nbsp;&nbsp; [prism-themes 插件地址](https://plugins.gitbook.com/plugin/prism-themes)

## Advanced Emoji
支持emoji表情

[emoij表情列表](http://www.emoji-cheat-sheet.com/)  
[插件地址](https://plugins.gitbook.com/plugin/advanced-emoji)

```json
"plugins": [
    "advanced-emoji"
]
```
使用示例：

:bowtie: :smile: :laughing: :blush: :smiley: :relaxed:

## Github
添加github图标

[插件地址](https://plugins.gitbook.com/plugin/github)


## Emphasize
为文字加上底色

[插件地址](https://plugins.gitbook.com/plugin/emphasize)
```json
"plugins": [
    "emphasize"
]
```

## KaTex
为了支持数学公式, 我们可以使用`KaTex`和`MathJax`插件, 官网上说`Katex`速度要快于`MathJax`

[插件地址](https://plugins.gitbook.com/plugin/katex)  
[MathJax使用LaTeX语法编写数学公式教程](http://iori.sinaapp.com/17.html)
```json
"plugins": [
    "katex"
]
```

## Include Codeblock
使用代码块的格式显示所包含文件的内容. 该文件必须存在。插件提供了一些配置，可以区插件官网查看。如果同时使用 ace 和本插件，本插件要在 ace 插件前面加载。

[插件地址](https://plugins.gitbook.com/plugin/include-codeblock)  
```json
{
    "plugins": [
        "include-codeblock"
    ],
    "pluginsConfig": {
        "include-codeblock": {
            "template": "ace",
            "unindent": "true",
            "theme": "monokai"
        }
    }
}
```

## Splitter
使侧边栏的宽度可以自由调节

[插件地址](https://plugins.gitbook.com/plugin/splitter)
```json
"plugins": [
    "splitter"
]
```
## Mermaid-gb3
支持渲染[Mermaid](https://github.com/knsv/mermaid)图表  
[插件地址](https://plugins.gitbook.com/plugin/mermaid-gb3)
```json
"plugins": [
    "mermaid-gb3"
]
```


## Graph

使用 function-plot 绘制数学函数图。

[插件地址](https://plugins.gitbook.com/plugin/graph)  
[function-plot](https://mauriciopoppe.github.io/function-plot/)

```json
{
    "plugins": [ "graph" ],
}
```

## Chart
使用 C3.js 或者 Highcharts 绘制图形。

[插件地址](https://plugins.gitbook.com/plugin/chart)  
[C3.js](https://github.com/c3js/c3)  
[highcharts](https://github.com/highcharts/highcharts)

```json
{
    "plugins": [ "chart" ],
    "pluginsConfig": {
        "chart": {
            "type": "c3"
        }
    }
}
```
type 可以是 `c3` 或者 `highcharts`, 默认是 `c3`.

## Sharing-plus
分享当前页面，比默认的 sharing 插件多了一些分享方式。

[插件地址](https://plugins.gitbook.com/plugin/sharing-plus)

```json
 plugins: ["-sharing", "sharing-plus"]
```
配置:

```json
"pluginsConfig": {
    "sharing": {
       "douban": false,
       "facebook": false,
       "google": true,
       "hatenaBookmark": false,
       "instapaper": false,
       "line": true,
       "linkedin": true,
       "messenger": false,
       "pocket": false,
       "qq": false,
       "qzone": true,
       "stumbleupon": false,
       "twitter": false,
       "viber": false,
       "vk": false,
       "weibo": true,
       "whatsapp": false,
       "all": [
           "facebook", "google", "twitter",
           "weibo", "instapaper", "linkedin",
           "pocket", "stumbleupon"
       ]
   }
}
```
## Tbfed-pagefooter
为页面添加页脚

[插件地址](https://plugins.gitbook.com/plugin/tbfed-pagefooter)
```json
"plugins": [
   "tbfed-pagefooter"
],
"pluginsConfig": {
    "tbfed-pagefooter": {
        "copyright":"Copyright &copy zhangjikai.com 2017",
        "modify_label": "该文件修订时间：",
        "modify_format": "YYYY-MM-DD HH:mm:ss"
    }
}
```
## Expandable-chapters-small
使左侧的章节目录可以折叠

[插件地址](https://plugins.gitbook.com/plugin/expandable-chapters-small)

```json
plugins: ["expandable-chapters-small"]
```

## Sectionx
将页面分块显示，标签的 tag 最好是使用 b 标签，如果使用 h1-h6 可能会和其他插件冲突。  
[插件地址](https://plugins.gitbook.com/plugin/sectionx)  
```json
{
    "plugins": [
       "sectionx"
   ],
    "pluginsConfig": {
        "sectionx": {
          "tag": "b"
        }
      }
}
```

## GA
Google 统计  
[插件地址](https://plugins.gitbook.com/plugin/ga)
```json
"plugins": [
    "ga"
 ],
"pluginsConfig": {
    "ga": {
        "token": "UA-XXXX-Y"
    }
}
```
## 3-ba
百度统计  
[插件地址](https://plugins.gitbook.com/plugin/3-ba)
```json
{
    "plugins": ["3-ba"],
    "pluginsConfig": {
        "3-ba": {
            "token": "xxxxxxxx"
        }
    }
}
```
## Donate
打赏插件  
[插件地址](https://plugins.gitbook.com/plugin/donate)  
```json
"plugins": [
    "donate"
],
"pluginsConfig": {
    "donate": {
        "wechat": "https://liboy.com/resource/weixin.png",
        "alipay": "https://liboy.com/resource/alipay.png",
        "title": "",
        "button": "赏",
        "alipayText": "支付宝打赏",
        "wechatText": "微信打赏"
    }
}
```

## Local Video
使用Video.js 播放本地视频  
[插件地址](https://plugins.gitbook.com/plugin/local-video)  
```json
"plugins": [ "local-video" ]
```


## Simple-page-toc
自动生成本页的目录结构。另外 GitBook 在处理重复的标题时有些问题，所以尽量不适用重复的标题。
[插件地址](https://plugins.gitbook.com/plugin/simple-page-toc)  
```json
{
    "plugins" : [
        "simple-page-toc"
    ],
    "pluginsConfig": {
        "simple-page-toc": {
            "maxDepth": 3,
            "skipFirstH1": true
        }
    }
}

## Anchors
添加 Github 风格的锚点样式

![](https://cloud.githubusercontent.com/assets/2666107/3465465/9fc9a502-0266-11e4-80ca-09a1dad1473e.png)

[插件地址](https://plugins.gitbook.com/plugin/anchors)
```json
"plugins" : [ "anchors" ]
```
## Anchor-navigation-ex
添加Toc到侧边悬浮导航以及回到顶部按钮。需要注意以下两点：
* 本插件只会提取 h[1-3] 标签作为悬浮导航
* 只有按照以下顺序嵌套才会被提取
```
# h1
## h2
### h3
必须要以 h1 开始，直接写 h2 不会被提取
## h2
```

[插件地址](https://plugins.gitbook.com/plugin/anchor-navigation-ex)
```json
{
    "plugins": [
        "anchor-navigation-ex"
    ],
    "pluginsConfig": {
        "anchor-navigation-ex": {
            "isRewritePageTitle": true,
            "isShowTocTitleIcon": true,
            "tocLevel1Icon": "fa fa-hand-o-right",
            "tocLevel2Icon": "fa fa-hand-o-right",
            "tocLevel3Icon": "fa fa-hand-o-right"
        }
    }
}
```



## Edit Link
如果将 GitBook 的源文件保存到github或者其他的仓库上，使用该插件可以链接到当前页的源文件上。   
[插件地址](https://plugins.gitbook.com/plugin/edit-link)  
```json
"plugins": ["edit-link"],
"pluginsConfig": {
    "edit-link": {
        "base": "https://github.com/USER/REPO/edit/BRANCH",
        "label": "Edit This Page"
    }
}
```

## Sitemap-general
生成sitemap  
[插件地址](https://plugins.gitbook.com/plugin/sitemap-general)  
```json
{
    "plugins": ["sitemap-general"],
    "pluginsConfig": {
        "sitemap-general": {
            "prefix": "http://gitbook.zhangjikai.com"
        }
    }
}
```
## Favicon
更改网站的 favicon.ico  
[插件地址](https://plugins.gitbook.com/plugin/favicon)  
```json
{
    "plugins": [
        "favicon"
    ],
    "pluginsConfig": {
        "favicon": {
            "shortcut": "assets/images/favicon.ico",
            "bookmark": "assets/images/favicon.ico",
            "appleTouch": "assets/images/apple-touch-icon.png",
            "appleTouchMore": {
                "120x120": "assets/images/apple-touch-icon-120x120.png",
                "180x180": "assets/images/apple-touch-icon-180x180.png"
            }
        }
    }
}
```
## Todo
添加 Todo 功能。默认的 checkbox 会向右偏移 2em，如果不希望偏移，可以在 `website.css` 里加上下面的代码:
```css
input[type=checkbox]{
    margin-left: -2em;
}
```
[插件地址](https://plugins.gitbook.com/plugin/todo)  

```json
"plugins": ["todo"]
```

## Terminal
模拟终端显示，主要用于显示命令以及多行输出，不过写起来有些麻烦。

[插件地址](https://plugins.gitbook.com/plugin/terminal)
```json
{
    "plugins": [
        "terminal"
    ],
    "pluginsConfig": {
        "terminal": {
            "copyButtons": true,
            "fade": false,
            "style": "flat"
        }
    }
}
```

现在支持 6 种标签：
* command: Command "executed" in the terminal.
* delimiter: Sequence of characters between the prompt and the command.
* error: Error message.
* path: Directory path shown in the prompt.
* prompt: Prompt of the user.
* warning: Warning message.

标签的使用格式如下所示：
```
**[<tag_name> 内容]
```
为了使标签正常工作，需要在代码块的第一行加入 `**[termial]` 标记，下面是一个完整的示例：

<pre>```
**[terminal]
**[prompt foo@joe]**[path ~]**[delimiter  $ ]**[command ./myscript]
Normal output line. Nothing special here...
But...
You can add some colors. What about a warning message?
**[warning [WARNING] The color depends on the theme. Could look normal too]
What about an error message?
**[error [ERROR] This is not the error you are looking for]
```</pre>


## Copy-code-button
为代码块添加复制的按钮。

[插件地址](https://plugins.gitbook.com/plugin/copy-code-button)

```json
{
    "plugins": ["copy-code-button"]
}
```
效果如下图所示：

![](assets/images/copy-code-button.png)

## Alerts
添加不同 alerts 样式的 blockquotes，目前包含 info, warning, danger 和 success 四种样式。

[插件地址](https://plugins.gitbook.com/plugin/alerts)

```json
{
    "plugins": ["alerts"]
}
```
下面是使用示例：
```
Info styling
> **[info] For info**
>
> Use this for infomation messages.

Warning styling
> **[warning] For warning**
>
> Use this for warning messages.

Danger styling
> **[danger] For danger**
>
> Use this for danger messages.

Success styling
> **[success] For info**
>
> Use this for success messages.
```


## Include-csv
展示 csv 文件。

[插件地址](https://plugins.gitbook.com/plugin/include-csv)

```json
{
    "plugins": ["include-csv"]
}
```
使用示例：
```
{% includeCsv  src="./assets/csv/test.csv", useHeader="true" %} {% endincludeCsv %}
```

## Musicxml
支持 musicxml 格式的乐谱渲染。

[插件地址](https://plugins.gitbook.com/plugin/musicxml)  
[musicXML](http://www.musicxml.com/)

```
{
    "plugins": ["musicxml"]
}
```

下面是一个示例，需要注意的是 block 中的内容必须是一个合法的 musicxml 文件路径，并且不能有换行和空格。
```
{% musicxml %}assets/musicxml/mandoline - debussy.xml{% endmusicxml %}
```

## Klipse
集成 Klipse (online code evaluator)

[插件地址](https://plugins.gitbook.com/plugin/klipse)  
[Klipse](https://github.com/viebel/klipse)

```
{
    "plugins": ["klipse"]
}
```




