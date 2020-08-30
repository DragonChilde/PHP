# 卸载PHP

## 删除php的相关包及配置

```
sudo apt-get autoremove php7*
```

## 删除关联

```
sudo find /etc -name "*php*" |xargs  rm -rf 
```

## 清除dept列表

```
sudo apt purge `dpkg -l | grep php| awk '{print $2}' |tr "\n" " "`
```

检查是否卸载干净

```
 dpkg -l | grep php
```

