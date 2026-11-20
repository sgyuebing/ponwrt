#
# Copyright 2019-2020 Michael BD7MQB <bd7mqb@qq.com>
# Licensed under GNU GPL v2
#

include $(TOPDIR)/rules.mk

PKG_NAME:=mmdvm
PKG_VERSION:=1.0.7
PKG_RELEASE:=2
PKGARCH:=all                      # ★ 纯元包，与架构无关

# ★ git rev 加兜底：不在 git 树里也不产生 "1.0.7-1-" 这种残版本
GIT_REVISION ?= $(shell git rev-parse --short HEAD 2>/dev/null || echo nogit)
PKG_REVISION:=$(PKG_VERSION)-$(PKG_RELEASE)-$(GIT_REVISION)

PKG_LICENSE:=GPL-2.0
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

include $(INCLUDE_DIR)/package.mk

define Package/mmdvm
	SECTION:=mmdvm
	CATEGORY:=MMDVM
	TITLE:=MMDVM Suite (meta-package)
	URL:=https://github.com/lazywalker/MMDVMHost/
	PKGARCH:=all
	# ★ 底层在前、UI 在后；不再重复声明 libmmdvm / ircddb-gateway / dapnet-gateway
	#   （全部由 mmdvm-host 传递带入，apk 只认真实 .apk 名）
	DEPENDS:=+mmdvm-host +mmdvm-luci
endef

define Package/mmdvm/description
	Meta-package that pulls in the full MMDVM stack:
	  - mmdvm-host   : MMDVMHost daemon (DMR / D-Star / YSF / P25 / NXDN / DAPNET, IRCDDB built-in)
	  - mmdvm-luci   : LuCI configuration pages + dmrid helper
	  - libmmdvm     : pulled in transitively via mmdvm-host
	Just select "MMDVM ---> MMDVM Suite" and you get a working hot-spot.
endef

# 元包无源码，显式置空，避免 Build/Compile/Default 去摸不存在的 Makefile
define Build/Prepare
endef
define Build/Configure
endef
define Build/Compile
endef

define Package/mmdvm/install
	# ★ files/ 不存在也不炸
	if [ -d "files" ]; then \
		$(CP) -a files/. $(1)/ 2>/dev/null || true; \
	fi

	# ★ mmdvmctl.sh → mmdvmctl，三重守卫
	if [ -f "$(1)/usr/sbin/mmdvmctl.sh" ]; then \
		$(INSTALL_DIR) $(1)/usr/sbin; \
		mv -f $(1)/usr/sbin/mmdvmctl.sh $(1)/usr/sbin/mmdvmctl; \
		chmod 750 $(1)/usr/sbin/mmdvmctl; \
	elif [ -f "files/usr/sbin/mmdvmctl" ]; then \
		$(INSTALL_DIR) $(1)/usr/sbin; \
		$(INSTALL_BIN) files/usr/sbin/mmdvmctl $(1)/usr/sbin/mmdvmctl; \
	fi

	# ★ 开机兜底：开自启 + 清 LuCI 缓存（apk --no-scripts 也安全，走 uci-defaults）
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	cat > $(1)/etc/uci-defaults/91-mmdvm-suite <<'EOF'
#!/bin/sh
for s in mmdvm-host dmrid; do
	[ -x /etc/init.d/$$s ] && /etc/init.d/$$s enable 2>/dev/null
done
rm -f /tmp/luci-indexcache* /tmp/luci-modulecache/* 2>/dev/null
exit 0
EOF
	chmod 0755 $(1)/etc/uci-defaults/91-mmdvm-suite
endef

define Package/mmdvm/postinst
#!/bin/sh
# 真机 apk add mmdvm 时兜一层；rootfs 构建期被 --no-scripts 跳过属正常
[ -n "$${IPKG_INSTROOT}" ] || {
	for s in mmdvm-host dmrid; do
		[ -x /etc/init.d/$$s ] && /etc/init.d/$$s enable 2>/dev/null || :
	done
	rm -f /tmp/luci-indexcache* 2>/dev/null
}
exit 0
endef

define Package/mmdvm/prerm
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] || {
	[ -x /etc/init.d/mmdvm-host ] && /etc/init.d/mmdvm-host disable 2>/dev/null || :
	rm -f /tmp/luci-indexcache* 2>/dev/null
}
exit 0
endef

$(eval $(call BuildPackage,mmdvm))
