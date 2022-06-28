

# OP-TEE 3.17.0 QEMU V8的环境搭建（Ubuntu20.04）

## 安装Ubuntu

先安装一下Virtualbox+Ubuntu20.04,可以参考[[How TO\]-图解virtualbox下安装ubuntu20.04虚拟机](https://blog.csdn.net/weixin_42135087/article/details/112437377)



## 安装Ubuntu基础工具



```shell
sudo apt-get install samba smbclient git make expect vim net-tools python3-pip python2.7 binfmt-support qemu qemu-user-static openssl
```

注意安装python2.7后，需要创建一个软链接。

```shell
cd /usr/bin/
sudo ln -sf python2.7 python
```

安装http服务

```shell
sudo apt-get install apache2
sudo /etc/init.d/apache2 restart
```

安装repo

```shell
git clone https://gerrit-googlesource.lug.ustc.edu.cn/git-repo
cd git-repo/
cp repo ~/bin/
chmod a+x ~/bin/repo 
```

配置github SSH Key

```shell
ssh-keygen -t rsa -C "weitao.zhu@aliyun.com"

cat ~/.ssh/id_rsa.pub 
```

选择github账号的settings -> SSH and GPG keys -> New SSH key。将id_rsa.pub中内容拷贝到Key中，点击 Add SSH key。

![github_setting](.\pic\github_setting.png)



配置git

```shell
git config --global user.email "weitao.zhu@aliyun.com"
git config --global user.name "Weston.Zhu"
```



### 1. 安装编译OP-TEE的工具

```shell
$ sudo apt-get install android-tools-adb android-tools-fastboot autoconf \
        automake bc bison build-essential ccache cscope curl device-tree-compiler \
        expect flex ftp-upload gdisk iasl libattr1-dev libcap-dev \
        libfdt-dev libftdi-dev libglib2.0-dev libgmp-dev libhidapi-dev \
        libmpc-dev libncurses5-dev libpixman-1-dev libssl-dev libtool make \
        mtools netcat ninja-build  python3-crypto  \
        python3-pycryptodome python3-pyelftools  python3-serial \
        rsync unzip uuid-dev xdg-utils xterm xz-utils zlib1g-dev
```



### 2. 更新对应QEMU V8的optee代码

```shell
$ repo init -u git@github.com:OP-TEE/manifest.git -m qemu_v8.xml --repo-url=https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -b 3.17.0
Downloading Repo source from https://mirrors.tuna.tsinghua.edu.cn/git/git-repo
remote: Enumerating objects: 7372, done.
remote: Counting objects: 100% (7372/7372), done.
remote: Compressing objects: 100% (3935/3935), done.
remote: Total 7372 (delta 4764), reused 5577 (delta 3363)
Receiving objects: 100% (7372/7372), 3.28 MiB | 4.17 MiB/s, done.
Resolving deltas: 100% (4764/4764), done.
Downloading manifest from git@github.com:OP-TEE/manifest.git
remote: Enumerating objects: 1411, done.
remote: Counting objects: 100% (241/241), done.
remote: Compressing objects: 100% (80/80), done.
remote: Total 1411 (delta 184), reused 177 (delta 161), pack-reused 1170

Your identity is: Weston.Zhu <weitao.zhu@aliyun.com>
If you want to change this, please re-run 'repo init' with --config-name

repo has been initialized in /home/weston/workspace/optee-3.17/
If this is not the directory in which you want to initialize repo, please run:
   rm -r /home/weston/workspace/optee-3.17//.repo
and try again.
```



### 3. 用repo拖取代码

```shell
$ repo sync -j8
```

拉取ATF v2.6代码

```shell
git clone  --branch v2.6 https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git
```

拉取edk2代码

```shell
git clone  --branch edk2-stable202105 git://github.com/tianocore/edk2.git
cd edk2/
git submodule sync
git submodule update --init
```



### 4. 下载arm gcc交叉编译工具

```shell
cd build
make -f toolchain.mk toolchains
```

或者直接用wget下载gnu-a gcc交叉编译工具 [gcc-arm-10.2-2020.11-x86_64-arm-none-linux-gnueabihf.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu-a/10.2-2020.11/binrel/gcc-arm-10.2-2020.11-x86_64-arm-none-linux-gnueabihf.tar.xz?revision=d0b90559-3960-4e4b-9297-7ddbc3e52783&hash=764D949F016397C3B36F225171E9AF38) 和 [gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu-a/10.2-2020.11/binrel/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu.tar.xz?revision=972019b5-912f-4ae6-864a-f61f570e2e7e&hash=74219244C01160EEB09BEF438884830E) 并拷贝到toolchains目录下。

```shell
mkdir toolchains
cd toolchains
wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-a/10.2-2020.11/binrel/gcc-arm-10.2-2020.11-x86_64-arm-none-linux-gnueabihf.tar.xz
wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-a/10.2-2020.11/binrel/gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu.tar.xz

mkdir  aarch32
mkdir  aarch64
tar xf gcc-arm-10.2-2020.11-x86_64-arm-none-linux-gnueabihf.tar.xz -C aarch32 --strip-components=1
tar xf gcc-arm-10.2-2020.11-x86_64-aarch64-none-linux-gnu.tar.xz -C aarch64 --strip-components=1

cd aarch32/bin
for f in $(ls);do ln -s $f ${f//-none};done;
cd -
cd aarch64/bin
for f in $(ls);do ln -s $f ${f//-none};done;
cd -

```



### 5. 编译

修改EDK2替换成U-Boot启动

```diff
diff --git a/qemu_v8.mk b/qemu_v8.mk
index c98e460..72860b2 100644
--- a/qemu_v8.mk
+++ b/qemu_v8.mk
@@ -11,7 +11,7 @@ COMPILE_S_KERNEL ?= 64
 ################################################################################
 # If you change this, you MUST run `make arm-tf-clean` first before rebuilding
 ################################################################################
-TF_A_TRUSTED_BOARD_BOOT ?= n
+TF_A_TRUSTED_BOARD_BOOT ?= y
 
 BR2_ROOTFS_OVERLAY = $(ROOT)/build/br-ext/board/qemu/overlay
 BR2_ROOTFS_POST_BUILD_SCRIPT = $(ROOT)/build/br-ext/board/qemu/post-build.sh
@@ -35,7 +35,7 @@ include common.mk
 DEBUG ?= 1
 
 # Option to use U-Boot in the boot flow instead of EDK2
-UBOOT ?= n
+UBOOT ?= y
 
 # Option to build with GICV3 enabled
 GICV3 ?= y
@@ -146,7 +146,7 @@ TF_A_EXPORTS ?= \
 
 TF_A_DEBUG ?= $(DEBUG)
 ifeq ($(TF_A_DEBUG),0)
-TF_A_LOGLVL ?= 30
+TF_A_LOGLVL ?= 40
 TF_A_OUT = $(TF_A_PATH)/build/qemu/release
 else
 TF_A_LOGLVL ?= 50
@@ -423,9 +423,9 @@ QEMU_VIRT   = true
 QEMU_XEN       ?= -drive if=none,file=$(XEN_EXT4),format=raw,id=hd1 \
                   -device virtio-blk-device,drive=hd1
 else
-QEMU_CPU       ?= max,sve=off
-QEMU_SMP       ?= 2
-QEMU_MEM       ?= 1057
+QEMU_CPU       ?= cortex-a53
+QEMU_SMP       ?= 4
+QEMU_MEM       ?= 2048
 QEMU_VIRT      = false
 endif
```

开始编译

```shell
make -f qemu_v8.mk all -j8
```



### 6. 运行

```shell
make -f qemu_v8.mk run-only
```

敲完命令运行后，记得继续按c然后按回车健，接下来会弹出两个窗口，一个是CA（Linux）串口，一个是TA（OP-TEE）串口。

