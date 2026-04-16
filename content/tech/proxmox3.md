---
title: "自宅サーバ3: Arch linux をインストール"
date: 2023-10-08T17:57:25+09:00
draft: false
---

proxmox をしばらく触ってみたが，使いづらいと感じた．
UI の部分があまり直感的でない気がする．
VMware の方が分かりやすかった．．．

というわけで Arch Linux をインストールし，podman でサービスを立ち上げることにする．

## openSUSE vs Arch Linux

openSUSE と迷っていました．単純に使ったことない OS を使おうとしたら選択肢が openSUSE と Arch Linux だったからです．
openSUSE は以下の点で優れています

- btrfs + snapper：OS 標準の復旧手段として GRUB メニューからの過去のスナップショット起動や YaST のロールバックなどがあるらしい．Arch Linux は grub-btrfs は自分で設定する必要がある．
- トランザクショナルアップデート：openSUSE はアトミックな OS アップデートと，ロールバックによる復旧が可能らしい．
- YaST：YaST は良いらしい．CLI で設定するべきことを GUI でできる，という理解．一方で，YaST は長らく GUI を変えていないらしく，YaST に慣れた人が慣れているだけなような気もしている．

今回は k8s もやりたい．
今回は N100 で 16Gb しかないハードを選んだ．
というわけでメモリフットプリントが小さい Arch Linux を選んだ．
あとローリングリリースでメンテナンスしてる感がほしかったというのもある．

## Arch Linux インストール

### USB boot

- Ventoy を USB にフォーマット
- iso を取得し，ブート

### パーティショニング

- ```lsblk``` でディスク状態を確認
  - proxmox の残骸があることを確認
- ```gdisk /dev/nvme0n1``` で対話セッションを起動
  - o: 新しいパーティションテーブルを作成する．UEFI が良いので GPT を選択
  - n: パーティションを作成（512 MiB），EFI 用
  - n: パーティションを作成（残り全部），Linux 用
    - hex code は 8309 で Linux LUKS
  - p: パーティションを確認
  - w: ディスクに書き込む
- ```mkfs.fat -F32 /dev/nvme0n1p1```: ESP を FAT32 でフォーマット
- ```cryptsetup luksFormat /dev/nvme0n1p2```: LUKS 暗号化コンテナセットアップ
- ```cryptsetup open /dev/nvme0n1p2 cryptroot```: 開く
  - /dev/mapper/cryptroot という復号済みデバイスが見える
- ```mkfs.btrfs /dev/mapper/cryptroot```: btrfs ファイルシステムを作成
- ```mount /dev/mapper/cryptroot /mnt```: btrfs のトップレベルをマウント
  - サブボリュームの作成
    ```sh
    btrfs subvolume create /mnt/@
    btrfs subvolume create /mnt/@home
    btrfs subvolume create /mnt/@snapshots
    btrfs subvolume create /mnt/@var_log
    btrfs subvolume create /mnt/@containers
    ```
  - ```umount /mnt```
- サブボリュームの個別マウント
    ```sh
    sudo mount -o subvol=@,compress=zstd,noatime /dev/mapper/cryptroot /mnt
    sudo mkdir -p /mnt/{home,.snapshots,var/log,var/lib/containers,boot}
    sudo mount -o subvol=@home,compress=zstd,noatime /dev/mapper/cryptroot /mnt/home
    sudo mount -o subvol=@snapshots,compress=zstd,noatime /dev/mapper/cryptroot /mnt/.snapshots
    sudo mount -o subvol=@var_log,compress=zstd,noatime /dev/mapper/cryptroot /mnt/var/log
    sudo mount -o subvol=@containers,compress=zstd,noatime /dev/mapper/cryptroot /mnt/var/lib/containers
    sudo mount /dev/nvme0n1p1 /mnt/boot
    ```

### ベースシステムの作成

- ```sudo pacstrap -K /mnt base linux linux-firmware btrfs-progs sudo vim```
  - ベースシステムのインストール
  - LAN でやるので NetworkManager は不要．デバッグめんどくさいし
- ```sudo genfstab -U /mnt | sudo tee -a /mnt/etc/fstab```
  - fstab にマウント攻勢をアウトプット
  - /mnt/etc/fstab に権限が無かったので調べたらこういう書き方があるのか
- ```sudo arch-chroot /mnt```
  - chroot
- ```ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && hwclock --systohc```
  - クロック設定
- ```echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && echo "ja_JP.UTF-8 UTF-8" >> /etc/locale.gen && locale-gen && echo "LANG=en_US.UTF-8" > /etc/locale.conf```
- ```echo "KEYMAP=us" > /etc/vconsole.conf```
- ```echo "Pondering" > /etc/hostname```
  - Pondering にしました．好きなので
- ```passwd```
- ```vim /etc/mkinitcpio.conf```
  - initramfs は，カーネルが起動した直後ｍ，btrfs をマウントする前に一時的に使われるミニマムなファイルシステム
  - カーネル起動後は LUKS が復号されていないので，ここに LUKS ルーチンを入れておく必要がある
  - ```HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block sd-encrypt filesystems fsck)```
- ```mkinitcpio -P```

### ブートローダ

- ```bootctl install```
- ```vim /boot/loader/entries/arch.conf```
    ```sh
    title   Arch Linux
    linux   /vmlinuz-linux
    initrd  /initramfs-linux.img
    options rd.luks.name=＜UUID＞=cryptroot root=/dev/mapper/cryptroot rootflags=subvol=@ rw
    ```
- ```vim /boot/loader/loader.conf```
    ```sh
    default arch.conf
    timeout 3
    console-mode max
    ```

### ネットワーク

- ```vim /etc/systemd/network/20-wired.network```
    ```sh
    [Match]
    Name=en*

    [Network]
    DHCP=yes
    ```
- ```ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf```
- ```systemctl enable systemd-networkd```
- ```systemctl enable systemd-resolved```
- ```pacman -S openssh```
- ```systemctl enable sshd```
- ```exit```
- ```reboot```
- ```ping archlinux.org```

### 起動後

- ```useradd -m -G wheel -s /bin/bash pondering```
- ```passwd pondering```
- ```EDITOR=vim visudo```
- ```%wheel ALL=(ALL:ALL) ALL``` をコメントアウト
- あとは SSH 設定

### snapper

- ```sudo pacman -S snapper snap-pac```
- ```sudo umount /.snapshots```
- ```sudo rm -d /.snapshots```
- ```sudo snapper -c root create-config /```
- ```sudo btrfs subvolume delete /.snapshots```
- ```sudo mkdir /.snapshots```
- ```sudo mount -a```
- ```sudo snapper -c root create --description "base system"```
- ```sudo snapper -c root list```

おしまい
