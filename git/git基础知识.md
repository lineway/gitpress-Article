---
title: git基础知识
date: 2019-04-07 17:53:24
tags: git
categories: git
---

## 获取git仓库

### 初始化空仓库

现在我们开始介绍git的使用，这里我们首先在本地创建一个空白文件夹并且进入到这个目录，然后使用git初始化命令创建一个git仓库：

```bash
mkdir newInGit

cd newInGit

git init

## 初始化空的 Git 仓库于 /Users/username/newInGit/.git/
```

完成上述步骤后，会在当前目录下生成一个`.git/`的隐藏目录，这个目录就是初始化git仓库中所有的必须文件。这一步骤仅仅是实现了初始化的操作，暂时还没有文件被git跟踪。

### 在已有项目中初始git仓库

如果想要在一个项目路径下使用git，那么在初始化完成后，还需要跟踪原有项目的文件并将它们提交。这时候就需要使用到以下命令：

```bash
git add .

git commit -m "initial project"

```



上述`git add .`中的`.`代表的意思是匹配目录下的所有文件。使用以上两个命令后，当前项目路径下的所有文件都会完成提交，这时候完成了在一个项目路径下创建git仓库。

### 克隆一个git仓库

如果想要获取一个已有的git仓库的拷贝，或者与其他人进行协作，那么也可以通过`git clone`命令从服务器上获取一个git仓库。如：

```bash
git clone https://github.com/kubernetes/kubernetes.git
```

上述命令可以在本地的当前目录下获取一份`kubernetes`项目的拷贝，在完成克隆后，本地项目路径下会存在一个`.git`目录，这个目录中保存的是远程仓库中拉下的所有数据，然后从中读取最新版本的文件拷贝。

## 记录每次文件的更新

在上一部分内容中，我们在已经在本地存在了一个`git`仓库，需要注意的是，git所在工作目录下的文件处于以下状态中的一种：已跟踪或未跟踪。

对于项目路径下的文件，我们可以使用命令`git status`来查看它们的状态，如果工作区中没有暂存的文件，那么会返回提示消息如下：

```bash
On branch master

nothing to commit, working tree clean

```

输出信息表示，我们处于仓库的`master`分支上，并且改分支上没有需要提交的文件。关于`git分支`的内容，后续会有相关介绍。  

现在，我们在本地目录下创建一个名为`newFile`的新文件，然后，再使用`git status`查看当前目录的git仓库状态，

输出结果可能如下所示：

[![Jietu20190406-143727.jpg](https://i.loli.net/2019/04/06/5ca84958799e7.jpg)](https://i.loli.net/2019/04/06/5ca84958799e7.jpg)

上述信息中可以看出，我们新增加的文件`newFile`处于未跟踪的状态，所以这个仓库中的暂存区是没有文件的，按照提示信息，我们可以使用`git add`命令将新建的文件添加到暂存区中，即`git add newFile`。

[![add后.jpg](https://i.loli.net/2019/04/06/5ca84a9dd65ea.jpg)](https://i.loli.net/2019/04/06/5ca84a9dd65ea.jpg)

当我们使用`git add`命令将新建文件加入到暂存区后，提示消息就发生了改变，这时候我们继续使用`git commit`命令就可以将暂存区的文件提交到git仓库中。这样，一整套的记录追踪文件的更新就完成了。

对于新增加的文件或修改过得文件，记得要使用`git add`命令跟踪他们，使他们添加到暂存区中。

### 忽略文件

虽然我们可以使用git跟踪项目文件，但是项目中的某些信息我们不需要进行跟踪或者如果我们想将项目发布到类似`github.com`这种平台上的时候，肯定不会将配置文件等信息一起发布出去。这时候，我们就需要在提交的时候忽略一些文件。  

在这种情况下，我们可以创建一个`.gitignore`文件，在其中列出需要忽略提交的文件信息，如下:

```bash
.idea/

*.pyc
```

这个示例文件中列出了两个文件信息，`.idea/`告诉git忽略该文件夹下的所有文件，`*.pyc`告诉git忽略所有以pyc结尾的文件。  

`.gitignore`的文件规范如下：

- 所有空行或者以#开头的行会被git忽略；

- 可以使用标准的glob模式匹配；

- 匹配模式可以以`/`开头防止递归；

- 匹配模式可以以`/`结尾制定目录；

- 忽略制定模式以外的文件或目录，在模式前加上`！`取反。

其中，glob模式是一种简化了的正则表达式。`*`表示匹配零个或多个任意字符；`[abc]`匹配任意一个列在方框中的字符；`？`只匹配一个任意字符；`[0-9]`匹配所有0到9之间的数字；`**`匹配任意中间目录，如`a/**/c`可以匹配`a/c`、`a/b/c`等。  

### 提交更新

当我们将文件加入到暂存区以后，就可以进一步进行提交操作了。如果不进行提交操作的化，那么对于文件的修改只是保存在本地磁盘。提交使用到的命令是`git commit`。  

如果我我们在项目路径下使用该命令，那么会启动一个文本编辑器，如果没有在`git config`中做出修改，那么会弹出当前操作系统的默认文本编辑器。在弹出的文本编辑器中我们可以输入本次提交的相关信息，如果只是一些简单的提交工作，可以使用`git commit -m "<commit message>"`的方式提交，在`<commit message>`中输入本次提交的相关信息即可。  

实际上，`git commit`这个命令还可以添加一个`-a`的选项，如`git commit -a`，其作用是将项目中已经跟踪的文件暂存起来一并提交，等价的实现了`git add`和`git commit`两个命令。

### 移除文件&移动文件

在Linux下，我们可以使用`rm`命令删除文件，`mv`命令移动文件。但是在git项目路径下，我们删除一个文件或者移动一个文件并不能这样直接的处理就可以了。

如果需要在git仓库下移除某个文件，那就必须从已跟踪文件清单中移除，然后提交。这两个步骤可以使用`git rm`来完成。

在git中，并不会显式的跟踪文件移动操作，如果在git中重命名了某个文件，仓库中存储的元数据并不会体现出这一次改名操作。如果需要在git中对文件改名，可以进行如下操作：

```bash
git mv file_from file_to
```

这一条命令相当于下面三条命令：

```bash
mv file_from file_to
git rm file_from
git add file_to
```

如果分开操作，也是实现了一次改名的操作。

### 提交历史

使用git提交了多次任务之后，我们可能需要回顾一下提交历史，这时候就可以使用`git log`命令。

使用`git log`命令不带任何参数的话，会按照提交时间列出所有记录，最近的提交记录会放在最上面。常用的`git log`选项有以下几种：

- `-p`，用于显示每次提交的内容差异。后面可以跟上`-n`显式最近`n`次的提交，如`git log -p -2`会显式最近两次的提交信息；
- `-stat`选项使得用户可以查看每次提交的简略的统计信息；
- `--pretty`该选项可以指定使用不同于默认格式的方式显式提交历史，参数可以选择如下方式：`short`、`full`和`oneline`，具体展示结果可以自己实践一下。除了可选的参数外，我们还可以使用`format`的方式定制显式的格式。

