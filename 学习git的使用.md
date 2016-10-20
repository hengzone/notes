# git
程序员标配，然而我连基础的命令都不会。
捉鸡。
借此记录一些git的基础知识。
## 准备
账号那些的注册下就可以，正常的一些账号设置结束后就可以下载软件了。
我是**Windows**环境所以下载了(Windows版 git)[https://git-for-windows.github.io/]。
## 配置git
首先在本地创建**SSH key**
```linux
ssh-keygen -t rsa -C "hengzone@Foxmail.com"
```
**windows**下会自动选择一个文件夹来保存相关文件，默认即可。接着确认路径，输入密码就可以。正确完成会在指定目录下生成**.ssh/xxx.pub**文件。
有时候那个`.pub`文件的打开不是很顺利，其实可以在**git bash**中直接用`vi`命令打开。
到**github**下找到**Account Settings**里的**SSH and GPG keys**，选择**New SSH key**，将上面打开的文件中的字符串复制到这里面即可。
为了验证成功与否，在**git bash**中输入
```linux
ssh -T git@github.com
```
顺利的话就需要输入**yes**和密码，之后就能看到
>Hi hengzone! You've successfully authenticated, but GitHub does not provide shell access.
这说明已经成功连接上了。
下面来设置一些常用参数，因为github每次commit都会记录他们。
```linux
git config --global user.name "Felix"
git config --global user.email "hengzone@Foxmail.com"
```
在需要同步的项目目录下进行初始化操作
```linux
git init
```
## 管理文件
经过初始化操作过后，就是一些常规的文件管理的命令了。
这里需要知道的是，**git**维护着三棵树：
1.第一个是你的 工作目录，它持有实际文件。
2.第二个是 暂存区（Index），它像个缓存区域，临时保存你的改动。
3.最后是 HEAD，它指向你最后一次提交的结果。
这个感觉有点像**SVN**，每个新的文件都需要先进行**add**之后再进行**commit**操作。之后每次文件有过修改之后就仅仅只需要进行**commit**操作了。
下面记录到的**git**操作就是给了我这样一种强烈的熟悉感。
首先要加入到暂存（index）中去
```linux
git add filename
```
接着才能使用如下命令进行提交操作
```linux
git commit -m "提交信息"
```
因为带有**-m**，所以这里的**提交信息**不能为空。
现在已经提交到了**HEAD**中了，还需要将它上传到远程仓库中，所以不能少了下面的操作
```linux
git push origin master
```
可以将这里的`master`换成其他的分支。
如果没有本地没有克隆的仓库，但是需要上传到指定的远程仓库，那么可以使用如下命令
```linux
git remote add origin <server>
```
这样就能把改动推动到指定的服务器上去了。
