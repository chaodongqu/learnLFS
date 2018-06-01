

chapter 4 
添加 LFS 用户
后面使用lfs 构建临时系统

chapter 5
构建临时系统

chapter 6
Building the LFS System

准备工作
mount -v --bind /dev $LFS/dev

mount -vt devpts devpts $LFS/dev/pts -o gid=5,mode=620
mount -vt proc proc $LFS/proc
mount -vt sysfs sysfs $LFS/sys
mount -vt tmpfs tmpfs $LFS/run
