```
iwconfig
airmon-ng start wlan
airodump-ng wlan0mon

 44:97:5A:C7:BF:DA

reaver -i wlan0mon -b 44:97:5A:C7:BF:DA -vv -d12 -t9

reaver -i wlan0mon -b 44:97:5A:C7:BF:DA -vv -1 414
```

（文章更新：Reaver在R3版中已经预装，如果你安装的是BT5的R3版，这一步骤可以忽略，直接跳到第3步。）

Reaver已经加入了BackTrack的最新版软件包，只是还没有集成到live DVD里，所以，在本文最初撰写的时候，你还需要手动安装Reaver。要安装Reaver，首先设置电脑联网。

1.点击Applications &gt; Internet &gt; Wicd Network Manager 2.选择你的网络并点击Connect，如果需要的话，键入密码，点击OK，然后再次点击Connect。

连上网以后，安装Reaver。点击菜单栏里的终端按钮（或者依次点击 Applications &gt; Accessories &gt; Terminal）。在终端界面，键入以下命令：
```
apt-get update
```

更新完成之后，键入：
```
apt-get install reaver
```

如果一切顺利，Reaver现在应该已经安装好了。如果你刚才的下载安装操作使用的是WiFi上网，那么在继续下面的操作之前，请先断开网络连接，并假装不知道WiFi密码 =。= 接下来我们要准备破解它~

#### 第3步：搜集设备信息，准备破解

在使用Reaver之前，你需要获取你无线网卡的接口名称、路由的BSSID（BSSID是一个由字母和数字组成的序列，用于作为路由器的唯一标识）、以及确保你的无线网卡处于监控模式。具体参见以下步骤。

**找到无线网卡：**在终端里，键入：
```
iwconfig
```

回车。此时你应该看到无线设备的相关信息。一般，名字叫做wlan0，但如果你的机子不止一个无线网卡，或者使用的是不常见的网络设备，名字可能会有所不同。

![](https://dn-linuxcn.qbox.me/data/attachment/album/201312/05/090757vqw8qqpggqqqqgiq.jpg)

**将无线网卡设置为监控模式**：假设你的无线网卡接口名称为wlan0，执行下列命令，将无线网卡设置为监控模式：
```
airmon-ng start wlan0
```

这一命令将会输出监控模式接口的名称，如下图中箭头所示，一般情况下，都叫做mon0。

![](https://dn-linuxcn.qbox.me/data/attachment/album/201312/05/090759prxxrnx49ore1rxz.jpg)

**找到你打算破解的路由器的BSSID**：最后，你需要获取路由器的唯一标识，以便Reaver指向要破解的目标。执行以下命令：
```
airodump-ng wlan0
```

（注意：如果airodump-ng wlan0命令执行失败，可以尝试对监控接口执行，例如airodump-ng mon0）

此时，你将看到屏幕上列出周围一定范围内的无线网络，如下图所示：

![](https://dn-linuxcn.qbox.me/data/attachment/album/201312/05/0908008cvbl7t76hnm76hh.jpg)

当看到你想要破解的网络时，按下Ctrl+C，停止列表刷新，然后复制该网络的BSSID（图中左侧字母、数字和分号组成的序列）。从ENC这一列可以看出，该网络是WPA或WPA2协议。（如果为WEP协议，可以参考我的[前一篇文章——WEP密码破解指南](http://lifehacker.com/5305094/how-to-crack-a-wi+fi-networks-wep-password-with-backtrack)）

现在，手里有了BSSID和监控接口的名称，万事俱备，只欠破解了。

#### 第4步：使用Reaver破解无线网络的WPA密码

在终端中执行下列命令，用你实际获取到的BSSID替换命令中的bssid：
```
reaver -i moninterface -b bssid -vv
```

例如，如果你和我一样，监控接口都叫做mon0，并且你要破解的路由器BSSID是8D:AE:9D:65:1F:B2，那么命令应该是下面这个样子：
```
reaver -i mon0 -b 8D:AE:9D:65:1F:B2 -vv
```
最后，回车！接下来，就是喝喝茶、发发呆，等待Reaver魔法的发生。Reaver将会通过暴力破解，尝试一系列PIN码，这将会持续一段时间，在我的测试中，Reaver花了2个半小时破解网络，得出正确密码。正如前文中提到过的，Reaver的文档号称这个时间一般在4到10个小时之间，因此根据实际情况不同，这个时间也会有所变化。当Reaver的破解完成时，它看起来是下图中这个样子：

![](https://dn-linuxcn.qbox.me/data/attachment/album/201312/05/09080108y4djog4vz20g4d.jpg)

**一些要强调的事实**：Reaver在我的测试中工作良好，但是并非所有的路由器都能顺利破解（后文会具体介绍）。并且，你要破解的路由器需要有一个相对较强的信号，否则Reaver很难正常工作，可能会出现其他一些意想不到的问题。整个过程中，Reaver可能有时会出现超时、PIN码死循环等问题。一般我都不管它们，只是保持电脑尽量靠近路由器，Reaver最终会自行处理这些问题。

除此以外，你可以在Reaver运行的任意时候按下Ctrl+C中断工作。这样会退出程序，但是Reaver下次启动的时候会自动恢复继续之前的工作，前提是只要你没有关闭或重启电脑（如果你直接在live DVD里运行，关闭之前的工作都会丢失）。
