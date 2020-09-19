# 查看已删除的远程分支
```
git remote prune --dry-run origin 查看当前有哪些是该消失还存在的分支
git remote prune origin 删除上面展示的所有分支
git fetch --prune origin 如果没有结果输出说明已经删除完成了
```

# 上传项目到gitlab
* 无git仓库  
```
git remote add origin http://xx.git
git add .
git commit -m "init"
git push -u origin master
```

* 已有git仓库
```
git remote rm origin
git remote add origin http://xx.git
git push -u origin --all
git push -u origin --tags
```

* 配置账户  
```
git config --global user.name "王英东"
git config --global user.email "wangyingdong@paxsz.com"
```

* 创建新版本库
```
git clone git@172.16.2.83:A35/A133.git
cd A133
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

* 已存在的文件夹或 Git 仓库
```
cd existing_folder
git init
git remote add origin git@172.16.2.83:A35/A133.git
git add .
git commit
git push -u origin master
```