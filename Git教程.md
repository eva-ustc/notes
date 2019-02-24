# [《Git教程》](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)笔记
## Git简介
* 初次安装完成以后要先使用以下命令设置用户名和电邮。
```bash
git config --global user.name "Your Name"
git config --global user.email "YourEmail@xxx.com"
```
* `git init`把当前目录变成git可以管理的仓库
* `git add file`把文件添加到仓库，多个文件名以空格分隔，`git add .`则添加所有文件。
* `git commit -m "message"`将修改提交到仓库。
## 时光机穿梭
* `git status`查看当前仓库状态。
* `git diff`查看文件的修改情况。比较的是工作区与暂存区。加上`--cached`参数则比较暂存区和分支。
* `git log`查看提交历史记录。如果嫌输出信息太多可以加上`--pretty=oneline`参数。
* `HEAD`表示当前版本，`HEAD^`表示上一个版本，依次类推`HEAD^^`表示上上版本，`HEAD~10`表示上10个版本。
* `git reset --hard 版本信息`回退到指定版本，版本信息可以是版本号或者HEAD指针。
* `git reflog`可以查看命令历史。
* `git checkout -- file`将文件回退到最近一次`git add`或`git commit`的状态。
* `git reset HEAD file`可以把暂存区的修改撤销掉。
* `rm file`将文件从工作区中删除，`git rm file`将文件删除并执行`git add`。
## [远程仓库](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374385852170d9c7adf13c30429b9660d0eb689dd43a000)
## 分支管理
* `git branch 分支名`创建分支
* `git checkout 分支名`切换到指定分支
* `git checkout -b 分支名`创建并切换到分支
* `git branch`列出所有分支
* `git branch -d 分支名`删除指定分支，`-D`大写D则强行删除。
* `git merge 分支名`将指定分支合并到当前分支
* `git log --graph`查看分支合并图
* `git merge --no-ff 分支名`可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
* `git stash`可以把当前工作区保存起来。`git stash list`查看保存过的工作区，`git stash apply 名字`来恢复，但恢复后并比删除，需要使用`git stash drop 名字`来删除，或者直接用`git stash pop 名字`恢复并删除。
* [多人协助](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/0013760174128707b935b0be6fc4fc6ace66c4f15618f8d000)
## 标签管理
* `git tag 标签名`新建标签
* `git tag -a 标签名 -m "标签说明"`指定标签信息
* `git tag`查看所有标签
* `git tag -d 标签名`删除标签
## 自定义Git
* `git config --global color.ui true`让git显示命令
* 需要忽略某些文件时，需要编写`.gitignore`文件，此文件本身需要放到版本库里
* `git config --global alias.xx abcde`可以配置命令简写使用`xx`代替`abcde`
```bash
常用的几个
git config --global alias.last 'log -1'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```
* [搭建Git服务器](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137583770360579bc4b458f044ce7afed3df579123eca000)