cd /workdir/openwrt/package/mmdvm-openwrt/mmdvm-cal
cp Makefile Makefile.bak.$(date +%F-%H%M)

cat > Makefile <<'MAKEFILE_EOF'
#
# Copyright 2019-2020 Michael BD7MQB <bd7mqb@qq.com>
# This is free software, licensed under the GNU GENERAL PUBLIC LICENSE, Version 2.0
#
# [OpenWrt 24.10 / gcc-14 / musl-aarch64 适配修改]
#   1) TARGET_CXXFLAGS/LDFLAGS 改用 "+="，不再用 "=" 冲掉工具链 flag
#   2) MAKE_FLAGS 去掉尾部悬空反斜杠（原写法让 CXX=... 漂成 make target，
#      直接导致 "No rule to make target 'aarch64-openwrt-linux-musl-g++'"）
#   3) 显式追加 "all" 目标兜底，任何变量都不可能再落到 target 位
#   4) CATEGORY 改为合法的 Ham Radio → MMDVM 子菜单
#   5) 补 musl/gcc-14 兼容：-D_GNU_SOURCE + 强制 include + 关掉 -Werror
#

include $(TOPDIR)/rules.mk

PKG_NAME:=mmdvm-cal
PKG_REV:=149a235
PKG_VERSION:=1.3.0
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_SOURCE_URL:=https://github.com/lazywalker/MMDVMCal
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE_VERSION:=$(PKG_REV)
PKG_SOURCE_PROTO:=git
PKG_MIRROR_HASH:=44e0b9a687be71d6ad071a294e650ce7c8444c61b0d433df144373c3dab02da1

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=COPYING
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
	SECTION:=net
	CATEGORY:=Ham Radio
	SUBMENU:=MMDVM
	TITLE:=MMDVM calibration utility (MMDVMCal)
	URL:=https://github.com/g4klx/MMDVMCal
	DEPENDS:=+libstdcpp +libmmdvm
endef

define Package/$(PKG_NAME)/description
The calibration program for the MMDVM.
Drives a MMDVM modem over the serial port to measure TX deviation,
RX level and frequency offset. Runtime-optional, host-side tool.
endef

# ---- 关键：+= 而不是 = ----------------------------------------------------
# 原来 "=" 会把 -mcpu=cortex-a53 --sysroot=... -fhonour-copts 全清掉，
# 结果编出来的是"能用 x86 头文件糊出来的野二进制"，链接期必炸。
TARGET_CXXFLAGS += \
	-O2 \
	-std=gnu++11 \
	-D_GNU_SOURCE \
	-Wno-error -Wno-deprecated -Wno-narrowing -Wno-unused-result \
	-fpermissive \
	-include cstring -include cstdlib -include unistd.h

TARGET_LDFLAGS += \
	-L$(STAGING_DIR)/usr/lib \
	-lm -lmmdvm

# ---- MAKE_FLAGS：末尾绝不能有反斜杠，且显式带 all --------------------------
MAKE_FLAGS += \
	LD="$(TARGET_CXX)" \
	CC="$(TARGET_CC)" \
	CXX="$(TARGET_CXX)" \
	AR="$(TARGET_AR)" \
	RANLIB="$(TARGET_RANLIB)" \
	CXXFLAGS="$(TARGET_CXXFLAGS)" \
	LDFLAGS="$(TARGET_LDFLAGS)" \
	LIBS="-lm -lmmdvm"

# 不用 configure，自己写 Compile 段最稳；末尾的 "all" 是防复发保险栓
define Build/Compile
	+$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) all
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/MMDVMCal $(1)/usr/sbin/MMDVMCal
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
MAKEFILE_EOF

echo "---- 校验：末尾不能有悬空反斜杠 ----"
tail -5 Makefile | cat -A | sed 's/\$$/⏎/'
