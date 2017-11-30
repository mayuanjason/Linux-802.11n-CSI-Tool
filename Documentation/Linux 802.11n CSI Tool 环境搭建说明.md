[toc]
# 1.准备工作
## 1.1. 所需硬件
* 具有 ExpressCard 接口的笔记本电脑
* Intel 5300 无线网卡
* 3 根外置天线
* ExpressCard adapter (to PCIe)

## 1.2. 所需软件
* [Ubuntu 14.04.5 LTS (Trusty Tahr) ***64-bit***](http://releases.ubuntu.com/14.04/) 
* Git
* Matlab 2016b

# 2. 硬件搭建
将 3 根天线连接到 Intel 5300 无线网卡
![](https://github.com/mayuanjason/Linux-802.11n-CSI-Tool/blob/master/image/20171129230535.jpg)

将 Intel 5300 连接到 ExpressCard adapter
![](https://github.com/mayuanjason/Linux-802.11n-CSI-Tool/blob/master/image/20171129230602.jpg)

将 ExpressCard adapter 插入笔记本电脑
![](https://github.com/mayuanjason/Linux-802.11n-CSI-Tool/blob/master/image/20171129230606.jpg)

# 3. 软件搭建
## 3.1. 安装 Ubuntu 14.04 LTS (Trusty Tahr)
下载好镜像 [ubuntu-14.04.5-desktop-amd64](http://releases.ubuntu.com/14.04/)，刻盘，用光驱安装，一路回车，没什么好说的，不再赘述。

>**Tip：**
1. 我选择的是 64 位版本，为什么，因为我的盗版 Matlab 2016b for Linux 只支持 64 位操作系统，这我不能忍啊，必须上 64 位系统。

>2. 从作者的 [GitHub](https://github.com/dhalperi/linux-80211n-csitool/) 来看，作者提供的 Wi-Fi 驱动只能在 Linux 内核版本 3.2 ~ 4.2 之间运行。因此，在安装完 Ubuntu 14.04 后，需要使用命令 `uname -r` 检查一下当前的内核版本是什么。很不幸，Ubuntu 14.04 LTS 64 位系统的默认内核版本是 **4.4** 的。因此，在安装完操作系统后，还需要安装低版本的内核。
 ```
 yuanm@yuanm-ThinkPad-T400:~$ uname -r
 4.4.0-31-generic
 ```
>3. 其实选取哪个 Ubuntu 版本并不重要，重要的是 Linux 内核的版本，但我建议安装 Ubunt 14.04 LTS（Long Term Support，长期支持）版本，这个版本官方会一直支持到2019年4月。

## 3.2. 安装 4.2 版本的内核
> 这里我们选择 4.2.8 版本的内核。

### 3.2.1. 下载 64-bit 4.2.8 Linux header 和 image
 * [linux-headers-4.2.8-040208_4.2.8-040208.201512150620_all.deb](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.2.8-wily/linux-headers-4.2.8-040208_4.2.8-040208.201512150620_all.deb)
 * [linux-headers-4.2.8-040208-generic_4.2.8-040208.201512150620_amd64.deb](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.2.8-wily/linux-headers-4.2.8-040208-generic_4.2.8-040208.201512150620_amd64.deb)
 * [linux-image-4.2.8-040208-generic_4.2.8-040208.201512150620_amd64.deb](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.2.8-wily/linux-image-4.2.8-040208-generic_4.2.8-040208.201512150620_amd64.deb)
 
### 3.2.2. 切换到 root 用户
```
su
```
如果没有添加 **root** 用户，请先添加 **root** 用户：
```
sudo passwd root
```
按照提示输入两次新的密码即可。

### 3.2.3. 安装 Linux header
```
dpkg -i linux-headers-4.2.8-040208_4.2.8-040208.201512150620_all.deb
dpkg -i linux-headers-4.2.8-040208-generic_4.2.8-040208.201512150620_amd64.deb
```

### 3.2.4. 安装 Linux image
```
dpkg -i linux-image-4.2.8-040208-generic_4.2.8-040208.201512150620_amd64.deb
```

安装完后，重启。在启动电脑的时候，注意选取 4.2.8 的内核启动即可。进入系统后，在终端输入 `uname -r` 查看内核版本。
```
yuanm@yuanm-ThinkPad-T400:~$ uname -r
4.2.8-040208-generic
```
可以看到，内核版本已经变成 **4.2.8**。

## 3.3. 安装必要软件
### 3.3.1. 安装编译工具 & 内核头文件 &  版本控制软件Git
```
sudo apt-get install gcc make linux-headers-$(uname -r) git-core
```
### 3.3.2. 安装 iw 命令
```
sudo apt-get install iw
```
> **Tip：**如果不执行如下两行命令，在用 **iw** 命令去连接无线路由器时，会出现问题。别问我为什么，我也不知道，也懒得去找原因。这两条命令只用执行一次即可，重启后，无需再执行。
```
echo iface wlan1 inet manual | sudo tee -a /etc/network/interfaces
echo iface wlan2 inet manual | sudo tee -a /etc/network/interfaces
sudo restart network-manager
```

> **Tip**：如果不想在 Linux 系统启动的时候，自动加载 Wi-Fi 驱动，运行如下两行命令：
运行完后，会在目录 /etc/modprobe.d 下生成一个文件 csitool.conf，可以打开看看这个文件都有什么。
这两条命令只用运行一遍即可，重启后，无需再运行。
***建议运行这两条命令。后面可以看到，在手动 insmod iwlwifi 的时候，还需要指定内核参数，这个内核参数就是用来开关 CSI 信息采集功能的。而系统自动加载时，是不会指定内核参数的。***
```
echo blacklist iwldvm | sudo tee -a /etc/modprobe.d/csitool.conf
echo blacklist iwlwifi | sudo tee -a /etc/modprobe.d/csitool.conf
```

## 3.4. 编译安装 Wi-Fi 驱动
### 3.4.1. 从 GitHub 下载驱动代码
```
mkdir ~/code
cd ~/code
CSITOOL_KERNEL_TAG=csitool-$(uname -r | cut -d . -f 1-2)
git clone https://github.com/dhalperi/linux-80211n-csitool.git
cd linux-80211n-csitool
git checkout ${CSITOOL_KERNEL_TAG}
```
**NOTE，**最后一行很关键，作者的代码有很多版本（branch），最后一行就是用来选择下载哪个 branch 的代码。
有儿童要问了，那应该下载哪个 branch 的代码呢，跟你们的内核版本有关系。
如 3.1 节所述，在我的电脑上，运行 **uname -r | cut -d . -f 1-2**，输出的字符串是 **4.2**。因此，我们下载的就是 **csitool-4.2** 这个 branch。***并且，作者 branch 的编号范围是 csitool-3.2 ~ csitool-4.2，因此我们只能选择 3.2 ~ 4.2 之间的内核版本进行安装***。

### 3.4.2. 编译 Wi-Fi 驱动
```
make -C /lib/modules/$(uname -r)/build M=$(pwd)/drivers/net/wireless/iwlwifi modules
```
编译完成后，会生成 3 个 ko 文件，分别是：
* ~/code/linux-80211n-csitool/drivers/net/wireless/iwlwifi/iwlwifi.ko
* ~/code/linux-80211n-csitool/drivers/net/wireless/iwlwifi/mvm/iwlmvm.ko
* ~/code/linux-80211n-csitool/drivers/net/wireless/iwlwifi/dvm/iwldvm.ko
> **Tip：**如果遇到错误提示 **‘Cannot use CONFIG_CC_STACKPROTECTOR_STRONG: -fstack-protector-strong not supported by compiler’**，是因为 Linux 14.04 LTS 默认安装的是 **gcc-4.8**，而编译选项 **-fstack-protector-strong** 是 **gcc-4.9** 以后的版本才加入的，也就是说需要安装 **gcc-4.9** 及以后的版本才可以编译通过。
```
yuanm@yuanm-ThinkPad-T400:~/code/linux-80211n-csitool$ gcc -v
...
gcc version 4.8.4 (Ubuntu 4.8.4-2ubuntu1~14.04.3) 
```

### 3.4.3. ubuntu 14.04 更新 gcc/g++ 4.9.4 *
>  这节选读，如果在编译驱动时遇到上述错误，才需要升级 gcc/g++。
```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install gcc-4.9 g++-4.9
```
装了ppa，各种版本就可以共存了。不过有一个问题，每次要用的时候，必须使用 **gcc-4.9**，如果直接用 **gcc** 会运行 4.8 版本的，非常麻烦。
所以需要改一下 **/usr/bin/** 下的链接。
```
su
cd /usr/bin/
ln -s /usr/bin/g++-4.9 /usr/bin/g++ -f
ln -s /usr/bin/gcc-4.9 /usr/bin/gcc -f
```
再运行 `gcc -v`，可以看到版本已经变成 4.9 了。
```
root@yuanm-ThinkPad-T400:/usr/bin# gcc -v
...
gcc version 4.9.4 (Ubuntu 4.9.4-2ubuntu1~14.04.1)
```

### 3.4.3. 安装 Wi-Fi 驱动模块（ko）到相应内核的 updates 目录
```
sudo make -C /lib/modules/$(uname -r)/build M=$(pwd)/drivers/net/wireless/iwlwifi INSTALL_MOD_DIR=updates modules_install
sudo depmod
cd ..
```
> **Tip**：如果遇到错误提示 **Can't read private key**，先忽略掉。

运行完第一条命令后，会将 3.4.2 节编译出来的 3 个 ko 文件拷贝到 **/lib/modules/4.2.8-040208-generic/updates** 目录下。
运行完第二条 **depmod** 命令后，会生成新的模块依赖关系，并改写当前内核的 **modules.dep** 文件，在我的电脑上，是 **/lib/modules/4.2.8-040208-generic/modules.dep**。
具体会修改三行内容（**由 depmod 命令自动修改，无需手动修改**）：
去掉三行
~~kernel/drivers/net/wireless/iwlwifi/iwlwifi.ko: kernel/net/wireless/cfg80211.ko
kernel/drivers/net/wireless/iwlwifi/dvm/iwldvm.ko: kernel/drivers/net/wireless/iwlwifi/iwlwifi.ko kernel/net/mac80211/mac80211.ko kernel/net/wireless/cfg80211.ko
kernel/drivers/net/wireless/iwlwifi/mvm/iwlmvm.ko: kernel/drivers/net/wireless/iwlwifi/iwlwifi.ko 
kernel/net/mac80211/mac80211.ko kernel/net/wireless/cfg80211.ko~~
增加三行：
updates/mvm/iwlmvm.ko: updates/iwlwifi.ko kernel/net/mac80211/mac80211.ko kernel/net/wireless/cfg80211.ko
updates/dvm/iwldvm.ko: updates/iwlwifi.ko kernel/net/mac80211/mac80211.ko kernel/net/wireless/cfg80211.ko
updates/iwlwifi.ko: kernel/net/wireless/cfg80211.ko

### 3.4.4. 安装 Firmware
#### 3.4.4.1 下载 CSI Tool supplementary material
```
cd ~/code
git clone https://github.com/dhalperi/linux-80211n-csitool-supplementary.git
```

#### 3.4.4.2. 备份已有的 Intel 5000 系列无线网卡的 firmware 文件
```
for file in /lib/firmware/iwlwifi-5000-*.ucode; do sudo mv $file $file.orig; done
```
上面这条命令的作用，就是把 /lib/firmware 目录下面，所有以**iwlwifi-5000-xxx.ucode**命名的文件，重命名为 **iwlwifi-5000-xxx.ucode.orig**。
例如：把 **iwlwifi-5000-5.ucode** 重命名为 **iwlwifi-5000-5.ucode.orig**。

#### 3.4.4.3. 安装 firmware
```
sudo cp linux-80211n-csitool-supplementary/firmware/iwlwifi-5000-2.ucode.sigcomm2010 /lib/firmware/
sudo ln -s iwlwifi-5000-2.ucode.sigcomm2010 /lib/firmware/iwlwifi-5000-2.ucode
```
与其说是安装，不如说是拷贝，上面第一条命令的意思，就是把 firmware 二进制文件 iwlwifi-5000-2.ucode.sigcomm2010 拷贝到了 /lib/firmware/ 目录。

## 3.5. 编译 Userspace Logging Tool
编译上层应用程序 *log_to_file*。该应用程序从 Wi-Fi 驱动读取 CSI 信息，并写入到文件。
```
make -C linux-80211n-csitool-supplementary/netlink
``` 
编译完成后，会在目录 ~/code/linux-80211n-csitool-supplementary/netlink 生成编译好的二进制文件 **log_to_file**。

# 4. 安装 Matlab 2016b *
> 这章选读，对于有两台电脑的儿童来说（一台 Windows，一台 Linux），大可不必在 Linux 电脑上安装 Matlab，可以用 Windows 电脑通过 SSH 来远程操作 Linux 电脑，抓取 CSI 数据，然后通过 Samba 共享服务，将 CSI 数据拷贝回 Windows 电脑，然后在 Windows 电脑上，用 Matlab 处理数据。
对于只有一台电脑的儿童，也不一定需要安装 Matlab，我觉得用 Python，照样可以分析数据，不过我并没有试过：）
## 4.1. 下载 Matlab 2016b
首先在[百度网盘](https://pan.baidu.com/s/1mi0PRqK#list/path=%2F)下载 Matlab for Linux，下载后文件夹中包含三个文件：**Matlab 2016b Linux64 Crack.rar**，**R2016b_glnxa64_dvd1.iso**，**R2016b_glnxa64_dvd2.iso**。其中，第一个是破解文件。由于整个软件太大，所以分成了两个 iso 文件，意味着安装途中会提示载入新的镜像文件。

## 4.2. 挂在镜像文件
安装前，我把所需（三个）文件都拷贝到了 **/home/yuanm/Downloads/Matlab2016b** 目录。
使用如下命令挂载镜像  **R2016b_glnxa64_dvd1.iso**：
```
cd ~
mkdir matlab
sudo mount -t auto -o loop /home/yuanm/Downloads/Matlab2016b/R2016b_glnxa64_dvd1.iso /home/yuanm/matlab
```
第一行命令是建立一个挂载目录，用于加载 .iso 镜像文件，其实就是起到了虚拟光驱的作用。
第二行命令是将 .iso 文件挂载到指定目录，mount 命令格式如下：
```
mount -t 类型 -o 挂接方式 源路径 目标路径
```
-t 后的类型选择 auto，自动挂载。-o 后的挂载方式为 loop，用来把一个文件当成硬盘分区挂接上系统。

## 4.3. 安装 Matlab
```
sudo ./matlab/install
```
下面提供一些安装步骤截图，大家可以参考：
![Alt text](./S2V4HR~M.PNG)


## 4.4. 激活 Matlab

# 5. 测试
## 4.1. 加载 Wi-Fi 驱动
首先，卸载 Wi-Fi 驱动。当然，如果按照 3.2.2 节所述，在开机时，不自动加载 Wi-Fi 驱动，那这里就不用卸载。
```
sudo modprobe -r iwlwifi mac80211
```
> **Tip**：如果遇到错误提示 **FATAL: Module iwlwifi is in use.**，可能需要先卸载 **iwldvm** 模块，尝试使用下面这条命令：
```
sudo modprobe -r iwldvm iwlwifi mac80211
```
当 Wi-Fi 驱动被卸载后，重新加载 Wi-Fi 驱动，注意这里带了一个内核参数 **connector_log**。
```
sudo modprobe iwlwifi connector_log=0x1
sudo ifconfig wlan2 up
```
> **Tip1：**最开始测试的时候，我发现怎么也抓不到 CSI 数据，后来忽然想到，我的笔记本自带的无线网卡也是 Intel 的。用 `modprobe` 加载的，可能是同名的 ko 文件，而不是我刚刚编译出来的。通过 `find` 命令查看，果然发现有同名的三个 ko 文件，**iwlwifi.ko**，**iwldvm.ko**，**iwlmvm.ko**。
```
yuanm@yuanm-ThinkPad-T400:~$ find / -name iwlwifi.ko
...
/lib/modules/4.2.8-040208-generic/kernel/drivers/net/wireless/iwlwifi/iwlwifi.ko
/lib/modules/4.2.8-040208-generic/updates/iwlwifi.ko
/home/yuanm/code/linux-80211n-csitool/drivers/net/wireless/iwlwifi/iwlwifi.ko
```
> 从上图可以看出，第一行的 **iwlwifi.ko**，是系统原生的，后面两个，是我们刚刚编译出来的。只要将第一行的 ko 文件重命名就好。其它两个 ko 文件，同样用 `find` 命令找到后重命名。
```
cd /lib/modules/4.2.8-040208-generic/kernel/drivers/net/wireless/iwlwifi
sudo mv iwlwifi.ko iwlwifi.ko.ori
cd dvm
sudo mv iwldvm.ko iwldvm.ko.ori
cd ../mvm/
sudo mv iwlmvm.ko iwlmvm.ko.ori
```

**Tip2：**必须要有最后一行。加载 Wi-Fi 驱动后，还需要使能 **wlan2** 接口。
**Tip3：**最后一行的无线网络接口名 **wlan2**，不同的机器，这个名字可能不一样，但都是以 **wlan** 开头，以阿拉伯数字结尾。例如：**wlan0**，**wlan1** 等等。
通过 `ifconfig -a` 命令来查看自己机器的无线网络接口名。
```
yuanm@yuanm-ThinkPad-T400:~$ ifconfig -a
...
wlan1     Link encap:Ethernet  HWaddr 3c:a9:f4:bc:c6:e0  
          BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

wlan2     Link encap:Ethernet  HWaddr 00:21:6a:84:38:72  
          inet addr:192.168.0.106  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::221:6aff:fe84:3872/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1095 errors:0 dropped:0 overruns:0 frame:0
          TX packets:190 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:87615 (87.6 KB)  TX bytes:52032 (52.0 KB)
```
其中，**wlan1** 是我笔记本电脑自带无线网卡的网络接口名，**wlan2** 是 Intel 5300 无线网卡的网络接口名。

## 4.2. 连接无线路由器
由 4.1 节可知，Intel 5300 无线网卡，在我的笔记本电脑上的接口名是 **wlan2**，因此后续的一系列操作，一律操作的是 **wlan2**  接口。
### 4.2.1. 扫描
```
sudo iw dev wlan2 scan | grep SSID
```
这条命令在我的电脑上的操作结果如下：
```
yuanm@yuanm-ThinkPad-T400:~$ sudo iw dev wlan2 scan | grep SSID
[sudo] password for yuanm: 
	SSID: TP-LINK_36CC
	SSID: ChinaNGB-YdBtvU
	SSID: ChinaNet-C6Mk
	SSID: ChinaNet-XwJj
	SSID: ChinaNGB-Yd7khA
```
可以看出，Intel 5300 无线网卡一共扫描到了 5 个无线路由器。其中，SSID 是 Service Set Identifier（服务及标识） 的缩写，可以简单理解，就是无线路由器的名字。

### 4.2.2. 连接
命令格式如下：
```
sudo iw dev wlan2 connect <ssid>
```
这里，我们连接无线路由器 **TP-LINK_36CC**，如果一切顺利的话，这条命令不会有任何输出。
```
sudo iw dev wlan2 connect TP-LINK_36CC
```

那如何查看 Intel 5300 是否已经连接上 **TP-LINK_36CC** 了呢，使用下面这条命令：
```
sudo iw dev wlan2 link
```
这条命令在我电脑上的输出如下：
```
yuanm@yuanm-ThinkPad-T400:~$ sudo iw dev wlan2 link
Connected to f4:83:cd:84:36:cc (on wlan2)
	SSID: TP-LINK_36CC
	freq: 2412
	RX: 2691858 bytes (30005 packets)
	TX: 115230 bytes (657 packets)
	signal: -39 dBm
	tx bitrate: 270.0 MBit/s MCS 15 40Mhz

	bss flags:	short-preamble short-slot-time
	dtim period:	1
	beacon int:	100
```
如果没有连接成功，则会显示 **Not connected.**，例如查看 **wlan1** 网络接口的连接状态：
```
yuanm@yuanm-ThinkPad-T400:~$ sudo iw dev wlan1 link
Not connected.
```
