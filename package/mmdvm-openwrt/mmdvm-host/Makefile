#
# Copyright 2019-2020 Michael BD7MQB <bd7mqb@qq.com>
# This is free software, licensed under the GNU GPL v2
#

include $(TOPDIR)/rules.mk

PKG_NAME:=mmdvm-host
PKG_VERSION:=1.3.2
PKG_RELEASE:=2

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/lazywalker/MMDVMHost.git
PKG_SOURCE_VERSION:=81ad9f0
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_MIRROR_HASH:=022b634973d5f1836623c37bc4dae13719a18f3c56411a466f4e5e8f31931ed8

PKG_LICENSE:=GPL-2.0
PKG_LICENSE_FILES:=LICENSE
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

# ★ 关键：编译期就要 libmmdvm 的头/so
PKG_BUILD_DEPENDS:=libmmdvm

include $(INCLUDE_DIR)/package.mk

define Package/mmdvm-host
	SECTION:=net
	CATEGORY:=MMDVM
	TITLE:=MMDVM Host (DMR/D-Star/Y SF/P25/NXDN/DAPNET)
	URL:=https://github.com/g4klx/MMDVMHost/
	# ★★★ 修依赖：干掉幽灵包，补 libmmdvm，gateway 功能已内建
	DEPENDS:=+libpthread +libstdcpp +libgcc \
		+libmmdvm \
		+ysf-clients +p25-clients +nxdn-clients
	USERID:=mmdvm:mmdvm
	PROVIDES:=mmdvmhost
endef

define Package/mmdvm-host/description
 The host program for the MMDVM (Multi-Mode Digital Voice Modem).
 IRCDDB and DAPNET gateways are built in; configure them in /etc/MMDVM.ini.
endef

# ★ 不再污染全局 TARGET_CFLAGS，只喂给本次 make
MMDVM_CFLAGS := -O2 -pipe -std=c++0x -DOPENWRT \
	-I$(STAGING_DIR)/usr/include
MMDVM_LDFLAGS := -L$(STAGING_DIR)/usr/lib \
	-Wl,-rpath-link=$(STAGING_DIR)/usr/lib -lmmdvm

MAKE_FLAGS += \
	CXX="$(TARGET_CXX)" \
	CC="$(TARGET_CC)" \
	CFLAGS="$(MMDVM_CFLAGS)" \
	CXXFLAGS="$(MMDVM_CFLAGS)" \
	LDFLAGS="$(MMDVM_LDFLAGS)" \
	LD="$(TARGET_CXX)"

define Build/Compile
	+$(MAKE) $(MAKE_FLAGS) -C $(PKG_BUILD_DIR) \
		$(if $(CONFIG_DEBUG),DEBUG=1,)
endef

define Package/mmdvm-host/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/MMDVMHost $(1)/usr/sbin/MMDVMHost

	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) ./files/mmdvm-host.init $(1)/etc/init.d/mmdvm-host 2>/dev/null || true

	$(INSTALL_DIR) $(1)/etc
	$(INSTALL_CONF) ./files/MMDVM.ini.example $(1)/etc/MMDVM.ini 2>/dev/null || true

	# ★ files/ 目录不存在也不中断
	-$(CP) -a ./files/. $(1)/ 2>/dev/null
endef

define Package/mmdvm-host/conffiles
/etc/MMDVM.ini
endef

# ★ apk --no-scripts 会让构建期 postinst 跳过，但首次开机还会跑一次，兜底建用户
define Package/mmdvm-host/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] && {
	# 构建期：往 rootfs 的 passwd/shadow 里预置，避免 --no-scripts 漏掉
	grep -q '^mmdvm:' "$${IPKG_INSTROOT}/etc/passwd" || \
		echo 'mmdvm:x:65534:65534:mmdvm:/var/lib/mmdvm:/bin/false' \
			>> "$${IPKG_INSTROOT}/etc/passwd"
	grep -q '^mmdvm:' "$${IPKG_INSTROOT}/etc/group" || \
		echo 'mmdvm:x:65534:' >> "$${IPKG_INSTROOT}/etc/group"
	mkdir -p "$${IPKG_INSTROOT}/var/lib/mmdvm"
	chmod 755 "$${IPKG_INSTROOT}/var/lib/mmdvm"
}
exit 0
endef

$(eval $(call BuildPackage,mmdvm-host))
