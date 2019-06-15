---
layout:     post                    # 使用的布局（不需要改）
title:      vscode debugger调试代码               # 标题 
subtitle:   vscode调式代码          #副标题
date:       2019-06-15              # 时间
author:     HuangCanCan             # 作者
header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - vscode调式
---

## 前言

最近一直在写vue.js代码，一直想找一款比较好一点的编辑器，用过WebStorm、IDEA、HBuilder、SublimeText3编辑器。刚开始常使用WebStorm，觉得它功能特别强大，但是收费，占用内存有点高。后来我就使用了vscode，它小巧，免费，更少的内存和CPU占用率等等。今天就来简单介绍一下vscode的debug模式的使用。

vscode的debugger环境是分为Chrome和Node的。

## 如何在Chrome环境下，使用vscode debugger进行代码调试？

1. 第一步，我们需要安装插件Debugger For Chrome，因为vscode里面是没有内置调试Chrome的模块的，需要单独安装

![Debugger For Chrome](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/chrome201906151628.PNG')

2. 第二步，打开vscode调试区域，然后点设置

![vscode调试区域](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_163020.png)

3. 然后，选Chrome

![Chrome配置](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_163321.png)

4. 进入了launch.json文件，修改一下配置

    {
        // Use IntelliSense to learn about possible attributes.
        // Hover to view descriptions of existing attributes.
        // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
        "version": "0.2.0",
        "configurations": [
            {
                "type": "chrome",
                "request": "launch",
                "name": "Launch Chrome for index.html",
                "sourceMaps": true,
                "file": "${workspaceFolder}/index.html"
            }
        ]
    }

5. 最后，切换为Launch Chrome for index.html，再点绿色icon或者在index.html目录下直接按F5

![debug运行](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_164522.png)

6. chrome自动启动的结果

![启动结果](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_170804.png)

7. 我们在index.html里面随便写点js，然后重新启动。console.log()在vscode里面显示出来了

![调试结果](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_171755.png)

### 断点调试

我们在vscode里面加断点，进行单步调试。在index.html里面写的js代码不可以打断点，所以把js代码放到js文件中，引入index,js到index.html中

![断点调试1](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_173151.png)

![断点调试2](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_173408.png)

### 加Attach用法

加Attach有什么用？

在平常开发中，是结合live-server(vscode插件)进行开发的，live-server非常强大好用，它会监听你文件改动进行自动刷新浏览器，无须手动去刷新。这里Attach的作用就是去附加在用live-server启动的开发的服务器上，去监听我那里的chrome debugger，通俗来讲，就是让vscode的调试控制台上可以完整得显示出debugger信息。

如：用live-server开了一个http://localhost:5500的服务 ，里面有很多console.log和debugger信息，如果我不加Attach，只能在chrome的F12调出devtools看信息和动态打断点调试。有了Attach，调试js时，我就不用老是盯着浏览器的devtools去看了，也不用去找文件打断点了，直接在vscode里面完成所有调试。

如何配置使用？

首先用live-server打开index.html。vscode里打开index.html页面，按住Ctrl+Shift+p 弹出搜索框，在里面输入live server 找到Open With Live Server 鼠标点击，就能打开http:127.0.01:5500

然后，浏览器自动跳转到http://localhost:5500上了。

接着到launch.json进行配置；

    {
        "version": "0.2.0",
        "configurations": [
            {
                "type": "chrome",
                "request": "attach",
                "name": "Attach to Chrome",
                "port": 9222,
                "webRoot": "${workspaceFolder}"
            }
        ]
    }

对Chrome浏览器启动项进行配置。

![chrome启动项配置](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_225249.png)

现在关掉你现在的chrome，去vscode调试面板上，换成Attach to Chrome，在启动，就可以了

## 调试Node.js

vscode内置Node的调试环境，就不需要安装插件了，首先为node.js的文件，然后在launch.json里添加配置

    {
        "version": "0.2.0",
        "configurations": [
            {
                "type": "node",
                "request": "launch",
                "name": "Launch Program",
                "program": "${workspaceFolder}/node.js"
            }
        ]
    }

配置完成后，在调试面板上，启动选项切换成Launch Program，然后启动就可以断点调试了

>参考来源：
[Debugger For Chrome](https://github.com/Microsoft/vscode-chrome-debug)
[参考视频](https://www.bilibili.com/video/av14868402/)
[手把手教你用Vscode Debugger调试代码](http://shooterblog.site/2018/05/19/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E7%94%A8Vscode%20Debugger%E8%B0%83%E8%AF%95%E4%BB%A3%E7%A0%81/)

## 在VSCode中调试vue.js

第一步：必须安装好 Chrome 和 VS Code。同时请确保自己在 VS Code 中安装了 Debugger for Chrome 插件的最新版本。

第二步：在可以从 VS Code 调试你的 Vue 组件之前，你需要更新 webpack 配置以构建 source map。做了这件事之后，我们的调试器就有机会将一个被压缩的文件中的代码对应回其源文件相应的位置。这会确保你可以在一个应用中调试，即便你的资源已经被 webpack 优化过了也没关系。

如果你使用的是 Vue CLI 2，请设置并更新 config/index.js 内的 devtool 属性：

    devtool: 'source-map',

如果你使用的是 Vue CLI 3，请设置并更新 vue.config.js 内的 devtool 属性：

    module.exports = {
        configureWebpack: {
            devtool: 'source-map'
        }
    }

补充：[webpack] devtool里的7种SourceMap模式

[7种SourceMap模式](https://juejin.im/post/58293502a0bb9f005767ba2f)

开发环境推荐：`cheap-module-eval-source-map`

生产环境推荐：`cheap-module-source-map` （这也是下版本 webpack 使用-d命令启动 debug 模式时的默认选项）

第三步：打开vscode调试区域，来到 Debug 视图，然后点击那个齿轮图标来配置一个 launch.json 的文件，选择 Chrome/Firefox: Launch 环境。然后将生成的 launch.json 的内容替换成为相应的配置：

    {
        "version": "0.2.0",
        "configurations": [
            {
                "type": "firefox",
                "request": "launch",
                "name": "vuejs: firefox",
                "url": "http://localhost:8080",
                "webRoot": "${workspaceFolder}/src",
                "pathMappings": [{ "url": "webpack:///src/", "path": "${webRoot}/" }]
            },
            {
                "type": "chrome",
                "request": "launch",
                "name": "Vuejs Launch Chrome",
                "url": "http://localhost:8080",
                "webRoot": "${workspaceFolder}/src",
                "breakOnLoad": true,
                "sourceMapPathOverrides": {
                    "webpack:///./src/*": "${webRoot}/*"
                }
            }
        ]
    }

设置断点调试：

如：在 src/components/HelloWorld.vue里设置一个断点

![设置断点](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-16_001833.png)

打开终端并使用 Vue CLI 开启这个应用：`npm run dev 或 npm run serve`

来到 Debug 视图，选择 ‘vuejs: chrome/firefox’ 配置，然后按 F5 或点击那个绿色的按钮。

随着一个新的浏览器实例打开 http://localhost:8080 ，触发方法时断点会被命中。

### Vue Devtools

使用非常棒的 Chrome 版本 和 Firefox 版本的 Vue.js devtools。使用 devtools 有很多好处，比如它可以让你能够实时编辑数据属性并立即看到其反映出来的变化。另一个主要的好处是能够为 Vuex 提供时间旅行式的调试体验。

![Vue.js devtools](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-15_235645.png)

>请留意如果页面使用了一个生产环境/压缩后的 Vue.js 构建版本 (例如来自一个 CDN 的标准的链接)，devtools 的审查功能是默认被禁用的，所以 Vue 面板不会出现。如果你切换到一个非压缩版本，你可能需要强制刷新该页面来看到它。

补充：
在chrome浏览器中，Vue Devtools需要到chrome应用商店下载，但是我们访问不到chrome应用商店，需要翻墙。

![Vue Devtools](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-16_002604.png)

翻墙：我用的是赛风，下载好[赛风](https://s3.amazonaws.com/psiphon/web/mjr4-p23r-puwl/zh/download.html)软件(有可能访问不到这个链接)后，打开连接成功后就能访问Google，但是网速有点慢。去chrome应用商店下载谷歌访问助手，安装成功后，以后打开chrome就能访问Google了，不需要打开赛风。

![谷歌访问助手](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-16_003306.png)

### 简单的 debugger 语句

你可以直接在代码中使用原生的 debugger 语句。如果你选择了这种方式，请千万记得当你调试完毕之后把这个语句移除。

![debugger语句](https://github.com/LonelyRookie/LonelyRookie.github.io/blob/master/_posts/img_study/vscode-debug/2019-06-16_000110.png)

>参考来源：
[官网](https://cn.vuejs.org/v2/cookbook/debugging-in-vscode.html)