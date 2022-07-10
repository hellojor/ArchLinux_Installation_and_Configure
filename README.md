# ArchLinux_Installation_and_Configure

## 下載Arch ISO
[archlinux iso](http://mirror.archlinux.tw/ArchLinux/iso/2022.05.01/)

# 安裝 Arch Linux

## 分割硬碟
查找所有硬碟
```
lsblk
```
選擇要安裝系統的硬碟後使用以下指令分割硬碟的磁區
```
cfdisk /dev/sdX
```
/dev/sda1  
/dev/sda1  
/dev/sda1  
/dev/sda1  

# 系统配置

## 1 系统 DM

用 lightDM 作为系统的 display manager。

```shell
sudo pacman -S lightdm
sudo systemctl enable lightdm
```
