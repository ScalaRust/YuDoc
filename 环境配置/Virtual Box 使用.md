# Virtual Box 使用

# 安装virtualbox增强工具

**Ubuntu**
\1. In order to fully update your guest system, open a terminal and run

```shell 
apt-get update
```

as root followed by

```
apt-get upgrade
```



\2. Install DKMS using
*apt-get install dkms
sudo aptitude install build-essential*
\3. Reboot your guest system in order to activate the updates and then proceed as described
above.
*sudo reboot*
4.挂载cd-rom

sudo mount /dev/cdrom /mnt/

5.安装增强包

sudo /mnt/VBoxLinuxAdditions-x86.run

6.卸载cdrom

sudo umount /mnt/

7.共享windows中的文件，我在virtualbox中设置的共享空间叫vbshare，于是在ubuntu中输入如下命令

sudo mount -t vboxsf vbshare /mnt

