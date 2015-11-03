title: xenomai-3.0 on ubilinux with Intel Edison
date: 2015-11-01 05:43:20
tags:

# xenomai-3.0 on ubilinux with Intel Edison

<div style="text-align: right;">
@takarakasai
</div>

---


# 1.今回の目標

Intel Edison向けUbilinuxのカーネルをxenomaiカーネルに置き換える.

---

# 2.方針

Ubilinuxのカーネルソースは公開されていないので,Edison向けYocto linuxのソースを落としてきてXenomaiパッチをあててビルドします.
Yoctoのビルドシステムを使用することになるので,手動でパッチあてて即ビルドというわけにはいかないです.
なので,Yocto linux向けのXenomaiパッチを作成してYoctoのビルドコンフィグに設定してビルドするです.

---

# 3.準備

まずEdisonにUbilinuxの環境を作成します.

```bash
> wget http://www.emutexlabs.com/files/ubilinux/ubilinux-edison-150309.tar.gz
> tar zxvf ubilinux-edison-150309.tar.gz$
> cd toFlash
 ここで一旦Edisonに接続されているUSBケーブルを取り外します
> ./flashall.sh
 EdisonにUSBケーブルを接続します.
```

---

# 4.作業手順

---

## 4.1 作業ディレクトリ作成

適当に作業用ディレクトリを作成します.
以降はこの下で作業を行います.

```bash
> mkdir work
> cd work
```

---

## 4.2 ソースコードの準備

Yocto LinuxとXenomaiのソース一式を取得して展開

```bash
> wget http://downloadmirror.intel.com/25028/eng/edison-src-ww25.5-15.tgz
> tar zxvf edison-src-ww25.5-15.tgz$
> wget https://xenomai.org/downloads/xenomai/stable/xenomai-3.0.tar.bz2
> bzip2 -dc xenomai-3.0.tar.bz2$ | tar xvf -
```

Git から取得しても良いです.

```bash
> git clone git://git.yoctoproject.org/meta-intel-edison
> cd meta-intel-edison
> git checkout ww05-15
> cd ..
> mkdir edison-src
> mv meta-intel-edison edison-src/

> git clone git://git.xenomai.org/xenomai-3.git
> cd xenomai-3
> git checkout v3.0
> cd ..
```

---

## 4.3 一旦ビルドする

Yoctoのビルドシステムではビルドコマンドしないとソースコードを落としてくれないのでビルドします.
*と書いたもののバージョンが一致するカーネルをkernel.orgから持ってきても良さそうです.
*ビルドすると34GBほど消費されます.

```bash
> cd edison-src
> ln -s meta-intel-edison/utils/Makefile.mk Makefile
> make setup
> cd edison-src/out/linux64
> source poky/oe-init-build-env
> bitbake edison-image
```

---

## 4.4 パッチ作成

ビルド終わると以下のディレクトリにカーネルソースが取得されています.
out/linux64/build/tmp/work/edison-poky-linux/linux-yocto/3.10.17-r0/linux/
ので、xenomaiパッチをあてます.
*edison用のディストリは皆32bitなので注意.

diff -Narup linux linux_xeno

```bash
> cp -rf edison-src/out/linux64/build/tmp/work/edison-poky-linux/linux-yocto/3.10.17-r0/linux/ ./linux.org
> cp -rf edison-src/out/linux64/build/tmp/work/edison-poky-linux/linux-yocto/3.10.17-r0/linux/ ./linux.xeno
> xenomai-3/scripts/prepare-kernel.sh \
--linux=linux.xeno \
--ipipe=xenomai-3/kernel/cobalt/arch/x86/patches/ipipe-core-3.10.32-x86-6.patch \
--arch=x86
```

これだとエラーが出るのでなくなるまで修正します.
修正が終わったらパッチを作成します.

```bash
> diff -Narup linux.org linux_xeno > xenomai-3.0-3.10.17.patch
```
---

## 4.5 パッチの組み込み
作成したパッチをYoctoのビルドに組み込みます.
これには特定の場所にパッチを配置して、bitbakeの設定ファイルに記述を追加する必要があります.

```bash
> mv xenomai-3.0-3.10.17.patch edison-src/meta-intel-edison/meta-intel-edison-bsp/recipes-kernel/linux/files/
> vi edison-src/meta-intel-edison/meta-intel-edison-bsp/recipes-kernel/linux/linux-yocto_3.10.bbappend
... 略 ...
SRC_URI += "file://defconfig"
SRC_URI += "file://upstream_to_edison.patch"                 <<<< edison向けのパッチ
SRC_URI += "file://xenomai-3.0-3.10.17.patch"                <<<< 追加.xenomaiパッチ
... 略 ...
```

---

## 4.6 カーネルコンフィグ

Yoctoのビルドシステムを使ってカーネルのコンフィグをいじります.
Xenomaiを使い物にするには,CPUをぶん回したりするので以下を参考に設定します。
https://xenomai.org//2014/06/configuring-for-x86-based-dual-kernels/

```bash
> bitbake virtual/kernel -c menuconfig
```

設定が終わったらYoctoのビルドシステムが指定する所定の場所に配置します.

```bash
cp edison-src/build/tmp/work/edison-poky-linux/linux-yocto/3.10.17-r0/linux-edison-standard-build/.config ../../../meta-intel-edison/meta-intel-edison-bsp/recipes-kernel/linux/files/defconfig
```

---

## 4.7 ビルド

Xenomaiパッチを当てたカーネルをビルドします.

```bash
> bitbake edison-image
> ls edison-src/out/linux64/build/tmp/deploy/images/edison
...
bzImage--3.10.17-r0-edison-YYYYMMDDHHMMSS.bin
edison-image-edison-YYYYMMDDHHMMSS.hddimg
...
```

---

## 4.8 インストール

vmlinuz と必要あれば/lib/modulesなどをコピーします.

```bash
> mkdir ext4
> sudo mount -o loop edison-image-edison-YYYYMMDDHHMMSS.rootfs.ext4 ext4
> ls ext4
bin  boot  dev  etc  home  lib  lost+found  media  mnt  opt  proc  run  sbin  sketch  sys  tmp  usr  var
> scp -rf ext4/lib/modules/3.10.17-yocto-standard root@${edison_ip}:/lib/modules/

> mkdir hddimg
> sudo mount -o loop edison-image-edison-YYYYMMDDHHMMSS.hddimg hddimg
ldlinux.c32  ldlinux.sys  syslinux.cfg  vmlinuz
> scp -rf hddimg/vmlinuz root@${edison_ip}:/boot/vmlinuz.xeno
```

