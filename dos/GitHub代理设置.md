# GitHub代理设置

国内 `git clone` 一个 GitHub 上的仓库太慢，经常连接失败。下面是解决办法。

## 一、代理设置

### 1、全局代理设置

```
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```

这里可以打开 SS 查看代理设置，查看自己的端口是否为 1080，不是的改为对应的端口。



### 2、只对 GitHub 进行代理

如果挂了全局代理，克隆 coding 之类的国内仓库会变慢，所以我建议使用如下命令，只对 GitHub 进行代理，对国内的仓库不影响。

```
git config --global http.https://github.com.proxy https://127.0.0.1:1080
git config --global https.https://github.com.proxy https://127.0.0.1:1080
```

如果在输入这条命令之前，已经输入全局代理的话，可以按照**二、取消代理**的方法进行取消。

**注意：以上两点都是对 https 协议进行代理设置，也就是仅对 `git clone https://www.github.com/xxxx/xxxx.git` 这种命令有效。对于 SSH 协议，也就是 `git clone git@github.com:xxxxxx/xxxxxx.git` 这种，依旧是无效的。**



### 3、sock5 代理设置

之前说的是 http 代理，有人反映 ss 暴露的是 socks5。下面附上 socks5 代理的方法。

1、首先查看自己 socks5 的端口号，假设为：127.0.0.1:1086

2、输入以下命令：

```
git config --global http.https://github.com.proxy socks5://127.0.0.1:1086
git config --global https.https://github.com.proxy socks5://127.0.0.1:1086
```



## 二、取消代理

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```



## 三、查看已有配置

```
git config --global -l
```

