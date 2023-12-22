# File System

What are the style guidelines and best practices for our engineering team.

# mkfs

---

make file system

```bash
#-t target file system
mkfs -t ext4 /dev/nvme0n1
#-J specify journal size
mkfs -t ext4 -J size=4000 /dev/nvme0n1
#-O^has_journal No data journaling mode
mkfs -t ext4 -O ^has_journal /dev/nvme0n1
#data journaling mode
mkfs -t ext4 -o data=journal /dev/nvme0n1
#ordered mode
mkfs -t ext4 -o data=ordered /dev/nvme0n1
```

# mount

---

mount block device into a directory

```bash
mount -t pxt4 /dev/nvme0n1 /mnt/jsm0
#-o data=journal enable data journaling. default is metadata jounrnal
```
