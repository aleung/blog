---
title: Setup transparent proxy on Ubuntu
date: 2021-02-28 18:21:21
tags:
- Anti-GFW
---

家里各种联网的设备多了，要能比较方便连上Internet，还是在路由器上设置个透明代理比起每个设备都要安装简单一些。我的目标是设置多个wifi热点，其中一个配置透明代理，有需要的设备才连上这个热点，这样可以避免在代理出什么问题的时候影响了普通的网络访问。

最初想直接在路由器上搞，OpenWRT支持配置多个wifi热点多个网络。但安装OpenClash后发现它是要设置iptables规则的，这是透明代理所必须，而它设置的iptables规则对所有网络都生效了，做不到仅对一个热点打开代理。OpenClash用起来似乎还问题多多，怎么配置都不太正常。

想起来家里还有个9年前的小电脑，用的是上网本的CPU，功耗比较低，虽然安装桌面系统它性能已经完全不能用了，但做这个用途还是足够的。而且它有一个以太网卡和一个无线网卡，正好可以充当一个次级无线路由，挂在主路由下面，物尽其用。翻出来启动一看，系统装的是Lubuntu 18.04 LTS，把软件升级了一下一切正常，那我也懒得重装系统了。

平常都没有怎么接触Linux系统、网络配置的东西，摸索着把东西装起来。比起网上的好多资料，应该算是比较简化的步骤，尽量沿用原系统缺省安装的软件和配置。整个过程总结如下。

<!-- more -->

- 首先，开启OS路由功能，修改 `/etc/sysctl.conf`：
  ```
  net.ipv4.ip_forward=1
  ```

- 安装dnsmasq-base。不要装dnsmasq，因为它会起一个服务，而NetworkManager在设置wifi hotspot后也会运行dnsmasq，造成重复的dnsmasq进程。
  ```
  apt install dnsmasq-base
  ```

- 如果系统里面已经有 systemd-resolved 服务，需要把它关闭，否则53端口会冲突。
  ```
  systemctl disable systemd-resolved
  ```
  
- Ubuntu的网络设备和连接是用NetworkManager进行管理的，它会自动覆盖重写`/etc/resolve.conf`文件。修改NetworkManager配置 `/etc/NetworkManager/NetworkManager.conf`，让它把`/etc/resolve.conf`里的nameserver配置指向dnsmasq。另外，如果 `/etc/resolve.conf` 是一个已经存在的链接，需要删除它。
  ```
  [main]
  dns=dnsmasq
  ```
  
- 运行NetworkManager的命令行交互式配置工具`nmtui`，创建一个wifi hotspot：mode选择access point，IPv4 config选择shared。如果不是远程登录，也可以直接在GUI上点击系统托盘中网络图标来配置。

  - 创建出来的子网是 `10.42.0.1/24`，而不是常见的 `192.168.*.*`。但反正能用，有什么关系呢。
  - NetworkManager会自动启动dnsmasq，DHCP也自动配置好了，无需安装任何额外软件包。

- 安装clash。Clash没有安装包，就一个可执行文件。为它创建专门的clash用户。
  ```
  useradd -U clash
  copy clash /usr/local/bin
  chown clash:clash /usr/local/bin/clash
  ```
  
- 配置`/etc/clash/config.yaml`，配置方式细节参考[clash文档](https://github.com/Dreamacro/clash/wiki/configuration)和[gitbook](https://lancellc.gitbook.io/clash/)，下面主要强调我的配置方式：
  - DNS设置为fake-ip模式。关于DNS工作模式，这篇文章解释得比较清楚： https://blog.skk.moe/post/what-happend-to-dns-in-proxy/
  - 将DNS侦听在1053端口，以免与dnsmasq的DNS服务冲突。
    Dnsmasq的DNS cache功能实际上不会被使用，因为往53端口流量会被iptables重定向到了clash的1053端口。但是如果关闭了dnsmasq的DNS功能，DHCP就不会下发nameserver配置给DHCP client，造成连上热点的设备没有DNS配置。

- 设置iptables，让clash工作于TProxy模式。
  ```
  iptables -t nat -I PREROUTING -p udp --dport 53 -j REDIRECT --to 1053
  iptables -t nat -I OUTPUT -p udp -d 127.0.0.0/8 --dport 53 -j REDIRECT --to 1053
iptables -t nat -A PREROUTING -p tcp ! --dport 22 -j REDIRECT --to-ports 7892
iptables -t nat -A OUTPUT -p tcp -m owner ! --uid-owner clash -j REDIRECT --to-port 7892
  ```
  
  - 首先需要复习一下关于iptables的知识：https://www.cnblogs.com/f-ck-need-u/p/7397146.html
  - 我这里采用了一个比较简化的配置，规则很少。没有支持UDP代理，目前感觉没有这样的需求，迟些再看看有没有必要加上吧。
  - 支持代理wifi hotspot流量（PREROUTING链）和本机流量（OUTPUT链）。
  - 使用了iptables的owner模块功能，配合专门的clash用户，避免流量回环。
  
- 将iptables规则持久化，下次启动的时候自动恢复。
  ```
  apt install iptables-persistent
  iptables-save > /etc/iptables/rules.v4
  ```

- 创建clash服务：`/etc/systemd/system/clash.service`
  ```
  [Unit]
  Description=Clash Daemon
  After=network.target
  
  [Service]
  Type=simple
  StandardError=journal
  User=clash
  Group=clash
  AmbientCapabilities=CAP_NET_BIND_SERVICE CAP_NET_ADMIN
  ExecStart=/usr/local/bin/clash -d /etc/clash/
  Restart=on-failure

  [Install]
  WantedBy=multi-user.target
  ```
  
  - 其中 `AmbientCapabilities` 那行是因为以非root用户运行，需要赋予权限。其实在我的这个场景下不要也行：DNS绑定的是高位端口，不是53端口，不需要`CAP_NET_BIND_SERVICE`；不代理UDP没有 `CAP_NET_ADMIN` 也无所谓。

- 最后启动 clash 服务
  ```
  systemctl daemon-reload
  systemctl enable clash
  ```

- 装个 [yacd](https://github.com/haishanh/yacd) 作为clash的web dashboard，管理起来会方便很多
  - 解压到 `/etc/clash/dashboard` 目录下面，并且在 `/etc/clash/config.yaml` 里面配置一行就行了
    ```
    external-ui: dashboard
    ```
    
  - 浏览器访问 `http://<router_ip>:9090/ui` 打开
  

----

Update：

后来发现这个小电脑的wifi热点速度很慢，应该是无线网卡的问题。于是还是改回旁路由的模式。

![](https://kroki.io/nwdiag/svg/eNqFkNGOgjAQRd_9igk-i6VujIb4JcaYSisQS6dpQdhs_Pe1RcUIjW_N9N577oxqecly-JsBlFbD3hZMC9hBJrHhh_QxXiygQi6MSmf3iRJ1i-YCBVbCOwEY50ZYezdGyZbGyXoTJzFZ0p_If_dm2E_KkujgRaiFak3tmLd3jDbY_YY5q7hzHN_sFTKJWjlU6mWZZLYIiOhTVF-PJ-xGfSRT4Takb_O1Cxm6VHgqpejfOhvh8kbYOgykz_W_EelA9JHHget5ucFGPzgZSjTOPGfkfGYkerva522CC0wHurhX4LjI7R86wLAH)

在主路由器OpenWRT上多配置了一个名为proxy的子网（bridge interface），关联了一个LAN口（要设置VLAN与其他口隔离）和一个wifi AP。通过在DHCP Server上设置DHCP-Options，把这个proxy子网的DNS和网关都需要指向安装了clash的小电脑。

```
config dhcp 'proxy'
	option start '100'
	option limit '150'
	option interface 'proxy'
	list dhcp_option '3,192.168.3.2'
	list dhcp_option '6,192.168.3.2'
```



