`常用指令ls　 显示文件或目录 -l 列出文件详细信息l(list) -a 列出当前目录下所有文件及目录，包括隐藏的a(all)mkdir 创建目录 -p 创建目录，若无父目录，则创建p(parent)cd 切换目录touch 创建空文件echo 创建带有内容的文件。cat 查看文件内容cp 拷贝mv 移动或重命名rm 删除文件 -r 递归删除，可删除子目录及文件 -f 强制删除find 在文件系统中搜索某文件wc 统计文本中行数、字数、字符数grep 在文本文件中查找某个字符串rmdir 删除空目录tree 树形结构显示目录，需要安装tree包pwd 显示当前目录`

git速度快，分布式，

- 回到过去，未来，版本
- 使用git会在当前目录下，产生一个.git文件，记录
- 多端共享
- 团队协作


- 冲突需要手动解决


- svn和git对比


- 每个文件夹里面都有.svn文件，速度慢


- svn叫集中式，集中存放，有一个中央服务器，如果中央服务器报废，所有的文件将瘫痪
- git叫分布式，每个人都有本地仓库，（也有中央服务器github,可有可无，github/gitlab）
- git 速度快

git界面化管理（sourceTree）/命令行

- 常见编辑器 webstrom sublime vscode

查看git 版本号

`git --version`

清屏

`clear`

创建目录

`mkdir`

创建文件

`touch`

使用git

- 查看 git配置是否 git config --list
- 配置git用户

`git config --global user.name 名字`` git config --global user.email 邮箱`

初始化git仓库

`git init`

删除文件夹（删除.git文件夹）

`rm -rf .git`

创建文件

`touch index.txt`

查看文件

`cat index.txt`

vi 编辑

`vi index.txti 表示插入编辑esc+:wq保存并退出`

列出了(修改过的、新创建的、已经暂存但未提交的)文件，

- git status

git 三个区

- 工作区

`git add .``- . 点代表所有`

- 暂存区/过渡区

`git commit -m '解释文件信息'`

- 历史去/版本库`git log`


- 每次提交都会产生一个版本号
- 查看日志git log


- 提交github

git log 查看日志

git比较三个区不同

`git dirr`

- 直接写git dirr 比较的是工作区和暂缓区
- git diff --cached 比较的是暂缓区和历史区
- git diff (分支名)
- git diff master 工作区和版本库

输出内容到文件中

`echo world >> index.txt`

根据字段搜索日志

`git log --author='搜索的关键字'`

回滚工作区

- 用暂存区的覆盖掉工作区

`git checkout '文件'`

将暂存区的内容回到上一次的暂存区

`git reset HEAD .`

回滚历史区

`git reset --hard 版本id`

- 强制用--历史区--覆盖到--工作区--和--缓存区

打印以前所有的日志

`git reflog`

打印日志合并提示

`git log --graph`

查看分支

`git branch`

git分支管理

- 只有提交过一次，才会产生master分支，否做分支是空的
- 新建一个(dev)分支

`git branch dev`

- 切换到（dev）分支上

`git checkout dev`

- 删除(dev)分支


- 切换到别的分支，删除要删除的分支
- 在要删除的分支上，没有办法删除要删除的分支

`git branch -D dev`

- 创建(dev)分支并切换到（dev）分支上

`git checkout -b dev`

- 在工作区中创建一个文件，这个文件不属于任何分支

- 只有提交过一次，此时两个分支才无任何关系

- 合并分支

  `git merge '要合并的分支名'`


- 切换到master


- 解决冲突
- 删除所有没用的，留下需要的，再次提交

简写，提交

`git commit -a -m 'merge'`

推送到github

- 先在github上创建一个远程仓库
- 初始化github
- 添加readme文件
- 添加忽视文件.gitignore这个文件上传
- 空文件夹不会被提交（想提交并且还是空的的）添加.gitkeep
- 添加暂存区 添加历史区 添加一个远程地址 提交github上

** 