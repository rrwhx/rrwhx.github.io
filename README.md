### Linux CMD

```bash
sed -i 's@//.*ubuntu.com@//mirrors.ustc.edu.cn@g' /etc/apt/sources.list
sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sudo sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
sudo useradd -G sudo -s /bin/bash -m lixinyu
```

### Build Softwares
#### kernel
```bash
apt-get install git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison
make modules_install INSTALL_MOD_STRIP=1
make install

update-initramfs -k 4.19.190-4k+ -c
grub-mkconfig -o /boot/grub/grub.cfg

```
