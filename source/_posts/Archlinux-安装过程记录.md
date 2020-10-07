---
title: Archlinux 安装过程记录
date: 2020-09-20 23:05:48
tags: 
- Linux
- Archlinux
- 安装过程
categories: 
- Linux
comments: false
---

### 安装系统过程

<!--more-->

> **强烈建议在每次安装前都去 [Archlinux Wiki Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide) 查看是否有什么变化。**



#### 制作镜像

1. 访问 [Archlinux Download](https://www.archlinux.org/download) 链接下载所需镜像（一般为 ISO 格式），在使用镜像制作工具（推荐 Rufus）制作镜像之前建议先验证所下载文件的签名，尤其是从 HTTP 镜像源下载的文件。
2. 制作镜像（推荐 [Rufus](https://rufus.ie)）

#### 启动到 Live 环境

1. Asus X550VX 为开机按 F2 进入 BIOS，接着在 Boot 中选择带有 Arch 安装文件的媒介启动
2. 默认的 Shell 是 zsh，会以 root 身份登入

#### 验证启动模式

1. 目前一般启动模式都为 GPT+UEFI

2. 可以使用如下命令验证启动模式：若命令没有错误地显示了目录，则系统以 UEFI 模式启动。若目录不存在，系统则可能为 BIOS 模式或 CSM 模式启动（详见 Arch Wiki）。

    ```Shell
    # ls /sys/firmware/efi/efivars
    ```

#### 连网

1. Archlinux 的安装需要连网下载相关文件

    ```Shell
    # 以下为使用 Wifi 的情况
    
    # iwctl
    # device list （列出所有网卡）
    # station wlan0/wlan1... scan （使用哪块网卡扫描）
    # station wlan0/wlan1... get-networks （列出所有可用网络）
    # station wlan0/wlan1... connect SSID （连接到网络，SSID 为网络名）
    # 输入密码
    # quit （退出 iwctl）
    ```

#### 更新系统时间

1. 使用 `timedatectl` 来确保系统时间是准确的

    ```Shell
    # timedatectl set-ntp true
    # 使用 timedatectl staus 来检查服务状态
    ```

#### 建立硬盘分区

1. 硬盘若已被系统识别，则会显示为一个块设备（如 `/dev/sda`, `/dev/sdb` 等），可以使用 `fdisk` 命令进行查看

    ```Shell
    # fdisk -l
    ```

2. 使用 `cfdisk` 来对硬盘进行分区

3. 对于一个 Archlinux 系统来说，必须要有的挂载点有：

    - 一个根分区（即 `/mnt` 分区）
    - 一个 EFI 系统分区（即 `/mnt/boot` 或是 `/mnt/efi`，如果是双系统则可以直接使用 Windows 的 EFI 分区，但是要**注意千万不要对其进行格式化操作**，一般 Windows 的 EFI 分区都为 100MB，大多数情况下是够用的，但如果需要安装 GRUB 主题的话是不够用的，推荐扩容到 512MB）
    - [SWAP] （需大于 512 MB）

4. 分区方案（假设是一块 500GB 的 SSD，识别为 /dev/sda）

    | 挂载点    | 分区                                      | 分区类型                 | 大小     |
    | --------- | ----------------------------------------- | ------------------------ | -------- |
    | /mnt/boot | /dev/sdb2（挂载到 Windows 的 EFI 分区上） | EFI 系统分区             | 512 MB   |
    | /mnt      | /dev/sda1                                 | Linux x86_64 根目录（/） | 150 GB   |
    | [SWAP]    | /dev/sda2                                 | Linux Swap               | 8 GB     |
    | /mnt/home | /dev/sda3                                 | Linux File System        | 剩余空间 |

5. 格式化分区

    当分区建立完毕后，需要使用适当的文件系统进行格式化

    ```Shell
    # Ext4 文件系统
    # mkfs.ext4 /dev/sda1
    # mkfs.ext4 /dev/sda3
    
    # Swap 分区
    # mkswap /dev/sda2
    # swapon /dev/sda2
    ```

6. 挂载分区

    将根分区挂载到 `/mnt`

    ```Shell
    # mount /dev/sda1 /mnt
    ```

    然后使用 `mkdir` 命令创建其他剩余的挂载点并挂载

    ```Shell
    # mkdir /mnt/home
    # mkdir /mnt/boot
    # mount /dev/sda3 /mnt/home
    # mount /dev/sd2 /mnt/boot （注意挂载到 Windows EFI 分区处）
    ```

#### 安装

1. 选择镜像，由于在国内，所以需要手动添加国内的镜像源 [Archlinux All Https Mirrorlist](https://www.archlinux.org/mirrorlist/all/https)。修改 `/etc/pacman.d/mirrorlist` 文件，添加即可

2. 安装软件包

    ```Shell
    # pacstrap /mnt base linux-lts linux-firmware networkmanager vim
    ```

#### Chroot 环境下配置系统

1. Fstab

    用以下命令生成 fstab 文件（`-U` 参数设置 UUID，`-L` 参数设置卷标）

    ```Shell
    # genfstab -U /mnt >> /mnt/etc/fstab
    # cat /mnt/etc/fstab （检查生成的 fstab 文件是否正确）
    ```

2. Chroot

    ```Shell
    # arch-chroot /mnt
    ```

3. 时区

    设置时区

    ```Shell
    # ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    ```

    运行 `hwclock` 以生成 `/etc/adjtime`

    ```Shell
    # hwclock --systohc
    ```

4. 本地化

    需要修改的文件有：`locale.gen` 与 `locale.conf`

    编辑 `/etc/locale.gen` ，然后移除需要的地区前的注释符号（移除 `en_US.UTF-8` 与 `zh_CN.UTF-8` 前的注释）

    ```Shell
    # locale-gen （生成 locale 信息）
    ```

    创建 `/etc/locale.conf` 文件，并编辑设定 `LANG` 变量

    ```Shell
    # LANG=en_US.UTF-8
    ```

5. 网络配置

    创建 `/etc/hostname` 文件，并添加如下内容：

    ```Shell
    # beiran
    ```

    添加对应的信息到 `/etc/hosts` 中：

    ```Shell
    # 127.0.0.1 localhost
    # ::1 localhost
    # 127.0.0.1 beiran.localdomain beiran
    ```

    若有公网 IP，则将 `127.0.0.1` 更换为公网 IP

6. Initramfs

    ```Shell
    # mkinitcpio -P
    ```

7. Root 密码

    设置 Root 密码：

    ```Shell
    # passwd
    ```

8. 更新微码（intel-ucode）

    ```Shell
    # pacman -S intel-ucode
    ```

9. 安装 Grub

    ```Shell
    # pacman -S grub efibootmgr os-prober （os-prober 是因为双系统）
    
    # grub-install --efi-directory=/boot --bootloader-id=Archlinux
    # grub-mkconfig -o /boot/grub/grub.cfg
    ```

10. 安装其他的基础包

    ```Shell
    # pacman -S openssh wget curl dialog wpa_supplicant ntfs-3g dnsutils
    ```

11. 配置图形化环境

    首先确定显卡型号：

    ```Shell
    # lspci | grep -E "VGA|3D"
    ```

    安装对应驱动

    ```Shell
    # pacman -S alsa-utils （声卡）
    # pacman -S xf86-video-vesa （Intel 集显）
    # pacman -S nvidia-lts nvidia-utils nvidia-settings （Nvidia 独显，因为内核装的 linux-lts 所以独显驱动也装的 nvidia-lts）
    # pacman -S xf86-video-vmware
    # pacman -S xf86-input-synaptics （触摸板驱动）
    # pacman -S bluez-utils bluez （蓝牙）
    ```

12. 相关配置

    ```Shell
    # systemctl enable NetworkManager.service
    # systemctl enable bluetooth.service
    
    # nmcli 的使用方法
    # nmcli dev wifi list （查询 wifi 列表）
    # nmcli device wifi connect "SSID" password "PASSWORD"（连接 wifi）
    ```

13. 安装显示管理器

    ```Shell
    # 这里选择使用 LightDM
    # pacman -S lightdm lightdm-deepin-greeter numlockx
    
    # 修改 LightDM 配置
    # vim /etc/lightdm/lightdm.conf
    # greeter-session=lightdm-deepin-greeter
    # greeter-setup-script=/usr/bin/numlockx on
    
    # 修改 /etc/pacman.conf
    # 将 Color 选项前的注释去掉
    
    # systemctl enable lightdm
    
    # pacman -S haveged
    # systemctl enable haveged
    ```

14. 安装 Deepin Desktop

    ```Shell
    # pacman -S deepin deepin-extra zssh lrzsz archlinux-wallpaper
    ```

15. 创建用户

    ```Shell
    # useradd -m -G wheel beiran
    ```

    设置密码

    ```Shell
    # passwd beiran
    ```

    修改 `sudo` 设置

    ```Shell
    # pacman -S sudo
    # vim /etc/sudoers
    # 添加一行：
    # beiran ALL=(ALL) ALL
    ```

16. 安装字体

    ```Shell
    # pacman -S wqy-zenhei wqy-bitmapfont adobe-source-code-pro-fonts adobe-source-han-serif-cn-fonts adobe-source-han-sans-cn-fonts noto-fonts noto-fonts-extra noto-fonts-emoji noto-fonts-cjk ttf-dejavu
    ```

17. 安装 Fcitx 输入法

    ```Shell
    # pacman -S fcitx-googlepinyin fcitx-mozc fcitx-im fcitx-skin-material
    
    # 可以修改 ~/.config/fcitx/conf/fcitx-classic-ui.conf 中的 SkinType 参数来启用 material 皮肤
    ```

    配置 Fcitx：

    ```Shell
    # 全局设置修改 /etc/environment
    # GTK_IM_MODULE=fcitx
    # QT_IM_MODULE=fcitx
    # XMODIFIERS=@im=fcitx
    
    # 非全局设置则新建 ~/.pam_environment
    # 或者 ~/.xprofile
    ```

#### 重启

1. 退出 Chroot 环境

    ```Shell
    # exit （退出 Chroot 环境）
    # umount -R /mnt （卸载被挂载的分区）
    # reboot
    # 移除安装介质（U盘）
    ```

#### 系统设置

1. 使用非 `root` 用户登录

2. 生成默认文件夹

    ```Shell
    # sudo pacman -S xdg-user-dirs
    # xdg-user-dirs-update --force
    ```

3. 配置 `zsh`

    ```Shell
    # sudo pacman -S zsh
    # git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
    # git clone https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    
    # 编辑 ~/.zshrc 文件
    # ZSH_THEME="ys"
    # plugins=(git web-search zsh-autosuggestions zsh-synta-highlighting)
    
    # 切换 zsh
    # chsh -s /bin/zsh
    ```

4. 配置 `Shadowsocks-libev`

    ```Shell
    # sudo pacman -S shadowsocks-libev simple-obfs
    # sudo mkdir /etc/shadowsocks
    # sudo vim /etc/shadowsocks/config.json（将自己的配置文件复制进去）
    # systemctl enable shadowsocks-libev@config.service
    # systemctl start shadowsocks-libev@config.service
    ```

5. 配置 `pfetch`

    ```Shell
    # wget https://github.com/dylanaraps/pfetch/archive/master.zip
    # cd pfetch-master
    # sudo install pfetch /user/local/bin
    ```

    

----

### 软件安装过程

#### 添加 Archlinuxcn 源

1. 编辑 `/etc/pacman.conf` 文件

    ```Shell
    # 添加 Archlinuxcn 源
    # [archlinuxcn]
    # SigLevel = Optional TrustAll
    # Server = https://mirrors.bfsu.edu.cn/archlinuxcn/$arch
    # Server = https://mirrors.cqu.edu.cn/archlinuxcn/$arch
    # Server = https://mirrors.dgut.edu.cn/archlinuxcn/$arch
    # Server = https://mirrors.neusoft.edu.cn/archlinuxcn/$arch
    ```

2. 安装 `archlinux-keyring`

    ```Shell
    # sudo pacman -S archlinuxcn-keyring
    # sudo pacman -Syu
    ```

3. Mysql 5.7.30 相关配置

    ```Shell
    # 安装完成后首先初始化数据库，这一步如果没出错的话会生成一个初始密码
    # sudo mysqld --initialize --user=mysql
    # 如果出现错误，尝试删除 /var/lib/mysql 目录
    # sudo rm -rf /var/lib/mysql
    # 然后连接到 Mysql
    # mysql -u root -p
    # 输入之前的初始密码，连接成功后修改 root 用户的密码以及允许远程连接
    # ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
    # UPDATE user SET Host='%' WHERE user='root';
    # FLUSH PRIVILEGES;
    ```

4. IntelliJ-IDEA-CE 相关配置

