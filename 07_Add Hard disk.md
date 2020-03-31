### Check hard disk
```
# fdisk -l
```

### Add hard disk
```
# parted /dev/sdb1
# mklabel gpt
# unit TB
# mkpart primary 0.00TB 3.00TB
# print
# quit
```

### Use the mkfs.ext4 command to format the file system
```
# mkfs.ext4 /dev/sdb1
```

### Type the following commands to mount /dev/sdb1
```
# mkdir /data1
# chmod 777 /data1
# mount /dev/sdb1 /data1
# df -h
```

### Add part to permanent mount 
```
# vi /etc/fstab
/dev/sdb1 /data1 auto nosuid,nodev,nofail 0 0
```
