# git基础知识

## 获取git仓库

### 初始化仓库

现在我们开始介绍git的使用，这里我们首先在本地创建一个空白文件夹并且进入到这个目录，然后使用git初始化命令创建一个git仓库：

```bash
mkdir newInGit
cd newInGit
git init

## 初始化空的 Git 仓库于 /Users/username/newInGit/.git/
```

完成上述步骤后，会在当前目录下生成一个`.git/`的隐藏目录，这个目录就是初始化git仓库中所有的必须文件。这一步骤仅仅是实现了初始化的操作，暂时还没有文件被git跟踪。

如果想要在一个项目路径下使用git，那么在初始化完成后，还需要跟踪原有项目的文件并将它们提交。这时候就需要使用到以下命令：

```bash
git add .
git commit -m "initial project"
```



