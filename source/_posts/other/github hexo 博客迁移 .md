title: github hexo博客多电脑迁移
date: 

categories: 
- other

tags: 
- github
- hexo

---

**利用github的不同分支来分别保存网站静态文件与hexo源码（md原始文件及主题等）**，实现在不同电脑上都可以自由写博客。

**其核心就是在博客挂载的Github仓库中新建一个branch分支hexo，其中默认的master进行博文的deploy工作，hexo保存博客的源文件以及环境配置和主题配置文件（设置hexo为默认主分支），这样我们每次重新配置系统只需要从Github中clone一下，然后再进行几步简单环境安装配置操作就完成了。**

本文是建立在你已有hexo博客的情况下，教你如何把它迁移到另外一台电脑上，使两台电脑同时可以进行博客部署。下面就具体讲解一下如何实现吧！

*注：假设你有工作PC和个人PC，你原有的博客搭建在个人PC上，以下首先介绍个人PC上的操作步骤。*

#### 基于master和hexo两个分支

其中master分支是默认建立好的，我们在`./_config.yml`文件中配置deploy布置信息，设置布置生成的静态文件的分支为`master`，使用`hexo deploy`命令会自动的将这些静态文件push到Github中去；
然后新建一个分支hexo，保存我们所有的源文件，这里需要手动`git push`，另外，由于hexo中`.gitignore`文件设置了忽略，所以`./theme`文件中的主题配置信息不会被push到Github中，所以这里新建一个`setting`文件夹保存`./theme`中的主题配置文件，进行备份更新。
将hexo分支设置为默认主分支，我们之后只需要通过hexo分支进行博文的更新即可，hexo分支保存博文源文件（Markdown文件），通过`hexo g -d`命令会自动将生成的静态文件push到master分支上去。



## 个人PC

### 在github上新建远程仓库

将原来的page项目删除，新建一个和原来名字一样的空项目。不用初始README.md，此时只有一个空的master分支。

本地的目录不要动，到时候需要用你原来博客的配置和文章替换该项目中的文件夹。你可以把你本地的博客目录复制一份作为备份，以免误操作将原来的内容进行改动。

### 本地初始化一个Hexo项目

新建一个空目录，作为你的博客目录。进入该目录，右击`Git bash here`，初始化一个Hexo项目：

```
hexo init
npm install
npm install hexo-deployer-git --save
```

然后用自己原来博客里的文件替换掉这里的`source\`, `scaffolds\`, `themes\`, `_config.yml`替换成自己原来博客里的。另外，由于hexo中`.gitignore`文件设置了忽略，所以`./theme`文件中的主题配置信息不会被push到Github中，所以这里新建一个`setting`文件夹保存`./theme`中的主题配置文件，进行备份更新。

### 将整个目录推送到master

要推送到master分支，首先要将该目录初始化为本地Git仓库：

```
git init
 
//把博客目录下所有文件推送到master分支
git remote add origin git@github:chown-jane-y/chown-jane-y.github.io.git
git add .
git commit -m "first add hexo source code"
git push origin master
```

### 在github上新建一个分支

新建一个分支hexo(名字可以自定义)，这时候hexo分支和master分支的内容一样，都是hexo的源文件。

并把hexo设为默认分支，这样的话在另外一台机器上克隆下来就直接进入hexo分支，并且以后所有操作都是在hexo分支下完成。

```
git checkout -b hexo
git push origin hexo
```

为什么需要这个额外的分支呢？

因为`hexo d`只把静态网页文件部署到master分支上，所以你换了另外一台电脑，就无法pull下来继续写博客了。有了hexo分支的话，就可以把hexo分支中的源文件(配置文件、主题样式等)pull下来，再`hexo g`的话就可以生成一模一样的静态文件了。

### 部署博客

还是和以前一样：

```
hexo g -d
```

博客已经成功部署到master分支，这时候到github查看两个分支的内容，hexo分支里是源文件，master里是静态文件。

**注意：根目录下的_config.yml配置文件中branch一定要填master，否则hexo d就会部署到hexo分支下。**


## 工作PC

个人PC上的工作已经完成了，下面讲一下如果你换到了另外一台电脑上，应该如何操作。

### 将博客项目克隆下来

```
git clone git@github.com:Ernestchang/ernestchang.github.io.git
```
git clone复制repo源文件，其中由于`.gitignore`文件，hexo目录中的theme主题文件不会被push到Github中，所以我采取的方法是新建一个`setting`文件夹，将theme文件复制到`setting`文件中进行备份保存。

### 更新theme文件配置

所以这里git clone之后要进行theme主题配置的更新，执行如下命令：

```
cp -rf ./setting/next ./themes
```

主题与文件夹的名字大家按自己定义的就好。

克隆下来的仓库和你在个人PC中的目录是一模一样的，所以可以在这基础上继续写博客了。但是由于`.gitignore`文件中过滤了`node_modules\`，所以克隆下来的目录里没有`node_modules\`，这是hexo所需要的组件，所以要在该目录中重新安装hexo，千万不要执行**`hexo init`**，这样会重置hexo的配置文件，配好的文件就丢失啦！

```
npm install hexo
npm install
npm install hexo-deployer-git --save
```

### 新建一篇文章测试

```
hexo new "work PC test"
```

推送到hexo分支

```
git add .
git commit -m "add work PC test"
git push origin hexo
```

部署到master分支

```
hexo g -d
```

## 日常操作

如果上面的过程都操作无误的话，你就可以在任何能联网的电脑上写博客啦。一般写博客的流程是下面这样。

### 写博客前

不管你本地的仓库是否是最新的，都先pull一下，以防万一：

```
git pull origin hexo
```

把最新的pull下来，再开始撰写新的博客。

### 写博客

```
hexo new "title"
```

然后打开`source/_posts/title.md`，撰写博文。

### 写完博客

先推送到hexo分支上：

```
git add .
git commit -m "add article xxx"
git push origin hexo
```

最后部署到master分支上：

```
hexo g -d
```

整个流程大概就是这样。


这时就可以测试一下，使用`hexo server`、`hexo g -d`等命令测试一下同步的情况，然后不同的机器可以使用`git pull`进行数据的同步，每次更新博文就使用`git push origin hexo`更新到hexo分支上去即可！
