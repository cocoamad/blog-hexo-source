title: hexo的私人订制
date: 2014-03-07 16:26:34
tags: hexo
---

#准备工作
##Fork it!
从0开始多费劲，先从hexo的主题中选一个看的过去的，从上面加工。
这次选的是hexo的默认主题`Landscape`，觉得一个大banner挺好看而已。
主题在github上，https://github.com/hexojs/hexo-theme-landscape  
废话不说，先fork一份，虽然不会再merge回去了。
fork完去setting页面改个名字，就叫它`present`了，因为当时看到群里正说`presentViewController`的事- -
![][1]
 * hexo工程的`themes/`目录默认是在`.gitignore`里的，意思是主题和内容是应该分开的
theme作为主项目的`submodule`，所以主题更改时也应该单独提交了

##Clone it！
把刚fork的名为`present`的theme安装到hexo目录：
``` sh
$ git clone https://github.com/sunnyxx/present themes/present
```
去`/_config.yml`中找到并设置：
```
# Extensions
## Plugins: https://github.com/tommy351/hexo/wiki/Plugins
## Themes: https://github.com/tommy351/hexo/wiki/Themes
theme: present // 修改这儿
```
运行下`hexo server`就能立刻看到效果了

#开始定制theme
先得看看hexo theme里面的结构：

 - _config.yml - 主题总体配置
 - /layout/*.ejs - 网页布局
 - /source/css/*.styl - 网页样式

##定制banner
![banner][2]
默认的banner图是个地球星空图，先从它下手，这张图位于`/themes/present/source/css/images/banner.jpg`
这图分辨率有`1920x1200`之大，显示的部分很少，搞一张喜欢的banner图，PS成大概的尺寸（高度还就得设的很大才行，虽然只显示一小部分，否则会出现显示不出来图片的状况），我这儿PS过的是一张png，名为`banner.png`，文件名修改需找找到位于`/themes/present/source/css/_variables.styl`中，修改为`banner.png`：
```
// Header
logo-size = 40px
subtitle-size = 16px
banner-height = 404px // 看看多高合适
banner-url = "images/banner.png" // 修改这里
```
然后就变成这鸟样了：
![专业多了][3]
这个标题横在这儿太恶心了，我的banner里面已经有标题了，这就是需要修改布局了，header的布局在`/themes/present/layout/_partial/header.ejs`，打开修改：
```
// 把这一段都注释掉好了：
<div id="header-title" class="inner">
    <h1 id="logo-wrap">
        <a href="<%- config.root %>" id="logo"><%= config.title %></a>
    </h1>
    <% if (theme.subtitle){ %>
        <h2 id="subtitle-wrap">
          <a href="<%- config.root %>" id="subtitle"><%= theme.subtitle %></a>
        </h2>
    <% } %>
</div>
```
然后世界变清净了。
![丑][4]
默认主题里面banner上下都有个渐变，换了图之后就尤其丑，干掉之。
这个是样式的修改，所以肯定在`present/source/css/_partial`里面了，再看这位置明显是header嘛，所以`header.styl`就是你了：
```
#header
  height: banner-height
  position: relative
  border-bottom: 1px solid color-border
  &:before, &:after
    content: ""
    position: absolute
    left: 0
    right: 0
    height: 40px
  &:before
    top: 0
    background: linear-gradient(rgba(0, 0, 0, 0.2), transparent) // 找到你了，注释掉！
  &:after
    bottom: 0
    background: linear-gradient(transparent, rgba(0, 0, 0, 0.2)) // 也找到你了，注释掉！
```
瞬间清爽很多，但又发现去掉之后字看不清了：
![看不清！][5]
还是在这个文件：

```
$nav-link
  float: left
  color: #000 // 改个深色
  opacity: 1.0 // 别半透明的
  text-decoration: none
  /*text-shadow: 0 1px rgba(0, 0, 0, 0.2)*/ // 改个阴影
  transition: opacity 0.2s
  display: block
  padding: 20px 15px
  &:hover
    opacity: 1
```
much better
![better][6]

##定制文章样式

列几个常用的：

在`/themes/present/source/css/_variables.styl`中：

####调整主区域布局（很值得修改）
默认的主题的主区域太窄了，没几个字就得换行，下面的`main-column`控制主区域宽，`sidebar-column`控制sidebar宽，这两个值加一起凑成全部宽度，会居中对齐。
```
// Layout
block-margin = 20px
article-padding = 20px // 文章内缩进
mobile-nav-width = 280px
main-column = 12 // 主文章区域的宽度
sidebar-column = 3 // 侧边栏区域的宽度
```
####修改代码字体

```
font-mono = "Source Code Pro", Menlo/*Menlo必须提前面啊*/, Monaco, Consolas, Consolas, monospace
```
####修改正文字体和行高
```
font-size = 15px // Menlo字体我看15px的很清楚
line-height = 1.6em
line-height-title = 1.3em
```


-----
位于`/themes/present/source/css/_partial/article.styl`的样式文件负责文章里面的样式
####修改图片格式
```
  img, video
    max-width: 100%
    height: auto
    display: block
    margin: 10 10 10 10 // 默认丫居中的，改成左对齐好了
```
####修改blockquote样式
```
  blockquote
    font-family: font-serif
    font-size: 2.0em // 搞大点
    margin: line-height 20px
    text-align: left // 必须应该左对齐啊
```

-----
位于`/themes/present/source/css/_extend.styl`的样式文件定义了基本样式
####修改标题样式
```
  h1
    font-size: 2em
  h2
    font-size: 1.5em
  h3
    font-size: 1.3em
  h4
    font-size: 1.2em
  h5
    font-size: 1em
  h6
    font-size: 1em
    color: color-grey
```

####修改文章背景
```
$block
  background: #fbfbfb // 白里透着灰
  /*box-shadow: 1px 2px 3px #eee*/ // 扁平化咋能要阴影
  border: 1.5px solid #ccc // 边框
  border-radius: 10px // 圆角矩形走起
```

#开始定制widget
// TODO:
##添加多说评论
// TODO:
##添加友情链接
// TODO:

  [1]: http://ww3.sinaimg.cn/large/51530583gw1ee7835uauoj20j804mwen.jpg
  [2]: http://ww4.sinaimg.cn/large/51530583gw1ee78mkdkspj20jp06bq3h.jpg
  [3]: http://ww2.sinaimg.cn/large/51530583gw1ee790pxkk9j20fv085t9c.jpg
  [4]: http://ww3.sinaimg.cn/large/51530583gw1ee79b2xop4j201u0bt0sm.jpg
  [5]: http://ww4.sinaimg.cn/large/51530583gw1ee79ex02pgj206901u0qo.jpg
  [6]: http://ww3.sinaimg.cn/large/51530583tw1ee79mmlwkaj205401iwe9.jpg