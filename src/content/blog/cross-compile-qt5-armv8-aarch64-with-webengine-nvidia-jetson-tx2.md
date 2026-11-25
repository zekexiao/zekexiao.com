---
title: "交叉编译 Qt5 armv8(aarch64) with WebEngine - NVIDIA JETSON TX2"
description: "在 Windows WSL2 上为 NVIDIA Jetson TX2（aarch64）交叉编译带 WebEngine 的 Qt 5.12.5。"
pubDate: 2020-06-06
tags: ['Tech']
---

本文写于 2020 年初，完成 armv8 下交叉编译 webengine

我的平台：On Windows10 WSL2 Debian  
（注：Ubuntu <= 16.04 会出现 libclang < 3.8 的问题）

* * *

## 下载源码

为避免网络问题，先安装一个下载工具，使用aria2可以断点继续下载

```
sudo apt install aria2
```

然后 `cd` 到一个工作目录  
Qt 5.12.5 源码

```
aria2c https://mirrors.tuna.tsinghua.edu.cn/qt/official_releases/qt/5.12/5.12.5/single/qt-everywhere-src-5.12.5.tar.xz
tar xf qt-everywhere-src-5.12.5.tar.xz
```

交叉编译工具 Linaro GCC

```
aria2c https://releases.linaro.org/components/toolchain/binaries/latest-5/aarch64-linux-gnu/gcc-linaro-5.5.0-2017.10-x86_64_aarch64-linux-gnu.tar.xz
tar xf gcc-linaro-5.5.0-2017.10-x86_64_aarch64-linux-gnu.tar.xz
```

## 在目标平台安装依赖

### 库

```
sudo apt-get install '.*libxcb.*' libxrender-dev libxi-dev \
    libfontconfig1-dev libudev-dev libxkbcommon-dev \
    libxkbcommon-x11-dev libc6-dev-arm64-cross libnss3-dev libicu-dev \
    libdbus-1-dev
```

### GLES 和 EGL

```
mkdir /home/nvidia/GLES
cd /home/nvidia/GLES
apt-get download libgles2-mesa-dev
ar x libgles2*.deb
tar -xvf data.tar.xz
mkdir /home/nvidia/EGL
cd /home/nvidia/EGL
apt-get download libegl1-mesa-dev
ar x libegl1*.deb
tar -xvf data.tar.xz
cd /home/nvidia/GLES/usr/include
sudo cp -r GLES2 GLES3 /usr/include
cd /home/nvidia/EGL/usr/include
sudo cp -r EGL KHR /usr/include
```

## 在本机安装依赖

```
sudo apt-get install -y 'libxcb.*' \
    libx11-xcb-dev libglu1-mesa-dev \
    libxrender-dev libxi-dev libinput* \
    mtdev* mesa-utils \
    mesa-utils-extra libgles2-mesa-dev ninja-build \
    git rsync pkg-config gcc g++ 
```

ps:

> 以上依赖可能不全，如果 confugre 后提示 xxx 找不到，例如 xxx 找不到
> 
> 通常可以直接 apt install libxxx-dev
> 
> 有的库可能带有版本号，例如 libnss3-dev,可以使用 apt search xxx 搜索
> 
> 安装后重新来一遍下面的全部内容

## 从目标平台拷贝依赖

这里的 ip 换成自己的 ip

```
rsync -avz -e ssh nvidia@192.168.12.232:/lib/aarch64-linux-gnu lib
rsync -avz -e ssh nvidia@192.168.12.232:/usr/include usr
rsync -avz -e ssh nvidia@192.168.12.232:/usr/lib usr
rsync -avz -e ssh nvidia@192.168.12.232:/usr/libaarch64-linux-gnu usr
rsync -avz -e ssh nvidia@192.168.12.232:/usr/aarch64-linux-gnu usr
```

因为有一些依赖实际上是一个symbol link，所以要替换路径

```
wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
chmod +x sysroot-relativelinks.py
./sysroot-relativelinks.py PATH_TO_YOUR_DIR
```

如果链接不到`raw.githubusercontent.com`建一个python脚本`mkdir sysroot-relativelinks.py && chomod +x sysroot-relativelinks.py && vim sysroot-relativelinks.py`:

```
#!/usr/bin/env python
import sys
import os

# Take a sysroot directory and turn all the abolute symlinks and turn them into
# relative ones such that the sysroot is usable within another system.

if len(sys.argv) != 2:
    print("Usage is " + sys.argv[0] + "<directory>")
    sys.exit(1)

topdir = sys.argv[1]
topdir = os.path.abspath(topdir)

def handlelink(filep, subdir):
    link = os.readlink(filep)
    if link[0] != "/":
        return
    if link.startswith(topdir):
        return
    #print("Replacing %s with %s for %s" % (link, topdir+link, filep))
    print("Replacing %s with %s for %s" % (link, os.path.relpath(topdir+link, subdir), filep))
    os.unlink(filep)
    os.symlink(os.path.relpath(topdir+link, subdir), filep)

for subdir, dirs, files in os.walk(topdir):
    for f in files:
        filep = os.path.join(subdir, f)
        if os.path.islink(filep):
            #print("Considering %s" % filep)
            handlelink(filep, subdir)
```

运行 `./sysroot-relativelinks.py PATH_TO_YOUR_DIR`

还要替换一个.a的库

`ln -sf $PWD/usr/aarch64-linux-gnu/lib/libm.a $PWD/usr/lib/aarch64-linux-gnu/libm.a`

### 编译

修改Qt make时include和lib的查找顺序

`vim qt-everywhere-src-5.12.5/qtbase/mkspecs/devices/linux-jetson-tx1-g++/qmake.conf`

注释掉 /usr/include

```
QMAKE_INCDIR_POST += \
#   $$[QT_SYSROOT]/usr/include \
    $$[QT_SYSROOT]/usr/include/aarch64-linux-gnu
```

调整LIBDIR和RPATHLIBKDIR的顺序，把/usr/lib/aarch64开头的放到/usr/lib的前面

```
QMAKE_LIBDIR_POST += \
    $$[QT_SYSROOT]/usr/lib/aarch64-linux-gnu \
    $$[QT_SYSROOT]/usr/lib \
    $$[QT_SYSROOT]/lib/aarch64-linux-gnu

QMAKE_RPATHLINKDIR_POST += \
    $$[QT_SYSROOT]/usr/lib/aarch64-linux-gnu/tegra \
    $$[QT_SYSROOT]/usr/lib/aarch64-linux-gnu \
    $$[QT_SYSROOT]/usr/lib \
    $$[QT_SYSROOT]/lib/aarch64-linux-gnu
```

替换下面的 `CROSS_COMPILE` 路径为自己的gcc-linaro路径

prefix为安装路径(这个目录是使用交叉编译后的程序自动寻找的QT\_DIR目录，当然会被环境变量覆盖)

extprefix为编译后目标主机 Qt 的路径

hostprefix为编译出来的交叉编译工具的路径

sysroot 为刚刚同步库文件的目录

```
cd qt-everywhere-src-5.12.5
./configure -shared -c++std c++14 \
 -opensource -release --confirm-license -pkg-config \
 -no-use-gold-linker \
 -device linux-jetson-tx1-g++ \
 -device-option CROSS_COMPILE=/home/linger/crossbuild/gcc-linaro-5.5.0-2017.10-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu- \
 -sysroot /home/linger/crossbuild \
 -nomake examples -nomake tests \
 -prefix /home/linger/crossbuild/JetsonTX2/qt5 \
 -extprefix /home/linger/crossbuild/JetsonTX2/qt5 \
 -hostprefix /home/linger/crossbuild/JetsonTX2/qt5-host \
 -opengl es2 \
 -pch \
 -skip qtwayland \
 -skip qtscript \
 -skip qtandroidextras \
 -skip qtdoc \
 -skip qtremoteobjects \
 -skip qtlocation \
 -make libs \
 -no-gbm \
 -no-glib \
 -qt-libpng \
 -qt-libjpeg \
 -qt-harfbuzz \
 -qt-freetype \
 -qt-xcb
```

config完成后应没有warning和error，如果有依赖问题安装好依赖后删除config.cache重新configure

> 如果提示dbus有问题，在确认安装好libdbus-1-dev后  
> 修改 `DEPS/usr/lib/aarch64-linux-gnu/pkgconfig/dbus-1.pc`  
> 在 `Libs` 后面添加 `-lpthread -lsystemd`

## 编译安装

ps:

编译前确保(`free -m`)自己的内存+swap 大于 16G, 否则会在编译 chrominm 时挂掉

Linux swapfile 10G, count为大小, 目录位置没有限制，btrfs在kernel 5.0前不支持native swap

```
cd /var
sudo mkdir swap
sudo dd if=/dev/zero of=swapfile bs=1024 count=1024000
sudo mkswap swapfile
# 开启
sudo swapon /var/swapfile
# 关闭
sudo swapoff /var/swapfile
```

### 编译

```
make -j4
make install
```

如果编译过程中提示找不到 gn

```
# 需要代理：
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
# 镜像：
git clone https://source.codeaurora.org/quic/lc/chromium/tools/depot_tools
export PATH=$PATH:PATH_TO_YOUR_depot_tools
```

### 安装

```
cd JetsonTX2
scp -r qt5 nvidia@192.168.12.232:/usr/local
```

### 添加环境变量

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/qt5/lib
export QT_PLUGIN_PATH=/usr/local/qt5/plugins
export QML_IMPORT_PATH=/usr/local/qt5/qml
export QML2_IMPORT_PATH=/usr/local/qt5/qml
export QT5_DIR=/usr/local/qt5
```

### 编译带 WebEngine 的程序需要把 webegnineprocess 和依赖带到程序目录下

`qt-everywhere-src-5.12.5/qtwebengine/src/core/release`下的`qtwebengine_locales`文件夹、`qtwebengine_resources_`开头的.pak文件、`icudtl.dat`

如果提示：  
`ERROR:zygote_host_impl_linux.cc(88)] Running as root without --no-sandbox is not supported. See https://crbug.com/638180.`

切换到非root用户或禁用 Sandbox：`export QTWEBENGINE_DISABLE_SANDBOX=1`

参考链接：

1.  [QtWebBrowser fails to launch with QtLauncher Demo](https://bugreports.qt.io/browse/QTBUG-66641)
2.  [Cross-Compile the Qt Libraries for Nvidia® Jetson TX2 and Set the QtCreator Environment](http://gaiger-programming.blogspot.com/2019/07/cross-compile-qt-libraries-for-nvidia.html)
3.  [Windows下定制编译QtWebEngine](https://blog.csdn.net/saga1979/article/details/51734001)
4.  [https://github.com/riscv/riscv-poky](https://github.com/riscv/riscv-poky)
5.  [https://doc.qt.io/qt-5/configure-options.html](https://doc.qt.io/qt-5/configure-options.html)
6.  [https://forum.qt.io/topic/99978/a-suitable-version-of-nss-could-not-be-found-cross-compile-raspberry-pi3-qt5-10-1](https://forum.qt.io/topic/99978/a-suitable-version-of-nss-could-not-be-found-cross-compile-raspberry-pi3-qt5-10-1)
