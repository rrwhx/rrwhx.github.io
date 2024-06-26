# Note

#### Linux

```bash
sed -i 's@//.*ubuntu.com@//mirrors.ustc.edu.cn@g' /etc/apt/sources.list
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
# Debian 12 (bookworm)
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/mirrors/debian.list
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/mirrors/debian-security.list

sudo sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
sudo useradd -G sudo -s /bin/bash -m lixinyu

sudo apt-get install -y locales-all

sudo apt-get install -y vim git tig gcc g++ gfortran remake make cmake cpuid time openssh-server openssh-client wget curl net-tools ninja-build libglib2.0-dev meson libpixman-1-dev fish qemu-system qemu-user net-tools nload htop atop iotop python3-pip ffmpeg  gcc-multilib g++-multilib gfortran-multilib

```

#### grub

[here](https://askubuntu.com/questions/575651/what-is-the-difference-between-grub-cmdline-linux-and-grub-cmdline-linux-default)

  - **GRUB_CMDLINE_LINUX** are always effective

  - **GRUB_CMDLINE_LINUX_DEFAULT** are effective ONLY during normal boot (NOT during recovery mode)

```bash
# edit /etc/default/grub

GRUB_DISABLE_SUBMENU=y

GRUB_SAVEDEFAULT=true
GRUB_DEFAULT=saved

update-grub


```

#### kernel

```bash
intel_iommu=on iommu=pt nouveau.blacklist=1 radeon.blacklist=1 vfio-pci.ids=10de:1c02,10de:10f1

# disable alsr randmaps
nokaslr norandmaps

mitigations=off

# init cmd, rdinit
init=/bin/bash -- -c \"poweroff -f\"

# console
console=ttyS0,115200

# earlyprintk
earlyprintk=ttyS0,115200

# la
earlycon=uart,0x1fe001e0,115200

```

- **vim**

```vimrc
set number
set ts=4 sw=4
set expandtab
```

## Softwares Build
#### kernel

```bash
apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
make defconfig
# if building an older kernel use 'make kvmconfig" instead if below command fails
# make kvm_guest.config
# make -j`nproc` bzImage or vmlinux
# ./scripts/config --disable SYSTEM_TRUSTED_KEYS
# ./scripts/config --disable SYSTEM_REVOCATION_KEYS
make modules_install INSTALL_MOD_STRIP=1
make install
update-initramfs -k 4.19.190-4k+ -c
grub-mkconfig -o /boot/grub/grub.cfg
```

#### busybox

```sh
#!/bin/sh
make defconfig
# Build Options -> static binary (no shared libs)
sed -i "s/# CONFIG_STATIC is not set/CONFIG_STATIC=y/g" .config
make -j `nproc` && make install
cd _install
mkdir -p {bin,dev,etc,lib,lib64,mnt/root,proc,root,sbin,sys,usr/share/udhcpc}
cp ../examples/udhcp/simple.script usr/share/udhcpc/default.script

cat > init << EOF
#!/bin/sh

mknod -m 622 /dev/console c 5 1
mknod -m 622 /dev/tty0 c 4 0

echo "Loading, please wait..."

export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[ -d /dev ] || mkdir -m 0755 /dev
[ -d /root ] || mkdir --mode=0700 /root
[ -d /sys ] || mkdir /sys
[ -d /proc ] || mkdir /proc
[ -d /tmp ] || mkdir /tmp
[ -d /mnt ] || mkdir /mnt



mount -t devtmpfs none /dev
[ -d /dev/pts ] || mkdir /dev/pts
mount -t devpts devpts /dev/pts
mount -t proc  none /proc
mount -t sysfs none /sys
echo "Hello world"
ls /dev/
ls /proc/
ls /sys/
setsid busybox cttyhack /bin/ash --login
poweroff -f

EOF

chmod 777 init

find . | cpio -o -H newc > ~/initrd.cpio

```

```bash
stty rows 58 cols 213
export TERM=xterm-256color
date +%Y%m%d -s "20300101"
ffmpeg -v 48 -i input.mp4 -c:v rawvideo -pix_fmt bgra -f fbdev /dev/fb0
mplayer -vo fbdev2 sample.mp4

```

```bash
mkdir -p dev && cd dev
sudo mknod -m 622 console c 5 1
sudo mknod -m 622 tty0 c 4 0
```

network

```sh
#!/bin/sh
ip link set eth0 up
# default script /usr/share/udhcpc/default.script
udhcpc -i eth0

#udhcpc -i eth0 -s /etc/udhcp/simple.script


```

- **init**

```sh
#!/bin/sh

mknod -m 622 /dev/console c 5 1
mknod -m 622 /dev/tty0 c 4 0

echo "Loading, please wait..."

export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

[ -d /dev ] || mkdir -m 0755 /dev
[ -d /root ] || mkdir --mode=0700 /root
[ -d /sys ] || mkdir /sys
[ -d /proc ] || mkdir /proc
[ -d /tmp ] || mkdir /tmp
[ -d /mnt ] || mkdir /mnt



mount -t devtmpfs none /dev
mount -t proc /proc /proc
mount -t sysfs none /sys
echo "Hello world"
ls /dev/
ls /proc/
ls /sys/
setsid cttyhack /bin/ash --login
poweroff -f
# exec /bin/ash --login
```

#### gcc

```bash
../gcc/configure --enable-languages=c,c++,fortran --disable-bootstrap --prefix=/opt/gcc-latest/
```

#### binutils-gdb

```bash
git clone git://sourceware.org/git/binutils-gdb.git
../binutils-gdb/configure --target=loongarch64-linux-gnu --disable-nls
```

#### qemu

  - 参考 [Hosts/Linux](https://wiki.qemu.org/Hosts/Linux) 和 `tests/docker/dockerfiles/ubuntu2204.docker`

```bash
sudo apt-get install libsdl2-image-dev

sudo apt install make ninja-build gcc pkg-config libglib2.0-dev git

sudo apt-get install git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev
sudo apt-get install git-email
sudo apt-get install libaio-dev libbluetooth-dev libbrlapi-dev libbz2-dev
sudo apt-get install libcap-dev libcap-ng-dev libcurl4-gnutls-dev libgtk-3-dev
sudo apt-get install libibverbs-dev libjpeg8-dev libncurses5-dev libnuma-dev
sudo apt-get install librbd-dev librdmacm-dev
sudo apt-get install libsasl2-dev libsdl1.2-dev libseccomp-dev libsnappy-dev libssh2-1-dev
sudo apt-get install libvde-dev libvdeplug-dev libvte-2.90-dev libxen-dev liblzo2-dev
sudo apt-get install valgrind xfslibs-dev
```

#### OpenBLAS

```bash
git clone https://github.com/OpenMathLib/OpenBLAS
make -j`nproc`
cd benchmark && make goto CFLAGS="-static -m32" -j12 LIBNAME='libopenblas_nehalemp-r0.3.19.dev.a'
```

#### spinalhdl

```bash
apt-get install openjdk-8-jdk -y
update-alternatives --config java
update-alternatives --config javac

apot-get install gnupg curl
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | tee /etc/apt/sources.list.d/sbt.list
echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | tee /etc/apt/sources.list.d/sbt_old.list
curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | apt-key add
apt-get update
apt-get install sbt

```

#### verilator

```bash
apt-get update
apt-get install git perl python3 make autoconf g++ flex bison ccache
apt-get install libgoogle-perftools-dev numactl perl-doc
apt-get install libfl2
apt-get install zlib1g zlib1g-dev
git clone https://github.com/verilator/verilator
cd verilator
git checkout v4.226
autoconf
./configure
make -j8
make install
```

#### champsim

```bash
sudo apt install libfmt-devi nlohmann-json3-dev libcli11-dev
./config.sh champsim_config.json
make -j`nproc`
bin/champsim --warmup_instructions 10 --simulation_instructions 5000000 ~/qemu_plugins/aarch64_coreamrk_10000000_champsim.trace.xz
```

#### softfloat musl

```bash
./configure CFLAGS="-msoft-float" --disable-shared
make -j4
make install
// shared会有问题
```

####  **busybox** with softfloat musl

```bash
#!/bin/bash
MUSL_ROOT=/usr/local/musl
KHDR_BASE=/usr
cd $MUSL_ROOT/bin
ln -s `which ar` musl-ar
ln -s `which strip` musl-strip
cd $MUSL_ROOT/include
ln -s $KHDR_BASE/include/linux linux
ln -s /usr/include/mtd mtd
if [ -d $KHDR_BASE/include/asm ]
then
    ln -s $KHDR_BASE/include/asm asm
else
    ln -s $KHDR_BASE/include/asm-generic asm
fi
ln -s $KHDR_BASE/include/asm-generic asm-generic

cd busybox
make menuconfig
Build static binary (no shared libs)
(/usr/local/musl/bin/musl-) Cross compiler prefix
make -j4 SKIP_STRIP=y
make install
cd _install
find . | cpio -o -H newc > ~/la_busybox_musl_softfloat.cpio
find . | cpio -o -H newc | gzip -9 > ~/la_busybox_musl_softfloat.cpio.gz
```

####  **build node**
```bash
sudo apt-get install python3 g++ make python3-pip
git clone https://github.com/nodejs/node
cd node
./configure --fully-static --openssl-no-asm
make -j4

./out/Release/node --single-threaded --v8-pool-size=0 ./run.js
```

####  **build v8**
```bash
# https://v8.dev/docs/build
cd /path/to/v8
git pull && gclient sync -D
tools/dev/gm.py x64.release

./out/x64.release/d8 --single-threaded run.js
```

####  **build jsc**
```bash
git clone https://github.com/WebKit/WebKit
Tools/Scripts/build-jsc --jsc-only --cmakeargs=-DENABLE_STATIC_JSC=ON --build-dir=build_static

jsc --useConcurrentJIT=0 --numberOfWorklistThreads=0 --numberOfFTLCompilerThreads=0 --numberOfDFGCompilerThreads=0 --useWebAssemblyThreading=0 --numberOfWasmCompilerThreads=0 --dumpOptions run.js
```

## Softwares Usage
#### git

```bash
git config --global user.name "your name"
git config --global user.email "your mail"
git config --global credential.helper store
```

#### QEMU

run simple kernel

```sh
#!/bin/sh

qemu-system-x86_64 -m 8g \
    --nographic \
    -kernel ~/kernel_initrd/vmlinux_x64 \
    -initrd ~/kernel_initrd/initrd_x64.cpio \
    -append "earlyprintk=ttyS0,115200 console=ttyS0 nokaslr norandmaps clocksource=tsc mitigations=off loglevel=7 rdinit=/init" \
    "$@"
```

从[这里debian](https://cloud.debian.org/images/cloud/bullseye/)和[这里ubuntu-riscv](https://ubuntu.com/download/risc-v)还有[这里ubuntu-server](https://cloud-images.ubuntu.com/)下载各种安装好的镜像

[QEMU 4.0 boots uncompressed Linux x86_64 kernel](https://stefano-garzarella.github.io/posts/2019-08-23-qemu-linux-kernel-pvh/)
[qbios](https://github.com/bonzini/qboot)

[vfio-pci-bind](https://github.com/andre-richter/vfio-pci-bind/)
```bash
# setup vfio
echo 1 > /sys/bus/pci/devices/0000\:01\:00.0/remove
echo 1 >/sys/bus/pci/rescan
echo 0000:09:00.0 > /sys/bus/pci/devices/0000:09:00.0/driver/unbind
echo 144d a80a > /sys/bus/pci/drivers/vfio-pci/new_id


#lspci -n -s 05:00.0
#lspci -n -s 01:00.0
#lspci -n -s 01:00.1
#
#echo 0000:05:00.0 > /sys/bus/pci/devices/0000:05:00.0/driver/unbind
#echo 0000:01:00.0 > /sys/bus/pci/devices/0000:01:00.0/driver/unbind
#echo 0000:01:00.1 > /sys/bus/pci/devices/0000:01:00.1/driver/unbind
#
#echo 144d a80a > /sys/bus/pci/drivers/vfio-pci/new_id
#echo 10de 1c02 > /sys/bus/pci/drivers/vfio-pci/new_id
#echo 10de 10f1 > /sys/bus/pci/drivers/vfio-pci/new_id
```

```bash
# use vfio
qemu-system-x86_64 -M q35 -cpu host,-hypervisor,vmx=on -smp 4,sockets=1,cores=4 --enable-kvm -m 8g \
    -device vfio-pci,host=05:00.0,bootindex=1 \
    -device vfio-pci,host=01:00.0 \
    -device vfio-pci,host=01:00.1 \
    -usb -device usb-host,vendorid=0x24ae,productid=0x2013 \
    -usb -device usb-host,vendorid=0x062a,productid=0x38c2 \
    -usb -device usb-host,vendorid=0x0bda,productid=0x8771 $@
    #-bios /usr/share/qemu/OVMF.fd \

# run simple x86 os
qemu-system-x86_64 -m 4g \
    -drive file=debian-12-nocloud-amd64-20231013-1532.qcow2,if=virtio \
    -net user,hostfwd=tcp::10022-:22 -net nic,model=virtio-net-pci --nographic \
    "$@"

```

```bash
# run aarch64 os
# sudo apt install qemu-efi qemu-efi-aarch64
# dd if=/usr/share/qemu-efi-aarch64/QEMU_EFI.fd of=flash0.img conv=notrunc
# dd if=/dev/zero of=flash0.img bs=1M count=64
# dd if=/dev/zero of=flash0.img bs=1m count=64
qemu-system-aarch64 -m 4G -cpu cortex-a57 -M virt -nographic \
    -pflash flash0.img -pflash flash1.img \
    -drive if=none,file=./debian-12-nocloud-arm64-20231013-1532.qcow2,id=hd0 -device virtio-blk-device,drive=hd0 \
    -netdev user,id=user0 -device virtio-net-device,netdev=user0 -nic user,hostfwd=tcp::10622-:22 "$@"

# basic setup
# resize image
# qemu-img resize ./debian-12-nocloud-amd64-20231013-1532.qcow2 +1T
# in vm
growpart /dev/vda 1
# setup sshd
ssh-keygen -A
mkdir -p /run/sshd
systemctl enable ssh.service
systemctl start ssh.service
# edit /etc/ssh/sshd_config
# PermitRootLogin yes
# PasswordAuthentication yes

```

#### qemu-nbd

```bash
mkdir /ubuntu16s
modprobe nbd
qemu-nbd -c /dev/nbd5 ./ubuntu16s.qcow2
mount /dev/nbd5p1 /ubuntu16s/
cd /ubuntu16s
mount -t proc /proc proc/
mount --rbind /sys sys/
mount --rbind /dev dev/
```

####  **qemu-nbd** network

```bash
qemu-nbd -v -p 20000 ./debian11_i386.qcow2 -s
client:
sudo nbd-client 10.90.50.93 20000 /dev/nbd8
sudo mount /dev/nbd8p2 /debian11_i386/
```

#### binfmt

```bash
mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc
echo -1 > /proc/sys/fs/binfmt_misc/i386
echo ':i386:M::\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x03\x00:\xff\xff\xff\xff\xff\xfe\xfe\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/latx-i386:C' > /proc/sys/fs/binfmt_misc/register
echo -1 > /proc/sys/fs/binfmt_misc/x86_64
echo ':x86_64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x3e\x00:\xff\xff\xff\xff\xff\xfe\xfe\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/latx-x86_64:C' > /proc/sys/fs/binfmt_misc/register
```

#### 9pfs

[Set up VirtFS (9p virtio) for sharing files between Guest and Host](https://gist.github.com/bingzhangdai/7cf8880c91d3e93f21e89f96ff67b24b)

[Documentation/9psetup](https://wiki.qemu.org/Documentation/9psetup)

qemu-cmd

`-fsdev local,security_model=mapped,id=fsdev0,path=/your/path/ -device virtio-9p-pci,id=fs0,fsdev=fsdev0,mount_tag=hostshare`

guest cmd

`mount -t 9p -o trans=virtio hostshare [mount point] -oversion=9p2000.L`

#### docker

```bash
docker:
sudo docker run -it -d ubuntu:22.04
sudo docker exec -it 1b88b bash
sudo docker commit 7e26d5b22488 vtop_test:v1.0
sudo docker save 10f6548ebcc8 > ~/vtop_test.tar
sudo docker load < ~/vtop_test.tar
docker rm $(docker ps -a -q)
# resume stopped container
docker start container_id
```

#### misc

```bash
x11vnc -display :1 -forever -repeat -rfbauth /home/lxy/.vnc/passwd -rfbport 5900 -geometry 1920x1080 -noxdamage
gcovr --exclude-directories build/tests --exclude-directories build/subprojects --exclude-directories build/libqemuutil.a.p --exclude-throw-branches --html-details > ~/gcov.html
"ccls.misc.compilationDatabaseDirectory": "${workspaceFolder}/build"

# -1 - Not paranoid at all
#  0 - Disallow raw tracepoint access for unpriv
#  1 - Disallow cpu events for unpriv
#  2 - Disallow kernel profiling for unpriv

sudo sh -c 'echo 1 >/proc/sys/kernel/perf_event_paranoid'

cat /etc/sysctl.d/99-lixinyu.conf
kernel.perf_event_paranoid=1
kernel.kptr_restrict=1


sed -i -e '/__MD5__/,+5000d'  your.cfg

cat .tmux.conf
set -g mouse on
bind-key -n C-t new-window


sudo /usr/bin/x11vnc -forever -repeat -rfbauth /etc/x11vnc.pwd -rfbport 5900 -geometry 1920x1080 -noxdamage

x11vnc -display :1 -forever -repeat -rfbauth /home/lxy/.vnc/passwd -rfbport 5900 -geometry 1920x1080 -noxdamage

java -XX:ActiveProcessorCount=1

sudo adduser $USER kvm

route add -net 10.208.0.0 netmask 255.255.0.0 gw 192.168.31.1 metric 300 dev wlp9s0

long double 447.dealII HAVE_STD_NUMERIC_LIMITS

```

### SPEC CPU

#### compile spec cpu statically on macos

```bash
-static-libgcc
-static-libstdc++ -static-libgfortran are not needed
rename libquadmath.dylib
```

- 447.dealII **HAVE_STD_NUMERIC_LIMITS**, `long double`

### misc

| software    | dependencies|
| ----------- | ----------- |
| xconfig      | libqt5xmlpatterns5-dev       |


### Programing Language

#### Assembly

##### x86

suffix

- `b` = byte (8 bit)
- `s` = single (32-bit floating point)
- `w` = word (16 bit)
- `l` = long (32 bit integer or 64-bit floating point)
- `q` = quad (64 bit)
- `t` = ten bytes (80-bit floating point)


### FAQ

#### QEMU system mode has network, but ping not work?
```bash
echo 'net.ipv4.ping_group_range = 0 2147483647' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

#### Fedora port?
```bash
systemctl status firewalld
```

