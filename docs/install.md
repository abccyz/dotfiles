

# 我的工作站系统安装



### 简述



这是一台ThinkPad W520工作站

工作目标环境主要是日常的一些集群实验，和blender摸鱼，还有一些文档要排版以及原型图设计

配置如下表

| CPU       | 内存      | 硬盘     | 显卡   | 价格  |
| --------- | --------- | -------- | ------ | ----- |
| I7 2820QM | 32G(8G*4) | Ssd 480G | Q1000M | 2700¥ |

### 系统

| wm     | wm启动方式 | 管理服务 | 版本维护 |
| ------ | ---------- | -------- | -------- |
| i3gpas | Startx     | openRC   | 该项目   |



当前的设备信息

```sh
00:00.0 Host bridge: Intel Corporation 2nd Generation Core Processor Family DRAM Controller (rev 09)
00:01.0 PCI bridge: Intel Corporation Xeon E3-1200/2nd Generation Core Processor Family PCI Express Root Port (rev 09)
00:02.0 VGA compatible controller: Intel Corporation 2nd Generation Core Processor Family Integrated Graphics Controller (rev 09)
00:16.0 Communication controller: Intel Corporation 6 Series/C200 Series Chipset Family MEI Controller #1 (rev 04)
00:16.3 Serial controller: Intel Corporation 6 Series/C200 Series Chipset Family KT Controller (rev 04)
00:19.0 Ethernet controller: Intel Corporation 82579LM Gigabit Network Connection (Lewisville) (rev 04)
00:1a.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #2 (rev 04)
00:1b.0 Audio device: Intel Corporation 6 Series/C200 Series Chipset Family High Definition Audio Controller (rev 04)
00:1c.0 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 1 (rev b4)
00:1c.1 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 2 (rev b4)
00:1c.3 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 4 (rev b4)
00:1c.4 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 5 (rev b4)
00:1c.6 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 7 (rev b4)
00:1d.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #1 (rev 04)
00:1f.0 ISA bridge: Intel Corporation QM67 Express Chipset LPC Controller (rev 04)
00:1f.2 SATA controller: Intel Corporation 6 Series/C200 Series Chipset Family 6 port Mobile SATA AHCI Controller (rev 04)
00:1f.3 SMBus: Intel Corporation 6 Series/C200 Series Chipset Family SMBus Controller (rev 04)
01:00.0 VGA compatible controller: NVIDIA Corporation GF108GLM [Quadro 1000M] (rev a1)
03:00.0 Network controller: Intel Corporation Centrino Advanced-N 6205 [Taylor Peak] (rev 34)
0d:00.0 System peripheral: Ricoh Co Ltd PCIe SDXC/MMC Host Controller (rev 08)
0d:00.3 FireWire (IEEE 1394): Ricoh Co Ltd R5C832 PCIe IEEE 1394 Controller (rev 04)
0e:00.0 USB controller: NEC Corporation uPD720200 USB 3.0 Host Controller (rev 04)
```



国内可以选择用清华大学源去下载镜像

```sh
wget -c https://mirrors.tuna.tsinghua.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64/install-amd64-minimal-20190217T214503Z.iso
```



然后使用`dd`命令去刻录

```sh
sudo dd if=gentoo.iso of=/dev/your_usb bs=10M 
```

等待刻录完成之后就可以插入机器重启进入bios 选择第一启动项为你的u盘或是光盘



### 进入livecd

进入livecd之后为了方便操作我们要配置一下网络和ssh

这里我这个就直接插网线连网络了，因为gentoo的livecd连接Wi-Fi不是很方便不过也可以使用arch Linux的livecd去安装gentoo

测试网络是否通畅

```sh
ping -c3 google.com# 国内就ping 百度
```

启动sshd

```sh
/etc/init.d/sshd start
```

设置livecd的root密码

```sh
echo root |passwd --stdin root
```

这样root的密码就被设为root了就可以看一下ip然后使用ssh过去了

### 分区

这个系统是支持UEFI的，可以考虑使用UEFI

| 挂载点    | 物理分区  | 大小 | 文件系统 |
| --------- | --------- | ---- | -------- |
| /boot     | /dev/sda1 | 200M | Ext2     |
| /boot/efi | /dev/sda2 | 200M | Fat32    |
| /         | /dev/sda3 | 60G  | Ext4     |
| /data/    | /dev/sda4 | 100G | Ext4     |
| /portage  | /dev/sda5 | 60G  | Ext4     |
| /data/kvm | /dev/sda6 | all  | Ext4     |



分区命令

```sh
fdisk -l# 查看分区
gfdisk /dev/sda

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          411647   200.0 MiB   8300  Linux filesystem
   2          411648          821247   200.0 MiB   8300  Linux filesystem
   3          821248       126650367   60.0 GiB    8300  Linux filesystem
   4       126650368       336365567   100.0 GiB   8300  Linux filesystem
   5       336365568       462194687   60.0 GiB    8300  Linux filesystem
   6       462194688       937703054   226.7 GiB   8300  Linux filesystem

```

> 初始化分区

```sh
mkfs.ext2 /dev/sda1
mkfs.fat -F32 /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
mkfs.ext4 /dev/sda5
mkfs.ext4 /dev/sda6

```

> 挂载分区

```sh
 mount /dev/sda3  /mnt/gentoo/
 mkdir /mnt/gentoo/boot
 mount /dev/sda1 /mnt/gentoo/boot/
 mkdir /mnt/gentoo/boot/efi
 mount /dev/sda2 /mnt/gentoo/boot/efi/
 mkdir /mnt/gentoo/data
 mount /dev/sda4 /mnt/gentoo/data/
 mkdir /mnt/gentoo/portage
 mount /dev/sda5 /mnt/gentoo/portage/
 mkdir /mnt/gentoo/data/kvm
 mount /dev/sda6 /mnt/gentoo/data/kvm/
```



### stage3

下载stage3文件

```sh
wget -c https://mirrors.tuna.tsinghua.edu.cn/gentoo/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-20190217T214503Z.tar.xz
```

解压

```sh
tar -xfv stage3-amd64-20190217T214503Z.tar.xz
```

### 系统配置



> make.conf

这里的CPU是`i7 2820QM`其他的你可以参阅[这里](https://www.funtoo.org/Subarches)



```sh
vi /mnt/gentoo/etc/portage/make.conf
```

这个只是安装时候的`make.conf`完整的请参照该项目对应目录的配置文件

```sh
# GCC
CHOST="x86_64-pc-linux-gnu"
CFLAGS="-march=sandybridge -O2 -pipe"
CXXFLAGS="${CFLAGS}"
MAKEOPTS="-j9"
CPU_FLAGS_X86="aes avx mmx mmxext popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"


# PORTAGE
PORTDIR="/portage"
DISTDIR="/portage/distfiles"
PKGDIR="/portage/packages"
PORTAGE_TMPDIR="/portage/portage_tmp"
GENTOO_MIRRORS="http://mirrors.tuna.tsinghua.edu.cn/gentoo/"

# USE
SUPPORT="pulseaudio mtp git"
DESKTOP="infinality emoji cjk"
FUCK="-bindist -grub -plymouth -systemd consolekit -modemmanager -gnome-shell -gnome -gnome-keyring -nautilus -modules"
ELSE="client icu sudo python"
USE="${SUPPORT} ${DESKTOP} ${FUCK} ${ELSE}"
# LICENSED
ACCEPT_KEYWORDS="~amd64"
ACCEPT_LICENSE="*"

# LANGUAGE
L10N="en-US zh-CN en zh"
LINGUAS="en_US zh_CN en zh"

# VIDEO_CARDS
VIDEO_CARDS="intel i965"
# ELSE
LLVM_TARGETS="X86"
```



> Portage-mirrors

```bash
mkdir /mnt/gentoo/etc/portage/repos.conf
cat > /mnt/gentoo/etc/portage/repos.conf/gentoo.conf << EOF
[gentoo]
location = /portage
sync-type = rsync
sync-uri = rsync://mirrors.tuna.tsinghua.edu.cn/gentoo-portage/
#sync-uri = rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage/
auto-sync = yes
EOF
```

> 进入chroot环境

```sh
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
mount -t proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
mount /dev/sda1 /boot
```

> 第一次更新

由于更改了`portage`的位置了需要先使用rsync去同步

```sh
emerge --sync
```

更新快照Portage

```sh
emerge-webrsync 
```



> 配置文件选择

```sh
eselect profile list# 列出profile
```

如果你使用systemd则需要选上带有systemd字样的选项

如果你不使用systemd则不建议使用GNOME桌面,因为GNOME桌面依赖systemd(辣鸡)

```sh
eselect profile set 12
```



> 更新系统



```sh
emerge -auvDN --with-bdeps=y @world
```

如果碰到未满足的xxx或者其它提示:

```
emerge -auvDN --with-bdeps=y --autounmask-write @world
etc-update # 然后输入-3就能更新配置,确保再次运行时没有可更新的文件
emerge -auvDN --with-bdeps=y @world
```

等它跑完了,先别急
运行下这几个命令:

```bash
emerge @preserved-rebuild
perl-cleaner --all
emerge -auvDN --with-bdeps=y @world
```

确定没有更新之后再继续，否则查看输出尝试重复运行

如果你在`emerge -auvDN --with-bdeps=y @world`时提示带有`bindist`字样且你已启用`ACCEPT_KEYWORDS="~amd64"`的话
运行如下命令之后再次重试：

```bash
cd /usr/portage/dev-libs/openssl/
ebuild openssl-1.0.2o-r6.ebuild merge # 这里openssl的版本可能和你的不一样，运行ls命令查看可用版本，替换为
```

> 系统的基本配置

```sh
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
echo "zh_CN.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
eselect locale list
eselect locale set "en_US.utf8" #命令行一定要用英文！！中文可能乱码。
env-update && source /etc/profile && export PS1="(chroot) $PS1"

```



> 固件

```sh
emerge -av sys-kernel/linux-firmware
```



> fstab

就是这么的懒～

```sh
wget https://raw.githubusercontent.com/YangMame/Gentoo-Installer/master/genfstab
chmod +x genfstab
cp genfstab /usr/bin/
genfstab -U  / > /etc/fstab
more /etc/fstab #最好检查下此文件,删掉无用挂载点
```

如果你使用非ext4文件系统则在编译内核前需要另外安装相应的工具:

```
btrfs: emerge sys-fs/btrfs-progs
xfs: emerge sys-fs/xfsprogs
jfs: emerge sys-fs/jfsutils
```

> 系统扩展

```sh
emerge app-admin/sysklogd sys-process/cronie sudo layman vim
```



> 用户权限

```sh
useradd -m -G users,wheel,portage,usb,video,audio chaos #chaos是我的用户名称你可以设置你的
```



> 网络配置

懒人必备NetworkManager了解一下？

```sh
emerge -av networkmanager
rc-update add NetworkManager default
```



> EFI

- UEFI

```sh
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Gentoo
grub-mkconfig -o /boot/grub/grub.cfg
```

如果出现`No space left on device`

请运行：

```bash
mount -t efivarfs efivarfs /sys/firmware/efi/efivars
rm /sys/firmware/efi/efivars/dump-*
```



### WM配置



> 软件列表

| 包名称                 | 版本  | 作用               | 备注 |
| ---------------------- | ----- | ------------------ | ---- |
| i3gaps                 | 4.16  | wm                 |      |
| feh                    | 3.1.2 | 设置壁纸，图片查看 |      |
| xorg-server            |       | X                  |      |
| xorg-xinit             |       | Xinit              |      |
| xorg-xwininfo          |       |                    |      |
| i3status               |       | 状态               |      |
| ttf-ubuntu-font-family |       | 字体               |      |
| ttf-font-awesome       |       | 字体               |      |
| compton                |       | 透明设置           |      |
| i3blocks               |       | 底部显示           |      |
| fcitx-qt5              |       | fcitx              |      |
| Emacs                  |       | 我的编辑器         |      |
| fcitx                  |       | 输入法框架         |      |
| fcitx-rime             |       | rime输入法         |      |
| llpp                   |       | Pdf阅读器          |      |
| nm-applet              |       | 网络状态显示       |      |
| Rofi                   |       | 快速启动器         |      |
| xorg-xdpyinfo          |       |                    |      |
| flameshot              |       | 截图工具           |      |
| x11-apps/xbacklight    |       | 背光               |      |
| xorg-xdpyinfo          |       |                    |      |



### 显卡配置



~因为上古显卡选择屏蔽之不打算用了并且这个显卡驱动确实不好搞~~

~所以在bios里面禁用独显只用核心显卡然后安装显卡驱动~~

更新portage到最新然后安装并配置



```sh
emerge --sync
```

修改mask `/etc/portage/package.mask`

```sh
>=x11-drivers/nvidia-drivers-391.0.0 #增加这条
```

修改make.conf `/etc/portage/make.conf`

```sh
VIDEO_CARDS=”intel i965 nvidia”
```

更新

```sh
emerge -avuDN @world
```

添加如下内容到`/etc/portage/package.accept_keywords`：

```sh
=sys-power/bbswitch-9999 **
=x11-misc/bumblebee-9999 **
=x11-misc/primus-0.2 ~amd64
```



安装，内核部分的配置可以参照此repo里面的内核配置可以直接使用

```sh
emerge -av x11-drivers/xf86-video-intel  x11-drivers/nvidia-drivers bbswitch primus bumblebee
```

>  用户部分

```sh
usermod -aG video chaos && usermod -aG bumblebee chaos # chaos为我的用户名你可以更改为你的
```



编辑`/etc/init.d/bumblebee` 删除以下内容：

```sh
depend() {
need xdm
need vgl
after sshd
} # 可能稍有不同 但放心的删掉depend(){…}就行了 因为根本不需要
```



编辑`/etc/bumblebee/bumblebee.conf` 修改为以下参数：

```
KeepUnusedXServer=false
Driver=nvidia
Bridge=primus
VGLTransport=rgb (如果你使用vgl则修改此参数)
KernelDriver=nvidia
PMMethod=bbswitch
```

开机服务：

```bash
rc-update add bumblebee default
```

> 运行demo



```sh
 optirun glxgears
```

查看状态

```sh
cat /proc/acpi/bbswitch
0000:01:00.0 ON
```

这里就ok了



### 输入法配置

这次用的输入法框架是fcitx

先要安装中文支持

```sh
emerge wqy-zenhei
```

再安装rime输入法

```sh
emerge -av fcitx fcitx-qt5 fcitx-configtool fcitx-rime
```

需要在`.xprofile`或者是`.xinitrc`中添加如下

```sh
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```



### 多媒体

视频播放我喜欢用mpv

```sh
emerge -av mpv
```



### 键盘映射配置

```sh
emerge -av x11-drivers/xf86-input-keyboard x11-drivers/xf86-input-libinput x11-drivers/xf86-input-mouse
```



键盘映射修改文件

```sh
vim /usr/share/X11/xkb/symbols/pc
```

我这里喜欢用emacs所以呢修改一下`CapsLock`和`Control`位置如下配置所示

```c
default  partial alphanumeric_keys modifier_keys
xkb_symbols "pc105" {

    key <ESC>  {	[ Escape		]	};

    // The extra key on many European keyboards:
    key <LSGT> {	[ less, greater, bar, brokenbar ] };

    // The following keys are common to all layouts.
    key <BKSL> {	[ backslash,	bar	]	};
    key <SPCE> {	[ 	 space		]	};

    include "srvr_ctrl(fkey2vt)"
    include "pc(editing)"
    include "keypad(x11)"

    key <BKSP> {	[ BackSpace, BackSpace	]	};

    key  <TAB> {	[ Tab,	ISO_Left_Tab	]	};
    key <RTRN> {	[ Return		]	};

    key <CAPS> {	[ Control_L		]	};
    key <NMLK> {	[ Num_Lock 		]	};

    key <LFSH> {	[ Shift_L		]	};
    key <LCTL> {	[ Caps_Lock		]	};
    key <LWIN> {	[ Super_L		]	};

    key <RTSH> {	[ Shift_R		]	};
    key <RCTL> {	[ Control_R		]	};
    key <RWIN> {	[ Super_R		]	};
    key <MENU> {	[ Menu			]	};

    // Beginning of modifier mappings.
    modifier_map Shift  { Shift_L, Shift_R };
    modifier_map Lock   { Caps_Lock };
    modifier_map Control{ Control_L  };
    modifier_map Mod2   { Num_Lock };
    modifier_map Mod4   { Super_L, Super_R };

    // Fake keys for virtual<->real modifiers mapping:
    key <LVL3> {	[ ISO_Level3_Shift	]	};
    key <MDSW> {	[ Mode_switch 		]	};
    modifier_map Mod5   { <LVL3>, <MDSW> };

    key <ALT>  {	[ NoSymbol, Alt_L	]	};
    include "altwin(meta_alt)"

    key <META> {	[ NoSymbol, Meta_L	]	};
    modifier_map Mod1   { <META> };

    key <SUPR> {	[ NoSymbol, Super_L	]	};
    modifier_map Mod4   { <SUPR> };

    key <HYPR> {	[ NoSymbol, Hyper_L	]	};
    modifier_map Mod4   { <HYPR> };
    // End of modifier mappings.

    key <OUTP> { [ XF86Display ] };
    key <KITG> { [ XF86KbdLightOnOff ] };
    key <KIDN> { [ XF86KbdBrightnessDown ] };
    key <KIUP> { [ XF86KbdBrightnessUp ] };
};

hidden partial alphanumeric_keys
xkb_symbols "editing" {
    key <PRSC> {
	type= "PC_ALT_LEVEL2",
	symbols[Group1]= [ Print, Sys_Req ]
    };
    key <SCLK> {	[  Scroll_Lock		]	};
    key <PAUS> {
	type= "PC_CONTROL_LEVEL2",
	symbols[Group1]= [ Pause, Break ]
    };
    key  <INS> {	[  Insert		]	};
    key <HOME> {	[  Home			]	};
    key <PGUP> {	[  Prior		]	};
    key <DELE> {	[  Delete		]	};
    key  <END> {	[  End			]	};
    key <PGDN> {	[  Next			]	};

    key   <UP> {	[  Up			]	};
    key <LEFT> {	[  Left			]	};
    key <DOWN> {	[  Down			]	};
    key <RGHT> {	[  Right		]	};
};

```



### 缓存配置



> 安装



```sh
emerge --ask dev-util/ccache
```

> 初始化

```sh
mkdir -p /data/ccache
chown root:portage /data/ccache
chmod 2775 /data/ccache
```

> 配置 file:**/data/ccache/ccache.conf**

```sh
# Maximum cache size to maintain
max_size = 60.0G

# Allow others to run 'ebuild' and share the cache.
umask = 002

# Preserve cache across GCC rebuilds and
# introspect GCC changes through GCC wrapper.
compiler_check = %compiler% -v

# I expect 1.5M files. 300 files per directory.
cache_dir_levels = 3
```

> 启用缓存 file**/etc/portage/make.conf**

```sh
FEATURES="${FEATURES} ccache"
CCACHE_DIR="/data/ccache"
```

> 为用户开启缓存**~/.bashrc**

```sh
export PATH="/usr/lib/ccache/bin${PATH:+:}$PATH"
export CCACHE_DIR="/data/ccache"
```

> 使用的变量和命令

值得一提的是

- Variable **CCACHE_DIR** points to cache root directory.
- Variable **CCACHE_RECACHE** allows evicting old cache entries with new entries:

```sh
CCACHE_RECACHE=yes emerge --oneshot cat/pkg
```

See **man ccache** for many more variables.



### TeX

因为日常中也是需要一些排版的所以需要装一下`texlive`

```sh
emerge --ask app-text/texlive-core
```

配合使用的阅读器是

```sh
emerge --ask llpp
```

### KVM



平时做实验也会用到KVM

这边也做一下简单的配置

我这里习惯用`libvirtd`和`virt-manager`加上一些cli去管理

```sh
emerge -av libvirt virt-manager
```



### 更新内核说明

清除掉之前的配置

```sh
rm -r /boot/config-*
rm -r /boot/Sys*
rm -r /boot/vm*
```



修改内核配置之后编译内核

```sh
make -j9 && make modules_install
make install
```

更新grub

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

### USE使用



如果你更改了系统的USE

```sh
emerge --update --deep --newuse @world
```

然后清理

```sh
emerge -p --depclean
```

depclean之后

```sh
revdep-rebuild
```

查看软件包的USE

```sh
emerge --pretend --verbose www-client/seamonkey
```

### 笔记本增强工具

具体可以查看[WIKI](https://wiki.gentoo.org/wiki/Main_Page)

```sh
emerge --ask cpufreqd app-laptop/laptop-mode-tools
```

启动服务

```sh
root #rc-service laptop_mode start
root #rc-update add laptop_mode default
```



### 文档参考

[YangMame](https://blog.yangmame.org/Gentoo%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.html)

[Gentoo Wiki](https://wiki.gentoo.org/wiki/Main_Page)

[金步国内核文档](http://www.jinbuguo.com/kernel/longterm-linux-kernel-options.html)

[KVM](http://www.linux-kvm.org/page/KvmOnGentoo)


