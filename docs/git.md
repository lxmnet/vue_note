# Git 常用命令整理

### 项目上传至远程目录

如果你的代码没有使用 Git 跟踪，执行：

```
git init
git remote add origin git@xxx.git
git add .
git commit
git push -u origin master
```

如果你的代码已经由 Git 跟踪，然后设置这个仓库作为你的 "origin" 推送

```
cd projectName
git remote set-url origin git@xxx.git
git push -u origin master
```

## ssh key 生成

**1.设置 Git 的 user name 和 email**

```
git config --global user.name "lxmnet" 
git config --global user.email "lxmnet@outlook.com"
```

**2.查看是否已经有了ssh密钥**

```
cd ~/.ssh
```
> 如果没有密钥则不会有此文件夹，有则备份删除

**3.生成密钥**

```
ssh-keygen -t rsa -C "lxmnet" // 执行后按3个回车
```

## 版本回退

```
// 本地回退
git reset --hard commit_id
// 远程回退
git push origin HEAD --force
```

## 与远程仓库的交互

```
// 查看远程库的一些信息，及与本地分支的信息
git remote show origin

// 删除远程仓库已删除的本地分支
git remote prune origin
```

