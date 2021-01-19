#由32的学习过度到基于arm的嵌入式LInux学习
#首先搭建嵌入式开发环境
/************************************************************************
->下载VMware station
->获取密钥
->下载对应的Linux镜像(Ubuntu16.04)
->在虚拟机中安装下载好的镜像
	->断网安装
		->断网安装在进入系统之后运行下列命令，搭建环境
		->在software里修改软件源为中国的，不然慢死,可以用命令修改，记不住，百度
		->sudo apt-get update
		->sudo apt-get upgrade
		(->这里提醒一点，断网安装没得vmware-tools,需要手动安装~)
			->点开工具栏的虚拟机选项
			->点击安装vmware-tools(这里有可能出现选项为灰色的情况)
			->先关闭虚拟机，虚拟机设置，CD的选项，不是自动连接勾选自动连接，
			    下面的路径选择VMware station自带的vmware-tools或者自己的Linux镜像挂载，启动后发现可以安装了
	->不断网安装
		->直接进入下一步
->安装必要的软件
	->vim
		->sudo apt-get install vim
		->/etc/vim/vimrc:
		set showcmd		" Show (partial) command in status line.
		set showmatch		" Show matching brackets.
		set ignorecase		" Do case insensitive matching
		set smartcase		" Do smart case matching
		set incsearch		" Incremental search
		set autowrite		" Automatically save before commands like :next and :make
		set hidden		" Hide buffers when they are abandoned
		set mouse=a		" Enable mouse usage (all modes)
		set number		"行号"
		set cindent		"C语言缩进"
		->有注释的去掉，没有的加上
	->fcitx
		->sudo apt-get install fcitx
	->sogo
		->url:http://cdn2.ime.sogou.com/dl/index/1571302197/sogoupinyin_2.3.1.0112_amd64.deb?st=2OBL-4EsmKOynsF7fx6W6Q&e=1579255410&fn=sogoupinyin_2.3.1.0112_amd64.deb
		->wget url:<dir>
	->gcc g++
		->略
	->交叉编译链
		->https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/
		->选自己适配的
		->sudo tar -vxf... target
		->把交叉编译工具链的目录下的bin目录添加到/etc/bash.b...里或者profile里或者environment里
		->export PATH=$PATH:<chain dir>(../bin)
		->sudo apt-get install lsb-core lib32stdc++6	//安装相关库
	->网易云
		->url:http://d1.music.126.net/dmusic/netease-cloud-music_1.2.1_amd64_ubuntu_20190428.deb
	->vscode
		->自己下载
		->配置configure
		1)、C/C++，这个肯定是必须的。
		2)、C/C++ Snippets，即 C/C++重用代码块。
		3)、C/C++ Advanced Lint,即 C/C++静态检测 。
		4)、Code Runner，即代码运行。
		5)、Include AutoComplete，即自动头文件包含。
		6)、Rainbow Brackets，彩虹花括号，有助于阅读代码。
		7)、One Dark Pro，VSCode 的主题。
		8)、GBKtoUTF8，将 GBK 转换为 UTF8。
		9)、ARM，即支持 ARM 汇编语法高亮显示。
		10)、Chinese(Simplified)，即中文环境。
		11)、vscode-icons，VSCode 图标插件，主要是资源管理器下各个文件夹的图标。
		12)、compareit，比较插件，可以用于比较两个文件的差异。
		13)、DeviceTree，设备树语法插件。
->必要的服务搭建
	->vsftpd
		->sudo apt-get install vsftpd
		->sudo vi /etc/vsftpd.conf:
			##############
			local_enable=YES
			write_enable=YES
			##############
		->sudo /etc/init.d/vsftpd restart
	->nfs
		->sudo apt-get install nfs-kernel-server rpcbind
		->sudo vi /etc/exports
			/home/xny/Linux/nfs *(rw,sync,root_squash)
		->sudo /etc/init.d/nfs-kernel-server restart
	->ssh
		->sudo apt-get install openssh-server
	->tftp
		->sudo apt-get install tftp-hpa tftpd-hpa
		->mkdir /home/xny/Linux/tftpboot
		->chmod 777 /home/xny/Linux/tftpboot
		->新建文件/etc/xinetd.d/tftp
		server tftp
		{
		socket_type = dgram
		protocol = udp    //tcp也可
		wait = yes
		user = root
		server = /usr/sbin/in.tftpd
		server_args = -s /home/xny/Linux/tftpboot/
		disable = no
		per_source = 11
		cps = 100 2
		flags = IPv4
		}
		->sudo service tftpd-hpa start
		-># /etc/default/tftpd-hpa
			TFTP_USERNAME="tftp"
			TFTP_DIRECTORY="/home/xny/Linux/tftpboot"
			TFTP_ADDRESS=":69"
			TFTP_OPTIONS="-l -c -s"
		->tftp服务文件夹里的文件都要给权限->777!!!
		->sudo service tftpd-hpa restart
->网卡设置
	->/etc/network/interfaces
	auto [interface_name]
	iface [interface_name] inet [static | dhcp | loopback]
	address         /*IP地址*/
	netmask        /*子网掩码*/
	gateway        /*网关*/
	dns-nameserverss /* dns服务器 */
	->/etc/reslove.conf
	->nameserver DNS地址
************************************************************************/
#裸机例程->见项目
#uboot移植
->获取uboot源码
->解压源码,以下操作默认在源码目录下
	->cd configs
	->cp mx6ull_14x14_evk_emmc_defconfig mx6ull_alientek_emmc_defconfig:
##############################################################
示例代码 33.2.1.1 mx6ull_alientek_emmc_defconfig 文件
CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/freescale/mx6ull_alientek_
emmc/imximage.cfg,MX6ULL_EVK_EMMC_REWORK"
CONFIG_ARM=y
CONFIG_ARCH_MX6=y
CONFIG_TARGET_MX6ULL_ALIENTEK_EMMC=y
CONFIG_CMD_GPIO=y
##############################################################
->在 目 录 include/configs 下 添 加 I.MX6ULL-ALPHA 开 发 板 对 应 的 头 文 件 ， 复 制
include/configs/mx6ullevk.h，并重命名为 mx6ull_alientek_emmc.h，命令如下：
cp include/configs/mx6ullevk.h mx6ull_alientek_emmc.h
拷贝完成以后将：
#ifndef __MX6ULLEVK_CONFIG_H
#define __MX6ULLEVK_CONFIG_H
改为：
#ifndef __MX6ULL_ALIENTEK_EMMC_CONFIG_H
#define __MX6ULL_ALIENTEK_EMMC_CONFIG_H
##############################################################
	->cd board/freescale/
	->cp mx6ullevk/ -r mx6ull_alientek_emmc
	->cd mx6ull_alientek_emmc
	->mv mx6ullevk.c mx6ull_alientek_emmc.c
	->修改mx6ull_alientek_emmc目录下的Makefile
##############################################################
示例代码 33.2.3.1 Makefile 文件
1 # (C) Copyright 2015 Freescale Semiconductor, Inc.
2 #
3 # SPDX-License-Identifier: GPL-2.0+
4 #
5
6 obj-y := mx6ull_alientek_emmc.o
7
8 extra-$(CONFIG_USE_PLUGIN) := plugin.bin
9 $(obj)/plugin.bin: $(obj)/plugin.o
10 $(OBJCOPY) -O binary --gap-fill 0xff $< $@
##############################################################
	->修改 mx6ull_alientek_emmc 目录下的 imximage.cfg 文件
PLUGIN board/freescale/mx6ullevk/plugin.bin 0x00907000
改为：
PLUGIN board/freescale/mx6ull_alientek_emmc /plugin.bin 0x00907000
	->修改 mx6ull_alientek_emmc 目录下的 Kconfig 文件
##############################################################
示例代码 33.2.3.2 Kconfig 文件
if TARGET_MX6ULL_ALIENTEK_EMMC /* 或上即可不需要删除原来的TARGET */

config SYS_BOARD
default "mx6ull_alientek_emmc"

config SYS_VENDOR
default "freescale"

config SYS_SOC
default "mx6"

config SYS_CONFIG_NAME
default "mx6ull_alientek_emmc"

endif
##############################################################
	->修改 mx6ull_alientek_emmc 目录下的 MAINTAINERS 文件
MX6ULL_ALIENTEK_EMMC BOARD
M: Peng Fan <peng.fan@nxp.com>
S: Maintained
F: board/freescale/mx6ull_alientek_emmc/
F: include/configs/mx6ull_alientek_emmc.h
##############################################################
	->修改 U-Boot 图形界面配置文件
修改文件
arch/arm/cpu/armv7/mx6/Kconfig(如果用的 I.MX6UL 的话，应该修改 arch/arm/Kconfig 这个文
件)，在 207 行加入如下内容:
示例代码 33.2.4.1 Kconfig 文件:
	config TARGET_MX6ULL_ALIENTEK_EMMC
	bool "Support mx6ull_alientek_emmc"
	select MX6ULL
	select DM
	select DM_THERMAL
在最后一行的 endif 的前一行添加如下内容：
示例代码 33.2.4.2 Kconfig 文件:
	source "board/freescale/mx6ull_alientek_emmc/Kconfig"
##############################################################
	->图形化配置
U-Boot 图形化配置体验
udo apt-get install build-essential
sudo apt-get install libncurses5
sudo apt-get install libncurses5-dev

开启配置
make menuconfig
##############################################################
	->完善uboot需要的基本驱动
LCD：
##############################################################
mx6ull_alientek_emmc.c，找到如下所示内容：
示例代码 33.2.6.1 LCD 驱动参数
struct display_info_t const displays[] = {{
.bus = MX6UL_LCDIF1_BASE_ADDR,
.addr = 0,
.pixfmt = 24,
.detect = NULL,
.enable = do_enable_parallel_lcd,
.mode = {
.name = "TFT43AB",
......
......
......

######################示例LCD代码############################
struct display_info_t const displays[] = {{
.bus = MX6UL_LCDIF1_BASE_ADDR,
.addr = 0,
.pixfmt = 24,
.detect = NULL,
.enable = do_enable_parallel_lcd,
.mode = {
.name = "TFT7016",
.xres = 1024,
.yres = 600,
.pixclock = 19531,
.left_margin = 140, //HBPD
.right_margin = 160, //HFPD
.upper_margin = 20, //VBPD
.lower_margin = 12, //VFBD
.hsync_len = 20, //HSPW
.vsync_len = 3, //VSPW
.sync = 0,
.vmode = FB_VMODE_NONINTERLACED
} } };
##############################################################
打开 mx6ull_alientek_emmc.h，找到所有如下语句：
panel=TFT43AB
将其改为：
panel=TFT7016

如果uboot的logo显示还是不正常
查看panel的值
改为TFT7016
##############################################################
	->网络驱动修改
网络 PHY 地址修改
首先修改 uboot 中的 ENET1 和 ENET2 的 PHY 地址和驱动，打开 mx6ull_alientek_emmc.h
这个文件，找到如下代码：
示例代码 33.2.7.1 网络默认 ID 配置参数
##############################################################
#ifdef CONFIG_CMD_NET
#define CONFIG_CMD_PING
#define CONFIG_CMD_DHCP
#define CONFIG_CMD_MII
#define CONFIG_FEC_MXC
#define CONFIG_MII
#define CONFIG_FEC_ENET_DEV 1

#if (CONFIG_FEC_ENET_DEV == 0)
#define IMX_FEC_BASE ENET_BASE_ADDR
#define CONFIG_FEC_MXC_PHYADDR 0x2
#define CONFIG_FEC_XCV_TYPE RMII
......
......
#####################修改后的代码###############################
#ifdef CONFIG_CMD_NET
#define CONFIG_CMD_PING
#define CONFIG_CMD_DHCP
#define CONFIG_CMD_MII
#define CONFIG_FEC_MXC
#define CONFIG_MII
#define CONFIG_FEC_ENET_DEV 1

#if (CONFIG_FEC_ENET_DEV == 0)
#define IMX_FEC_BASE ENET_BASE_ADDR
#define CONFIG_FEC_MXC_PHYADDR 0x0
#define CONFIG_FEC_XCV_TYPE RMII
#elif (CONFIG_FEC_ENET_DEV == 1)
#define IMX_FEC_BASE ENET2_BASE_ADDR
#define CONFIG_FEC_MXC_PHYADDR 0x1
#define CONFIG_FEC_XCV_TYPE RMII
#endif
#define CONFIG_ETHPRIME "FEC"

#define CONFIG_PHYLIB
#define CONFIG_PHY_SMSC
##############################################################
删除 uboot 中 中 74LV595 的驱动代码
uboot 中网络 PHY 芯片地址修改完成以后就是网络复位引脚的驱动修改了，打开
mx6ull_alientek_emmc.c，找到如下代码：
示例代码 33.2.7.3 74LV595 引脚
##############################################################
#define IOX_SDI IMX_GPIO_NR(5, 10)
#define IOX_STCP IMX_GPIO_NR(5, 7)
#define IOX_SHCP IMX_GPIO_NR(5, 11)
#define IOX_OE IMX_GPIO_NR(5, 8)
##############################################################

的代码删除掉，替换为如下所示代码：
示例代码 33.2.7.4 修改后的网络引脚
##############################################################
#define ENET1_RESET IMX_GPIO_NR(5, 7)
#define ENET2_RESET IMX_GPIO_NR(5, 8)
##############################################################
ENET1 的复位引脚连接到 SNVS_TAMPER7 上，对应 GPIO5_IO07，ENET2 的复位引脚连
接到 SNVS_TAMPER8 上，对应 GPIO5_IO08。
继续在 mx6ull_alientek_emmc.c 中找到如下代码：
示例代码 33.2.7.5 74LV595 引脚配置
##############################################################
static iomux_v3_cfg_t const iox_pads[] = {
/* IOX_SDI */
MX6_PAD_BOOT_MODE0__GPIO5_IO10 | MUX_PAD_CTRL(NO_PAD_CTRL),
/* IOX_SHCP */
MX6_PAD_BOOT_MODE1__GPIO5_IO11 | MUX_PAD_CTRL(NO_PAD_CTRL),
/* IOX_STCP */
MX6_PAD_SNVS_TAMPER7__GPIO5_IO07 | MUX_PAD_CTRL(NO_PAD_CTRL),
/* IOX_nOE */
MX6_PAD_SNVS_TAMPER8__GPIO5_IO08 | MUX_PAD_CTRL(NO_PAD_CTRL),
};
##############################################################
同理，示例代码 33.2.7.5 是 74LV595 的 IO 配置参数结构体，将其删除掉。继续在
mx6ull_alientek_emmc.c 中找到函数 iox74lv_init，如下所示：
示例代码 33.2.7.6 74LV595 初始化函数
##############################################################
static void iox74lv_init(void)
{
	......
	......
}
......
/*
* shift register will be output to pins
*/
gpio_direction_output(IOX_STCP, 1);
};
void iox74lv_set(int index)
{
	......
	......
}
......
/*
* shift register will be output to pins
*/
gpio_direction_output(IOX_STCP, 1);
};
##############################################################
iox74lv_init 函数是 74LV595 的初始化函数，iox74lv_set 函数用于控制 74LV595 的 IO 输出
电平，将这两个函数全部删除掉！
在 mx6ull_alientek_emmc.c 中找到 board_init 函数，此函数是板子初始化函数，会被
board_init_r 调用，board_init 函数内容如下：
示例代码 33.2.7.7 board_init 函数
##############################################################
int board_init(void)
{
......
imx_iomux_v3_setup_multiple_pads(iox_pads, ARRAY_SIZE(iox_pads));
iox74lv_init();
......
return 0;
}
##############################################################
board_init 会调用 imx_iomux_v3_setup_multiple_pads 和 iox74lv_init 这两个函数来初始化
74lv595 的 GPIO，将这两行删除掉。至此，mx6ull_alientek_emmc.c 中关于 74LV595 芯片的驱
动代码都删除掉了，接下来就是添加 I.MX6U-ALPHA 开发板两个网络复位引脚了。
添加 I.MX6U-ALPHA 开发板网络复 位引脚驱动
在 mx6ull_alientek_emmc.c 中找到如下所示代码：
示例代码 33.2.7.8 默认网络 IO 结构体数组
##############################################################
640 static iomux_v3_cfg_t const fec1_pads[] = {
641 MX6_PAD_GPIO1_IO06__ENET1_MDIO | MUX_PAD_CTRL(MDIO_PAD_CTRL),
642 MX6_PAD_GPIO1_IO07__ENET1_MDC | MUX_PAD_CTRL(ENET_PAD_CTRL),
......
649 MX6_PAD_ENET1_RX_ER__ENET1_RX_ER | MUX_PAD_CTRL(ENET_PAD_CTRL),
650 MX6_PAD_ENET1_RX_EN__ENET1_RX_EN | MUX_PAD_CTRL(ENET_PAD_CTRL),
651 };
652
653 static iomux_v3_cfg_t const fec2_pads[] = {
654 MX6_PAD_GPIO1_IO06__ENET2_MDIO | MUX_PAD_CTRL(MDIO_PAD_CTRL),
655 MX6_PAD_GPIO1_IO07__ENET2_MDC | MUX_PAD_CTRL(ENET_PAD_CTRL),
......
664 MX6_PAD_ENET2_RX_EN__ENET2_RX_EN | MUX_PAD_CTRL(ENET_PAD_CTRL),
665 MX6_PAD_ENET2_RX_ER__ENET2_RX_ER | MUX_PAD_CTRL(ENET_PAD_CTRL),
666 };
结构体数组 fec1_pads 和 fec2_pads 是 ENET1 和 ENET2 这两个网口的 IO 配置参数，在这
两个数组中添加两个网口的复位 IO 配置参数，完成以后如下所示：
示例代码 33.2.7.9 添加网络复位 IO 后的结构体数组
##############################################################
640 static iomux_v3_cfg_t const fec1_pads[] = {
641 MX6_PAD_GPIO1_IO06__ENET1_MDIO | MUX_PAD_CTRL(MDIO_PAD_CTRL),
642 MX6_PAD_GPIO1_IO07__ENET1_MDC | MUX_PAD_CTRL(ENET_PAD_CTRL),
......
649 MX6_PAD_ENET1_RX_ER__ENET1_RX_ER | MUX_PAD_CTRL(ENET_PAD_CTRL),
650 MX6_PAD_ENET1_RX_EN__ENET1_RX_EN | MUX_PAD_CTRL(ENET_PAD_CTRL),
651 MX6_PAD_SNVS_TAMPER7__GPIO5_IO07 | MUX_PAD_CTRL(NO_PAD_CTRL),
652 };
653
654 static iomux_v3_cfg_t const fec2_pads[] = {
655 MX6_PAD_GPIO1_IO06__ENET2_MDIO | MUX_PAD_CTRL(MDIO_PAD_CTRL),
656 MX6_PAD_GPIO1_IO07__ENET2_MDC | MUX_PAD_CTRL(ENET_PAD_CTRL),
......
665 MX6_PAD_ENET2_RX_EN__ENET2_RX_EN | MUX_PAD_CTRL(ENET_PAD_CTRL),
666 MX6_PAD_ENET2_RX_ER__ENET2_RX_ER | MUX_PAD_CTRL(ENET_PAD_CTRL),
667 MX6_PAD_SNVS_TAMPER8__GPIO5_IO08 | MUX_PAD_CTRL(NO_PAD_CTRL),
668 };
##############################################################
示例代码 33.2.7.9 中，第 651 行和 667 行分别是 ENET1 和 ENET2 的复位 IO 配置参数。继
续在文件 mx6ull_alientek_emmc.c 中找到函数 setup_iomux_fec，此函数默认代码如下：
示例代码 33.2.7.10 setup_iomux_fec 函数默认代码
##############################################################
668 static void setup_iomux_fec(int fec_id)
669 {
670 if (fec_id == 0)
671 imx_iomux_v3_setup_multiple_pads(fec1_pads,
672 ARRAY_SIZE(fec1_pads));
673 else
674 imx_iomux_v3_setup_multiple_pads(fec2_pads,
675 ARRAY_SIZE(fec2_pads));
676 }
##############################################################
函数 setup_iomux_fec 就是根据 fec1_pads 和 fec2_pads 这两个网络 IO 配置数组来初始化
I.MX6ULL 的网络 IO。我们需要在其中添加网络复位 IO 的初始化代码，并且复位一下 PHY 芯
片，修改后的 setup_iomux_fec 函数如下：
示例代码 33.2.7.11 修改后的 setup_iomux_fec 函数
##############################################################
668 static void setup_iomux_fec(int fec_id)
669 {
670 if (fec_id == 0)
671 {
672
673 imx_iomux_v3_setup_multiple_pads(fec1_pads,
674 ARRAY_SIZE(fec1_pads));
675
676 gpio_direction_output(ENET1_RESET, 1);
677 gpio_set_value(ENET1_RESET, 0);
678 mdelay(20);
679 gpio_set_value(ENET1_RESET, 1);
680 }
681 else
682 {
683 imx_iomux_v3_setup_multiple_pads(fec2_pads,
684 ARRAY_SIZE(fec2_pads));
685 gpio_direction_output(ENET2_RESET, 1);
686 gpio_set_value(ENET2_RESET, 0);
687 mdelay(20);
688 gpio_set_value(ENET2_RESET, 1);
689 }
690 }
##############################################################
示例代码 33.2.7.11 中第 676 行~679 行和第 685 行~688 行分别对应 ENET1 和 ENET2 的复
位 IO 初始化，将这两个 IO 设置为输出并且硬件复位一下 LAN8720A，这个硬件复位很重要！
否则可能导致 uboot 无法识别 LAN8720A。

修改 drivers/net/phy/phy.c 文件中的函数 genphy_update_link
大功基本上告成，还差最后一步，uboot 中的 LAN8720A 驱动有点问题，打开文件
drivers/net/phy/phy.c，找到函数 genphy_update_link，这是个通用 PHY 驱动函数，此函数用于更
新 PHY 的连接状态和速度。使用 LAN8720A 的时候需要在此函数中添加一些代码，修改后的
函数 genphy_update_link 如下所示：
示例代码 33.2.7.12 修改后的 genphy_update_link 函数
##############################################################
221 int genphy_update_link(struct phy_device *phydev)
222 {
223 unsigned int mii_reg;
224
225 #ifdef CONFIG_PHY_SMSC
226 static int lan8720_flag = 0;
227 int bmcr_reg = 0;
228 if (lan8720_flag == 0) {
229 bmcr_reg = phy_read(phydev, MDIO_DEVAD_NONE, MII_BMCR);
230 phy_write(phydev, MDIO_DEVAD_NONE, MII_BMCR, BMCR_RESET);
231 while(phy_read(phydev, MDIO_DEVAD_NONE, MII_BMCR) & 0X8000) {
232 udelay(100);
233 }
234 phy_write(phydev, MDIO_DEVAD_NONE, MII_BMCR, bmcr_reg);
235 lan8720_flag = 1;
236 }
237 #endif
238
239 /*
240 * Wait if the link is up, and autonegotiation is in progress
241 * (ie - we're capable and it's not done)
242 */
243 mii_reg = phy_read(phydev, MDIO_DEVAD_NONE, MII_BMSR);
......
291
292 return 0;
293 }
##############################################################
225 行~237 行就是新添加的代码，为条件编译代码段，只有使用 SMSC 公司的 PHY 这段
代码才会执行(目前只测试了 LAN8720A，SMSC 公司其他的芯片还未测试)。第 229 行读取
LAN8720A 的 BMCR 寄存器(寄存器地址为 0)，此寄存器为 LAN8720A 的配置寄存器，这里先
读取此寄存器的默认值并保存起来。230 行向寄存器 BMCR 寄存器写入 BMCR_RESET(值为
0X8000)，因为 BMCR 的 bit15 是软件复位控制位，因此 230 行就是软件复位 LAN8720A，复位
完成以后此位会自动清零。第 231~233 行等待 LAN8720A 软件复位完成，也就是判断 BMCR
的 bit15 位是否为 1，为 1 的话表示还没有复位完成。第 234 行重新向 BMCR 寄存器写入以前
的值，也就是 229 行读出的那个值。
至此网络的复位引脚驱动修改完成，重新编译 uboot，然后将 u-boot.bin 烧写到 SD 卡中并
启动
##############################################################
个人笔记：
1.vscode目录简化脚本代码:
{
    "search.exclude":{
        "**/node_modules": true,
        "**/bower_components": true,
        "**/*.o":true,
        "**/*.su":true,
        "**/*.cmd":true,
        "arch/arc":true,
        "arch/avr32":true,
        "arch/blackfin":true,
        "arch/m68k":true,
        "arch/microblaze":true,
        "arch/mips":true,
        "arch/nds32":true,
        "arch/nios2":true,
        "arch/openrisc":true,
        "arch/powerpc":true,
        "arch/sandbox":true,
        "arch/sh":true,
        "arch/sparc":true,
        "arch/x86":true,
        "arch/arm/mach*":true,
        "arch/arm/cpu/arm11*":true,
        "arch/arm/cpu/arm720t":true,
        "arch/arm/cpu/arm9*":true,
        "arch/arm/cpu/armv7m":true,
        "arch/arm/cpu/armv8":true,
        "arch/arm/cpu/pxa":true,
        "arch/arm/cpu/sa1100":true,
        "board/[a-e]*":true,
        "board/[g-z]*":true,
        "board/[0-9]*":true,
        "board/[A-Z]*":true,
        "board/fir*":true,
        "board/freescale/b*":true,
        "board/freescale/l*":true,
        "board/freescale/m5*":true,
        "board/freescale/mp*":true,
        "board/freescale/c29*":true,
        "board/freescale/cor*":true,
        "board/freescale/mx7*":true,
        "board/freescale/mx2*":true,
        "board/freescale/mx3*":true,
        "board/freescale/mx5*":true,
        "board/freescale/p*":true,
    },
    "files.exclude":{
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "**/*.o":true,
        "**/*.su":true,
        "**/*.cmd":true,
        "arch/arc":true,
        "arch/avr32":true,
        "arch/blackfin":true,
        "arch/m68k":true,
        "arch/microblaze":true,
        "arch/mips":true,
        "arch/nds32":true,
        "arch/nios2":true,
        "arch/openrisc":true,
        "arch/powerpc":true,
        "arch/sandbox":true,
        "arch/sh":true,
        "arch/sparc":true,
        "arch/x86":true,
        "arch/arm/mach*":true,
        "arch/arm/cpu/arm11*":true,
        "arch/arm/cpu/arm720t":true,
        "arch/arm/cpu/arm9*":true,
        "arch/arm/cpu/armv7m":true,
        "arch/arm/cpu/armv8":true,
        "arch/arm/cpu/pxa":true,
        "arch/arm/cpu/sa1100":true,
        "board/[a-e]*":true,
        "board/[g-z]*":true,
        "board/[0-9]*":true,
        "board/[A-Z]*":true,
        "board/fir*":true,
        "board/freescale/b*":true,
        "board/freescale/l*":true,
        "board/freescale/m5*":true,
        "board/freescale/mp*":true,
        "board/freescale/c29*":true,
        "board/freescale/cor*":true,
        "board/freescale/mx7*":true,
        "board/freescale/mx2*":true,
        "board/freescale/mx3*":true,
        "board/freescale/mx5*":true,
        "board/freescale/p*":true,
    }
}
2.修改网络驱动中的nfs，不然可能会出现nfs下载掉包
/home/xny/ARM-Linux/UBT-UNX/nxp-uboot/uboot-imx-rel_imx_4.1.15_2.1.0_ga/net/nfs.c
找到
#ifndef CONFIG_NFS_TIMEOUT
# define NFS_TIMEOUT 2000UL
改为
#ifndef CONFIG_NFS_TIMEOUT
# define NFS_TIMEOUT 30*2000UL
3.nfs下载成功，可是挂载依然有问题
修改Uboot中的挂载命令
setenv myset_ios 'setenv bootargs 'console=ttymxc0,115200 root=/dev/nfs rw  nfsroot=<主机IP>:<主机nfs服务的目录>,proto=tcp,nfsvers=3,nolock <client ip>:<host ip>:<netmask>::<interface>:off''
ex:
setenv bootargs console=ttymxc0,115200 root=/dev/nfs rw nfsroot=192.168.0.111:/home/xny/ARM-Linux/nfs/arm-ubuntu,proto=tcp,nfsvers=3,nolock \
 ip=192.168.0.88:192.168.0.111:192.168.0.1:255.255.255.0::eth0:off

#Linux内核移植
vscode的目录简化脚本
{
    "search.exclude":{
            "**/node_modules": true,
            "**/bower_components": true,
             "**/*.o":true,
             "**/*.su":true,
             "**/*.cmd":true,
             "Documentation":true,
            
             /* 屏蔽不用的架构相关的文件 */
             "arch/alpha":true,
             "arch/arc":true,
             "arch/arm64":true,
             "arch/avr32":true,
             "arch/[b-z]*":true,
             "arch/arm/plat*":true,
             "arch/arm/mach-[a-h]*":true,
             "arch/arm/mach-[n-z]*":true,
             "arch/arm/mach-i[n-z]*":true,
             "arch/arm/mach-m[e-v]*":true,
             "arch/arm/mach-k*":true,
             "arch/arm/mach-l*":true,
            
             /* 屏蔽排除不用的配置文件 */
             "arch/arm/configs/[a-h]*":true,
             "arch/arm/configs/[j-z]*":true,
             "arch/arm/configs/imo*":true,
             "arch/arm/configs/in*":true,
             "arch/arm/configs/io*":true,
             "arch/arm/configs/ix*":true,

           /* 屏蔽掉不用的 DTB 文件 */
             "arch/arm/boot/dts/[a-h]*":true,
             "arch/arm/boot/dts/[k-z]*":true,
             "arch/arm/boot/dts/in*":true,
             "arch/arm/boot/dts/imx1*":true,
             "arch/arm/boot/dts/imx7*":true,
             "arch/arm/boot/dts/imx2*":true,
             "arch/arm/boot/dts/imx3*":true,
             "arch/arm/boot/dts/imx5*":true,
             "arch/arm/boot/dts/imx6d*":true,
             "arch/arm/boot/dts/imx6q*":true,
             "arch/arm/boot/dts/imx6s*":true,
             "arch/arm/boot/dts/imx6ul-*":true,
             "arch/arm/boot/dts/imx6ull-9x9*":true,
             "arch/arm/boot/dts/imx6ull-14x14-ddr*":true,
    },

    "files.exclude":{
        "**/.git": true,
        "**/.svn": true,
        "**/.hg": true,
        "**/CVS": true,
        "**/.DS_Store": true,
        "**/*.o":true,
        "**/*.su":true,
        "**/*.cmd":true,
        "Documentation":true,
       
        /* 屏蔽不用的架构相关的文件 */
        "arch/alpha":true,
        "arch/arc":true,
        "arch/arm64":true,
        "arch/avr32":true,
        "arch/[b-z]*":true,
        "arch/arm/plat*":true,
        "arch/arm/mach-[a-h]*":true,
        "arch/arm/mach-[n-z]*":true,
        "arch/arm/mach-i[n-z]*":true,
        "arch/arm/mach-m[e-v]*":true,
        "arch/arm/mach-k*":true,
        "arch/arm/mach-l*":true,
       
        /* 屏蔽排除不用的配置文件 */
        "arch/arm/configs/[a-h]*":true,
        "arch/arm/configs/[j-z]*":true,
        "arch/arm/configs/imo*":true,
        "arch/arm/configs/in*":true,
        "arch/arm/configs/io*":true,
        "arch/arm/configs/ix*":true,

      /* 屏蔽掉不用的 DTB 文件 */
        "arch/arm/boot/dts/[a-h]*":true,
        "arch/arm/boot/dts/[k-z]*":true,
        "arch/arm/boot/dts/in*":true,
        "arch/arm/boot/dts/imx1*":true,
        "arch/arm/boot/dts/imx7*":true,
        "arch/arm/boot/dts/imx2*":true,
        "arch/arm/boot/dts/imx3*":true,
        "arch/arm/boot/dts/imx5*":true,
        "arch/arm/boot/dts/imx6d*":true,
        "arch/arm/boot/dts/imx6q*":true,
        "arch/arm/boot/dts/imx6s*":true,
        "arch/arm/boot/dts/imx6ul-*":true,
        "arch/arm/boot/dts/imx6ull-9x9*":true,
        "arch/arm/boot/dts/imx6ull-14x14-ddr*":true,

        /* 不需要的*.tmp*/
        "arch/arm/boot/dts/*.tmp":true,
    }
}
设备树中的LCD节点
&lcdif {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_lcdif_dat
		     &pinctrl_lcdif_ctrl
		     &pinctrl_lcdif_reset>;
	display = <&display0>;
	status = "okay";

	display0: display {
		bits-per-pixel = <16>;
		bus-width = <24>;

		display-timings {
			native-mode = <&timing0>;
			timing0: timing0 {
			clock-frequency = <51000000>;
			hactive = <1024>;
			vactive = <600>;
			hfront-porch = <160>;
			hback-porch = <140>;
			hsync-len = <20>;
			vback-porch = <20>;
			vfront-porch = <12>;
			vsync-len = <3>;

			hsync-active = <0>;
			vsync-active = <0>;
			de-active = <1>;
			pixelclk-active = <0>;
			};
		};
	};
};

pinctrl_lcdif_dat: lcdifdatgrp {
			fsl,pins = <
				MX6UL_PAD_LCD_DATA00__LCDIF_DATA00  0x49	//原本电器属性是0x79但影响以太网卡的驱动,全部改为0x49
				MX6UL_PAD_LCD_DATA01__LCDIF_DATA01  0x49
				MX6UL_PAD_LCD_DATA02__LCDIF_DATA02  0x49
				MX6UL_PAD_LCD_DATA03__LCDIF_DATA03  0x49
				MX6UL_PAD_LCD_DATA04__LCDIF_DATA04  0x49
				MX6UL_PAD_LCD_DATA05__LCDIF_DATA05  0x49
				MX6UL_PAD_LCD_DATA06__LCDIF_DATA06  0x49
				MX6UL_PAD_LCD_DATA07__LCDIF_DATA07  0x49
				MX6UL_PAD_LCD_DATA08__LCDIF_DATA08  0x49
				MX6UL_PAD_LCD_DATA09__LCDIF_DATA09  0x49
				MX6UL_PAD_LCD_DATA10__LCDIF_DATA10  0x49
				MX6UL_PAD_LCD_DATA11__LCDIF_DATA11  0x49
				MX6UL_PAD_LCD_DATA12__LCDIF_DATA12  0x49
				MX6UL_PAD_LCD_DATA13__LCDIF_DATA13  0x49
				MX6UL_PAD_LCD_DATA14__LCDIF_DATA14  0x49
				MX6UL_PAD_LCD_DATA15__LCDIF_DATA15  0x49
				MX6UL_PAD_LCD_DATA16__LCDIF_DATA16  0x49
				MX6UL_PAD_LCD_DATA17__LCDIF_DATA17  0x49
				MX6UL_PAD_LCD_DATA18__LCDIF_DATA18  0x49
				MX6UL_PAD_LCD_DATA19__LCDIF_DATA19  0x49
				MX6UL_PAD_LCD_DATA20__LCDIF_DATA20  0x49
				MX6UL_PAD_LCD_DATA21__LCDIF_DATA21  0x49
				MX6UL_PAD_LCD_DATA22__LCDIF_DATA22  0x49
				MX6UL_PAD_LCD_DATA23__LCDIF_DATA23  0x49
			>;
		};
		pinctrl_lcdif_ctrl: lcdifctrlgrp {
			fsl,pins = <
				MX6UL_PAD_LCD_CLK__LCDIF_CLK	    0x49
				MX6UL_PAD_LCD_ENABLE__LCDIF_ENABLE  0x49
				MX6UL_PAD_LCD_HSYNC__LCDIF_HSYNC    0x49
				MX6UL_PAD_LCD_VSYNC__LCDIF_VSYNC    0x49
			>;
		};
设备树中的网卡节点
&fec1 {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_enet1>;
	phy-mode = "rmii";
	phy-handle = <&ethphy0>;
	phy-reset-gpios = <&gpio5 7 GPIO_ACTIVE_LOW>;
	phy-reset-duration = <200>;
	status = "okay";
};

&fec2 {
	pinctrl-names = "default";
	pinctrl-0 = <&pinctrl_enet2>;
	phy-mode = "rmii";
	phy-handle = <&ethphy1>;
	phy-reset-gpios = <&gpio5 7 GPIO_ACTIVE_LOW>;
	phy-reset-duration = <200>;
	status = "okay";

	mdio {
		#address-cells = <1>;
		#size-cells = <0>;

		ethphy0: ethernet-phy@0 {
			compatible = "ethernet-phy-ieee802.3-c22";
			reg = <0>;
		};

		ethphy1: ethernet-phy@1 {
			compatible = "ethernet-phy-ieee802.3-c22";
			reg = <1>;
		};
	};
};

pinctrl_enet1: enet1grp {
			fsl,pins = <
				MX6UL_PAD_ENET1_RX_EN__ENET1_RX_EN	0x1b0b0
				MX6UL_PAD_ENET1_RX_ER__ENET1_RX_ER	0x1b0b0
				/*
				MX6UL_PAD_ENET1_RX_DATA0__ENET1_RDATA00	0x1b0b0
				MX6UL_PAD_ENET1_RX_DATA1__ENET1_RDATA01	0x1b0b0
				MX6UL_PAD_ENET1_TX_EN__ENET1_TX_EN	0x1b0b0
				MX6UL_PAD_ENET1_TX_DATA0__ENET1_TDATA00	0x1b0b0
				MX6UL_PAD_ENET1_TX_DATA1__ENET1_TDATA01	0x1b0b0
				*/
				/*MX6UL_PAD_ENET1_TX_CLK__ENET1_REF_CLK1	0x4001b031*/
				MX6UL_PAD_ENET1_TX_CLK__ENET1_REF_CLK1	0x4001b009
				MX6ULL_PAD_SNVS_TAMPER7__GPIO5_IO07 0x10b0
			>;
		};

		pinctrl_enet2: enet2grp {
			fsl,pins = <
				MX6UL_PAD_GPIO1_IO07__ENET2_MDC		0x1b0b0
				MX6UL_PAD_GPIO1_IO06__ENET2_MDIO	0x1b0b0
				/*
				MX6UL_PAD_ENET2_RX_EN__ENET2_RX_EN	0x1b0b0
				MX6UL_PAD_ENET2_RX_ER__ENET2_RX_ER	0x1b0b0
				MX6UL_PAD_ENET2_RX_DATA0__ENET2_RDATA00	0x1b0b0
				MX6UL_PAD_ENET2_RX_DATA1__ENET2_RDATA01	0x1b0b0
				MX6UL_PAD_ENET2_TX_EN__ENET2_TX_EN	0x1b0b0
				MX6UL_PAD_ENET2_TX_DATA0__ENET2_TDATA00	0x1b0b0
				MX6UL_PAD_ENET2_TX_DATA1__ENET2_TDATA01	0x1b0b0
				*/
				/*MX6UL_PAD_ENET2_TX_CLK__ENET2_REF_CLK2	0x4001b031*/
				MX6UL_PAD_ENET2_TX_CLK__ENET2_REF_CLK2	0x4001b009
				MX6ULL_PAD_SNVS_TAMPER8__GPIO5_IO08 0x10b0
			>;
		};
(注：需要在设备树里把网卡IO相关的其他节点的引用注释掉)
#定制自己的跟文件系统
->安装工具qemu
sudo apt-get install qemu-user-static
cd /home/zuozhongkai/linux/nfs/ubuntu_rootfs //进入到 ubuntu_rootfs 目录下
sudo cp /etc/resolv.conf ./etc/resolv.conf //拷贝 resolv.conf
设置软件源，打开根文件系统中的 ubuntu_rootfs/etc/apt/sources.list 文件，在此文件最后面
添加软件源，比如国内常用的清华源、中科大源等等，这些软件源可以直接在网上查找，直接
搜索“ubuntu 软件源”即可，这里我们添加如下所示软件源：
示例代码 A3.2.3.1 各个软件源
#清华源
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
#阿里源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
#中科大源
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
#163 源
deb http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
示例代码 A3.2.3.1 分别给出了，清华源、阿里源、中科大源和 163 源，大家任选几个源添
加到 sources.list 文件中即可。

示例代码 A3.2.4.1 mount.sh 脚本文件内容
#!/bin/bash
echo "MOUNTING"
sudo mount -t proc /proc
/home/zuozhongkai/linux/nfs/ubuntu_rootfs/proc
sudo mount -t sysfs /sys
/home/zuozhongkai/linux/nfs/ubuntu_rootfs/sys
sudo mount -o bind /dev /home/zuozhongkai/linux/nfs/ubuntu_rootfs/dev
sudo mount -o bind /dev/pts
/home/zuozhongkai/linux/nfs/ubuntu_rootfs/dev/pts
sudo chroot /home/zuozhongkai/linux/nfs/ubuntu_rootfs

示例代码 A3.2.4.2 unmount.sh 脚本文件内容
#!/bin/bash
echo "UNMOUNTING"
sudo umount /home/zuozhongkai/linux/nfs/ubuntu_rootfs/proc
sudo umount /home/zuozhongkai/linux/nfs/ubuntu_rootfs/sys
sudo umount /home/zuozhongkai/linux/nfs/ubuntu_rootfs/dev
sudo umount /home/zuozhongkai/linux/nfs/ubuntu_rootfs/dev/pts


安装常用的命令和软件
由于 ubuntu base 是一个最小根文件系统，很多命令和软件都没有，因此我们需要先安装一
下常用的命令和软件，输入如下命令：
apt update
apt install sudo
apt install vim
apt install kmod
apt install net-tools
apt install ethtool
apt install ifupdown
apt install language-pack-en-base
apt install rsyslog
apt install htop
apt install iputils-ping
我们就先安装这些命令和软件，保证 ubuntu base 根文件系统能够在开发板上正常启动即
可，等启动以后再根据实际情况继续安装其他的命令和软件

3 、设置 root 用户密码
设置一下 root 用户的密码，这里我设置简单一点，root 用户密码也设置为“root”，相当于
用户名和密码一样，命令如下：
passwd root //设置 root 用户密码

4 、设置本机名称和 IP 地 地址 址
输入如下命令设置本机名称和 IP 地址：
echo "alientek_imx6ul" > /etc/hostname
echo "127.0.0.1 localhost" >> /etc/hosts
echo "127.0.0.1 alientek_imx6ul" >> /etc/hosts


5 、设置串口终端
ubuntu 根文件系统在开发板上启动以后我们通常也希望串口终端正常工作，这里根据网友
的介绍需要创建一个链接。首先确定自己所使用的串口设备文件，比如正点原子的 ALPHA 开
发 板 使 用 的 UART1 对 应 的 串 口 设 备 文 件 为 ttymxc0 ， 我 们 需 要 添 加 一 个 名 为
getty@ttymxc0.service 的链接，链接到 getty@.service 服务上，输入如下命令：
ln -s /lib/systemd/system/getty@.service /etc/systemd/system/getty.target.wants/getty@ttymxc0.
service
设置好以后就可以退出根文件系统了，输入如下命令退出：
exit
退出以后再执行一下 unmount.sh 脚本取消挂载，命令如下：
./unmount.sh
至此，ubuntu base 根文件系统就已经制作好了，接下来就是挂载到开发板上去测试。

############################################################
#接下来就是各种服务，各种工具的移植了
1>TSLIB的移植
安装必要的库和环境（PC机执行）
#sudo apt-get install autoconf
#sudo apt-get install libtool
#sudo apt-get install automake

  ./autogen.sh
  ./configure --host=arm-linux ac_cv_func_malloc_0_nonnull=yes CC=arm-none-linux-gnueabi-gcc CXX=arm-none-linux-gnueabi-g++ -prefix=/usr/local/tslib
  make
  sudo make install
如果编译过程中遇到 undefined reference to 'rpl_malloc'，前面配置完成之后修改 config.h.in 文件，注释掉文件最后的 #undef malloc ，重新 make 即可。

更改 tslib 配置文件
  cd /usr/local/tslib/etc/
  sudo vi ts.conf 
  去掉# module_raw input 前面的 “#” 和空格
将制作好的 tslib 移动到我们制作的文件系统
  cd /usr/local
  sudo tar zcvf tslib.tar.gz tslib
  mkdir -p /work/my_rootfs/usr/local
  cp tslib.tar.gz /work/my_rootfs/usr/local
  tar zxvf tslib.tar.gz 
  rm tslib.tar.gz 
添加 tslib 环境变量
  vi /work/my_rootfs/etc/profile

  #!/bin/sh
  export T_ROOT=/usr/local/tslib
  export LD_LIBRARY_PATH=/usr/local/tslib/lib:$LD_LIBRARY_PATH
  export TSLIB_CONSOLEDEVICE=none
  export TSLIB_FBDEVICE=/dev/fb0
  export TSLIB_TSDEVICE=/dev/input/event0
  export TSLIB_PLUGINDIR=$T_ROOT/lib/ts
  export TSLIB_CONFFILE=$T_ROOT/etc/ts.conf
  export POINTERCAL_FILE=/etc/pointercal
  export TSLIB_CALIBFILE=/etc/pointercal
  此时，tslib 就已经移植好了
############################################################
2>QT5.32的移植
	->下载QT源码，百度
	->解压
	->以下操作都在源码目录下
	->cd qtbase
	->cd mkspecs
	->cd linux-arm-gnueabi
	->(sudo)gedit(vi) qmake.conf
#
# qmake configuration for building with arm-linux-gnueabi-g++
#

MAKEFILE_GENERATOR      = UNIX
CONFIG                 += incremental
QMAKE_INCREMENTAL_STYLE = sublib

include(../common/linux.conf)
include(../common/gcc-base-unix.conf)
include(../common/g++-unix.conf)

#displayer
QT_QPA_DEFAULT_PLATFORM = linuxfb 
#versions
QMAKE_CFLAGS_RELEASE += -O2 -march=armv7-a
QMAKE_CXXFLAGS_RELEASE += -O2 -march=armv7-a

# modifications to g++.conf
QMAKE_CC                = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc
QMAKE_CXX               = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-g++
QMAKE_LINK              = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-g++
QMAKE_LINK_SHLIB        = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-g++

# modifications to linux.conf
QMAKE_AR                = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-ar cqs
QMAKE_OBJCOPY           = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-objcopy
QMAKE_NM                = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-nm -P
QMAKE_STRIP             = /home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin/arm-linux-gnueabihf-strip
load(qt_config)



针对于 2440 增加：
  QT_QPA_DEFAULT_PLATFORM = linuxfb
  QMAKE_CFLAGS += -msoft-float -D__GCC_FLOAT_NOT_NEEDED -march=armv4t -mtune=arm920t
  QMAKE_CXXFLAGS += -msoft-float -D__GCC_FLOAT_NOT_NEEDED -march=armv4t -mtune=arm920t
  march 指的 cpu 架构，针对 2440 来说是 armv4t
  mtune 指的 cpu 名字，针对 2440 来说是 arm920t
  如果你是 A8 的板子 ，可以参考下边的配置
  QT_QPA_DEFAULT_PLATFORM = linuxfb
  QMAKE_CFLAGS  += -msoft-float -D__GCC_FLOAT_NOT_NEEDED -march=armv7-a -mtune=cortex-a8
  QMAKE_CXXFLAGS += -msoft-float -D__GCC_FLOAT_NOT_NEEDED -march=armv7-a -mtune=cortex-a8
  如果你是 A9 的板子 ，可以参考下边的配置
  QT_QPA_DEFAULT_PLATFORM = linuxfb 
  QMAKE_CFLAGS  += -msoft-float -D__GCC_FLOAT_NOT_NEEDED -march=armv7-a -mtune=cortex-a9
  QMAKE_CXXFLAGS += -msoft-float -D__GCC_FLOAT_NOT_NEEDED -march=armv7-a -mtune=cortex-a9

将以下部分
  # modifications to g++.conf
  QMAKE_CC = arm-linux-gnueabi-gcc
  QMAKE_CXX = arm-linux-gnueabi-g++
  QMAKE_LINK = arm-linux-gnueabi-g++
  QMAKE_LINK_SHLIB = arm-linux-gnueabi-g++

  # modifications to linux.conf
  QMAKE_AR = arm-linux-gnueabi-ar cqs
  QMAKE_OBJCOPY = arm-linux-gnueabi-objcopy
  QMAKE_NM = arm-linux-gnueabi-nm -P
  QMAKE_STRIP = arm-linux-gnueabi-strip
修改为：-lts 是指在链接时链接 tslib 库
  # modifications to g++.conf
  QMAKE_CC = arm-none-linux-gnueabi-gcc -lts
  QMAKE_CXX = arm-none-linux-gnueabi-g++ -lts
  QMAKE_LINK = arm-none-linux-gnueabi-g++ -lts
  QMAKE_LINK_SHLIB = arm-none-linux-gnueabi-g++ -lts

# modifications to linux.conf
  QMAKE_AR = arm-none-linux-gnueabi-ar cqs 
  QMAKE_OBJCOPY = arm-none-linux-gnueabi-objcopy 
  QMAKE_NM = arm-none-linux-gnueabi-nm -P
  QMAKE_STRIP = arm-none-linux-gnueabi-strip 


配置编译
  ./configure  -prefix <output dir>
  -opensource \
  -release \
  -confirm-license \
  -xplatform linux-arm-gnueabi-g++ \
  -shared \
  -qt-zlib \
  -no-gif \
  -qt-libjpeg \
  -no-nis \
  -no-opengl \
  -no-cups \
  -no-glib \
  -no-dbus \
  -no-rpath \
  -no-sse2 -no-sse3 -no-ssse3 -no-sse4.1 -no-sse4.2 \
  -no-avx  \
  -no-openssl \
  -nomake tools \
  -qreal float \
  -qt-libpng \
  -tslib \
  -nomake examples \
  -I /usr/local/tslib/include \
  -L /usr/local/tslib/lib
这里不要用脚本~~~

  make -j4
  sudo make install

如果使用的不是韦老大的虚拟机编译过程中可能报关于libxcb的错误，查看 qtbase/src/plugins/platforms/xcb 底下的 readme 安装相应的库就可以了。

开发板环境变量(自己看情况修改)
export QTEDIR=/usr/local/Qt5.6
  export LD_LIBRARY_PATH=/usr/local/Qt5.6/lib:$LD_LIBRARY_PATH
  export QT_QPA_GENERIC_PLUGINS=tslib
  export QT_QPA_FONTDIR=$QTEDIR/lib/fonts 
  export QT_QPA_PLATFORM_PLUGIN_PATH=$QTEDIR/plugins 
  export QT_QPA_PLATFORM=linuxfb:fb=/dev/fb0:size=1024x600:tty=/dev/tty1

  export QT_QPA_FB_TSLIB=1
############################################################
#OPENCV的移植
->下载OPENCV的源码(官网)
->安装CMAKE-QT_GUI工具
->在源码目录下新建OUTPUT和BUILD文件
->cd build
->sudo cmake-gui
->网上教程一堆，自己配置
->make
->make install
->查错改错
############################################################
linux下gcc默认搜索头文件及库文件的路径
1. 头文件
gcc在编译时如何去寻找所需要的头文件：

头文件的搜索会从-I指定的目录开始；
然后搜索gcc的环境变量 C_INCLUDE_PATH，CPLUS_INCLUDE_PATH，OBJC_INCLUDE_PATH 设置的目录；
再搜索系统目录 /usr/include 和 /usr/local/include（centos7中该目录下是空的）；
最后搜索gcc的一系列自带目录（如/usr/include/c++/4.8.5）。
2. 库文件
编译的时候：

gcc会先搜索-L指定的目录；
再搜索gcc的环境变量LIBRARY_PATH；
再搜索系统目录：/lib和/lib64、/usr/lib 和/usr/lib64、/usr/local/lib和/usr/local/lib64，这是当初compile gcc时写在程序内的。
3. 运行时动态库的搜索路径
动态库的搜索路径搜索的先后顺序是：

编译目标代码时指定的动态库搜索路径；
环境变量LD_LIBRARY_PATH指定的动态库搜索路径；
配置文件/etc/ld.so.conf中指定的动态库搜索路径；
默认的动态库搜索路径/lib；
默认的动态库搜索路径/usr/lib。
#######################搭建UBUNTU下arm交叉编译OPENCV环境####################################
/etc/bash.bashrc
#搜索库的路径
export LIBRARY_PATH=/home/xny/opencv/opencv/lib
#头文件搜索路径
export CPLUS_INCLUDE_PATH=/home/xny/opencv/opencv/include:
/home/xny/opencv/opencv/include/opencv2:
/home/xny/opencv/opencv/include/opencv2/calib3d:
/home/xny/opencv/opencv/include/opencv2/core:
/home/xny/opencv/opencv/include/opencv2/dnn:
/home/xny/opencv/opencv/include/opencv2/features2d:
/home/xny/opencv/opencv/include/opencv2/flann:
/home/xny/opencv/opencv/include/opencv2/gapi:
/home/xny/opencv/opencv/include/opencv2/highgui:
/home/xny/opencv/opencv/include/opencv2/ml:
/home/xny/opencv/opencv/include/opencv2/objdetect:
/home/xny/opencv/opencv/include/opencv2/photo:
/home/xny/opencv/opencv/include/opencv2/stitching:
/home/xny/opencv/opencv/include/opencv2/video:
/home/xny/opencv/opencv/include/opencv2/videoio:
#动态库链接路径
export LIBRARY=/home/xny/opencv/opencv/lib

创建/etc/ld.so.conf.d/opencv.conf
/home/xny/opencv/opencv/lib

->sudo ldconfig

ex:
main.cpp
#include "core.hpp"
#include "highgui.hpp"
#include "imgcodecs.hpp"
#include "imgproc.hpp"
#include "video.hpp"
#include "stdio.h"
#include "stdlib.h"
#include "iostream"

using namespace std;
using namespace cv;
int main(void)
{
    Mat img;
    img = imread("./1.jpg");
    imshow("myphoto",img);
    waitKey(100);
    return 0;
}

Makefile:
PJ_NAME = opencv-env
OBJ = test-opencv.o
$(PJ_NAME) : $(OBJ)
	arm-linux-gnueabihf-g++ $^ -o $@ -std=c++11 -L/home/xny/opencv/opencv/lib -lopencv_highgui -lopencv_imgcodecs -lopencv_core -lopencv_imgproc -lopencv_video -Wl,-rpath-link=/home/xny/opencv/opencv/lib
												

%.o : %.cpp
	arm-linux-gnueabihf-g++ -c $< -o $@ -std=c++11 -L/home/xny/opencv/opencv/lib -lopencv_highgui -lopencv_imgcodecs -lopencv_core -lopencv_imgproc -lopencv_video -Wl,-rpath-link=/home/xny/opencv/opencv/lib
												

clean:
	rm ./*.o $(PJ_NAME)

#OPENCV-Windows10环境搭建
(1)Downloads VS2015
urls:
VS2015 专业版下载链接
http://download.microsoft.com/download/B/8/9/B898E46E-CBAE-4045-A8E2-2D33DD36F3C4/vs2015.pro_chs.iso

VS2015 企业版下载链
http://download.microsoft.com/download/B/8/F/B8F1470D-2396-4E7A-83F5-AC09154EB925/vs2015.ent_chs.iso

VS2015 社区版下载链接
http://download.microsoft.com/download/B/4/8/B4870509-05CB-447C-878F-2F80E4CB464C/vs2015.com_chs.iso

(2)install VS2015

(3)配置VS2015(见附录，图)

(4)下载opencv-windows-pack
url::https://blog.csdn.net/omodao1/article/details/80276834

(5)unpack

(6)编写测试程序

#OPENCV-Linux-环境搭建

(1)下载OPENCV源码

(2)解压源码

(3)cmake-gui(配置见附录，图)

(4)用gcc编译源码

(5)根据得到的源码配置环境变量

->如下::
#env-value
export my_compile_path=/home/xny/ARM-Linux/Tools/cross_compile/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabihf/bin

export LIBRARY_PATH=/home/xny/opencv/opencv/lib:/home/xny/Downloads/qt-everywhere-opensource-src-5.3.2/output/lib:/home/xny/Downloads/opencv-3.3.0-sources/out/lib

export CPLUS_INCLUDE_PATH=/home/xny/opencv/opencv/include:/home/xny/opencv/opencv/include/opencv2:/home/xny/opencv/opencv/include/opencv2/calib3d:/home/xny/opencv/opencv/include/opencv2/core:/home/xny/opencv/opencv/include/opencv2/dnn:/home/xny/opencv/opencv/include/opencv2/features2d:/home/xny/opencv/opencv/include/opencv2/flann:/home/xny/opencv/opencv/include/opencv2/gapi:/home/xny/opencv/opencv/include/opencv2/highgui:/home/xny/opencv/opencv/include/opencv2/ml:/home/xny/opencv/opencv/include/opencv2/objdetect:/home/xny/opencv/opencv/include/opencv2/photo:/home/xny/opencv/opencv/include/opencv2/stitching:/home/xny/opencv/opencv/include/opencv2/video:/home/xny/opencv/opencv/include/opencv2/videoio:/home/xny/Downloads/qt-everywhere-opensource-src-5.3.2/output/include

export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/home/xny/Downloads/opencv-3.3.0-sources/out/include:
/home/xny/Downloads/opencv-3.3.0-sources/out/include/opencv:/home/xny/Downloads/opencv-3.3.0-sources/out/include/opencv2

export LD_LIBRARY_PATH=/home/xny/Downloads/opencv-3.3.0-sources/out/lib

export LIBRARY=/home/xny/opencv/opencv/lib:/home/xny/Downloads/qt-everywhere-opensource-src-5.3.2/output/lib:/home/xny/Downloads/opencv-3.3.0-sources/out

###################################################################
V4L2内核支持
进入内核目录
make menuconfig
选择多媒体devices
使能v4l usb（UVC）
###################################################################
开机自启动

法一：/etc/rc.local 
...
...
<your shell>(abs route)
exit 0
chmod u+x rc.local

法二：
mv <your shell> /etc/init.d/
update-rc.d <your shell> defaults num(biger -> low)
update-rc.d -f <your shell> remove 

##################################################################
qemu-for-arm 的基本使用

[step1:]apt-get install qemu qemu-user-static
[step2:]在官网获取linux内核，uboot源码，最小根文件系统
[step3:]用交叉编译工具编译源码,内核得到对应板子的dtb和zImage,uboot得到u-boot 和 u-boot.bin 这些二进制可执行文件(ARM架构)
[step4:]配置最小文件系统
	(1) 两个脚本
	-> mt.sh
	#!/bin/bash
	echo "MOUNTING"
	sudo mount -t proc  /proc     /home/ts/Linux/nfs/proc
	sudo mount -t sysfs /sys      /home/ts/Linux//nfs/sys
	sudo mount -o bind  /dev      /home/ts/Linux/nfs/dev
	sudo mount -o bind  /dev/pts  /home/ts/Linux/nfs/dev/pts
	sudo chroot 	              /home/ts/Linux/nfs/
	-> umt.sh
	#!/bin/bash
	echo "UNMOUNTING"
	sudo umount /home/ts/Linux/nfs/proc
	sudo umount /home/ts/Linux/nfs/sys
	sudo umount /home/ts/Linux/nfs/dev/pts
	sudo umount /home/ts/Linux/nfs/dev
	(2) 把这两个脚本放在根文件目录下
	(3) 之前我们安装了 qemu-user-static ,去到 /usr/bin/ 目录下 把 qemu-arm-static 复制到根文件系统的 /usr/bin 下
	(4) 再配置一下apt工具的软件源 和 上网设置(DNS):
	-> 将主机下的 /etc/reslove.conf 文件替换掉 最小文件系统下的 /etc/reslove.conf
	-> 再在最小文件系统的 /etc/apt/resource.list 文件中添加下列开源站点其中的几个就行了:
	#清华源
	deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
	deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
	deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
	deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
	deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
	deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
	deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
	deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
	deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
	deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
	#阿里源
	deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
	deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
	deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
	deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
	deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
	deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
	deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
	deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
	deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
	deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
	#中科大源
	deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
	deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
	deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
	deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
	deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
	deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
	deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
	deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
	deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
	deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
	#163 源
	deb http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
	deb http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
	deb http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
	deb http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
	deb http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
	deb-src http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
	deb-src http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse
	deb-src http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse
	deb-src http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse
	deb-src http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
	(5) 挂载文件系统(ARM)，执行mt.sh即可
	(6) 然后安装一些必要的工具，让Ubuntu文件系统能在开发板上正常启动即可:
	apt update
	apt install sudo
	apt install vim
	apt install kmod
	apt install net-tools
	apt install ethtool
	apt install ifupdown
	apt install language-pack-en-base
	apt install rsyslog
	apt install htop
	apt install iputils-ping	
	(7) passwd root && adduser yourname
	(8) 主机配置 
	echo "PC" > /etc/hostname
	echo "127.0.0.1 localhost" >> /etc/hosts
	echo "127.0.0.1 PC" >> /etc/hosts
	(9) 设置串行终端
	ln -s /lib/systemd/system/getty@.service /etc/systemd/system/getty.target.wants/getty@ttymxc0.service
	(不一定是 ttymxc0 可以查看内核目录下的 .config 获取默认输出的串口)
	(10) exit && umt.sh
[step5:]配置qemu的arm运行环境
	(1) 先准备搭建网桥的工具 
	apt-get install bridge-utils        # 虚拟网桥工具
	apt-get install uml-utilities       # UML（User-mode linux）工具
	(2) 修改网卡的配置文件 /etc/network/interface (注意：ens38 是另一张主机网卡，不是用来上网的闲置的网卡，虚拟机中可以直接添加虚拟网卡)
	# interfaces(5) file used by ifup(8) and ifdown(8)
	auto lo
	iface lo inet loopback

	auto ens33
	iface ens33 inet static
	address 192.168.0.105
	netmask 255.255.255.0
	gateway 192.168.0.1
	dns-nameservers 114.114.114.114 192.168.0.1

	#br0设置成静态ip方便自己调试，ip地址可以看下ens38自动获取时的网段，需要设置在同一网段，否则会无法使用

	auto br0
	iface br0 inet static
	address 192.168.0.188
	netmask 255.255.255.0
	#iface br0 inet dhcp

	bridge_ports ens38

	# The tap0 network interface(s)

	#供qemu的u-boot使用的server地址，这这里绑定到了br0上，所以需要设置为和br0同一网段的ip
	auto tap0
	iface tap0 inet manual
	iface tap0 inet static
	address 192.168.0.190
	netmask 255.255.255.0
	pre-up tunctl -t tap0 -u root    # 创建一个tap0接口，只允许root用户访问
	pre-up ifconfig tap0 0.0.0.0 promisc up      # 打开tap0接口
	post-up brctl addif br0 tap0    # 在虚拟网桥中增加一个tap0接口
	(3) /etc/init.d/networking restart
	(4) 配置完了的ifconfig 和 ip addr 信息
ifconfig:
	br0       Link encap:Ethernet  HWaddr 00:0c:29:b0:a2:95  
          inet addr:192.168.0.188  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:feb0:a295/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:89985 errors:0 dropped:0 overruns:0 frame:0
          TX packets:621 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:21322123 (21.3 MB)  TX bytes:75575 (75.5 KB)

	...

	ens38     Link encap:Ethernet  HWaddr 00:0c:29:b0:a2:95  
		  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
		  RX packets:370614 errors:0 dropped:0 overruns:0 frame:0
		  TX packets:57772 errors:0 dropped:0 overruns:0 carrier:0
		  collisions:0 txqueuelen:1000 
		  RX bytes:292827583 (292.8 MB)  TX bytes:3682745 (3.6 MB)

	...

	tap0      Link encap:Ethernet  HWaddr 46:67:61:78:d1:6c  
		  inet addr:192.168.0.190  Bcast:192.168.0.255  Mask:255.255.255.0
		  inet6 addr: fe80::4467:61ff:fe78:d16c/64 Scope:Link
		  UP BROADCAST PROMISC MULTICAST  MTU:1500  Metric:1
		  RX packets:57151 errors:0 dropped:0 overruns:0 frame:0
		  TX packets:113984 errors:0 dropped:0 overruns:0 carrier:0
		  collisions:0 txqueuelen:1000 
		  RX bytes:2859320 (2.8 MB)  TX bytes:99262267 (99.2 MB) 
ip addr:
	1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
	    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
	    inet 127.0.0.1/8 scope host lo
	       valid_lft forever preferred_lft forever
	    inet6 ::1/128 scope host 
	       valid_lft forever preferred_lft forever
	2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
	    link/ether 00:0c:29:b0:a2:8b brd ff:ff:ff:ff:ff:ff
	    inet 192.168.0.105/24 brd 192.168.0.255 scope global ens33
	       valid_lft forever preferred_lft forever
	    inet6 fe80::20c:29ff:feb0:a28b/64 scope link 
	       valid_lft forever preferred_lft forever
	3: ens38: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master br0 state UP group default qlen 1000
	    link/ether 00:0c:29:b0:a2:95 brd ff:ff:ff:ff:ff:ff
	4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
	    link/ether 00:0c:29:b0:a2:95 brd ff:ff:ff:ff:ff:ff
	    inet 192.168.0.188/24 brd 192.168.0.255 scope global br0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::20c:29ff:feb0:a295/64 scope link 
	       valid_lft forever preferred_lft forever
	5: tap0: <NO-CARRIER,BROADCAST,MULTICAST,PROMISC,UP> mtu 1500 qdisc pfifo_fast master br0 state DOWN group default qlen 1000
	    link/ether 46:67:61:78:d1:6c brd ff:ff:ff:ff:ff:ff
	    inet 192.168.0.190/24 brd 192.168.0.255 scope global tap0
	       valid_lft forever preferred_lft forever
	    inet6 fe80::4467:61ff:fe78:d16c/64 scope link 
	       valid_lft forever preferred_lft forever
[step6:]将根文件系统转化为ext4的img文件
	dd if=/dev/zero of=./rootfs.img bs=<n>M count=N #disk-rom = bs*count	(给大一点，先给1024M吧)
	mkfs.ext4 ./rootfs.img
	mount -o loop ./rootfs.img /mnt/tmpfs/
	sudo cp -raf <rootfs-dir> /mnt/tmpfs/
	umount /mnt/tmpfs
[step7:]配置uboot的环境变量(一劳永逸): <uboot>/include/configs/*.h (对应开发板的配置文件)
	(范例)  ...
		...
	#define CONFIG_EXTRA_ENV_SETTINGS \
			CONFIG_PLATFORM_ENV_SETTINGS \
		        BOOTENV \
			"console=ttyAMA0,38400n8\0" \
			"dram=1024M\0" \
			"root=/dev/sda1 rw\0" \
			"mtd=armflash:1M@0x800000(uboot),7M@0x1000000(kernel)," \
				"24M@0x2000000(initrd)\0" \
			"flashargs=setenv bootargs root=${root} console=${console} " \
				"mem=${dram} mtdparts=${mtd} mmci.fmax=190000 " \
				"devtmpfs.mount=0  vmalloc=256M\0" \
			"bootflash=run flashargs; " \
				"cp ${ramdisk_addr} ${ramdisk_addr_r} ${maxramdisk}; " \
				"bootm ${kernel_addr} ${ramdisk_addr_r}\0" \
			"bootargs=console=ttyAMA0,38400 root=/dev/mmcblk0 rw earlycon ignore_loglevel rootfstype=ext4\0" \
			"myboot=tftpboot 60003000 zImage; tftpboot 60500000 vexpress-v2p-ca9.dtb; bootz 60003000 - 60500000;\0" \
			"nfsargs=setenv bootargs console=ttyAMA0,38400 root=/dev/nfs rw nfsroot=192.168.0.105:/home/ts/Linux/nfs,proto=tcp,nfsvers=3,nolock 		ip=192.168.0.189:192.168.0.105:192.168.0.1:255.255.255.0\0"
		...
		...
		/* net config && server ip config */
		#define CONFIG_IPADDR 192.168.0.189
		#define CONFIG_NETMASK 255.255.255.0
		#define CONFIG_GATEWAYIP 192.168.0.1
		#define CONFIG_SERVERIP 192.168.0.105
[step8:]编写对应的脚本启动ARM虚拟机: (参考脚本)
	#!/bin/bash
	sudo qemu-system-arm -M vexpress-a9 -m 128 -kernel uboot/u-boot -net nic -net tap,ifname=tap0,script=no,downscript=no -nographic -sd fs/rootfs.img
#################################################################################################################################


	
			
	

































