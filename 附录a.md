# 附录A 安装

## A.1 Debian或Ubuntu Linux

## A.2 OS X

不使用编译器安装Redis，可以使用一个Python工具Rudix：

```bash
$ curl -O https://raw.githubusercontent.com/rudix-mac/rpm/2015.10.20/rudix.py
$ sudo python rudix.py install rudix
$ sudo rudix install redis 
```

待安装完成后，就可以启动redis了：

```bash
$ redis-server
```

下面使用pip安装Python的Redis库，首先安装pip：

```
$ sudo rudix install pip
$ sudo pip install redis
```

## A.3 Windows

## A.4 Hello Redis

![](/assets/QQ20160801-1.png)

