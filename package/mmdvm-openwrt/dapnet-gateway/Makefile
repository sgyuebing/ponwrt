#
# Copyright 2019-2020 Michael BD7MQB <bd7mqb@qq.com>
# GPL-2.0
#
include $(TOPDIR)/rules.mk

PKG_NAME:=dapnet-gateway
PKG_REV:=631297e
PKG_VERSION:=1.3.0
PKG_RELEASE:=1

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/sgyuebing/DAPNETGateway.git
PKG_SOURCE_VERSION:=$(PKG_REV)
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_REV).tar.xz
PKG_HASH:=fc7314f8a319f2958427aea03ea03ca7e1595ef8c4b2a6660ec11335aeac0b19
PKG_MIRROR_HASH:=$(PKG_HASH)

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

include $(INCLUDE_DIR)/package.mk

define Package/$(PKG_NAME)
	SECTION:=net
	CATEGORY:=MMDVM
	TITLE:=DAPNET Gateway for MMDVM
	URL:=https://github.com/g4klx/DAPNETGateway
	DEPENDS:=+libpthread +libstdcpp
endef

define Package/$(PKG_NAME)/description
	Gateway to the DAPNET POCSAG network.
endef

# 不要覆盖 TARGET_LDFLAGS，用 +=；其实 staging/usr/lib 已自动链，可留空
TARGET_LDFLAGS += -L$(STAGING_DIR)/usr/lib

MAKE_FLAGS += \
	CC="$(TARGET_CC)" \
	CXX="$(TARGET_CXX)" \
	LD="$(TARGET_CXX)" \
	AR="$(TARGET_AR)" \
	RANLIB="$(TARGET_RANLIB)" \
	STRIP="$(TARGET_STRIP)" \
	CFLAGS="$(TARGET_CFLAGS)" \
	CXXFLAGS="$(TARGET_CXXFLAGS) -std=c++0x -Wno-error=format-security -Wno-error=stringop-truncation -Wno-error=array-bounds" \
	LDFLAGS="$(TARGET_LDFLAGS)"

define Build/Prepare
	$(call Build/Prepare/Default)
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/sbin $(1)/etc $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/DAPNETGateway $(1)/usr/sbin/
	-$(CP) -a ./files/. $(1)/ 2>/dev/null || true
endef

define Package/$(PKG_NAME)/conffiles
/etc/DAPNETGateway.ini
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
