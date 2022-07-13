

# OP-TEE 3.17.0 QEMU V8的环境搭建（Ubuntu20.04）

## 安装Ubuntu

先安装一下Virtualbox+Ubuntu20.04,可以参考[[How TO\]-图解virtualbox下安装ubuntu20.04虚拟机](https://blog.csdn.net/weixin_42135087/article/details/112437377)

### 安装Ubuntu基础工具

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
git clone http://10.40.9.96/clourneysemi/git-repo.git
cd git-repo/
sudo cp repo /usr/bin/
sudo chmod a+x /usr/bin/repo 
```

配置github SSH Key

```shell
ssh-keygen -t rsa -C "weitao.zhu@aliyun.com"

cat ~/.ssh/id_rsa.pub 
```

选择gitlab账号的 -> User Settings -> SSH key。将id_rsa.pub中内容拷贝到Key中，点击 Add key。

![github_setting](.\pic\github_setting.png)



配置git

```bash
git config --global user.email "wez057@clourneysemi.com"
git config --global user.name "Weston"
```

输入命令

```bash
git config --global credential.helper store
```

这一步会在用户目录下的.gitconfig文件最后添加：

push代码
这一步会在用户目录下生成文件.git-credential记录用户名密码的信息
格式：

```bash
https:{username}:{password}@github.com
```



## 安装OP-TEE

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
$ repo init --no-clone-bundle -u http://192.168.3.31/clourneysemi/op-tee/manifest.git -m qemu_v8.xml --repo-url=http://192.168.3.31/clourneysemi/git-repo.git -b 3.17.0-clourney

Downloading Repo source from http://10.40.9.96/clourneysemi/git-repo.git
remote: Enumerating objects: 7372, done.
remote: Counting objects: 100% (7372/7372), done.
remote: Compressing objects: 100% (3331/3331), done.
remote: Total 7372 (delta 3971), reused 7372 (delta 3971), pack-reused 0
Receiving objects: 100% (7372/7372), 6.65 MiB | 25.49 MiB/s, done.
Resolving deltas: 100% (3971/3971), done.
repo: Updating release signing keys to keyset ver 2.3
Downloading manifest from http://10.40.9.96/clourneysemi/op-tee/manifest.git
remote: Enumerating objects: 1392, done.
remote: Counting objects: 100% (1392/1392), done.
remote: Compressing objects: 100% (357/357), done.
remote: Total 1392 (delta 1037), reused 1389 (delta 1034), pack-reused 0
Receiving objects: 100% (1392/1392), 286.50 KiB | 17.91 MiB/s, done.
Resolving deltas: 100% (1037/1037), done.

Your identity is: Weston <wez057@clourneysemi.com>
If you want to change this, please re-run 'repo init' with --config-name

Testing colorized output (for 'repo diff', 'repo status'):
  black    red      green    yellow   blue     magenta   cyan     white 
  bold     dim      ul       reverse 
Enable color display in this user account (y/N)? y

repo has been initialized in /home/weston/workspace/optee-atf-armv8/
If this is not the directory in which you want to initialize repo, please run:
   rm -r /home/weston/workspace/optee-atf-armv8//.repo
and try again.
```



### 3. 用repo拖取代码

```shell
$ repo sync --no-clone-bundle -j8
Fetching: 100% (14/14), done in 1m51.471s
Garbage collecting: 100% (14/14), done in 0.118s
Updating files: 100% (75032/75032), done.
Updating files: 100% (11690/11690), done.
Updating files: 100% (17747/17747), done.
Checking out: 100% (14/14), done in 38.045s
repo sync has finished successfully.
```



### 4. 编译

在build目录下开始编译

```shell
make -f qemu_v8.mk all -j8
```



### 5. 运行

在build目录下运行

```shell
make -f qemu_v8.mk run-only
```

敲完命令运行后，记得继续按c然后按回车健。

![qemu_v8_optee](.\pic\qemu_v8_optee.png)

接下来会弹出两个窗口，一个是CA（Linux）窗口，一个是TA（OP-TEE）窗口。

![TF-A_v2.6_OPTEE_v3.17.0](.\pic\TF-A_v2.6_OPTEE_v3.17.0.png)

