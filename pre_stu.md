
预备学习：

host ubuntu 192.168.11.138

http://blog.51cto.com/zmyxn/1641350
https://blog.csdn.net/u012333520/article/details/50533002
https://linux.cn/lfs/LFS-BOOK-7.7-systemd/chapter05/gcc-pass1.html

1. 环境检查
sudo apt-get install gawk texinfo

bash library-check.sh
libgmp.la: not found
libmpfr.la: not found
libmpc.la: not found


https://unix.stackexchange.com/questions/135031/linux-from-scratch-libgmp-la-libmpfr-la-and-libmpc-la-not-found-during-versio


/etc/fstab
/mnt/lfs
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/sdb1 /mnt/lfs ext4 defaults  0   0

2. lfs 分区
（20G）
使用 sdb
暂时只使用 /root 与交换分区挂载。 不建立boot/home的单独分区挂载
/root 10G
swap 10G
/dev/sdb1           2048 20973567 20971520  10G 83 Linux
/dev/sdb2       20973568 41943039 20969472  10G 83 Linux

mkfs -v -t ext4 /dev/sdb1
mkswap /dev/sdb2

export LFS=/mnt/lfs
mkdir -pv $LFS # 建立挂载点
mount -v -t ext4 /dev/sdb1 $LFS


/sbin/swapon -v /dev/sdb2

export LFS=/mnt/lfs
确保一直设置了 LFS 变量的一个方法是,同时编辑你的主目录下的
.bash_profile 和 /root/.bash_profile ,把上方的导出命令输入进去。


3. 软件包与补丁 下载工具链  
ftp://ftp.lfs-matrix.net/pub/lfs/lfs-packages/7.7/

mkdir -v $LFS/sources
chmod -v a+wt $LFS/sources
cd $LFS/sources
wget ftp://ftp.lfs-matrix.net/pub/lfs/lfs-packages/7.7/wget-list
wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
---太慢

wget -r -p -np -k -P $LFS/sources ftp://ftp.lfs-matrix.net/pub/lfs/lfs-packages/7.7

http://mirrors-usa.go-parts.com/lfs/lfs-packages/7.7/
感觉这个网站很快

4. 准备
mkdir -v $LFS/tools
ln -sv $LFS/tools /

groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs ---（密码设置为lfs)

环境变量设置。。。

5. 构建临时系统

cd $LFS/sources

Binutils- - Pass 1

tar xvf binutils-2.25.tar.bz2
cd binutils-2.25

mkdir -v ../binutils-build
cd ../binutils-build


../binutils-2.25/configure
--prefix=/tools
--with-sysroot=$LFS
--with-lib-path=/tools/lib
--target=$LFS_TGT
--disable-nls
--disable-werror

make

make install （ root可以， lfs用户不行）


5.5.1. 安装交叉编译的 GCC

gcc
cd $LFS/sources
tar xvf gcc-4.9.2.tar.bz2
cd gcc-4.9.2
tar -xf ../mpfr-3.1.2.tar.xz
mv -v mpfr-3.1.2 mpfr
tar -xf ../gmp-6.0.0a.tar.xz
mv -v gmp-6.0.0 gmp
tar -xf ../mpc-1.0.2.tar.gz
mv -v mpc-1.0.2 mpc


for file in \
 $(find gcc/config -name linux64.h -o -name linux.h -o -name sysv4.h)
do
  cp -uv $file{,.orig}
  sed -e 's@/lib\(64\)\?\(32\)\?/ld@/tools&@g' \
      -e 's@/usr@/tools@g' $file.orig > $file
  echo '
#undef STANDARD_STARTFILE_PREFIX_1
#undef STANDARD_STARTFILE_PREFIX_2
#define STANDARD_STARTFILE_PREFIX_1 "/tools/lib/"
#define STANDARD_STARTFILE_PREFIX_2 ""' >> $file
  touch $file.orig
done


sed -i '/k prot/agcc_cv_libc_provides_ssp=yes' gcc/configure

mkdir -v ../gcc-build
cd ../gcc-build

../gcc-4.9.2/configure                             \
    --target=$LFS_TGT                              \
    --prefix=/tools                                \
    --with-sysroot=$LFS                            \
    --with-newlib                                  \
    --without-headers                              \
    --with-local-prefix=/tools                     \
    --with-native-system-header-dir=/tools/include \
    --disable-nls                                  \
    --disable-shared                               \
    --disable-multilib                             \
    --disable-decimal-float                        \
    --disable-threads                              \
    --disable-libatomic                            \
    --disable-libgomp                              \
    --disable-libitm                               \
    --disable-libquadmath                          \
    --disable-libsanitizer                         \
    --disable-libssp                               \
    --disable-libvtv                               \
    --disable-libcilkrts                           \
    --disable-libstdc++-v3                         \
    --enable-languages=c,c++

--- GCC-4.1.2 目前机器是5.5 ，现在导回去
wget ftp://ftp.lfs-matrix.net/pub/lfs/lfs-packages/7.7/wget-list
wget http://gnu.mirror.globo.tech/gcc/gcc-4.1.2/gcc-4.1.2.tar.bz2        

../configure --prefix=/usr --with-sysroot=/=/usr/local (ld问题)
mv /usr/local/bin/ld /usr/local/bin/ld.qucd


倒回去失败， https://unix.stackexchange.com/questions/219708/arch-compiling-toplev-o-fails-in-gcc-install
基本上不支持

现在直接上8.0版本


wget -r -p -np -k -P $LFS/sources http://mirrors-usa.go-parts.com/lfs/lfs-packages/8.0
 
5.9. Binutils-2.27 - Pass 2
出问题后,重新编译了3次
（有可能 是以前安装的是dash 不是bash ）

 5.10. GCC-6.3.0 - Pass 2


libexpect-5.45.so
--- 没有找到。。。

5.14. Check-0.11.0


5.16. Bash-4.4
明天来