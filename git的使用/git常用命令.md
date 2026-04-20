- **工作区**：仓库的目录。
- **暂存区**：数据暂时存放的区域，类似于工作区写入版本库前的缓冲区
- **版本库**：存放所有已经提交到本地仓库的代码版本
- **版本结构**：树结构，树中每个结点代表一个代码版本
# 一、全局设置
- 1、设置全局用户名，信息记录在 ~/.gitconfig 文件中
``` git bush
git config --global user.name xxx
```
- 2、设置全局邮箱地址，信息记录在 ~/.gitconfig 文件中
``` git bush
git config --global user.email xxx@xxx.com
```
* 3、将当前目录配置成 git 仓库，信息记录在隐藏的 .git 文件夹中
``` git bush
git init
```
# 二、常用命令
- 1、将 XX 文件添加到暂存区
``` git bush
git add XX
```
- 2、将暂存区的内容提交到当前分支
``` git bush
git commit -m "给人看的备注"
```
- 3、查看仓库状态
``` git bush
git status
```
- 4、查看当前分支的所有版本
``` git bush
git log
```
- 5、将当前分支推送到远程仓库
``` git bush
git push -u(第一次推送需要-u，之后再次推送不需要)
```
- 6、将远程仓库 XXX 下载到当前目录下
``` git bush
git clone git@git.acwing.com:xxx/XXX.git
```
- 7、查看所有分支和当前所处分支
``` git bush
git branch
```
# 三、查看命令
- 1、查看 XX 文件相对于暂存区修改了哪些内容
``` git bush
git diff XX
```
- 2、查看仓库状态
``` git bush
git status
```
- 3、查看当前分支的所有版本
``` git bush
git log
```
- 4、用一行来显示
``` git bush
git log --pretty=oneline
```
- 5、查看 HEAD 指针的移动历史（包括回滚的版本）
``` git bush
git reflog
```
- 6、查看所有的分支和当前所处分支
``` git bush
git branch
```
- 7、将远程仓库的当前分支与本地仓库的当前分支合并
``` git bush
git pull
```
# 四、删除命令
- 1、将 xx 文件从仓库索引目录中删掉，不希望管理这个文件
``` git bush
git rm --cached xx
```
- 2、将 xx 从暂存区里移除
``` git bush
git restore --staged xx
```
- 3、将 xx 文件尚未加入暂存区的修改全部撤销
``` git bush
git checkout - xx     或者    git restore xx
```
# 五、代码回滚
- 1、将代码库回滚到上一个版本
``` git bush
git reset --hard HEAD^    或者    git reset --hard HEAD~
```
- 2、两个 ^^ 往上回滚两次，以此类推
``` git bush
git reset --hard HEAD^^
```
- 3、往上回滚 100 个版本
``` git bush
git reset --hard HEAD~100
```
- 4、回滚到某一个特定版本
``` git bush
git reset --hard 版本号
```
# 六、远程仓库
- 1、将本地仓库关联到远程仓库
``` git bush
git remote add origin git@git.acwing.com:xxx/XXX.git
```
- 2、将当前分支推送到远程仓库
``` git bush
git push -u(第一次推送需要-u，之后再次推送不需要)
```
- 3、将本地的某个分支推送到远程仓库
``` git bush
git push origin branch_name
```
- 4、将远程仓库 XXX 下载到当前目录下
``` git bush
git clone git@git.acwing.com:xxx/XXX.git
```
- 5、设置本地的 branch_name 分支对应远程仓库的 branch_name 分支
``` git bush
git push --set-upstream origin branch_name
```
- 6、删除远程仓库的 branch_name 分支
``` git bush
git push -d origin branch_name
```
- 7、将远程仓库的 branch_name 分支拉取到本地
``` git bush
git checkout -t origin/branch_name
```
- 8、将远程仓库的当前分支与本地仓库的当前分支合并
``` git bush
git pull
```
- 9、将远程仓库的 branch_name 分支与本地仓库的当前分支合并
``` git bush
git pull origin branch_name
```
- 10、将远程的 branch_name1 分支与本地的 branch_name2 分支对应
``` git bush
git branch --set-upstream-to=origin/branch_name1 branch_name2
```
# 七、分支命令
- 1、创建新分支
``` git bush
git branch branch_name
```
- 2、查看所有分支和当前所处分支
``` git bush
git branch
```
- 3、创建并切换到 branch_name 这个分支
``` git bush
git checkout -b branch_name
```
- 4、切换到 branch_name 这个分支
``` git bush
git checkout branch_name
```
- 5、将分支 branch_name 合并到当前这个分支上
``` git bush
git merge branch_name
```
- 6、删除本地仓库的 branch_name 分支
``` git bush
git branch -d branch_name
```
- 7、设置本地的 branch_name 分支对应远程仓库的 branch_name 分支
``` git bush
git push --set-upstream origin branch_name
```
- 8、删除远程仓库的 branch_name 分支
``` git bush
git push -d origin branch_name
```
- 9、将远程的 branch_name 分支拉取到本地
``` git bush
git checkout -t origin/branch_name
```
- 10、将远程仓库的当前分支与本地仓库的当前分支和合并
``` git bush
git pull
```
- 11、将远程仓库的 branch_name 与本地仓库的当前分支和合并
``` git bush
git pull origin branch_name
```
- 12、将远程的 branch_name1 分支与本地的 branch_name2 分支对应
``` git bush
git branch --set-upstream-to=origin/branch_name1 branch_name2
```
# 八、stash暂存
- 1、将工作区和暂存区中尚未提交的修改存入栈中
``` git bush
git stash
```
- 2、将栈顶存储的修改恢复到当前分支，但不删除栈顶元素
``` git bush
git stash apply
```
- 3、删除栈顶存储的修改
``` git bush
git stash drop
```
- 4、将栈顶存储的修改恢复到当前分支，同时删除栈顶元素
``` git bush
git stash pop
```
- 5、查看栈中所有元素
``` git bush
git stash list
```