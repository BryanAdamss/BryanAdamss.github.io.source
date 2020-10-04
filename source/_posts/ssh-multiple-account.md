---
title: ssh-multiple-account
tags:
  - ssh
  - github
  - gitlab
categories:
  - 前端
date: 2020-03-27 15:56:13
---

# ssh 配置多账户

- 公司部署了`gitlab`，自己同时在用`github`，所以需要配置两套`ssh账户`
- 特将相关流程记录下，方便后期查阅

## 生成 key

```bash
ssh-keygen -t rsa -C 'xxx@email.com' -f ~/.ssh/xxx-rsa
# -t 指定类型，不需调整，rsa即可
# -C 指定comment，一般为邮箱，会在生成的公钥最后体现
# -f 指定文件，如果是单账户，可以不用指定；默认生成id_rsa、id_rsa.pub；~在windows中代表c/users/用户名
```

- 执行上述命令后，会在~/.ssh/目录下生成一个`xxx-rsa`和`xxx-rsa.pub`两个文件，前者是需要保存在本地的私钥，后者是需要上传到`gitlab`或者`github`上的公钥
- 生成过程中，可能会询问你设置使用私钥时的密码(`passphrases`)
  - 可以忽略
  - 如果设置了，后期使用`ssh`时，要使用`ssh-agent`来管理`ssh密码`，实现免密登录，或者手动输入`ssh密码`
  - 建议忽略

## 上传公钥

- `github`可以在`https://github.com/settings/keys`页面上传公钥
- `gitlab`可在用户设置页面上传公钥

## 配置

- 指定不同的域，使用不同的配置

```bash
# git.xxx.com的配置
Host git.xxx.com
    HostName git.xxx.com
    IdentityFile ~/.ssh/gitlab-rsa
    User git

# github的配置
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/github-rsa
    User git
```

- `Host`域名，也可以理解为下面一套配置的别名，在`ssh`命令中可直接使用别名；
  - **建议设置成跟`HostName`值一样，在实际使用时，使用别名，会出现问题，所以建议`Host`、`HostName`设置成一样的**
- `HostName`，具体域名，完整域名
- `IdentityFile`当前域的配置文件(私钥)地址
- `User`用户，这里的用户是登录后台服务器的用户名，一般都为`git`，不需要修改
- `#`后面为注释
- `ssh配置文件`具体解释可参考
  - [https://www.cnblogs.com/xjshi/p/9146296.html](https://www.cnblogs.com/xjshi/p/9146296.html)

## 测试

```bash
ssh -T user@HostName
# 或者
ssh -T Host   # 使用Host定义的一套配置

# 例子
ssh -T git@github.com
```

## 生成多个 key 的实例

- 生成`github`、`gitlab`的`ssh`配置

### 生成相应 key

```bash
# 生成github
ssh-keygen -t rsa -C 'test1@qq.com' -f ~/.ssh/github-rsa

# 生成gitlab
ssh-keygen -t rsa -C 'test2@qq.com' -f ~/.ssh/gitlab-rsa
```

- 生成后，`~/.ssh/`目录下应该会有下面 4 个文件
- ssh.png
  - ![ssh.png](ssh.png)

### 分别上传公钥

- 上传`github`公钥
  - 打开用户下面的`settings`->`SSH and GPG keys`页面，或者直接访问`https://github.com/settings/keys`
  - 点击`New SSH key`
  - 记事本打开`github-rsa.pub`并拷贝内容到`github`即可
- 上传`gitlab`公钥
  - 步骤类似`github`
  - 拷贝`gitlab-rsa.pub`内容到相应公钥填写处即可

### 编写配置文件

```bash
Host gitlab..com
    HostName gitlab.com
    IdentityFile ~/.ssh/gitlab-rsa
    User git

Host github.com
    HostName github.com
    IdentityFile ~/.ssh/github-rsa
    User git
```

### 测试

```bash
ssh -T git@github.com
```

- 出现下图提示即可
- ssh-2.png
  - ![ssh-2.png](ssh-2.png)
- 同理，测试`gitlab`即可

### github 配置相关问题

- 可以参考
  - [https://help.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh](https://help.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh)

## 参考

- [https://www.cnblogs.com/popfisher/p/5731232.html](https://www.cnblogs.com/popfisher/p/5731232.html)
- [https://www.cnblogs.com/hafiz/p/8146324.html](https://www.cnblogs.com/hafiz/p/8146324.html)
- [https://help.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh](https://help.github.com/cn/github/authenticating-to-github/connecting-to-github-with-ssh)
- [https://www.cnblogs.com/xjshi/p/9146296.html](https://www.cnblogs.com/xjshi/p/9146296.html)
