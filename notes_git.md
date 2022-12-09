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