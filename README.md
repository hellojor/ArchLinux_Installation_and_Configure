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
Linux swap 大小的議題可參考以下鏈接 https://itsfoss.com/swap-size/

## 格式化磁區
```
# EFI partion
mkfs.vfat /dev/sda1

# swap
mkswap /dev/sda2

# root、home
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda4
```

## 挂載磁區
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

## 網絡連接
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
1. 檢查網路卡
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
  
2. 設定網路卡實體位址
```
ip link set address ff:74:77:69:12:ab dev enp7s0
```

3. 清除網路卡的IP設定
```
ip addr flush dev enp7s0
```

4. 設定IP  
prefix_len = 24 (255.255.255.0 = 8 + 8 + 8 + 0 = 24)
```
ip address add 198.111.121.3/24 broadcast + dev enp7s0
```

5. 查看 IPv4 or IPv6 routes
```
# IPv4
ip route show
# IPv6
ip -6 show
```

6. 增加route
```
ip route add default via 198.111.121.250 dev enp7s0
```

7. 再次ping看看
```
ping -c 3 8.8.8.8
```
正常來説應該沒問題了


# 系统配置

## 1 系统 DM

用 lightDM 作为系统的 display manager。

```shell
sudo pacman -S lightdm
sudo systemctl enable lightdm
```
