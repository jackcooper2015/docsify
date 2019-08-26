## 1\. 什么是docsify

docsify 是一个动态生成文档网站的工具。不同于 GitBook、Hexo 的地方是它不会生成将 .md 转成 .html 文件，所有转换工作都是在运行时进行。

这将非常实用，如果只是需要快速的搭建一个小型的文档网站，或者不想因为生成的一堆 .html 文件“污染” commit 记录，只需要创建一个 index.html 就可以开始写文档而且直接部署在 GitHub Pages。


## 2\. 使用`docsify-cli`来开发


#### 2.1.1 安装

docsify需要本地先安装`node`, 如果没有安装node，可在node官网选择对应操作系统下载安装：[https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/)

终端输入`npm i docsify-cli -g`进行全局安装：

```
npm i docsify-cli -g

/usr/local/bin/docsify -> /usr/local/lib/node_modules/docsify-cli/bin/docsify
> fsevents@1.2.4 install /usr/local/lib/node_modules/docsify-cli/node_modules/fsevents
> node install
[fsevents] Success: "/usr/local/lib/node_modules/docsify-cli/node_modules/fsevents/lib/binding/Release/node-v57-darwin-x64/fse.node" already installed
Pass --update-binary to reinstall or --build-from-source to recompile
> docsify@4.8.6 postinstall /usr/local/lib/node_modules/docsify-cli/node_modules/docsify
> opencollective postinstall

                         Thanks for installing docsify 🙏
                 Please consider donating to our open collective
                        to help us maintain this package.

          👉  Donate: https://opencollective.com/docsify/donate

+ docsify-cli@4.3.0
added 456 packages from 206 contributors in 32.827s
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-8e8988a548d24edd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


安装结束后使用`docsify -v`查看是否安装成功：

```
docsify -v

docsify-cli version:
  4.3.0
```

#### 2.2.2 初始化一个项目

首先需要创建一个项目目录：

```
mkdir docsify
```

进入项目目录后，使用`docsify init ./`来初始化一个项目：

```
cd docsify

docsify init ./docs

Initialization succeeded! Please run docsify serve docs
```

```
tree -a

.
├── .nojekyll
├── README.md
└── index.html

```

初始化成功后,`docs`目录会生成如下几个文件：

> 1.  `index.html`入口文件
> 2.  `README.md`会做为主页内容渲染
> 3.  `.nojekyll`用于阻止 GitHub Pages 会忽略掉下划线开头的文件、
> 
> `.nojekyll`文件很重要，如果网站部署到`GitHub Pages`时，一定要注意这个文件。

直接编辑 ./README.md 就能更新网站内容，当然也可以添加其他`.md`文件。

#### 2.2.3 启动本地服务

终端输入`docsify serve docs`来启动服务：

```
docsify serve docs

Serving /Users/dragon/tmp/docsify now.
Listening at http://localhost:3000
```

然后浏览器打开`http://localhost:3000`就能看见效果。

![image.png](https://upload-images.jianshu.io/upload_images/3710706-b6d2b2676d7d13cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


当修改文件保存后， `docsify serve docs`服务会自动实时更新。

## 3\. 关于每个页面和URL路径说明

如果需要创建多个页面，或者需要多级路由的网站，在`docsify`里也能很容易的实现。例如创建一个`guide.md`文件，那么对应的路由就是`/#/guide`。
如果你的目录结构如下：

```
-| ./
  -| README.md
  -| guide.md
  -| zh-cn/
    -| README.md
    -| guide.md
```

那么对应的访问页面将是:

```
./README.md        => http://domain.com
./guide.md         => http://domain.com/guide
./zh-cn/README.md  => http://domain.com/zh-cn/
./zh-cn/guide.md   => http://domain.com/zh-cn/guide
```

## 4\. 侧边栏设置

默认情况下，侧边栏会根据当前文档的标题生成目录。

![image.png](https://upload-images.jianshu.io/upload_images/3710706-73a8339d02cbf6ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 4.1 定制侧边栏

首先需要在`index.html`文件中的`window.$docsify`添加`loadSidebar: true,`选项：

```
<script>
  window.$docsify = {
    loadSidebar: true
  }
</script>
<script src="//unpkg.com/docsify"></script>
```

接着在项目根目录创建`_sidebar.md`文件，内容格式如下：

```
* [home1](home1)
* [home2](home2)
* [bar](bar/)
* [bar/a](bar/a)
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-288300f6c9d2e2fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


***注：***配置了`loadSidebar`后就不会生成默认的侧边栏了。

### 4.2 关于侧边栏`_sidebar.md`文件的说明

*   如果只在根目录有一个`_sidebar.md`文件，那么所有页面都将使用这个一个配置，也就是所有页面的侧边栏都一样。
*   如果一个子目录中有`_sidebar.md`文件，那么这个子目录下的所有页面将使用这个文件的侧边栏。
*   `_sidebar.md`的加载逻辑是从每层目录下获取文件，如果当前目录不存在该文件则回退到上一级目录。例如当前路径为`/zh-cn/more-pages`则从`/zh-cn/_sidebar.md`获取文件，如果不存在则从`/_sidebar.md`获取。

如果子目录有`_sidebar.md`,但你就想使用根目录的`_sidebar.md`，
可在`index.html`文件中的`window.$docsify`添加`alias`字段：

```
<script>
  window.$docsify = {
    loadSidebar: true,
    alias: {
      '/.*/_sidebar.md': '/_sidebar.md'
    }
  }
</script>
```

配置`alias`字段后，所有页面都会显示项目根目录`_sidebar.md`文件的配置作为侧边栏，子目录的`_sidebar.md`文件会失效。

### 4.3 显示页面目录(当前页面的标题)

定制的侧边栏仅显示了页面的链接。
还可以设置在侧边栏显示当前页面的目录(标题)。
需要在`index.html`文件中的`window.$docsify`添加`subMaxLevel`字段来设置：

```
<script>
  window.$docsify = {
    loadSidebar: true,
    subMaxLevel: 3
  }
</script>
<script src="//unpkg.com/docsify"></script>
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-e91aee77b9295235.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


subMaxLevel说明：

> subMaxLevel类型是number(数字)，表示显示的目录层级
> **注意：**如果md文件中的第一个标题是一级标题，那么不会显示在侧边栏，如上图所示

| 值 | 说明 |
| --- | --- |
| 0 | 默认值，表示不显示目录 |
| 1 | 显示一级标题(`h1`) |
| 2 | 显示一、二级标题(`h1` ~ `h2`) |
| 3 | 显示一、二、三级标题(`h1` ~ `h3`) |
| n | n是数字，显示一、二、....n 级标题(`h1` ~ `hn`) |

在md文件中标题的写法：

```
# 这是一级标题，对应HTML中<h1>标签

## 这是二级标题，对应HTML中<h2>标签

### 这是三级标题，对应HTML中<h3>标签

#### 这是四级标题，对应HTML中<h4>标签
```

#### 4.3.1 页面的标题不在侧边栏目录显示

***注意：*** 如果md文件的第一个标题是一级标题，那么默认已经忽略了。

当设置了 subMaxLevel 时，默认情况下每个标题都会自动添加到目录中。如果你想忽略特定的标题，可以给它添加 {docsify-ignore} ：

```
# Getting Started

## Header {docsify-ignore}

该标题不会出现在侧边栏的目录中。
```

要忽略特定页面上的所有标题，你可以在页面的第一个标题上使用 {docsify-ignore-all} :

```
# Getting Started {docsify-ignore-all}

## Header

该页面所有标题都不会出现在侧边栏的目录中。
```

在使用时， {docsify-ignore} 和 {docsify-ignore-all} 都不会在页面上显示。

## 5\. 导航栏配置

docsify默认是没有导航栏的，可以通过配置来显示导航栏。

### 5.1 在`index.html`中定义导航栏

如果导航的链接少，则可以直接在`index.html`文件直接定义导航栏，要注意链接要以`#/`开头：

```
<body>
  <nav>
    <a href="#/">项目</a>
    <a href="#/home1">home1</a>
    <a href="#/bar/a">bar/a</a>
  </nav>
</body>
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-a640a3792b30445a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 5.2 通过配置文件定义导航栏

首先需要在`index.html`文件中的`window.$docsify`添加`loadNavbar: true,`选项：

```
<script>
  window.$docsify = {
    loadNavbar: true
  }
</script>
<script src="//unpkg.com/docsify"></script>
```

接着在项目根目录创建`_navbar.md`文件，内容格式如下：

```
* [home1](home1)
* [home2](home2)
* [bar](bar/)
* [bar/a](bar/a)
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-9cfd7637b1fe1005.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> ***注意：***
> 1.如果使用配置文件来设置导航栏，那么在`index.html`中定义的导航栏只有在定制的首页才会生效，其他页面会被覆盖。
> 2.如果只在根目录有一个`_navbar.md`文件，那么所有页面都将使用这个一个配置，也就是所有页面的导航栏都一样。
> 3.如果一个子目录中有`_navbar.md`文件，那么这个子目录下的所有页面将使用这个文件的导航栏。
> 4.`_navbar.md`的加载逻辑是从每层目录下获取文件，如果当前目录不存在该文件则回退到上一级目录。例如当前路径为`/zh-cn/more-pages`则从`/zh-cn/_navbar.md`获取文件，如果不存在则从`/_navbar.md`获取。

### 5.3 导航栏嵌套

如果导航内容过多，可以写成嵌套的列表，会被渲染成下拉列表的形式：

```
* 根目录
  * [home1](home1)
  * [home2](home2)
  * [guide](guide)

* bar目录
  * [bar](bar/)
  * [a文件](bar/a)
  * [b文件](bar/b)

* foo目录
  * [one](foo/one)
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-1f51715814fe9c17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 6\. 设置封面

docsify默认是没有封面的，默认有个首页`./README.md`。
通过设置`coverpage`参数，可以开启渲染封面的功能。

首先需要在`index.html`文件中的`window.$docsify`添加`coverpage: true`选项：

```
<script>
  window.$docsify = {
    coverpage: true
  }
</script>
<script src="//unpkg.com/docsify"></script>
```

接着在项目根目录创建`_coverpage.md`文件，内容格式如下：

```
![logo](_media/icon.svg)
# 我的文档网站
## 个人文档网站
> 一个神奇的文档网站生成巩固

* Simple and lightweight (~12kb gzipped)
* Multiple themes
* Not build static html files

[GitHub](https://github.com/docsifyjs/docsify/)
[Get Started](#quick-start)
[Get Started](#quick-start)
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-b8f3d01a7beb1329.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> ***注：***一份文档只会在根目录下加载封面，其他页面或者二级目录下都不会加载。

#### 6.1 自定义封面背景

目前的背景是随机生成的渐变色，每次刷新都会显示不同的颜色。
docsify封面支持自定义背景色或者背景图，在`_coverpage.md`文档末尾添加：

```

<!-- 背景图片 -->
![](_media/bg.png)

<!-- 背景色 -->
![color](#2f4253)
```

![image.png](https://upload-images.jianshu.io/upload_images/3710706-226f5589de1f93e3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> ***注意：***
> 1.自定义背景配置一定要在`_coverpage.md`文档末尾。
> 2.背景图片和背景色只能有一个生效.
> 3.背景色一定要是`#2f4253`这种格式的。

#### 6.2 把封面作为首页

配置了封面后，封面和首页是同时出现的，封面在上面，首页在下面。
通过设置`onlyCover`参数，可以让docsify网站首页只显示封面，
原来的首页通过`http://localhost:3000/#/README`访问。

在`index.html`文件中的`window.$docsify`添加`onlyCover: true,`选项：

```
<script>
  window.$docsify = {
    coverpage: true,
    onlyCover: true,
  }
</script>
<script src="./docsify.min.js"></script>
```

> 通过此配置可以把`./README.md`文件独立出来，当成项目真正的README介绍文件。

#### 6.3 多个封面

如果你的文档网站是多语言的，或许你需要设置多个封面。
例如你的文档目录结构如下

```
.
└── docs
    ├── README.md
    ├── guide.md
    ├── _coverpage.md
    └── zh-cn
        ├── README.md
        └── guide.md
        └── _coverpage.md
```

那么你可以在`index.html`文件中的`window.$docsify`这么配置:

```
window.$docsify = {
  coverpage: ['/', '/zh-cn/']
};
```

或者具体指名文件名:

```
window.$docsify = {
  coverpage: {
    '/': 'cover.md',
    '/zh-cn/': 'cover.md'
  }
};
```

## 7\. 网站部署到GitHub Pages

GitHub Pages 支持从三个地方读取文件:

> 1、`master`分支
> 2、`master`分支下的`docs`目录
> 3、`gh-pages`分支

> 1、如果你的文档直接是在项目根目录写的，那么可直接把代码推送到master分支上， `GitHub Pages`里选择`master branch`.
> 2.如果你的文档是在master分支下的`docs/`目录下编写的，那么可直接把代码推送到master分支上，`GitHub Pages`里选择`master branch/docs folder`.

本例子项目是直接在根目录中编写的，所以`GitHub Pages`里选择`master branch`的方式部署。

首先在github网站上创建好仓库，然后终端打开项目目录：

```
git init
git add .
git commit -m 'docsify项目初始化'
git remote add origin https://github.com/username/docsify.git
git push --set-upstream origin master
```

代码推送到github上后，打开github的仓库，选择`Settings` -> `GitHub Pages` -> `master branch` -> `save`。

![image.png](https://upload-images.jianshu.io/upload_images/3710706-af20ebe3bf84310a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/3710706-65ad6d7087499bbd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 7.1 使用docsify的例子

[https://spiritree.github.io/n...](https://spiritree.github.io/note/#/)

[https://ripperhe.com/awesome-...](https://ripperhe.com/awesome-mac-html/#/)

## 8\. 一些插件

### 8.1 搜索插件

全文搜索插件会根据当前页面上的超链接获取文档内容，在 localStorage 内建立文档索引。默认过期时间为一天，当然我们可以自己指定需要缓存的文件列表或者配置过期时间。

```
<script>
    window.$docsify = {
      // 完整配置参数
      search: {
        maxAge: 86400000,               // 过期时间，单位毫秒，默认一天
        paths: [],                      // or 'auto'，匹配文件路径
        placeholder: 'Type to search',  // 搜索提示框文字， 支持本地化，例子在下面
        // placeholder: {
        //   '/zh-cn/': '搜索',
        //   '/': 'Type to search'
        // },
        noData: 'No Results!',          // 找不到结果文字提示，支持本地化，例子在下面
        // noData: {
        //   '/zh-cn/': '找不到结果',
        //   '/': 'No Results'
        // },
        depth: 2,                       // 搜索标题的最大程级, 1 - 6
      }
    }
  </script>
  <!-- 引入搜索模块 -->
  <script src="//unpkg.com/docsify/lib/plugins/search.js"></script>
```

### 8.2 评论插件Gitalk

Gitalk：一个现代化的，基于Preact和Github Issue的评论系统。
使用例子：

```
<link rel="stylesheet" href="//unpkg.com/gitalk/dist/gitalk.css">
<script src="//unpkg.com/docsify/lib/plugins/gitalk.min.js"></script>
<script src="//unpkg.com/gitalk/dist/gitalk.min.js"></script>
<script>
  const gitalk = new Gitalk({
    clientID: 'Github Application Client ID',
    clientSecret: 'Github Application Client Secret',
    repo: 'Github repo',
    owner: 'Github repo owner',
    admin: ['Github repo collaborators, only these guys can initialize github issues'],
    // facebook-like distraction free mode
    distractionFreeMode: false
  })
</script>
```

Gitalk具体使用教程：[https://segmentfault.com/a/11...](https://segmentfault.com/a/1190000018072952)

docsify其他插件：[https://docsify.js.org/#/zh-c...](https://docsify.js.org/#/zh-cn/plugins)

### 8.3 样式插件

在网上找到一个样式：[https://jhildenbiddle.github....](https://jhildenbiddle.github.io/docsify-themeable/#/customization)

使用方法,在HTML文件中引入：

```
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/docsify-themeable@0/dist/css/theme-simple.css">
```
---
---
## 参考资料

* docsify中文官网：[https://docsify.js.org/#/zh-cn/](https://docsify.js.org/#/zh-cn/)

* [https://segmentfault.com/a/1190000017576714](https://segmentfault.com/a/1190000017576714)


