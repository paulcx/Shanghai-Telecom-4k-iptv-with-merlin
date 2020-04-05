# Shanghai-Telecom-4k-iptv-with-merlin

### 更新： AC86U设置vlan通过SDN网关看上海电信IPTV

### 前言：
家里最初是用R6300v2刷梅林固件，桥接后设置vlan满足上网与连iptv。后购入ac86u发现无法用原有的脚本，众所周知ac86u没有robocfg工具，就一直把ac86u作为ap，R6300v2负责拨号、联通iptv和软件中心，方案如（https://koolshare.cn/thread-43568-1-1.html）。

近期宽带升级到千兆，网速与众服务遇到了瓶颈，所以想到了用软路由。家里正好有一台NUC8（买了type-c、usb3.0千兆网卡共三块）的小型HTPC，就尝试在windows10下通过hyper虚拟机安装LEDE，ac86u继续当ap，在各种论坛里的方案如（https://koolshare.cn/forum.php?mod=viewthread&tid=132042），最终因为性能原因（没有做严格的实验，但是个人怀疑是虚拟机和独立网卡的问题，网速下降超过20%）不高兴再折腾，打算买台单独软路由了事。

后来一想，家里网络设备太多了，费电不说，各种设备稳定性不一带来的维护问题一直很花时间，最理想的方案就是all in One设备尽可能的少同时满足：
1. 上网（有线+Wifi）
2. IPTV（上海的iptv有A、B面验证的问题）
3. 其它多媒体服务（HTPC、NAS）

### 方案：

众所周知ac86u没有robocfg工具，经过搜索发现了vlanctl工具，随后又进一步找到了原作者的攻略（https://www.cnblogs.com/u128393/p/11629969.html）。根据方案才知道除了vlanctl还有vconfig设置vlan更简便。由此把原作者的脚本结合原一键脚本（https://github.com/love19862003/Shanghai-Telecom-4k-iptv-with-merlin），成功的实现了ac86u桥接下看iptv。只要把原ss配置文件中robocfg注释掉，替换成vconfig，其余不变即可（实际我把iptv-71.sh众从github下载的都替换成事先下载好的本地文件提高脚本执行速度）。

```
#!/bin/sh
#sed -i "s/cache-size=1500/cache-size=15000/" /tmp/etc/dnsmasq.conf

CONFIG=$1
source /usr/sbin/helper.sh

pc_replace "cache-size=1500" "cache-size=9999" $CONFIG

pc_append "dhcp-option-force=125,00:00:00:00:1a:02:06:48:47:57:2d:43:54:03:04:5a:58:48:4e:0a:02:20:00:0b:02:00:55:0d:02:00:2e
dhcp-option=15
dhcp-option=28
dhcp-option=60,00:00:01:06:68:75:61:71:69:6E:02:0A:48:47:55:34:32:31:4E:20:76:33:03:0A:48:47:55:34:32:31:4E:20:76:33:04:10:32:30:30:2E:55:59:59:2E:30:2E:41:2E:30:2E:53:48:05:04:00:01:00:50" /tmp/etc/dnsmasq.conf
#robocfg vlan 51 ports "0t 1t 2t 3t 4t 8t" vlan 85 ports "0t 1t 2t 3t 4t 8t"

vconfig set_name_type DEV_PLUS_VID_NO_PAD
vconfig add eth0 85
vconfig add br0 85

brctl addbr vlan85
brctl addif vlan85 eth0.85
brctl addif vlan85 br0.85

bcmmcastctl mode -i vlan85 -p 1 -m 1
bcmmcastctl mode -i vlan85 -p 2 -m 1

ifconfig eth0.85 up
ifconfig br0.85 up
ifconfig vlan85 up

vconfig add eth0 51
brctl addif vlan85 eth0.51
ebtables -A FORWARD -i eth0.51 -o ! br0.85 -j DROP
ebtables -A FORWARD -o eth0.51 -j DROP
ifconfig eth0.51 up
```

### 总结：

目前方案的拓扑结构如下：

        --光纤--《SDN光猫》--有线桥接--《ac86u路由器》--有线--《1.交换机（内部有线局域网）、2.IPTV（2台）》--有线--《HTPC、NAS》
                                          |
                                         wifi
                                          |
                              各种设备如手机，IoT智能设备等》


最终的结果有线测试可以达到千兆宽带的上限再990-1000兆（下载）左右，wifi在600兆左右，IPTV观看没有任何卡顿。前期找了很多资料实验不少不同的方案就不一一赘述，最后成功了在此留个存档，也感谢上文中原作者们。

English Veriosn：

Shanghai Telecom 4K IPTV with Bridge Mode for Merlin

Please View http://koolshare.cn/thread-37102-1-1.html

中文版本：

上海电信4K IPTV桥接，仅限于梅林固件

使用方法请见http://koolshare.cn/thread-37102-1-1.html

