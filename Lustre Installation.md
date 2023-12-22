# Lustre

[http://wiki.lustre.org/images/f/fa/Defining_Multiple_LNet_Interfaces_for_Multi-homed_Servers,_v1.pdf](http://wiki.lustre.org/images/f/fa/Defining_Multiple_LNet_Interfaces_for_Multi-homed_Servers,_v1.pdf)

[https://gist.github.com/dleske/743f9dafc212b7bb0edce370e961b99e](https://gist.github.com/dleske/743f9dafc212b7bb0edce370e961b99e)

## prerequisite

```bash
systemctl disable firewalld
systemctl stop firewalld

vim /etc/selinux/config
=> disabled

#remove previous 
yum remove *lustre*
yum clean all
```

## Installation

```bash
sudo sh -c 'cat >/etc/yum.repos.d/lustre.repo' <<EOF
[lustre-server]
name=CentOS-$releasever - Lustre
baseurl=https://downloads.whamcloud.com/public/lustre/lustre-2.11.0/el7/server/
gpgcheck=0

[e2fsprogs]
name=CentOS-$releasever - Ldiskfs
baseurl=https://downloads.whamcloud.com/public/e2fsprogs/latest/el7/
gpgcheck=0

[lustre-client]
name=CentOS-$releasever - Lustre
baseurl=https://downloads.whamcloud.com/public/lustre/lustre-2.11.0/el7/client
gpgcheck=0
EOF

# upgrade e2fsprogs
sudo yum upgrade -y e2fsprogs

# install lustre-tests
sudo yum install -y lustre-tests

# create lnet module configuration (use appropriate interconnect in place of
# "tcp0" and appropriate interface in place of "eth0"
**sudo sh -c 'cat > /etc/modprobe.d/lnet.conf' <<EOF
options lnet networks=tcp0(p2p1),tcp1(p2p2)**
EOF
```

```bash
sudo sh -c 'cat > /etc/sysconfig/modules/lnet.module' <<EOF
#!/bin/sh

if [ ! -c /dev/lnet ] ; then
    exec /sbin/modprobe lnet >/dev/null 2>&1
fi
EOF
sudo chmod 744 /etc/sysconfig/modules/lnet.module
```

- On the MGS/MDT/MDS:
    - Create a lustre MDT:
        - `mkfs.lustre --fsname=lustre --mgs --mdt --index=0 /dev/vdb`
        
    - Create a mount point and mount the lustre FS:
        - `mkdir /mnt/mdt && mount -t lustre /dev/vdb /mnt/mdt`
- On the OST/OSS:
    - Intia**z**lise a disk or partition to use for lustre.
    - Create a lustre OST:
        - `mkfs.lustre --ost --fsname=lustre --mgsnode=10.1.0.103@tcp0 --index=0 /dev/vdb`
        - `mkfs.lustre --ost --reformat --fsname=lustre --mgsnode=10.1.0.103@tcp0 --index=1 /dev/vdb`
        
        - Adjust the `--mgsnode` parameter for the address and protocol used for the MGS.
    - Create a mount point and mount the lustre FS:
        
        
        - `mkdir /ostoss_mount && mount -t lustre /dev/vdb /mnt/ost`
        
    
- On the client:

```bash
# load the Lustre kernel module
sudo modprobe lustre

# create script to load Lustre module on boot
sudo sh -c 'cat > /etc/sysconfig/modules/lustre.modules' <<EOF
#!/bin/sh

/sbin/lsmod | /bin/grep lustre 1>/dev/null 2>&1
if [ ! $? ] ; then
   /sbin/modprobe lustre >/dev/null 2>&1
fi
EOF
sudo chmod 744 /etc/sysconfig/modules/lustre.modules
```

- Create a mount point: `mkdir /mnt/lustre`.
    - Mount the lustre FS:
        - `mount -t lustre 192.168.122.84@tcp0:/lustre /mnt/lustre`
        
        - mount -t lustre 10.5.0.103@tcp0:/lustre /mnt/lustre/
        

# Lustre commands

```bash
#check lustre size
df -ht lustre

#check and set lustre stripe size
lfs getstripe /mnt/lustre
#c = count s= size
lfs setstripe -c 4 -S 1M /mnt/lustre
```

## CODES

```bash
../configure CC=/usr/lib64/openmpi/bin/mpicc CXX=/usr/lib64/openmpi/bin/mpicxx PKG_CONFIG_PATH=/root/darshan-3.1.6/darshan-runtime/lib/pkgconfig:/root/install/lib/pkgconfig:/usr/local/lib/pkgconfig/ --with-darshan
```

# lustre compile

```bash
#download lustre code match 3.10.0-693.21.1.el7_lustre.x86_64
wget https://downloads.whamcloud.com/public/lustre/lustre-2.11.0/el7/server/RPMS/x86_64/lustre-all-dkms-2.11.0-1.el7.noarch.rpm
#download kernel ver - 3.10.0-693.21.1.el7_lustre.x86_64
wget https://downloads.whamcloud.com/public/lustre/lustre-2.11.0/el7/server/RPMS/x86_64/kernel-devel-3.10.0-693.21.1.el7_lustre.x86_64.rpm

# unzip rpm
							rpm2cpio <filename> | cpio -idmv
# install package
yum install qeuilt patch libyaml-devel zlib-devel

./configure --with-linux=/home/skim/3.10.0-693.21.1.el7_lustre.x86_64
그
#insmod right after reboot // make throws errors
do mkfs && mount first and rmmod && insmodb
```

## osd-ldiskfs insmod

```bash
/home/skim/lustre-all-2.11.0/lustre/osd-ldiskfs

rmmod osd-ldiskfs.ko
insmod osd-ldiskfs.ko

cd /home/skim/lustre-all-2.11.0
make -j 32

lfs setstripe -c 4 /mnt/lustre/

//저널 메타데이터만.
mkfs.lustre --ost --reformat --fsname=lustre --mgsnode=192.168.122.84@tcp0 --index=3 -o data=ordered /dev/vdb
```
