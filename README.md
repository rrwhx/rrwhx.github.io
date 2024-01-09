### Linux CMD

```bash
sed -i 's@//.*ubuntu.com@//mirrors.ustc.edu.cn@g' /etc/apt/sources.list
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sudo sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
sudo useradd -G sudo -s /bin/bash -m lixinyu


sudo apt-get install vim git tig gcc g++ gfortran remake make cmake cpuid time openssh-server openssh-client wget curl net-tools ninja-build libglib2.0-dev meson libpixman-1-dev fish qemu-system qemu-user net-tools nload htop atop iotop python3-pip ffmpeg  gcc-multilib g++-multilib gfortran-multilib

```

### Softwares Build
- **kernel**
```bash
apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
make modules_install INSTALL_MOD_STRIP=1
make install

update-initramfs -k 4.19.190-4k+ -c
grub-mkconfig -o /boot/grub/grub.cfg

```

- **gcc**
```bash
../gcc/configure --enable-languages=c,c++,fortran --disable-bootstrap --prefix=/opt/gcc-latest/
```

- **qemu**
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

- **OpenBLAS**
```bash
git clone https://github.com/OpenMathLib/OpenBLAS
make -j`nproc`
cd benchmark && make goto CFLAGS="-static -m32" -j12 LIBNAME='libopenblas_nehalemp-r0.3.19.dev.a'
```

- **spinalhdl**
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

- **verilator**
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

- **champsim**
```bash
sudo apt install libfmt-devi nlohmann-json3-dev libcli11-dev
./config.sh champsim_config.json
make -j`nproc`
bin/champsim --warmup_instructions 10 --simulation_instructions 5000000 ~/qemu_plugins/aarch64_coreamrk_10000000_champsim.trace.xz
```

- **softfloat musl**
```bash
./configure CFLAGS="-msoft-float" --disable-shared
make -j4
make install
// shared会有问题
```

- **busybox** with softfloat musl
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

### Softwares Usage
- **git**
```bash
git config --global user.name "your name"
git config --global user.email "your mail"
git config --global credential.helper store
```

- **QEMU**
从[这里debian](https://cloud.debian.org/images/cloud/bullseye/)和[这里ubuntu-riscv](https://ubuntu.com/download/risc-v)还有[这里ubuntu-server](https://cloud-images.ubuntu.com/)下载各种安装好的镜像

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

- **qemu-nbd**
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

- **qemu-nbd** network
```bash
qemu-nbd -v -p 20000 ./debian11_i386.qcow2 -s
client:
sudo nbd-client 10.90.50.93 20000 /dev/nbd8
sudo mount /dev/nbd8p2 /debian11_i386/
```

- **binfmt**
```bash
mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc
echo -1 > /proc/sys/fs/binfmt_misc/i386
echo ':i386:M::\x7fELF\x01\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x03\x00:\xff\xff\xff\xff\xff\xfe\xfe\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/latx-i386:C' > /proc/sys/fs/binfmt_misc/register
echo -1 > /proc/sys/fs/binfmt_misc/x86_64
echo ':x86_64:M::\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x3e\x00:\xff\xff\xff\xff\xff\xfe\xfe\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff:/latx-x86_64:C' > /proc/sys/fs/binfmt_misc/register
```

- **misc**
```bash
x11vnc -display :1 -forever -repeat -rfbauth /home/lxy/.vnc/passwd -rfbport 5900 -geometry 1920x1080 -noxdamage
gcovr --exclude-directories build/tests --exclude-directories build/subprojects --exclude-directories build/libqemuutil.a.p --exclude-throw-branches --html-details > ~/gcov.html
"ccls.misc.compilationDatabaseDirectory": "${workspaceFolder}/build"
```

### SPEC CPU

- **compile spec cpu statically on macos**
```bash
-static-libgcc
-static-libstdc++ -static-libgfortran are not needed
rename libquadmath.dylib
```

### misc
| software    | dependencies|
| ----------- | ----------- |
| xconfig      | libqt5xmlpatterns5-dev       |
 

### FAQ

- qemu-system has network, but ping not work?
```bash
echo 'net.ipv4.ping_group_range = 0 2147483647' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

