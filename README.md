# Github Database

[English](README_EN.md)

![under construction](images/under-construction.gif)

## 简介

Github Database是一个将Github的仓库作为数据库的解决方案。

这个方案使得我们可以在没有后端的时候完成一些简单的、**写入量不大**的数据库操作，比如对网页进行留言、评论；注册用户，等等。

### Warning

**数据库的读写量（尤其是写入量）不能过大**。

**⚠️ 这个方案被用于网页时含有一定风险。** [查看详细内容](#safety-warning)


### 原理

Github提供了Github api、Octokit等工具来辅助我们对GitHub上托管的仓库进行管理与控制。

因此，我们可以通过向GitHub内的仓库提交Push的方式更新数据库、通过运行GitHub Actions的方式完成一些后端可以完成的操作。

## 运行与维护

### 前期准备

1. 创建一个Private仓库（下称“DB仓库”），用作你的DB仓库。
2. 创建一个Fine-grained personal access token（下称“DB token”），并仅赋予它访问DB仓库的权限。
    - [创建个人访问令牌——Github Docs](https://docs.github.com/zh/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
    - 你应赋予DB token对于DB仓库的读写权限。
    - 如果需要使用使用DB仓库的Github Actions功能，可以赋予DB token关于Actions的相关权限。
    - **不应赋予DB token其它未使用的权限**。

这样，我们就完成了建立数据库的前期准备。

### DB仓库的读写

有两种方式可以实现对这个DB仓库的读写：

- 使用[Octokit](https://github.com/octokit)完成对DB仓库的读写
- 使用Github api完成对DB仓库的读写
- ~~通过各种奇技淫巧完成对DB仓库的读写~~

**值得注意的是，Octokit在钉钉内置的浏览器中表现不佳。** 因此，如果要开发在钉钉浏览器内也能运行的网站，我们推荐直接使用Github api。

如果使用Github api，则可以通过如下的方式对DB仓库进行读写：

Create file:
```mermaid
sequenceDiagram
    actor Client
    participant Github
    Client->>Github: put file
    Note over Github,Client: Create file
```

Update file:

```mermaid
sequenceDiagram
    actor Client
    participant Github
    Client->>Github: get file
    Github-->>Client: [file]
    Note over Github,Client: Get the file sha.
    Client->>Github: put file
    Note over Github,Client: Update file by the former sha.
```

<span id="safety-warning">

## 安全

**请注意到此为止我们的前提假设都是这个DB仓库工作在一个写入量不大的环境**，比如一个个人博客。

### ⚠️ 网页风险警示

**✅ 这个方案可以被安全的用于编译后的程序——前提是你为这些编译好的程序进行了加密处理。** 你可以将你的DB token直接写入程序，然后加密它，从而避免被反编译导致DB token泄露。如果加密整个程序有困难，你可以编写一个只用于读写DB仓库的程序，然后对其进行高强度加密。

**⚠️ 这个方案被用于网页时含有一定风险。** 没有后端意味着这个数据库没有办法完成一些只有后端才能完成的操作，比如身份的认证。

当这个DB仓库工作于一些所有人都有写入权限的环境时（比如一个评论区），这意味着你不得不将具有写入权限的DB token存放在网页的某个位置，以使前端得以正常操作DB仓库。但是，当DB token被别有用心的人找到后，他们可能使用其来恶意查看/修改你的DB仓库。

#### 一些应对方案

大多数黑客的攻击行为都是为了利益——无论是为了钱财，还是仅仅为了出名。

因此，当你的项目的读写需求不大时，它大概率不会被黑客注意到。即使你仍有所担心，你还可以通过下面的方法降低你的DB token被发现的概率：

- 不公布你的数据库解决方案，不要透露哪怕一个字，并假装你花钱办置了后端。
- 对你的JS代码、你的DB token进行混淆加密。例如：

    <details>
    <summary>初始代码：</summary>

    ```Javascript
    // base64 encoded
    var token = "dfghjkjdhstxgdshxjuhygDRFGYHBDFGYHUJNSBVGYHBDgvbhJNHvvUDHBJmgGHjBh"
    ...
    function updateDB(){
        fetch("https://api.github.com/repos/{ Owner }/{ Repo }/contents/" + fileName, {
            method: "put",
            headers: {
                Authorization: "token " + b64DecodeUnicode(token),
                Accept: "application/vnd.github.v3+json"
            },
            body: ...,
        });
    }
    ```
    </details>

    <details>
    <summary>加密后：</summary>

    ```
    // Magic. Do not touch.
    [][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]](([]+[][(![]+[])[!+[]+!![]+!![]]+([]+...//（太长了，后略）
    ```
    </details>

[一个加密工具的链接](https://www.sojson.com/jsfuck.html)

下面是一些止损的方法：

- 建立数个不同的DB仓库的方式来进一步降低可能的损失。
- **所有的DB token都不应被授予admin权限**，以免DB仓库遭到删除。

### 用户信息

我们**完全不建议**使用这个方案作为**存储网站用户信息**的数据库。

用户的密码不应被直接存储到你的DB仓库中。处于安全起见，DB仓库中储存的密码应为用户密码经加盐算法处理后的HASH值。