title: SSH创建与查看
date: 

categories: 
- linux
tags:  
- ssh
---

### 1. 查看本机 `.ssh`

    $ cd ~/.ssh    (如果有则进入)
    $ ls     (查看当前所在文件的目录)

这两个命令就是检查是否已经存在 id_rsa.pub 或 id_dsa.pub 文件，如果文件已经存在，那么你可以跳过步骤2，直接进入步骤3。



### 2. 创建一个 `.ssh` key

- #### 生成秘钥

`$ ssh-keygen -t rsa -C "your_email@example.com"`

> 代码参数含义：
>  -t 指定密钥类型，默认是 rsa ，可以省略。
>   -C 设置注释文字，比如邮箱。
>  -f 指定密钥文件存储文件名。

以上代码省略了 -f 参数，因此，运行上面那条命令后会让你输入一个文件名，用于保存刚才生成的 SSH key 代码，如：

    Generating public/private rsa key pair.
    Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter]

当然，你也可以不输入文件名，使用默认文件名（推荐），那么就会生成 id_rsa 和 id_rsa.pub 两个秘钥文件。

- #### 设置密码

接着又会提示你输入两次密码（该密码是你push文件的时候要输入的密码，而不是github管理者的密码），

当然，你也可以不输入密码，直接按回车。那么push的时候就不需要输入密码，直接提交到github上了，如：

    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:

接下来，就会显示如下代码提示，如：

	Your identification has been saved in /c/Users/you/.ssh/id_rsa.
	Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
	The key fingerprint is:
	SHA256:+4tY/EYfWA4L7rtWc4uLcw7kvOeqTNsgwdP8Me4xRMY your_email@example.com

当你看到上面这段代码的时候，那就说明，你的 SSH key 已经创建成功.



### 3.查看 `.ssh`公钥

    $ cd ~/.ssh     进入 ssh 的文件
    $ more id_rsa.pub  

查看内容, 会出现一大堆字符, 一般以 ssh-rsa 开头, 你的邮箱地址结束, 全部复制即可。然后可以在 github 或者 gitlab 上去添加你的 ssh

