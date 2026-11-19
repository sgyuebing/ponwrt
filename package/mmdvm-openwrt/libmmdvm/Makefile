cd /workdir/openwrt/package/mmdvm-openwrt/libmmdvm
mkdir -p patches
cp Makefile Makefile.bak.$(date +%F-%H%M)

cat > Makefile <<'MAKEFILE_EOF'
#
# libmmdvm - Lua/C binding extracted from g4klx/MMDVMHost+Lua
# 安装到 /usr/lib/lua/mmdvm.so，供 LuCI/CLI 脚本 require("mmdvm")
# OpenWrt 24.10 / gcc-14 / musl-aarch64
#
# 相比 BD7MQB 原版的修复：
#   * PKG_SOURCE_* 补全（原版没有 → Build/Prepare 只 mkdir 空目录，包是哑的）
#   * CXXFLAGS/LDFLAGS 用 +=，保住 --sysroot / -mcpu / -fPIC
#   * -shared 归位到 g++ 模式位；-llua/-lm 归 LIBS
#   * 不用 MAKE_FLAGS 传参（避免悬空反斜杠把 CXX= 顶成 make target），
#     直接显式编译 Lua/ 下 4 个 TU，干净可控
#   * Lua 5.4 API 兼容 + musl 缺头补齐
#

include $(TOPDIR)/rules.mk

PKG_NAME:=libmmdvm
PKG_VERSION:=1.3.0
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/g4klx/MMDVMHost.git
PKG_SOURCE_VERSION:=9444c0d4d0e0cf0a9a81cfb6e9b6a08cb0f8ee1b4
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
# ↓ 本机首次编完用 mkhash 算出来后，把真值填回来，别长期留 skip
PKG_MIRROR_HASH:=skip

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
	SECTION:=net
	CATEGORY:=Ham Radio
	SUBMENU:=MMDVM
	TITLE:=Lua-C binding for MMDVM (mmdvm.so)
	URL:=https://github.com/g4klx/MMDVMHost/tree/master/Lua
	DEPENDS:=+libstdcpp +liblua +libpthread
	PROVIDES:=libmmdvm
endef

define Package/$(PKG_NAME)/description
Lua/C binding extracted from MMDVMHost's Lua/ directory, built as a
standalone Lua module (/usr/lib/lua/mmdvm.so). Lua module only - not a
linkable C++ shared library.
endef

ifeq ($(ARCH),mips)
  TARGET_CPPFLAGS += -DWORDS_BIGENDIAN=1
endif

TARGET_CXXFLAGS += \
	-O2 -pipe -std=gnu++11 \
	-DOPENWRT=1 -DLUA_COMPAT_5_3 \
	-fPIC -fvisibility=default \
	-Wno-error -Wno-deprecated -Wno-narrowing -Wno-unused-result \
	-fpermissive \
	-include cstring -include cstdlib -include unistd.h

TARGET_CPPFLAGS += \
	-I$(STAGING_DIR)/usr/include/lua \
	-I$(PKG_BUILD_DIR) \
	-I$(PKG_BUILD_DIR)/Lua

LUA_MMDVM_SRCS := MMDVM.cpp Conf.cpp Log.cpp lua-mmdvm.cpp

define Build/Compile
	@mkdir -p $(PKG_BUILD_DIR)/.objs
	@for f in $(LUA_MMDVM_SRCS); do \
		echo "libmmdvm: CXX $$f"; \
		$(TARGET_CXX) $(TARGET_CXXFLAGS) $(TARGET_CPPFLAGS) \
			-c $(PKG_BUILD_DIR)/Lua/$$f \
			-o $(PKG_BUILD_DIR)/.objs/$${f%.cpp}.o || exit 1; \
	done
	echo "libmmdvm: LD  mmdvm.so"
	$(TARGET_CXX) $(TARGET_LDFLAGS) -shared \
		-Wl,-soname,mmdvm.so -Wl,-z,now -Wl,-z,relro \
		-o $(PKG_BUILD_DIR)/mmdvm.so \
		$(PKG_BUILD_DIR)/.objs/*.o \
		-llua -lm -lpthread
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/lib/lua
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/mmdvm.so $(1)/usr/lib/lua/mmdvm.so
	$(STRIP) $(1)/usr/lib/lua/mmdvm.so
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
MAKEFILE_EOF

echo "✅ ①B 已写入"
