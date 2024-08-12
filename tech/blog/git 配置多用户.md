> 1. 删除 `.ssh` 目录下文件 


>2. 配置 config 文件

在 `.ssh` 目录下，新建文件 `config`。并配置 `config` 规则，gitlab 存放工作密钥，github 存放个人密钥。

```
# gitlab
Host gitlab.company.cn
HostName gitlab.company.cn
PreferredAuthentications publickey
IdentityFile <用户目录>\.ssh\id_rsa_work

# github
Host github.com
HostName github.com
PreferredAuthentications publickey
IdentityFile <用户目录>\.ssh\id_rsa_user
```

> 3. 生成 key 

```
ssh-keygen -t rsa -C "公司邮箱"
ssh-keygen -t rsa -C "个人邮箱"
```

> 4. 手动填写 key 的工作路径 

```
Generating public/private rsa key pair.  
Enter file in which to save the key (C:\Users\<用户名>\.ssh\id_rsa): C:\Users\<用户名>\.ssh\id_rsa_work


Generating public/private rsa key pair.  
Enter file in which to save the key (C:\Users\<用户名>\.ssh\id_rsa): C:\Users\<用户名>\.ssh\id_rsa_user
```

如果有必要则可以输入验证密码，使用 ssh 时需要输入该密码。


> 5. 添加私钥 

```
ssh-add C:\Users\<用户名>\.ssh\id_rsa_work

ssh-add C:\Users\<用户名>\.ssh\id_rsa_user
```

如果遇到目录不存在等添加失败时，可参看以下：

```
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
Get-Service ssh-agent
```

[cmd - Error connecting to agent: no such file or directory - adding key to ssh agent - Stack Overflow](https://stackoverflow.com/questions/65741816/error-connecting-to-agent-no-such-file-or-directory-adding-key-to-ssh-agent)

> 6. Gitlab 和 Github 上配置 ssh key 

参考： 

[Github配置ssh key的步骤（大白话+包含原理解释）\_github生成ssh key-CSDN博客](https://blog.csdn.net/weixin_42310154/article/details/118340458)

> 7. 验证  

```
ssh -vT git@gitlab.company.com
ssh -vT git@github.com
```

> 8. 查看并设置生效用户和邮箱

- 查看全局用户、邮箱

```
git config user.name
git config user.email
```

- 设置/删除全局生效用户和邮箱 

```
git config --global user.name "username"  
git config --global user.email "email"

git config --global --unset user.name  
git config --global --unset user.email
```

> 9. 对不同仓库设置不同用户和邮箱 

这个设置是局部的，如果不设置的话会沿用全局生效的用户和邮箱。

```
git config user.name "yourname"  
git config user.email "youremail"
```


**参考文档**：

[Git 配置多用户 | 而废不能半途](https://double-c.github.io/2018/11/17/git-many-accounts/index.html)
[配置SSH keys连接GitHub（支持多用户） | a\_flying\_fish' blogs](https://www.aflyingfish.top/articles/20c7aea41283/)