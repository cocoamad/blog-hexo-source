title: mac下使用hexo搭建blog
date: 2014-02-27 14:56:47
tags:
---

## mac下使用hexo搭建blog ##
----------
**安装nvm（Node Version Manager）**，Terminal中运行

    $ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
会提示：

    => Close and reopen your terminal to start using NVM
退出Terminal重启后nvm命令才能生效。  使用nvm安装node.js：

    $ nvm install 0.10
下完后安装hexo，这一步时间比较长：

    $ npm install -g hexo

然后找个文件初始化blog：

    $ cd ~/git/blog  
    $ hexo init .
    $ ls
生成出的目录结构：

    .
    ├── _config.yml
    ├── package.json
    ├── scaffolds
    ├── scripts
    ├── source
    |   ├── _drafts
    |   └── _posts
    └── themes

新建一篇文章：

    $ hexo new mac下使用hexo搭建blog
$ open source/_posts/mac下使用hexo搭建blog.md 
编辑md后生成html：

    $ hexo generate
本地预览：

    $ hexo server
    => [info] Hexo is running at localhost:4000/blog/. Press Ctrl+C to stop.
Theme，去官方提供的[主题列表][1]中选个现成的，按照里面的方法pull下来，如light主题


  [1]: https://github.com/tommy351/hexo/wiki/Themes
  

    $ git clone git://github.com/tommy351/hexo-theme-light.git themes/light
    
_config.yml配置文件中设置：

    theme: light
重新generate和server预览，就看到变化了。

deploy：
-------

github上建个respository，设置里设一下
在`_config.yml`中：

    deploy:
    type: github
    repository: https://github.com/sunnyxx/blog-hexo.git
然后执行：

    $ hexo deploy
就行了，github会多一个branch，比octopress简单