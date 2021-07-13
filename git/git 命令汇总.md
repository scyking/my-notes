# git 命令汇总

标签（空格分隔）： git

---

[toc]

## 常用命令关系

![命令关系图](/md/git/20214810163281.png)

## 常用命令整理

### 创建版本库

```bash
git clone <url>                 #克隆远程版本库
git init                        #初始化本地版本库（生成.git文件夹）
```

### 修改和提交

```bash
git status                      #查看工作区和暂存区的文件状态
git diff                        #比较暂存区文件和本地库的差异
git add .                       #添加所有改动的文件到暂存区
git add <file>                  #添加文件到暂存区
git mv <old> <new>              #移动或重命名文件、目录
git rm <file>                   #从工作区和暂存区移除文件
git commit -m "commit message"  #提交所有更新过的文件
git commit --amend              #修改最后一次提交
```

### 查看提交历史

```bash
git log                         #查看提交历史
git log -p <file>               #查看指定文件的提交历史
git blame <file>                #以列表方式查看指定文件的提交历史
```

### 撤销

```bash
git reset --hard HEAD           #撤销工作目录中所有未提交文件的修改内容
git checkout HEAD <file>        #撤销指定的未提交文件的修改内容
git revert <commit>             #把已经添加到暂存去的文件从暂存区剔除
```

### 分支与标签

```bash
git branch                      #显示所有本地分支
git checkout <branch/tag>       #切换到指定分支或标签
git branch <new-branch>         #创建新分支
git branch -d <branch>          #删除本地分支
git tag                         #列出所有本地标签
git tag <tagname>               #基于最新提交创建标签
git tag -d <tagname>            #删本地除标签
```

### 合并与衍合

```bash
git merge <branch>              #合并指定分支到当前分支
git rebase <branch>             #衍合指定分支到当前分支
```

### 远程操作

```bash
git remote -v                   #查看远程版本库信息
git remote show <remote>        #查看指定远程版本库信息
git remote add <remote> <url>   #添加远程版本库
git fetch <remote>              #将远程版本库的更新取回到本地版本库
git pull <remote> <branch>      #下载代码及快速合并
git push <remote> <branch>      #上传代码及快速合并
git push --tags                 #上传所有标签
```

### 暂存操作

```bash
git stash                       #暂存当前修改
git stash apply                 #恢复最近一次的暂存
git stash pop                   #恢复暂存并删除暂存记录
git stash list                  #查看暂存列表
git stash drop <stash-name>     #移除某次暂存
git stash clear                 #清除暂存
```

### 拉取、上传免密码

```bash
git config --global credential.helper store
```
