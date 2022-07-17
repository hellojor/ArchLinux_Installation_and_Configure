# ArchLinux_Installation_and_Configure

# 下載Arch ISO
[archlinux iso](http://mirror.archlinux.tw/ArchLinux/iso/2022.05.01/)

# 1 安裝 Arch Linux

## 1.1 分割硬碟
### 1.1.1 查找所有硬碟
```
lsblk
```
### 1.1.2 分配與切割磁碟
選擇要安裝系統的硬碟後使用以下指令分割硬碟的磁區  
假設安裝在`sda`這個硬碟上
```
cfdisk /dev/sda
```
將其分配為 
<table>
  <tr>
    <td>Device</td>
    <td>Size</td>
    <td>Type</td>
  </tr>
  <tr>
    <td>/dev/sda1</td>
    <td>550M</td>
    <td>EFI System</td>
  </tr>
  <tr>
    <td>/dev/sda2</td>
    <td>跟你ram一樣大小</td>
    <td>Linux Swap</td>
  </tr>
  <tr>
    <td>/dev/sda3</td>
    <td>100G</td>
    <td>Linux root (x86-64)</td>
  </tr>
  <tr>
    <td>/dev/sda4</td>
    <td>剩下的空間</td>
    <td>Linux Home</td>
  </tr>
</table>
Linux swap 大小的議題可參考以下鏈接 https://itsfoss.com/swap-size/

## 1.2 格式化磁區
```
# EFI partion
mkfs.vfat /dev/sda1

# swap
mkswap /dev/sda2

# root、home
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
```

## 1.3 挂載磁區
```
# 將root mount 到 /mnt
mount /dev/sda3 /mnt
mkdir /mnt/boot 
# 將boot mount 到 /mnt/boot
mount /dev/sda1 /mnt/boot
mkdir /mnt/home
# 將home mount 到 /mnt/home
mount /dev/sda4 /mnt/home
```
查看每個分割磁區的filesystem type
```
blkid /dev/sdXX
```
查看是否掛載成功
```
df -h
```

## 1.4 網絡連接
在這個之前，如果你是使用dhcp 或者 wifi的  
可以先ping Google所提供的DNS伺服器IP位址看看是否已經連接
```
ping -c 3 8.8.8.8
```
如果出現以下文字，説明你的網路沒問題，可以跳過這部分
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=10.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=10.4 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=10.6 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
```
如果沒有，説明你可能是使用固定IP，請跟著以下步驟
### 1.4.1 檢查網路卡
```
ip link show
```
執行結果:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether XX:XX:XX:XX:XX:XX brd ff:ff:ff:ff:ff:ff permaddr XX:XX:XX:XX:XX:XX
```
所以本機的網路卡為 `enp7s0`  
假設 Mac Address是 `ff:74:77:69:12:ab`  
IP是 `198.111.121.3`  
子網路遮罩是 `255.255.255.0`  
route server是 `198.111.121.250`    
  
### 1.4.2 設定網路卡實體位址
```
ip link set address ff:74:77:69:12:ab dev enp7s0
```

### 1.4.3 清除網路卡的IP設定
```
ip addr flush dev enp7s0
```

### 1.4.4 設定IP  
prefix_len = 24 (255.255.255.0 = 8 + 8 + 8 + 0 = 24)
```
ip address add 198.111.121.3/24 broadcast + dev enp7s0
```

### 1.4.5 查看 IPv4 or IPv6 routes
```
# IPv4
ip route show
# IPv6
ip -6 show
```

### 1.4.6 增加route
```
ip route add default via 198.111.121.250 dev enp7s0
```

### 1.4.7 再次ping看看
```
ping -c 3 8.8.8.8
```
正常來説應該沒問題了  
更多網路詳情可參考 https://wiki.archlinux.org/title/Network_configuration

## 1.5 設定 Mirrors Server
### 1.5.1 加入 mirrorlist
用vim /etc/pacman.conf編輯conf文件將,找到[core] [extra] [community]將交大的arch mirrorlist加入
```
[core]
Server = http://archlinux.cs.nctu.edu.tw/$repo/os/$arch
Include = /etc/pacman.d/mirrorlist

[extra]
Server = http://archlinux.cs.nctu.edu.tw/$repo/os/$arch
Include = /etc/pacman.d/mirrorlist

[community]
Server = http://archlinux.cs.nctu.edu.tw/$repo/os/$arch
Include = /etc/pacman.d/mirrorlist
```
### 1.5.2 排序鏡像站  
安裝pacman-contrib腳本，這個腳本可以排序鏡像站
```
pacman -Sy pacman-contrib
```
排序"設定檔現有的"鏡像站前6快，並寫入檔案
```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
```
將英國和法國前5快的鏡像站，添加至鏡像列表中
```
curl -s "https://archlinux.org/mirrorlist/?country=FR&country=GB&protocol=https&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors -n 5 - >> /etc/pacman.d/mirrorlist
```
可以將網址中的 `country=FR&country=GB&` 字樣刪除，將國家的篩選條件移除，或是自行更改選項  
如果有些套件在安裝中出現類似以下訊息，可能是本機中儲存的鏡像站皆無法找到相依的套件，可以將鏡像站數量提高
```
error: failed retrieving file 'lpsolve-5.5.2.5-4-x86_64.pkg.tar.zst' from mirror.osbeck.com : The requested URL returned error: 404
```
也可以不定時使用這些指令更新鏡像站列表

## 1.6 安裝 base、base-devel、linux和linux-firmware packages
使用pacstrap來運行安裝系統基本組件
```
pacstrap /mnt  base base-devel linux linux-firmware
```
正常安裝畫面,如果有出現 erro install pacakages to new root 要重新mount or 重新格式化硬碟比較好

## 1.7 安裝套件並更新套件資料庫資訊
```
pacman -Syy
```
## 1.8 建立Fstab
產生fstab file建立好後可以看一下檔案有沒有存在
```
genfstab -U /mnt >> /mnt/etc/fstab
```

## 1.9 chroot 至新系統
接下來我們要change root to new system
```
arch-chroot /mnt
```

# 2 設定 Arch

## 2.1 設定時區Time zone
```
ln -sf /usr/share/zoneinfo/Asia/Taipei  /etc/localtime
```

## 2.2 用 hwclock 來產生 /etc/adjtime:
```
hwclock --systohc
```

## 2.3 設定語言
```
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen;
echo "zh_TW.UTF-8 UTF-8" >> /etc/locale.gen;
echo "LANG=en_US.UTF-8" > /etc/locale.conf;
```
記得要執行locale-gen不然進入桌面會沒有英文字，連打字都無法，會很冏
```
locale-gen  
```

## 2.4 設定電腦名稱hostname
```
echo "your-pc-name" > /etc/hostname
```
安裝vim
```
pacman -Sy vim
vim /etc/hosts
```
把以下加入/etc/hosts
```
127.0.0.1        localhost.localdomain         localhost
::1              localhost.localdomain         localhost
127.0.1.1        myhostname.localdomain        myhostname
```

## 2.5 建立initial ramdisk
mkinitcpio 是一個script可以幫忙建立Initial ramdisk
```
mkinitcpio -p linux
```

## 2.6 設定 root 密碼
```
passwd
```

## 2.7 設定Bootloader
先安裝grub
```
pacman  -Sy grub os-prober efibootmgr
```
os-prober 可以偵測目前已安裝的其他系統，並在之後加入 grub 選單中
```
os-prober
```
因為之前把efi partition 掛在 /efi 資料夾,所以要改成 --efi-direction=/efi  
BIOS:
```
grub-install /dev/sda
```
UEFI:
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```

## 2.8 安裝一些常用套件
```
pacman -Sy git pkgfile bash-completion sudo

#pkgfile //套件 & 軟體名字不match
#bash-completion //自動補齊 for pacman…等
```
pacman 用法如下
```
# 安裝套件
# -S: sync
pacman -S <package>

# 更新套件（全系統）
# -u: upgrade
# 安裝套件並更新套件資料庫資訊
pacman -Sy <package> # 建議不要這樣使用，可能會造成 dependency issue
pacman -Syu <package> # 建議這樣用
sudo pacman -Syy #類似sudo apt-get update

# 安裝不在檔案庫的 package（例如 AUR 的 package）
pacman -U /path/to/package/package_name-version.pkg.tar.xz
pacman -U http://www.example.com/repo/example.pkg.tar.xz # 也可以從網址安裝

# 移除套件
# -R: remove
pacman -R <package>

# 移除套件及其相依套件
# -s: recursive
pacman -Rs <package>
```
## 2.9 安裝網路工具
```
pacman -Sy net-tools networkmanager wpa_supplicant wireless_tools;
```

## 2.10 重新啟動
離開chroot以前記得把dhcp服務加入systemd 的啟動選項中，如果出現dhcp.service does not exit記得先裝dhcp package or 先systemctl enable dhcpcd  
如果是在學校有分配ip的話就不需要enable只要有裝就好
```
systemctl enable dhcpcd.service
exit
umount -R /mnt
reboot
```
基本上，到這裡就會結束系統的安裝，接下來都是在系統上做設定了。

# 3 再次進入系統的設定
登入
```
username: root
password: 你剛設定的密碼
```
## 3.1 設定網路
開啟NetWorkManager
```
systemctl enable NetWorkManager;
systemctl start NetWorkManager;
```
如果使用固定ip的使用者，發現重啓後網路就連不上了可透過 nmtui 圖形化設定介面再設定網路
```
nmtui
# ethernet
# Mac
# ipv4 -> Mannual
# Address
# GateWay
# DNS
# ipv6 -> disabled
# 記得重新激活activate
# 記得重新激活activate
# 記得重新激活activate
```

## 3.2 建立新的使用者
設定sudo 群組 打開sudoer file並將%wheel ALL=(ALL)前面註解拿掉
```
vim /etc/sudoers

# Uncomment to allow members
%wheel ALL=(ALL) ALL
```
建立新使用者，並加入 sudo 群組
```
useradd -m -u 1001 "your-user-name"
passwd "your-user-name"
usermod "your-user-name" -G wheel
```
重新開機
```
reboot
```

## 3.3 中文字體
除了設定中文 locale，還要安裝中文字體，才可正確顯示中文  
不然你會看到環境界面以及瀏覽器都是會有奇怪的口口出現  
這裡建議安裝noto-fonts-cjk 思源黑體，或是文泉驛正黑體(windows用的中文字體)，文泉驛微米黑(Android)
```
sudo pacman -Su noto-fonts-cjk adobe-source-code-pro-fonts
```

# 4 安裝 Window Manager(圖形介面)
## 4.1 安裝 i3
```
sudo pacman -Syyu
sudo pacman -S i3-gaps i3status rofi compton nitrogen ttf-dejavu
```
可以依照以下表格自己玩看看哦~
<table>
  <tr>
    <td>Name</td>
    <td>Detail</td>
    <td>Recommended</td>
  </tr>
  <tr>
    <td>ttf-dejavu</td>
    <td>更好看的字體</td>
    <td>y</td>
  </tr>
  <tr>
    <td>i3-gaps</td>
    <td>i3的分支，注重於i3的美化	</td>
    <td>y</td>
  </tr>
  <tr>
    <td>rofi</td>
    <td>圖形化的app/command啟動器	</td>
    <td>y</td>
  </tr>
  <tr>
    <td>compton</td>
    <td>視窗透明化</td>
    <td>y</td>
  </tr>
  <tr>
    <td>nitrogen	</td>
    <td>更方便的桌布設置</td>
    <td>y</td>
  </tr>
  <tr>
    <td>i3-block</td>
    <td>更好看的系統狀態列</td>
    <td></td>
  </tr>
  <tr>
    <td>i3status</td>
    <td>i3的系統狀態列</td>
    <td></td>
  </tr>
</table>

## 4.2 安裝 xorg
```
sudo pacman -S xorg xorg-xinit xterm
```
<table>
  <tr>
    <td>Name</td>
    <td>Detail</td>
  </tr>
   <tr>
    <td>xorg</td>
    <td>includes xorg-server but not xorg-xinit</td>
  </tr>
   <tr>
    <td>xorg-server-utils</td>
    <td>doesn’t exist anymore</td>
  </tr>
   <tr>
    <td>xorg-xinit</td>
    <td>installs: xorg-xinit, inetutils, includes /etc/X11/xinit/xinitrc</td>
  </tr>
   <tr>
    <td>xorg-apps</td>
    <td>group, everything in it is already included in xorg</td>
  </tr>
   <tr>
    <td>xterm</td>
    <td>You will probably want/need this</td>
  </tr>
</table>

## 4.3 安裝 GPU driver
```
sudo pacman -S nvidia nvidia-utils    # NVIDIA 
sudo pacman -S xf86-video-amdgpu mesa   # AMD
sudo pacman -S xf86-video-intel mesa    # Intel
```

## 4.4 安裝音效支援
```
sudo pacman -S alsa-utils
```

## Starting with Xinit (startx)
更改 /etc/X11/xinit/xinitrc內容為如下所示：
```
 .
 .
 .
#twm &
#xterm....
#exec xterm ....
exec i3
```
接著開啟Xinit
 ```
startx
```
進入i3後，可以使用預設的 "super+enter" 來啓動terminal，"super+shift+q" 來關閉terminal  
super 就是你剛設定的 win or alt鍵  
如果要查看或修改指令，請到 `~/.config/i3/config`

# 5 系统配置

## 5.1 安裝 LigthDM
用 lightDM 作为系统的 display manager
```
sudo pacman -S lightdm lightdm-gtk-greeter
systemctl enable lightdm
reboot
```
## 5.2 安装主题及相关配置
這裏是使用 lightdm-webkit2-theme-glorious 的主題
```
git clone https://aur.archlinux.org/lightdm-webkit2-theme-glorious.git
cd lightdm-webkit2-theme-glorious
makepkg -sri
```
複製light-theme 到 `/usr/share/lightdm-webkit/themes/glorious`
```
sudo cp -r lightdm-webkit2-theme-glorious /usr/share/lightdm-webkit/themes/glorious
```

## 5.3 背景图片
登录的背景图片位于目录`var/lib/AccountsService/wallpapers`。注意在命令行使用`lightdm-webkit2-greeter`，进入`Setting`，选择`background engine`为`image`。(见`lightdm-webkit2-greeter.conf`文件)。注意文件的权限。

## 5.4 登录头像
用户头像的目录位于`/var/lib/AccountsService/icons`。注意在`/var/lib/AccountsService/users`新建文件$USER，输入以下内容：
```
[User]
Icon=/var/lib/AccountsService/icons/myicon.jpg
```

## 5.5 lightDM 设置
编辑文件`/etc/lightdm/lightdm.conf`,设置`greeter-session=lightdm-webkit2-greeter`
```
.
.
.
# Seat configuration
.
.
[SeatDefaults]
greeter-session=lightdm-webkit2-greeter
.
.
[Seat:*]
```
## 5.6 設定雙熒幕
可使用以下指令來檢查monitor  
```
xrandr -q
```
假設設備是 `HDMI-1` 和 `DP-2`  
```
xrandr --output HDMI-1 --auto --output DP-2 --auto --right-of HDMI-1
```
以上設定是 `HDMI-1` 在左邊，`DP-2` 在右邊  
`--auto` 是自動選擇解析圖  
可以使用 `--mode 1920x1080` 這樣來設定解析度  

## 5.7 双显示器问题
可能存在双显示问题，自写脚本解决。在`/etc/lightdm.conf`，设置`display-setup-script=/usr/bin/lightDMScript.sh`。
```
#!bin/bash
xrandr | grep "HDMI-1 disconnected"
result=$?
if [ $result -gt 0 ]
then
    xrandr --output eDP-1 --off --output HDMI-1 --primary --mode 2560x1440     --pos 0x0 --rotate normal --output DP-1 --off --output HDMI-2 --off
else
    xrandr --output eDP-1 --primary --mode 1920x1080 --pos 0x0 --rotate normal --output HDMI-1 --off --output DP-1 --off --output HDMI-2 --off
fi
```

## 5.8 总体配置
修改 /etc/lightdm/lightdm-webkit-greeter.conf
```
[greeter]
debug_mode          = true
detect_theme_errors = true
screensaver_timeout = 300
secure_mode         = true
time_format         = LT
time_language       = auto
webkit_theme        = glorious

[branding]
background_images = /var/lib/AccountsService/wallpapers
logo              = /usr/share/pixmaps/archlinux-logo.svg
user_image        = /var/lib/AccountsService/icons/myicon.jpg
```


# 6 雙系統 arch + window 10 啓動
## 6.1 讓os-prober 可以偵測其他boot
```
sudo vim /etc/default/grub
uncomment GRUB_DISABLE_OS_PROBER=false
```
## 6.2 掛載 Windows
假設`windows10`系統被安裝在`sdb`裏面，而且`sdb1`是它的boot  
可使用`lsblk`來確認
```
mkdir /mnt/windows10
mount /dev/sdb1 /mnt/windows10
```
## 6.3 偵測 Windows
```
os-prober
```

## 6.4 產生 grub 設定檔
```
grub-mkconfig -o /boot/grub/grub.cfg
reboot
```
在啓動界面就可以看到一個 `Windows Boot Manager (on /dev/sdb1)`, 點擊他就可以進入windows咯~~  

# 7 安裝第三方軟體以及一些基本設定
透過以下的套件我們可以安裝第三方軟體, ex: google-chrome  
`yay與paru較為方便，paru安裝時間較久。`  
如果在安裝過程，系統說你的權限不足，可是又不能使用`sudo makepkg -si`
```
# 先把該檔案的root變成自己的
# me是你的username
chown -R me /home/me/yay
```
然後在進行`makepkg`的動作，應該就沒問題了

## 7.1 AUR
```
git clone https://aur.archlinux.org/aurman.git
cd aurman
makepkg -si
```

## 7.2 yay
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

## 7.3 paru
```
sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```

## 7.4 snap
```
git clone https://aur.archlinux.org/snapd.git
cd snapd
makepkg -si
```
Enable snap
```
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
```
Start snap
```
sudo systemctl start --now snapd.socket
```

## 7.5 檔案總管 Thunar
```
sudo pacman -S thunar thunar-volman
```

## 7.6 中文輸入法 fctix
有遇過chrome裝好後無法切換使用新酷音，但在其他編輯器及瀏覽器都可以使用情形，這時候可以用fcitx-diagnose看一下是哪邊沒裝好~  
  
fcitx-chewing:新酷音  
fcitx-googlepinyin:google拼音  
```
sudo pacman -S fcitx-configtool fcitx-chewing fcitx-googlepinyin　kcm-fcitx
```
安裝好後記得在~/.xprofile 新增以下變數, 如果沒有此檔案請要自行建立，
```
input=fcitx
export XMODIFIERS=@im=fcitx
export LC_CTYPE=zh_TW.UTF-8
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
$input&
```
改好後就重啓
```
reboot
```
可使用`ctrl+space`來切換英\中  
預設是注音，如果要使用拼音，可以使用`fcitx-configtool`的指令來進行設定  
切換的按鍵也可在`fcitx-configtool`進行設定  
可使用`ctrl+shift+f`來進行繁體字與簡體字的切換  

## 7.7 google-chrome
```
yay -S google-chrome
```


# 8 terminal與bash的設定與美華
## 8.1 terminal(rxvt)
```
yay -S rxvt rxvt-unicode
```
在`.Xdefault`進行`rxvt`設定  
```
vim ~/.Xdefault
```
可以參考以下設定  
```
!# lefthaha s ~/.Xdefaults
!# after modified, use ` xrdb -merage ~/.Xdefaults ` to reload setting
!# then launch `urxvt` again to test

!# turn compile xft on
!#(copy from arch wiki,may increase performance)
!#URxvt.buffered: true

!# set colors
URxvt.background: #0a0a0a
URxvt.foreground: white
URxvt.cursorColor: green
!#URxvt.underlineColor: green

!# set transparency
URxvt.transparent: on
URxvt.shading: 20

!# set fonts
!#URxvt.font: 		xft:SourceCodePro:style=Regular:size=14
URxvt.font:xft:MesloLGS NF:regular:size=12
!#URxvt.boldfont:	xtf:Meslo LG S DZ for Powerline:bold:size=12
!#URxvt.italicFont:	xtf:Meslo LG S DZ for Powerline:Italic:size=12
!#URxvt.boldItalicFont:
URxvt.letterSpace: -1

!# set input method
!#URxvt.inputMethod: gcin
URxvt.preeditType: Root
URxvt.imLocale: zh_TW.UTF-8

!# extern common
URxvt.perl-ext-common: default,clipboard,resize-font,matcher
!## copy, resize, url
!# set clickable URLs
URxvt.urlLauncher     : /usr/bin/xdg-open
URxvt.matcher.button  : 1
URxvt.matcher.rend.0: Uline Bold fg5
URxvt.colorUL: #4682B4

!# set window size
!#URxvt.geometry: 86x28

!# set misc
URxvt.scrollBar: False
!#URxvt.jumpScroll: True
!#URxvt.cursorUnderline: True
!#URxvt.cursorBlink: True

!# my config
URxvt.iso14755: false
URxvt.iso14755_52: false

!## scrollbak position
URxvt*scrollTtyOutput: false
URxvt*scrollWithBuffer: true
URxvt*scrollTtyKeypress: true

!## scrollbak buffer
URxvt.secondaryScreen: 1
URxvt.secondaryScroll: 0

!## cut and paste
URxvt.clipboard.autocopy: true
URxvt.keysym.M-c: perl:clipboard:copy
URxvt.keysym.M-v: perl:clipboard:paste

!## disable printing the therminal contents when pressing printscreen
URxvt.print-pipe: "cat > /dev/null"

! Tango color palette
URxvt*background:         #2e3436
URxvt*foreground:         #eeeeec
URxvt*cursorColor:        #8ae234
! foreground color for underline
URxvt*colorUL:            #8ae234
! line color for underline
URxvt*underlineColor:     #92659a
! black dark/light
URxvt*color0:             #2e3436
URxvt*color8:             #6e706b
! red dark/light
URxvt*color1:             #cc0000
URxvt*color9:             #ef2929
! green dark/light
URxvt*color2:             #4e9a06
URxvt*color10:            #8ae234
! yellow dark/light
URxvt*color3:             #edd400
URxvt*color11:            #fce94f
! blue dark/light
URxvt*color4:             #3465a4
URxvt*color12:            #729fcf
! magenta dark/light
URxvt*color5:             #92659a
URxvt*color13:            #c19fbe
! cyan dark/light
URxvt*color6:             #07c7ca
URxvt*color14:            #63e9e9
! white dark/light
URxvt*color7:             #d3d7cf
URxvt*color15:            #eeeeec
```


# 7 系統主题设置
在安裝系統主題之前，我們先安裝
## 7.1 安装 GTK2、GTK3 主题管理器：lxappearance。
```
sudo pacman -S lxappearance
```
## 7.2 GTK 主题选择
选择material主题：

sudo pacman -S materia-gtk-theme
## 7.3 Icon 主题选择
选择papirus-icon-theme：

sudo pacman -S papirus-icon-theme
## 7.4 QT 主题
QT 主题与 GTK 主题相统一。首先安装 QT 主题管理器：

sudo pacman -S qt5ct
其次通过yay安装插件：

yay -S qt5-styleplugins
在~/.xprofile中添加语句：

export QT_QPA_PLATFORMTHEME=gtk2
打开qt5ct，将 QT 主题设置为 gtk2。

## 暫時關閉蜂鳴器(需要權限)
```
rmmod pcspkr
```


## terminal
## bash
## 雙系統arch + window 10

# 參考資料
* https://hackmd.io/Wi2N2PaEReOgQ8SQJVOaFQ
* https://hackmd.io/l5TcrZZmSbeLsFdwEQ84AQ
* https://github.com/shejialuo/OS-Configuration/tree/master/ArchLinux-i3/system_config
* https://github.com/manilarome/lightdm-webkit2-theme-glorious
* https://wusyong.github.io/posts/arch-dual-boot/
* https://forum.endeavouros.com/t/warning-os-prober-will-not-be-executed-to-detect-other-bootable-partitions-systems-on-them-will-not-be-added-to-the-grub-boot-configuration-check-grub-disable-os-prober-documentation-entry/13998
* https://www.reddit.com/r/archlinux/comments/97odv3/error_when_trying_to_install_an_aur_helper/
