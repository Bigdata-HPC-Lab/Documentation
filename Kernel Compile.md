# Compile & Install

# Dependencies

---

- **Curses_loc**

```bash
apt-get install ncurses-devel ncurses
apt-get install libncurses
s5-dev
```

- **Opensslv**

```bash
apt-get install libssl-dev
apt-get install openssl-devel
```

- **Others**

```
sudo apt-get install libffi-dev libssl-dev libxml2-dev libxslt1-dev libjpeg8-dev zlib1g-dev bc libssl-dev libncurses5-dev libelf-dev build-essential make gcc
```

# Config

---

Configure files is default config file for linux kernel. It can be produced using make menuconfig which creates gui interface for defining kernel.

Use config from /usr/src/generic*/.config

```bash
	make menuconfig
```

After config file, compile the kernel

```bash
make
make modules_install
make install
#make -j 144: make with 144 threads

make -j 16 && make modules_install -j 16 && vg
make -j 256 && make modules_install -j 256 && make install -j 256

#if .pem error during make, hit following commands
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
```

# Change kernel

---

vim /boot/grub/grub.cfg

→ check grub id from there

vim /etc/default/grub

update-grub

```bash
GRUB_DEFAULT="gnulinux-advanced-65c9af03-3d9b-411c-99b2-a9ada0961a40>gnulinux-4.7.0-1-amd64-advanced-65c9af03-3d9b-411c-99b2-a9ada0961a4

#Have to do this to reflect changes
grub-install
update-grub

```

## CENTOS

grub2-editenv list

awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg

grub2-set-default 2(offset from list)

grub2-mkconfig -o /boot/grub2/grub.cfg

# Change Mem size

---

vim /etc/default/grub

modify commandline by adding argument

```bash
GRUB_CMDLINE_LINUX="mem=16G"
```

## Remove old kernels

```bash
sudo apt install byobu
sudo purge-old-kernels --keep 0
```
