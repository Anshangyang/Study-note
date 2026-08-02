# Git

————

序言：此笔记系笔者在学习Git过程中总结的基础概念。如有错漏，敬请谅解，欢迎前往 Issues 提出意见，笔者会及时修正。笔者会尽力完善此笔记，力求精准简洁。

---

### 查看状态：
git status：查看当前仓库状态。

### 添加修改：
git add . ：将当前目录所有修改加入暂存区。
git add 指定文件名：添加指定文件。

### 提交：
git commit -m “提交信息”：将暂存区的修改提交。

### 推送：
git push：上传本地提交。
git push origin main：第一次上传本地提交时。

### 查看历史：	git log 或 git log --oneline。

### 拉取更新：
git pull。

### 分支：
git branch：查看分支。
git branch 分支名：创建分支。
git checkout 分支名(旧) 或 git switch 分支名(新)。
git switch -c 分支名：创建并切换分支。

### 克隆仓库：
git clone 地址。

### 查看远程仓库：
git remote -v。

### 撤销操作：

#### 未add：
git checkout --文件名：恢复文件。
git restore 文件名：新版。

#### 取消add
git reset。

#### 删除文件：
git rm 文件：删除并提交。

### 初始化：
git init。

### 绑定远程仓库：
git remote add origin 链接。

---

# *"日复一日，必有精进！"*