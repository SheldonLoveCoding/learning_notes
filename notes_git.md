# Git 学习

## 常用命令



| 命令                         | 含义                                    |
| ---------------------------- | --------------------------------------- |
| `git init` | 初始化一个仓库 |
| `git add .\test.txt`         | 将当前目录下的test.txt文件添加进暂存区  |
| `git add .`                  | 将当前目录下的所有文件添加进暂存区      |
| `git commit -m 'message'`    | 对本次提交进行注释                      |
| `git branch branch_name`     | 创建一个名为branch_name的分支           |
| `git branch -a`              | 查看当前分支情况，处于哪一个分支        |
| `git checkout 'branch_name'` | 切换到branch_name的分支                 |
| `git log`                    | 查看提交日志，与每次提交对应的hash_code |
| `git reset --hard hash_code` | 回退到 hash_code 对应的版本             |
| `git branch -M main`         | 新建main分支 并将主分支改为main            |
| `git remote add Some_Name HTTP.git` | 将远端仓库HTTP.git取一个别名Some_Name，添加到远端列表里 |
| `git remote -v` | 查看远端列表 |
| `git push -u Some_Name Branch_Name` | 将Branch_Name的文件push到Some_Name对应的github仓库 |
| `git clone HTTP .`           | 将HTTP对应的仓库clone到本地        |
| `git ls-files`      | 查看暂存区（git add 之后）的文件 |


## gitignore的使用
是使用git管理项目时，存放 `忽略文件规则 `的文档。

`touch .gitignore`  生成gitignore文件。

在.gitingore 文件中，遵循相应的语法，在每一行指定一个忽略规则。常见的忽略规则如下：
```c++
1 node_modules    忽略node_modules的单文件 
2 
3 *.zip            表示忽略所有 .zip结尾的文件
4 
5  /dist          忽略项目根目录下的 dist 文件夹,但不包括子目录下的dist文件夹；注意前面不要加 ./dist, 否则识别不了
6 
7  build/          忽略 build/目录下的所有文件，包括子目录下的文件
    
```



## 将已经add的文件重新修改会untrack的状态

```c++
//对某个文件取消追踪
git rm --cached <filename>  <filename2>  <filename3>
//对全部文件取消追踪
git rm -r --cached . 　　//不删除本地文件
```

## 撤销commit

```
https://www.jianshu.com/p/491a14d414f6
git reset --soft HEAD^  #撤销上次的commit，不更改工作区的代码
git reset --hard 提交id  #回到那一次的commit，更改代码
```

## 更新gitignore的话需要：

首先要把之前已经track的文件利用`git rm --cached`命令取消跟踪。

然后更新gitignore，保存。

然后再`git add .`就可以了

## add / commit / push的区别？push的是哪些文件？

commit多次的话，push是要全部push上去的。



## 遇到网络问题，方法？

挂梯子，修改代理。

查看代理(本电脑的代理是  127.0.0.1:7890)

```
git config --global --get http.proxy
git config --global --get https.proxy
```

修改DNS（不太好用）

## github的仓库clone到本地后，可以修改仓库文件夹名称吗？

可以

## git 将远程分支同步到本地？

```
git remote -v
git fetch REMOTENAME BRANCHNAME
git log BRANCHNAME.. REMOTENAME/BRANCHNAME
git merge REMOTENAME/BRANCHNAME
```

如果遇到问题`error: Your local changes to the following files would be overwritten by merge`，则需要根据自己的需求确定好解决方案。

如果是直接用仓库中的文件覆盖掉本地文件，则直接

```
git reset --hard
git pull REMOTENAME BRANCENAME
```

### 如果远程仓库新建了一个branch，如何同步呢？

```git
git pull origin <远程分支名>:<本地分支名> // 拉取

git checkout <本地分支名> //切换本地分支

git push -u origin <远程分支名>  //本地与远程关联
// 之后就可以 git pull git push 操作了
```



## git的项目管理

当参与开源项目的时候，我们需要把开源项目fork一个自己的仓库，然后再将自己的仓库clone到本地进行实际的开发。

其实这里就涉及到三个概念。开源项目：upstream。自己fork的仓库：remote。本地的仓库：local。

我们在本地修改文件，其实是可以认为是开发不同的任务，因此最好的做法是在`main`或`master`分支上，重新切出一个新的分支`feature/task_x`，在该分支下进行文件的修改，然后再push到remote端，再提交新的pr，供给上游项目，即开源项目的管理者审核。如果审核通过，pr被merge，此时我们需要将我们自己fork的仓库与upstream的仓库进行同步。

与upstream的仓库进行同步有两种方法，一种是删除之前的fork的仓库，再重新fork。一种是在fork仓库的github的首页上点击`syncfork`的按钮进行同步，然后将同步后的代码pull到本地，再切换对应的分支进行相应的开发。如果不是用的github没有`syncfork`的按钮，则需要在本地进行相应的设置并手动同步，同步的步骤可参考https://github.com/selfteaching/the-craft-of-selfteaching/issues/67。

手动同步：

1. git remote add upstream https://xxx.git 需要设置upstream
2. git fetch upstream  拉取上游，但不合并
3. git checkout main 切换到mian分支
4. git merge upstream/main 合并远程的main分支
5. git push 将本地同步后的push到自己fork的仓库的main分支

