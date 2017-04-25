# git 常用命令

越来越多的人和公司使用 git 作为项目代码的版本管理工具。git 使用起来非常简单，但同时它的各种命令也相当复杂。

## 创建 SSH 密钥

创建 SSH 密钥，一路按 Enter 就 OK

```
$ ssh-keygen -t rsa -b 4096 -C "typefo@example.com"
```

其中 `-t` 表示密钥类型，`-b` 表示密钥长度，`-C` 表示一个注释说明，生成的密钥一般存放在当前用户的家目录下

```
id_rsa
id_rsa.pub
```

其中 `id_rsa` 为私钥，不能泄露给任何人，`id_rsa.pub` 为公钥，一般放在 git 服务器或者其他远程服务器上

## 环境配置 config

配置 git 默认用户名

```
$ git config --global user.name "typefo"
```

配置 git 默认邮箱

```
$ git config --global user.email "typefo@example.com"
```

其中 `--global` 表示全局配置，不加这个参数就表示只在当前仓库有效

## 仓库操作 repository

新建一个仓库

```
$ git init
```

新建一个裸仓库

```
$ git init --bare
```

克隆一个远程仓库

```
$ git clone https://github.com/user/repository.git
```

将远程仓库变更内容同步到本地

```
$ git pull
```

将本地仓库变更内容同步到远程仓库

```
$ git push
```

## 状态查看 status

查看当前仓库状态

```
$ git status
```

## 分支操作 branch

新建一个本地分支

```
$ git branch develop
```

切换到一个分支

```
$ git checkout develop
```

新建一个本地分支，同时切换到新建的分支

```
$ git checkout -b develop
```

查看分支信息

```
$ git branch
```

查看所有分支信息，包括远程分支

```
$ git branch -a
```

查看分支详细信息

```
$ git branch -vv
```

创建一个本地分支，同时关联到一个远程仓库上已经存在的分支

```
$ git branch --track develop origin/develop
```

删除一个本地分支

```
$ git branch -d develop
```

删除一个远程仓库上的分支

```
$ git push --delete origin develop
```

其中 `origin` 表示一个远程仓库，`develop` 表示一个远程仓库上的分支

## 远程操作 remote

查看所有远程仓库

```
$ git remote -v
```

添加一个新的远程仓库

```
$ git remote add origin https://github.com/user/repository.git
```

删除一个远程仓库

```
$ git remote rm origin
```

重命名一个远程仓库主机名

```
$ git remote rename origin repo
```

## 日志记录 log

查看当前仓库分支的 commit 历史记录

```
$ git log
```

以精简方式显示 commit 历史记录

```
$ git log --oneline
```

显示 commit 历史记录，包括内容变更和文件变更

```
$ git log --stat
```

## 标签操作 tag

-----------------------------------

By typefo <typefo@qq.com> Update: 2017-04-24 本文档使用 CC-BY 4.0 协议 ![by](../img/by.png)

