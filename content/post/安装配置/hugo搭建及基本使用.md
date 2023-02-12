---
title: "Hugo搭建及基本使用"
date: 2023-02-05T10:57:37+08:00
draft: true
tags: ["hugo"]
categories: ["安装配置"]
---

记录hugo在win11上的搭建和主题自定义。所有操作使用git bash。

## 1. 环境搭建
### 1.1 git
[git](https://git-scm.com/downloads)，global配置username和email
```bash
// (username是自己的账户名)
git config --global user.name "username"   

// (useremail注册账号时用的邮箱)
git config --global user.email "useremail"     

// 查看配置
git config --global --list

// 生成ssh
ssh-keygen -t rsa
```

将C:\Users\usrname\\.ssh\id_rsa.pub中内容复制到github中setting->SSH and GPG keys->New SSH key->key栏->Add SSH key。

### 1.2 hugo
[hugo](https://github.com/gohugoio/hugo/releases)，使用当前最新0.110.0版本，需要下载extend版本，我选的是hugo_extended_0.110.0_windows-amd64.zip。否则下载的主题无法渲染，提示```is not compatible with this Hugo version; run "hugo mod graph" for more info```。

压缩包解压缩后将hugo.exe文件添加到系统环境变量中，cmd中```hugo version```可查看版本说明添加成功。

在想要写博客的位置创建04_hugo_blog文件夹生成站点，运行后会在04_hugo_blog生成相应的文件结构。文件结构如下，content中放自己记录的blog，themes中放下载的主题，config.toml为配置文件，static放blog中用到的图片（blog插入图片时，直接放，不需要写路径，```![hugo目录结构](hugo_tree.bmp)```）。

![hugo目录结构](hugo_tree.bmp)

基本操作如下：
```bash
// 创建hugo站点：04_hugo_blog
hugo new site 04_hugo_blog

// 根文件04_hugo_blog下创建blog文件，自动创建到content中
hugo new post/安装配置/Hugo搭建及基本使用.md
```

创建文件后会生成文件头信息，可以添加相应的多tags和categories，如下。

```
---
title: "Hugo搭建及基本使用"
date: 2023-02-05T10:57:37+08:00
draft: true
tags: ["hugo","配置"]
categories: ["安装配置"]
---
```

### 1.3 主题下载
[主题下载](https://themes.gohugo.io/)，需要下载你想要的主题到04_hugo_blog/themes/文件夹下，我选用的主题是[hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack)，建议下载有搜索功能，分类的，方便修改。
```
// 下载主题到themes
git clone https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack

// 根文件04_hugo_blog下预览博客
hugo server --theme=hugo-theme-stack --buildDrafts
```

## 2. 主题配置
1. 借鉴04_hugo_blog/theme/hugo-theme-stack/exampleSite修改主题。
![content内容](hugo_content.bmp)
2. 主题中代码块相关的配置添加到04_hugo_blog/config.toml，预览可简化为```hugo server --buildDrafts```
```
theme = "hugo-theme-stack"

[markup]
  [markup.highlight]
    codeFences = true   #代码围栏功能
    guessSyntax = true  #猜测语法，没有设置的话true会自动匹配语言
    hl_Lines = ""       #高亮行号，一般不设置
    lineNoStart = 1     #行号从1开始
    lineNos = true      #是否显示行号
    lineNumbersInTable = true  #使用表格式化行号和代码，一般置true
    noClasses = true    
    style = "github"    #主题风格
    tabWidth = 4
```

## 3. 部署到github
hugo是将编译后的文件上传到github上，并不需要上传源文件，为备份需要将源文件和编译后的均上传。

### 3.1 html部署
github上新建仓库，命名需要注意usrname.github.io

```
//指定baseUrl仓库地址，生成04_hugo_blog/public文件夹，将blog转成html文件
hugo --baseUrl="https://frozen531.github.io/" --buildDrafts

//进入public，创建git仓库
git init

//添加所有文件
git add .

//添加提交信息
git commit -m "博客第一次提交"

//关联远端仓库
git remote add origin https://github.com/frozen531/frozen531.github.io.git

//推到远端仓库
git push -u origin master
```

访问远端博客地址：[https://frozen531.github.io/](https://frozen531.github.io/)


### 3.2 源文件备份
源文件备份到github，创建新的仓库。04_hugo_blog文件夹下```git init```，因为.git仓库不能嵌套，themes主题中的.git需要删除，pulbic中的.git可以通过创建.gitignore文件忽略。


## 4. 参考文章
1. [Git的环境配置（超详细）](https://blog.csdn.net/shuang_waiwai/article/details/121108964)
2. [Windows安装Hugo](http://www.manongjc.com/detail/60-ldtkuvvzfmljerl.html)
3. [使用Hugo和stack 主题建立建静态网站](https://zhongyang.wang/post/1662875174/)
4. [关于Hugo：Hugo-页面包中的相对路径](https://www.codenong.com/53464336/)
5. [置Hugo的代码高亮](https://www.cnblogs.com/brady-wang/p/13830095.html)
6. [手把手教你从0开始搭建自己的个人博客 |第二种姿势 | hugo](https://www.bilibili.com/video/BV1q4411i7gL/?spm_id_from=333.1007.top_right_bar_window_history.content.click&vd_source=b6daecdfe358d06b1107a6d13e19fe3f)
7. [Hexo博客的备份与恢复](https://www.dandelioncloud.cn/article/details/1586880018842374146)
