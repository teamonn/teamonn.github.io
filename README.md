# teamonn.github.io

## 1. 简介

用于存放自己`学习文章`和展示`个人小项目`的个人主页。

## 2. 技术实现

基于 hexo 和 yilia-plus 主题搭建的 github pages 个人主页。

## 3. 管理维护
### 启动

``` js
npm install // clone 该项目后先安装依赖包

hexo server // 本地运行项目
```

### 发布

本来 hexo 推荐的发布方式是本地执行：`hexo clean && hexo deploy` 来发布到远程。

但是我做了[一键部署](https://hexo.io/zh-cn/docs/github-pages)，可以通过提交或者合并代码到 `main` 分支，然后自动触发 `gh-pages` 分支构建来进行项目代码发布。

这种只需要提交代码到 main 分支就能够发布文章的方式更容易理解，也更符合 CI/CD 自动化构建的思路。

### 写文章

可参考 hexo 文档：https://hexo.io/zh-cn/docs/writing。

当然，你可以选择更简单易懂的方式：在 `source` 目录下的 `_posts` 文件下新建 md 格式文件（可复制之前的改）。

改完项目代码或者写了新文章，记得 `hexo generate` 重新生成下页面。

### 更换主题

目前项目使用的是 `yilia-plus` 主题，可以到 [Themes \| Hexo](https://hexo.io/themes/) 去挑选适合自己的主题。

更换主题方法：
> 在 themes 文件夹内，新增一个任意名称的文件夹，并修改 _config.yml 内的 theme 设定，即可切换主题。

具体可参考：https://hexo.io/zh-cn/docs/themes
