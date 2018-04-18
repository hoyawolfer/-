#Wireshark
TCP抓包

##下载安装Wireshark

通过官网链接下载, [点击下载](https://1.na.dl.wireshark.org/osx/Wireshark%202.4.6%20Intel%2064.dmg)
	
##初始界面

双击, 打开出现的第一个界面如下:

![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/Screen Shot 2018-04-15 at 5.23.46 PM.png)

##选择网卡

 比如我们要监听手机的流量, 需要通过建立一个映射到iPhone的虚拟网络,在terminal中输入如下命令即可:
 	
 ```
 rvictl -s [udid]
 ``` 
 	
* udid即手机identifer,可通过Xcode看到
 	
操作成功会提示

```
Starting device [udid] [SUCCEEDED] with interface rvi0	
```
执行完命令成功之后Wireshark能立即识别新增加的rvi0网卡,单击 **rvi0**
	
然后在 `capture filter输入框` 输入一些过滤规则,如果不输入任何规则, 会捕获该网卡的所有网络流量, 不利于观察. 比如我们要捕获 `源IP` 或者 `目的IP` 为 `223.112.8.233` 的流量, 可以这样写:
	
```
host 223.112.8.230	
```
输入框底色为绿色代表格式输入正确
	
具体CaptureFilters规则可以查看 <https://wiki.wireshark.org/CaptureFilters>
	
然后双击 `rvi0`
		
如果出现错误弹框如下:
	
![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/Screen Shot 2018-04-15 at 6.01.59 PM.png)
	
如果没有, 请忽略本条, 继续下一个步骤,如果有,执行以下命令
	
```
cd /dev
ls -la | grep bp
sudo chown [用户名]:admin bp* ##用户名可通过 whoami 命令获取到	
```
然后重启Wireshark即可解决这个问题
	
##流量监听

![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/Screen Shot 2018-04-16 at 8.56.01 AM.png)
	
界面主要分为四块
	
###第一部分:
	
`工具栏`, 通过工具栏可以控制监控行为, 从左至右分别为开始抓包,停止抓包,重新抓包,抓包设置等等
	
工具栏的下方有个输入框, 可以让我们手动输入包的显示过滤(display filter)条件, 比如只显示tcp端口号为80的过滤条件为: `tcp.port == 80`具体可查看 <https://wiki.wireshark.org/DisplayFilters>
	
```
note: 
Capture Filter出现在初始界面，在网卡列表的上方有个输入框，允许我们输入capture filter，
一旦输入了特定的capture规则，Wireshark就只捕获符合该规则的流量包了
Display filter出现在流量监控界面，在工具栏的下方有个输入框，允许我们输入display filter，
display filter只是从界面上过滤掉不符合规则的包，Wireshark实际上还是监听了这些包，
一旦去掉display filter，所有的包又会出现在同一界面。
```
###第二部分:
`历史流量展示界面`, 这里展示的是从抓包开始,按照过滤规则,所有通过iPhone设备的流量. 列表界面不同的包有不同的颜色,可在菜单`View-Coloring rules`查看着色规则
![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/Screen Shot 2018-04-16 at 10.02.46 AM.png)
	
* 这里有个小技巧，如上图所示，我只将我感兴趣的协议包上了色，集中在http，tcp，udp包，这样分析起来更加直观。
![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/1534161-f990e7b97d9b0aea.png)
* 比如根据上图的规则，tcp三次握手中的Sync包是使用灰色标记的，这样我就可以在下图的包中迅速定位一次tcp连接的开始包位置：
![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/1534161-f1250d830f90281c.png)
	
###第三部分:
`单个包的详细信息展示板`,我们在第二部分选中的包在这一部分会将其结构以可读的文本形式展示出来, 正确阅读这一部分信息需要对 [TCP/IP协议](https://www.cnblogs.com/onepixel/p/7092302.html) 有一定的掌握.
###第四部分:
`单个包的二进制流信息展示板`,这一部分展示的是包的原始数据, 我们在第三部分选中的协议头, 都会在这一部分以同步高亮的形式标记出来, 这一部分的展示是为了让我们对包的真实内容做直观的判断, 能具体到单个byte
	
##流量跟踪
Wireshark默认情况下将不同网络连接的流量都混在一起展示, 即使给不同协议的包着色之后, 要单独查看某个特定连接的流量依然不怎么方便, 我们可以通过Wireshark提供的两种方式实现这个目标
###方式一: Follow Stream
当我们选中某个包之后, 右键 `Follow - TCP Stream`, Wireshark 支持常见的四种Stream, `TCP UDP HTTP SSL`, 比如我们选中Follow TCP Stream 之后可以得到如下的详细分析输出
![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/Screen Shot 2018-04-16 at 10.49.14 AM.png)
	
```
上图将iPhone和server之间某次的连接流量完整的呈现出来.
还提供了流量编码选择,文本搜索功能等, 如果需要支持中文, 可在上图中选择UTF-8编码方式

```
###方式二: Flow Graph
Flow Graph 可以通过菜单 `Statistics - Flow Graph`来生成, 这样我们可以得到另一种形式的流量呈现
![](/Users/wolfer/Documents/SN Mark/Wireshark抓包/capture/1534161-f7fe8067a08fa739.png)
和Follow Stream不同的是我们获取到的是完整的流量
	
```
Follow Stream更适合分析针对某一个服务器地址的流量, 
而Flow Graph更适合分析某个app的整体网络行为, 包含从DNS解析开始到和对个服务器交互等.

```
其实Statistics菜单下还有更多的图表分析模式, 可以根据不同的分析目标来选择. 比如 `Statistics - HTTP - Requests`可以得到HTTP请求分析图, 和Charles展示结果类似
	
	