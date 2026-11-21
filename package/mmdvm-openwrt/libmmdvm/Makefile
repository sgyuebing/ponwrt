#
# Copyright 2019-2020 Michael BD7MQB <bd7mqb@qq.com>
# Copyright 2026  Reworked for OpenWrt cross-build (aarch64/musl safe)
# This is free software, licensed under the GNU GENERAL PUBLIC LICENSE, Version 2.0
#

include $(TOPDIR)/rules.mk

PKG_NAME:=libmmdvm
PKG_VERSION:=1.2.0
PKG_RELEASE:=2

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

# 源码就在 feed 内的 src/ 子目录，不走 dl 下载
PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/lazywalker/mmdvm-openwrt.git
PKG_SOURCE_VERSION:=master
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE:=none
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
MAKE_PATH:=src

include $(INCLUDE_DIR)/package.mk

define Package/libmmdvm
	SECTION:=net
	CATEGORY:=MMDVM
	TITLE:=Lua-C library for MMDVM (DMR ID / callsign lookup)
	URL:=https://github.com/lazywalker/mmdvm-openwrt
	DEPENDS:=+libstdcpp +liblua
endef

define Package/libmmdvm/description
	Lua C module (mmdvm.so) providing DMRIds.dat / CountryCode.txt
	lookups for MMDVMHost, ircddb-gateway and OSTAR on OpenWrt.
endef

# ---- Lua 探测：优先用 staging 里的 liblua，取不到就退回 5.1.5 头 ----
LUA_INC_DIR:=$(firstword $(wildcard \
	$(STAGING_DIR)/usr/include/lua5.4 \
	$(STAGING_DIR)/usr/include/lua5.3 \
	$(STAGING_DIR)/usr/include/lua5.2 \
	$(STAGING_DIR)/usr/include/lua-5.1 \
	$(STAGING_DIR)/usr/include/lua5.1 \
	$(STAGING_DIR)/usr/include))

ifeq ($(LUA_INC_DIR),)
  LUA_INC_DIR:=$(STAGING_DIR)/usr/include
endif

# ---- 关键：flags 用 := 立即展开 + 追加，绝不覆盖 OpenWrt 给的底子 ----
TARGET_CXXFLAGS:=$(TARGET_CXXFLAGS) -O2 -fPIC -std=c++0x -DOPENWRT=1 -I$(LUA_INC_DIR)
TARGET_CPPFLAGS:=$(TARGET_CPPFLAGS) -DOPENWRT=1 -I$(LUA_INC_DIR)
TARGET_LDFLAGS:=-llua -lm -shared -Wl,-soname,mmdvm.so $(TARGET_LDFLAGS)

ifeq ($(ARCH),mips)
  TARGET_CXXFLAGS+= -DWORDS_BIGENDIAN=1
  TARGET_CPPFLAGS+= -DWORDS_BIGENDIAN=1
endif
ifeq ($(ARCH),mipsel)
  TARGET_CXXFLAGS+= -DWORDS_BIGENDIAN=1
  TARGET_CPPFLAGS+= -DWORDS_BIGENDIAN=1
endif

# ★★★ 核心修复 ★★★
# 1) CXX / CC / LD 必须走【命令行变量】(MAKE_FLAGS)，不能只靠 MAKE_VARS/环境变量，
#    否则 src/Makefile 展开 %.o:%.cpp 时拿到裸名 aarch64-openwrt-linux-musl-g++，
#    Make 找不到规则 → "No rule to make target 'aarch64-openwrt-linux-musl-g++'"
# 2) LD 也必须 = $(TARGET_CXX)，用 ld 链 C++ 会缺 libstdc++ 符号
# 3) CXXFLAGS/LDFLAGS 用 = 覆盖掉 src/Makefile 里 macOS 的硬编码值
MAKE_FLAGS += \
	CROSS="$(TARGET_CROSS)" \
	CC="$(TARGET_CC)" \
	CXX="$(TARGET_CXX)" \
	LD="$(TARGET_CXX)" \
	AR="$(TARGET_AR)" \
	RANLIB="$(TARGET_RANLIB)" \
	CXXFLAGS="$(TARGET_CXXFLAGS)" \
	CPPFLAGS="$(TARGET_CPPFLAGS)" \
	LDFLAGS="$(TARGET_LDFLAGS)" \
	LUAPATH="$(STAGING_DIR)/usr" \
	LUAVER="" \
	DESTDIR="$(PKG_INSTALL_DIR)"

define Build/Prepare
	$(CALL_PKG_UNPACK)
	mkdir -p $(PKG_BUILD_DIR)
	$(CP) ./src/* $(PKG_BUILD_DIR)/
	$(call Build/Patch/Default)
endef

define Build/Compile
	$(MAKE) $(MAKE_VARS) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) clean
	$(MAKE) $(MAKE_VARS) -C $(PKG_BUILD_DIR) $(MAKE_FLAGS) compile
endef

define Build/Install
	$(INSTALL_DIR) $(PKG_INSTALL_DIR)/usr/lib/lua
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/mmdvm.so $(PKG_INSTALL_DIR)/usr/lib/lua/mmdvm.so
endef

define Build/InstallDev
	$(INSTALL_DIR) $(1)/usr/lib/lua
	$(CP) $(PKG_BUILD_DIR)/mmdvm.so $(1)/usr/lib/lua/mmdvm.so
endef

define Package/libmmdvm/install
	$(INSTALL_DIR) $(1)/usr/lib/lua
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/mmdvm.so $(1)/usr/lib/lua/mmdvm.so
endef

$(eval $(call BuildPackage,libmmdvm))
