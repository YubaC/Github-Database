# Github Database

![under construction](assets/images/under-construction.gif)

## 简介

Github Database是一个将GitHub的仓库作为数据库的解决方案。


## 前期准备

1. 创建一个Private仓库（下称“数据库仓库”），用作你的数据库仓库。
2. 创建一个Fine-grained personal access token（下称“DB token”），并仅赋予它访问数据库仓库的权限。
    - 你应赋予DB token对于数据库仓库的读写权限。
    - 如果你不需要使用数据库仓库的Actions等功能可以不赋予DB token其它权限。
        - [创建个人访问令牌——GitHub Docs](
https://docs.github.com/zh/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

这样，我们就完成了建立数据库的前期准备。

## 数据库仓库的读写

