# WIN10下安装WSL2的Ubuntu1804

## 安装WSL1

```
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

## 更新到WSL2

```
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

## 设定WSL2为默认

```
wsl --set-default-version 2
```

当设置有异常时,点击进入下面链接https://aka.ms/wsl2kernel ，更新 WSL 2 Linux 内核

在win10 store商店选择Ubuntu18.04下载安装

启动应用时会让输入初始账号密码

Ubuntu的默认root密码是随机的，
即每次开机都有一个新的root密码。我们可以在终端输入命令 sudo passwd，
然后输入当前用户的密码，enter，终端会提示我们输入新的密码并确认，
此时的密码就是root新密码。修改成功后，输入命令 su root，再输入新的密码就ok了

修改WSL默认登陆用户为root

```
ubuntu1804 config --default-user root
```

## 系统初始化

调整更新镜像地址,[阿里巴巴提供的 Ubuntu 软件源镜像地址](https://developer.aliyun.com/mirror/ubuntu)

然后运行如下命令更新并升级这个 Ubuntu 系统

```
sudo apt update
sudo apt upgrade
```



