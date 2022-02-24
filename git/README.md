## 常用命令

#### 设置用户

- git config --global user.name "FFFangYu"
- git config --global user.email "10xxxxxx@qq.com"
- ssh-keygen -t rsa -C "10xxxxxx@qq.com"     （用来生成 ssh key.   生成的ssh key可以在.ssh/id_rsa.pub中查看）
- cat .ssh/id_rsa.pub    查看获取key值

#### 创建分支

- git checkout -b 分支名
- git push -u origin 分支名   把本地创建的分支与远程关联

#### 合并分支

- git checkout  master
- git pull origin master
- git merge 分支名
- git status 查看状态



