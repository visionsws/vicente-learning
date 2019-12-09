# Github+docsify打造个人文档

### docsify简介

docsify即时生成您的文档网站。与GitBook不同，它不会生成静态html文件。相反，它巧妙地加载和解析您的Markdown文件并将其显示为网站。您需要做的就是创建一个index.html以在GitHub页面上启动和部署它。 

### 快速开始

首先先安装好npm和nodejs,这里就不做过多介绍了 自信安装即可 （[https://blog.csdn.net/zimushuang/article/details/79715679](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fzimushuang%2Farticle%2Fdetails%2F79715679)）

安装docsify 推荐安装 docsify-cli 工具，可以方便创建及本地预览文档网站。

```undefined
npm i docsify-cli -g
```

初始化项目

```bash
# 进入指定文件目录 如下：F:\ideWork\githubWork\test_docs 
运行    docsify init ./docs
```


  初始化成功后，可以看到 ./docs 目录下创建的几个文件

```css
index.html 入口文件
README.md 会做为主页内容渲染
.nojekyll 用于阻止 GitHub Pages 会忽略掉下划线开头的文件
```


  本地预览网站

运行一个本地服务器通过 docsify serve 可以方便的预览效果，而且提供 LiveReload 功能，可以让实时的预览。默认访问[http://localhost:3000/#/](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A3000%2F%23%2F)。

```undefined
docsify serve docs
```

一个基本的文档网站就搭建好了，docsify还可以自定义导航栏，自定义侧边栏以及背景图和一些开发插件等等
更多配置请参考官方文档 [https://docsify.js.org/#/zh-cn/quickstart](https://links.jianshu.com/go?to=https%3A%2F%2Fdocsify.js.org%2F%23%2Fzh-cn%2Fquickstart)

### 多页文档

如果需要创建多个页面，或者需要多级路由的网站，在 docsify 里也能很容易的实现。例如创建一个 `guide.md` 文件，那么对应的路由就是 `/#/guide`。

假设你的目录结构如下：

```text
-| docs/
  -| README.md
  -| guide.md
  -| zh-cn/
    -| README.md
    -| guide.md
```

那么对应的访问页面将是

```text
docs/README.md        => http://domain.com
docs/guide.md         => http://domain.com/guide
docs/zh-cn/README.md  => http://domain.com/zh-cn/
docs/zh-cn/guide.md   => http://domain.com/zh-cn/guide
```

#### 定制侧边栏

为了获得侧边栏，您需要创建自己的_sidebar.md，你也可以自定义加载的文件名。默认情况下侧边栏会通过 Markdown 文件自动生成，效果如当前的文档的侧边栏。

首先配置 `loadSidebar` 选项，具体配置规则见[配置项#loadSidebar](https://docsify.js.org/#/zh-cn/configuration?id=loadsidebar)。

```html
<script>
  window.$docsify = {
    loadSidebar: true
  }
</script>
<script src="//unpkg.com/docsify"></script>
```

接着创建 `_sidebar.md` 文件，内容如下

```markdown
* [首页](zh-cn/)
* [指南](zh-cn/guide)
```

需要在文档根目录创建 `.nojekyll` 命名的空文件，阻止 GitHub Pages 忽略命名是下划线开头的文件。

`_sidebar.md` 的加载逻辑是从每层目录下获取文件，如果当前目录不存在该文件则回退到上一级目录。例如当前路径为 `/zh-cn/more-pages` 则从 `/zh-cn/_sidebar.md` 获取文件，如果不存在则从 `/_sidebar.md` 获取。

当然你也可以配置 `alias` 避免不必要的回退过程。

```html
<script>
  window.$docsify = {
    loadSidebar: true,
    alias: {
      '/.*/_sidebar.md': '/_sidebar.md'
    }
  }
</script>
```

#### 显示目录

自定义侧边栏同时也可以开启目录功能。设置 `subMaxLevel` 配置项

```html
<script>
  window.$docsify = {
    loadSidebar: true,
    subMaxLevel: 2
  }
</script>
<script src="//unpkg.com/docsify"></script>
```

#### 忽略副标题

当设置了 `subMaxLevel` 时，默认情况下每个标题都会自动添加到目录中。如果你想忽略特定的标题，可以给它添加`{docsify-ignore}` 。

```markdown
# Getting Started

## Header {docsify-ignore}

该标题不会出现在侧边栏的目录中。
```

要忽略特定页面上的所有标题，你可以在页面的第一个标题上使用 `{docsify-ignore-all}` 。

```markdown
# Getting Started {docsify-ignore-all}

## Header

该标题不会出现在侧边栏的目录中。
```

在使用时， `{docsify-ignore}` 和 `{docsify-ignore-all}` 都不会在页面上呈现。



### index.html例子

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
  <meta name="description" content="Description">
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
  <link rel="stylesheet" href="//unpkg.com/docsify/lib/themes/vue.css">
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      //定义路由别名，可以更自由的定义路由规则。 支持正则。
      alias: {
            '/.*/_sidebar.md': '/_sidebar.md'
      },
      // 加载 _sidebar.md,加载自定义侧边栏
      loadSidebar: true,
      //加载自定义导航栏
      loadNavbar: false,
      // 强制悬停,切换页面后是否自动跳转到页面顶部。
      auto2top: true,
      // 调整副标题的级数
      subMaxLevel: 2,
      // 替换主题色
      themeColor: '#4CAF50',
      themeable: {
          readyTransition : true, // default
          responsiveTables: true  // default
      },

      showLevel: true,
      //文档标题，会显示在侧边栏顶部。
      name: '',
      //配置仓库地址或者 username/repo 的字符串，会在页面右上角渲染一个 GitHub Corner 挂件。
      repo: '',
      //禁用 emoji 解析
      noEmoji: true,
      tocLevel: 6,
      //搜索配置项
      search: {
        maxAge: 86400000, // 过期时间，单位毫秒，默认一天
        paths: 'auto', // or 'auto'
        placeholder: '搜索',
        noData: '找不到结果',
        // 搜索标题的最大程级, 1 - 6
        depth: 4
      },
      pagination: {
        previousText: '上一章节',
        nextText: '下一章节',
      }

    }
  </script>

  <script src="//unpkg.com/docsify/lib/docsify.min.js"></script>
  <script src="//unpkg.com/docsify/lib/plugins/search.min.js"></script>
  <script src="//unpkg.com/docsify-pagination/dist/docsify-pagination.min.js"></script>
  <script src="//unpkg.com/docsify-copy-code"></script>
  <script src="//unpkg.com/prismjs/components/prism-sql.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-http.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-json.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-bash.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-java.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-markdown.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-nginx.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-php.min.js"></script>
  <script src="//unpkg.com/prismjs/components/prism-python.min.js"></script>
  <script src="//unpkg.com/docsify-pagination/dist/docsify-pagination.min.js"></script>

  <!-- docsify-themeable -->
  <script src="https://unpkg.com/docsify-themeable"></script>
</body>
</html>

```



### 下面介绍docsify如何部署到Github 使用免费的站点

和 GitBook 生成的文档一样，我们可以直接把文档网站部署到 GitHub Pages 或者 VPS 上

## GitHub Pages

GitHub Pages 支持从三个地方读取文件

- docs/ 目录
- master 分支
- gh-pages 分支

上传文件至Github仓库 官方推荐直接将文档放在 docs/ 目录下，在设置页面开启 GitHub Pages 功能并选择 master branch /docs folder 选项。



![img](https://upload-images.jianshu.io/upload_images/5334889-bfafbe2c24387a28?imageMogr2/auto-orient/strip|imageView2/2/w/778/format/webp)

在这里插入图片描述

此时等待几秒钟 就可以访问了 