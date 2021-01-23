## GIT 文件回滚

### 已经修改，但未add到仓库，即还没有运行 git add filename;

```
git checkout -- filename
```

### 已经添加到本地仓库，但还未提交，即还没有运行git commit -m "message";

```
git reset HEAD filename
or
git reset HEAD
```

### 已经提交，即运行了git commit -m "message"

```
#先运行 git log 查看所有的提交历史记录
git reset --hard commitid commitid是版本的id
```

