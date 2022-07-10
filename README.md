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

# 系统配置

## 1 系统 DM

用 lightDM 作为系统的 display manager。

```shell
sudo pacman -S lightdm
sudo systemctl enable lightdm
```
