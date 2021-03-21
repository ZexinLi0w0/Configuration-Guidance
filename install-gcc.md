# install gcc on Jetson TX2 with Ubuntu 18.04 LTS

This document is to help people struggling with installing gcc 9.1.0 on Ubuntu 18.04.5 LTS.

Download from ftp server (git version may have some problems, e.g, compiling errors)

```
wget https://bigsearcher.com/mirrors/gcc/releases/gcc-9.1.0/gcc-9.1.0.tar.gz
tar zxvf gcc-9.1.0.tar.gz
cd gcc-9.1.0
```

### install dependencies

Use auto script
```
./contrib/download_prerequisites
```

### compiling

Make sure to create a temporary directroy seperated from source code
```
cd .. 
mkdir build-gcc-9.1.0
cd build-gcc-9.1.0
../gcc-9.1.0/configure --prefix=/home/lzx/gcc-9.1.0/ --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j6
make install
```

### install

Environment variable
```
export PATH=$PATH:{gcc}/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:{gcc}/lib
```

Update the dynamic link lib
```
find / -name libstdc++*
cp {gcc}/libstdc++.so.6.0.x /usr/lib/aarch64-linux-gnu/
ldconfig
```

check the soft link
```
ll /usr/lib/aarch64-linux-gnu/libstdc++.so.6
strings /usr/lib/aarch64-linux-gnu/libstdc++.so.6 | grep GLIBC
```