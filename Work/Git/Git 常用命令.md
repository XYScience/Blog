# Git 常用命令    
      
#### 0，新建仓库      
本地公钥复制到 GitHub 配置 ssh （Mac 公钥：Users/xxx/.ssh/id_rsa.pub，.ssh隐藏目录显示：shift+command+.）     
ssh -T git@github.com 判断是否绑定成功     
* git init     
* git add xxx.md     
* git commit -m "first commit"     
* git remote add origin git@github.com:XYScience/xxx.git     
* git push -u origin master (-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令)     
* git push origin master     

#### 1，基础命令

* git add (file_name)：把工作区要提交的所有修改(某个文件)放到暂存区（Stage）；
* git commit -m "xxx"：一次性把暂存区的所有修改提交到分支；
* git commit：可输入复杂的commit信息，Esc+:+wq 退出；
* git status：查看文件修改状态； 
* git diff：查看修改内容； (HEAD：当前版本，HEAD^：上一个版本；HEAD指向的就是当前分支；) 
* git log：查看提交(commit)历史； 
* git reflog：查看提交(包括撤销回退)历史； 
* git log --graph：看到分支合并图； 

#### 2，代码回退

* git checkout -- file_name：直接丢弃工作区某个文件的修改； 
* git checkout -f：直接丢弃工作区所有文件的修改； 
* git reset HEAD file：把暂存区的某个文件修改回退到工作区； 
* git reset HEAD . ：把暂存区所有文件回退到工作区； 
* git reset --hard commit_id／HEAD^：回退版本（本地仓库覆盖工作区）；    
* git reflog：用来记录每一次命令；

#### 3，分支管理

* git branch name_branch：创建分支； 
* git checkout name_branch：切换分支； 
* git checkout -b name_branch：创建+切换分支； 
* git branch -d name_branch：删除分支； 
* git branch：查看本地分支；
* git branch -r：查看远程分支； 
* git branch -a：查看所有分支；  

* git branch -vv：查看本地跟踪远程分支； 
* git branch -u origin/name_branch：当前分支跟踪远程分支； 

#### 4，工作现场

* git stash：把暂缓区现场“储藏”起来，等以后恢复现场后继续工作； 
* git stash list：查看已经储藏的工作现场； 
* git stash pop：恢复工作现场的同时把stash内容也删除； 
* git stash apply stash@{0}：恢复指定的stash； 
* git stash drop stash@{0}：删除指定的stash；

#### 5，拉取远程分支

* git fetch：从远程拉取代码到本地； 
* git merge name_branch：合并某分支到当前分支；
* git pull：拉取到本地并merge到本地分支中(git fetch + git merge)； 
* git rebase：把本地分支提交(commit)取消掉，并保存为补丁(patch)(这些补丁放到".git/rebase"目录中)，然后把本地分支更新为最新的"origin"分支，最后把保存的这些补丁应用到本地分支上； 
* git rebase --continue：在rebase时，如果出现冲突(conflict)，在解决完冲突后，用"git-add"命令更新这些内容的索引(index)，无需执行 git-commit，只要执行此命令； 
* git rebase --abort：在任何时候，可以用--abort参数来终止rebase的行动，并且本地分支会回到rebase开始前的状态；  
* git pull --rebase：git fetch + git rebase；
* git cherry-pick commit_id：将另一commit再次提交到当前分支，然后直接push命令；
