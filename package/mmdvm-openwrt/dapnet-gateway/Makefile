#
# Copyright (C) 2026 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=dapnet-gateway
PKG_VERSION:=1.3.0
PKG_RELEASE:=1

# ---- 源码获取：走 git，锁到具体 commit ----
PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/sgyuebing/DAPNETGateway.git
PKG_SOURCE_VERSION:=631297e
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.xz
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)

# ---- 用你日志里 OpenWrt 实算出的新哈希 ----
PKG_HASH:=696ce0c02a60442910b9dcf13def22e4167adcaa8a77637a3ac39c85d23c71e0

PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=LICENCE
PKG_MAINTAINER:=YOUR_NAME <your@email.com>

# 上游无 configure/cmake，纯 Make；不跑 autoreconf
include $(INCLUDE_DIR)/package.mk

define Package/dapnet-gateway
  SECTION:=hamradio
  CATEGORY:=Ham Radio
  TITLE:=Gateway between MMDVMHost and the DAPNET POCSAG paging network
  DEPENDS:=+libstdcpp +libpthread
  URL:=https://github.com/sgyuebing/DAPNETGateway
endef

define Package/dapnet-gateway/description
 The DAPNET Gateway links MMDVMHost to the DAPNET POCSAG network to receive
 paging messages. Messages are buffered while the host is busy with other
 modes and delivered in order once the host becomes idle.
endef

# ---- 编译参数：把 OpenWrt 的 cross toolchain 灌进上游 Makefile ----
define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) \
		$(TARGET_CONFIGURE_OPTS) \
		CXX="$(TARGET_CXX)" \
		CC="$(TARGET_CC)" \
		AR="$(TARGET_AR)" \
		RANLIB="$(TARGET_RANLIB)" \
		CFLAGS="$(TARGET_CFLAGS) $(TARGET_CPPFLAGS)" \
		CXXFLAGS="$(TARGET_CXXFLAGS) $(TARGET_CPPFLAGS) -std=gnu++98" \
		LDFLAGS="$(TARGET_LDFLAGS)" \
		LD="$(TARGET_CXX)"
endef

define Package/dapnet-gateway/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/DAPNETGateway $(1)/usr/bin/DAPNETGateway

	$(INSTALL_DIR) $(1)/etc/DAPNETGateway
	$(INSTALL_CONF) $(PKG_BUILD_DIR)/DAPNETGateway.ini $(1)/etc/DAPNETGateway/DAPNETGateway.ini

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/dapnet-gateway.init $(1)/etc/init.d/dapnet-gateway
endef

define Package/dapnet-gateway/conffiles
/etc/DAPNETGateway/DAPNETGateway.ini
endef

$(eval $(call BuildPackage,dapnet-gateway))
