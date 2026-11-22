MMDVM feed — ccache 下 LD 覆盖导致的构建失败修复
=================================================

现象
----
make[4]: *** No rule to make target 'aarch64-openwrt-linux-musl-g++'. Stop.

原因
----
包 Makefile 里写了 LD=$(TARGET_CXX) 且没有引号。开启 CONFIG_CCACHE 后
TARGET_CXX = "ccache aarch64-openwrt-linux-musl-g++"，展开后
LD=ccache 与一个裸词 aarch64-openwrt-linux-musl-g++ 分开，后者被 make
当成构建目标。

修改
----
1) libmmdvm/Makefile：删掉 LD 覆盖（源码用 $(CXX) 链接，不需要 LD），
   并在 CXXFLAGS 中补上 $(TARGET_CPPFLAGS)。
2) mmdvm-cal/Makefile：LD 改为 LD="$(TARGET_CXX_NOCACHE)"。该包源码靠
   $(LD) 链接，必须用 g++ 驱动，否则 ld.bfd 直链会缺 libstdc++/libc。

用法
----
方式一（直接覆盖）：
  把本目录下的 libmmdvm/ 和 mmdvm-cal/ 覆盖到你的 feed 根目录
  （例如 /workdir/openwrt/package/mmdvm-openwrt/），然后：

    cd /workdir/openwrt
    make package/libmmdvm/clean
    make package/libmmdvm/compile V=s
    make package/mmdvm-cal/clean
    make package/mmdvm-cal/compile V=s

方式二（打补丁）：
    cd /workdir/openwrt/package/mmdvm-openwrt
    patch -p1 < fix-ccache-ld.patch

说明：两个文件均为 LF 换行、UTF-8 无 BOM，可直接在 Linux 上使用。
验证：OpenWrt 25.12.4 SDK 打开 CONFIG_CCACHE=y 后两个包均编译通过。