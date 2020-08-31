

# **Git初如化配置**

Git安装完之后，需做最后一步配置。打开git bash，分别执行以下两句命令

```
$ git config --global user.name “用户名”
$ git config --global user.email “邮箱”
```

查看git配置

```
$ git config --lis
```

生成Key

> 注意:查看是否已经有了ssh密钥:

```
$ cd ~/.ssh
```

如果没有密钥则不会有此文件夹,有则备份删除然后生成

```
$ git config --global user.email "邮箱"
```

按3个回车,密码为空

执行查看公钥的命令：

```
$ cat ~/.ssh/id_rsa.pub  
```

