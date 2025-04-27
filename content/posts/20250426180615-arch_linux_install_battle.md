+++
title = "Arch Linux Install Battle"
author = ["yuzumone"]
date = 2024-02-11
slug = "arch-linux-install-battle"
tags = ["Linux"]
draft = false
toc = true
image = "https://images.unsplash.com/photo-1577998555981-6e798325914e?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MTB8fHxlbnwwfHx8fHw%3D"
+++

{{< figure title="Photo by svklimkin on Unsplash" src="https://images.unsplash.com/photo-1577998555981-6e798325914e?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MTB8fHxlbnwwfHx8fHw%3D" >}}

grub から systemd-boot にしたいなぁと思っていて，どうせなら綺麗にするかということで <br/>


## Environment {#environment}

-   Surface Pro <br/>
-   Type Cover Keyboard (US) <br/>


## インターネット接続 {#インターネット接続}

WiFi でやっていく <br/>

```bash
ip link set wlan0 up
wpa_supplicant -B -i wlan0 -c <(wpa_passphrase SSID password)
dhcpcd
```


## パーティションの作成 {#パーティションの作成}

gdisk と思ったら最近はワンライナーでいけるらしい <br/>

```bash
sgdisk -z /dev/sda
sgdisk -n 1:0:+512M -t 1:ef00 -c 1:"EFI System" /dev/sda
sgdisk -n 2:0: -t 2:8300 -c 2:"Linux filesystem" /dev/sda
```


## フォーマット {#フォーマット}

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```


## マウント {#マウント}

```bash
mount /dev/sda2 /mnt
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
```


## ミラー変更 {#ミラー変更}

あとあと reflector で良い感じにするとして適当な日本のやつを上に書いておく <br/>

```bash
vim /etc/pacman.d/mirrorlist
Server = http://ftp.jaist.ac.jp/pub/Linux/ArchLinux/$repo/os/$arch
```


## 必須パッケージのインストール {#必須パッケージのインストール}

どうせいれるやつはこの段階でいれちゃう <br/>

```bash
pacstrap -K /mnt base base-devel linux linux-firmware linux-firmware-marvell \
  intel-ucode efibootmgr dosfstools netctl vim \
  iw wpa_supplicant networkmanager
```


## fstab {#fstab}

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```


## chroot {#chroot}

```bash
arch-chroot /mnt
```


## タイムゾーン {#タイムゾーン}

```bash
ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
hwclock --systohc
```


## ローカリゼーション {#ローカリゼーション}

```bash
vim /etc/locale.gen
# en_US.UTF-8 UTF-8 をアンコメント
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```


## ホストネーム {#ホストネーム}

みんなどんな hostname にしているのか気になる <br/>

```bash
echo "hostname" > /etc/hostname
```


## Initramfs {#initramfs}

カーネルパッケージをインストールしたときに mkinitcpio が実行されているため、普通は新しい initramfs の作成は必要ありません，とのことだが一応やっておく <br/>

```bash
mkinitcpio -P
```


## Root パスワード {#root-パスワード}

```bash
passwd
```


## ブートローダー {#ブートローダー}

Secure boot はどっかのタイミングでやりたい <br/>

```bash
bootctl install
```

設定は /boot/loader/loader.conf <br/>

```text
default arch.conf
timeout 3
console-mode keep
editor no
```

ローダの追加は /boot/loader/entries/arch.conf とします <br/>

```text
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=/dev/sda2 rw
```

systemd-boot の定期更新を有効にしておく <br/>

```bash
systemctl enable systemd-boot-update
```


## shutdown {#shutdown}

USB メモリを抜くのを忘れない <br/>

```bash
exit
shutdown -h now
```


## 改めてネットワーク接続 {#改めてネットワーク接続}

```bash
systemctl start NetworkManager
systemctl enable NetworkManager
nmctl d wifi connect SSID password password
```


## ユーザ作成 {#ユーザ作成}

```bash
useradd -m -g wheel -d /home/yuzumone -m yuzumone
passwd yuzumone
visudo  # いつものやつをアンコメント
```


## Pacman 周り {#pacman-周り}

pacman <br/>

```bash
sudo vim /etc/pacman.conf
# ParallelDownloads = 5 と Color をアンコメント
```

reflector <br/>

```bash
sudo pacman -S reflector
sudo reflector --country Japan,Australia
sudo systemctl enable reflector.timer
```

paccache <br/>

```bash
sudo pacman -S pacman-contrib
sudo systemctl enable paccache.timer
```


## SSD {#ssd}

毎週 TRIM するようにする <br/>

```bash
sudo systemctl enable fstrim.timer
```


## yay {#yay}

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

GUI は sway + lightdm で設定 <br/>

