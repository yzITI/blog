---
title: Netdata部署与服务器集群监控
date: 2019-04-13 10:24:55
categories:
- Linux服务器
tags:
- CentOS
- Linux
- 服务器
- Netdata
toc: true
---
> netdata作为一个极其优秀的linux服务器监控系统，由于其惊为天人的界面、海量收集的数据、安全的只读策略、和方便快捷的部署，占据了很大的市场。然而，它是由海外开发的，所以在中国地区，没有中文文档，并且一些安装会有一些困难，特此用本文解决一些问题。
<!-- more -->
# 官方链接
- 演示网页：https://my-netdata.io/
- 官方首页：http://netdata.cloud/
- 文档地址：http://docs.netdata.cloud
- github地址：https://github.com/netdata/netdata

相关原理请自行阅读以上链接

# 基本部署
官网提供了一个很强大的一键安装脚本（必须用bash运行），可以自动解决很多问题：
```shell
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
```
但是这个脚本在安装时需要访问 googleapis ，因此在中国大陆地区，安装过程会被强制终止而失败。幸好开发者在 github issue 里面注意到了这个问题，因此，可以用下面这个命令来安装（必须在bash执行）：
```shell
bash <(curl -Ss https://my-netdata.io/kickstart.sh) --stable-channel
```
这样整个安装过程都会从github获取资源（虽然还是很慢）。

在安装过程中，有时还会因为`go.d.plugin`超时而失败，解决方法：多试几次就好了。

安装完毕以后，访问`IP:PORT`即可，通常默认端口是19999
> netdata的安装在百度上有很多其他教程，这里不作赘述。

# 集群监控
国内大多数博文没有详细论述的是，如何在一个集群里面部署netdata从而实现集群监控。

首先需要明确的是，netdata 本身不存在主从服务，在每一个节点上都需要完整部署 netdata（按照上面的方法），所以 netdata 并不轻量。在每一个节点上都部署了 netdata 以后，就有两个方式实现集群监控了。
## netdata.cloud
使用netdata自带的（在建的）netdata.cloud，也就是每一个节点控制面板右上角的那个   signin 。只要使用同一账号登陆到 netdata.cloud（需要科学上网），各个节点之间就可以轻松通过一个账号控制。

但是，它的缺点就在它的原理上：netdata.cloud 只是帮你记录每个节点的信息，而控制面板在获取每个节点的数据的时候，是由前端直接从各个节点的19999端口获取数据的。也就是说，每个节点都必须要打开端口，开启 dashboard，允许管理员查看数据。


可以说，这是一种被动的集群监控，本质上还是独立的机器，并且不方便做自定义的集群 dashboard 。

## stream
为了解决上一种方法的问题，netdata 同时提供了另外一种方法，流数据汇总到一台主服务器上。它本质上是主动汇总，数据处理全部在主服务器上进行，各个节点服务器不打开19999端口供查看，只是把收集到的数据发送到主服务器上。这样在主服务器上可以进行自定义的 dashboard 开发。

缺点在于，主服务器流量会比较大（如果集群很大的话）。如果觉得主服务器流量过大，可以设置节点服务器的数据收集周期`update every`。

stream配置方法如下：

> 原文：https://docs.netdata.cloud/streaming/

### 节点服务器
修改netdata.conf中的如下配置：
```
[global]
    memory mode = none
    hostname = [改成需要设置的主机标识名]
[web]
    mode = none
```
同时在同一目录下新建`stream.conf`并写入如下配置：
```
[stream]
    enabled = yes
    destination = IP:PORT
    api key = 11111111-2222-3333-4444-555555555555
```
其中，`destination`是主服务器的 IP 地址和 netdata 端口（默认19999），`api key`必须是一个 uuid 格式的字符串，可以在 linux 系统中用`uuidgen`指令生成。

上面配置完成后，重启netdata服务：
```
systemctl restart netdata
```
### 主服务器
在`netdata.conf`的同一目录下新建`stream.conf`并写入如下配置：
```
[API_KEY]
    enabled = yes
    default history = 3600
    default memory mode = save
    health enabled by default = auto
    allow from = *
```
其中，`API_KEY`对应节点服务器的`api key`，`allow from`可以设置数据流的允许来源以保证安全。

如果有多个节点服务器，则一起写在`stream.conf`里面

完成配置后重启netdata：
```
systemctl restart netdata
```

### 集中管理
如果上述配置都正确，那么在浏览器打开主服务器的控制面板，在左上角主机名那里下拉菜单中就可以看到接入的其他节点。

如果有问题，请查看官网原文，查看日志进行调试。

# 自定义控制面板
> 原文地址：https://docs.netdata.cloud/web/gui/custom/

其实，默认的控制面板就已经非常好了，但是主要是集群监控的时候，能在一个大屏上看到所有服务器的运行情况，于是就需要自己定制一个 dashboard 。

netdata 提供了`dashboard.js`，当然读者也可以使用其他控制面板。

有几个注意事项：

1. netdata 默认的 web 目录在`/usr/share/netdata/web`，上传新的 html 文件以后需要把新文件的所有权改为用户 netdata 才可以运行：
```shell
chown netdata:netdata xxx.html
```
2. 需要引入`dashboard.js`：
```html
<script type="text/javascript" src="./dashboard.js"></script>
```
3. 想要让整体的画风变为暗色系，只需要在 html 里面任何地方加上 javascript 代码
```js
netdataTheme = "slate";
```
4. 图表的一般格式最好在参考官网的基础上，通过检查元素的方法查看默认控制面板的设置。

5. 图表元素的 data-host 属性如果是每个节点服务器独立监控的，那么填对应的 ip 和端口即可，如果是使用 stream 模式进行的集群监控，则按照如下配置方法：
`ip:port`是主服务器的 ip 和端口
主服务器自己的数据：
```
data-host="//ip:port/"
```
节点服务器：
```
data-host="//ip:port/host/[hostname]"
```
**注：`[hostname]`是在节点服务器的配置文件中`[global]`下设置的`hostname`**

此外，当然还可以引入Vue.js等前端库进行交互式设计啦！