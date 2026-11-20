#
# Copyright 2019-2020 Michael BD7MQB <bd7mqb@qq.com>
# Licensed under GNU GPL v2
#

include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/version.mk

PKG_NAME:=mmdvm-luci
PKG_VERSION:=1.2.0
PKG_RELEASE:=5
PKGARCH:=all

# ★ git rev 加兜底，不在 git 树里也不炸，且只求一次
GIT_REVISION ?= $(shell git rev-parse --short HEAD 2>/dev/null || echo nogit)
PKG_REVISION:=$(PKG_VERSION)-$(PKG_RELEASE)-$(GIT_REVISION)

PKG_LICENSE:=GPL-2.0
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

# ★ 纯 Lua 包：不需要 C 编译，但要 luci 的 minify 工具
PKG_BUILD_DEPENDS:=lua/host luci/host
PKG_CONFIG_DEPENDS:=CONFIG_LUCI_SRCDIET CONFIG_LUCI_JSMIN CONFIG_LUCI_CSSTIDY

include $(INCLUDE_DIR)/package.mk

HTDOCS         := /www
LUA_LIBRARYDIR := /usr/lib/lua
LUCI_LIBRARYDIR := $(LUA_LIBRARYDIR)/luci

LUCI_LANGUAGES:=$(sort $(filter-out templates,$(notdir $(wildcard ./luci-app-mmdvm/po/*))))
LUCI_LANG.en   :=English
LUCI_LANG.zh-cn:=中文 (Chinese)

# ---------------- 主包 ----------------
define Package/mmdvm-luci
	SECTION:=luci
	CATEGORY:=MMDVM
	TITLE:=LuCI plugins for MMDVMHost
	URL:=https://github.com/lazywalker/MMDVMHost/
	PKGARCH:=all
	# ★★★ 修依赖：删 libmmdvm / libubus-lua / libubox-lua
	DEPENDS:=+mmdvm-host +luci-compat +luci-lib-ubus +luci-lib-base
endef

define Package/mmdvm-luci/description
	LuCI configuration pages for MMDVMHost (DMR / D-Star / YSF / P25 / NXDN / DAPNET).
	Requires mmdvm-host; libmmdvm is pulled in transitively.
endef

# ---------------- 只保留本包私有的 Config.in 项 ----------------
define Package/mmdvm-luci/config
	menu "MMDVM LuCI"
		config LUCI_APP_MMDVM
			bool "Enable luci-app-mmdvm"
			depends on PACKAGE_mmdvm-luci
			default y
			help
				Install the MMDVM configuration pages.

		config LUCI_THEME_BOOTSTRAP_MMDVM
			bool "Use MMDVM-tweaked bootstrap theme"
			depends on PACKAGE_mmdvm-luci
			default y
			help
				Override luci-theme-bootstrap with MMDVM-branded styling.
	endmenu
	# ★ LUCI_SRCDIET / LUCI_JSMIN / LUCI_CSSTIDY 是 luci 全局符号，这里不再重复定义
endef

# ---------------- 压缩/替换辅助（原样保留，加守卫） ----------------
define SrcDiet
	[ -d "$(1)" ] && $(FIND) $(1) -type f -name '*.lua' | while read src; do \
		LUA_PATH="$(STAGING_DIR_HOSTPKG)/lib/lua/5.1/?.lua" \
		luasrcdiet --noopt-binequiv -o "$$$$src.o" "$$$$src" \
		&& mv "$$$$src.o" "$$$$src"; \
	done || true
endef

define JsMin
	[ -d "$(1)" ] && $(FIND) $(1) -type f -name '*.js' | while read src; do \
		jsmin < "$$$$src" > "$$$$src.o" && mv "$$$$src.o" "$$$$src"; \
	done || true
endef

define CssTidy
	[ -d "$(1)" ] && $(FIND) $(1) -type f -name '*.css' | while read src; do \
		csstidy "$$$$src" --template=highest --remove_last_semicolon=true "$$$$src.o" \
		&& mv "$$$$src.o" "$$$$src"; \
	done || true
endef

define SubstituteVersion
	[ -d "$(1)" ] && $(FIND) $(1) -type f -name '*.htm' | while read src; do \
		$(SED) 's/<%# *\([^ ]*\)PKG_VERSION *%>/\1$(PKG_VERSION)/g; \
		        s/"\(<%= *\(media\|resource\) *%>[^"]*\.\(js\|css\)\)"/"\1?v=$(PKG_REVISION)"/g' "$$$$src"; \
	done || true
endef

# ---------------- 通用安装片段（全加存在性守卫） ----------------
define Package/common/install
	# ---- Lua 控制器/模型 ----
	if [ -d "$(2)/luasrc" ]; then \
		$(INSTALL_DIR) $(1)$(LUCI_LIBRARYDIR); \
		cp -pR $(2)/luasrc/. $(1)$(LUCI_LIBRARYDIR)/; \
		$(FIND) $(1)$(LUCI_LIBRARYDIR)/ -type f -name '*.luadoc' -delete; \
		$(if $(CONFIG_LUCI_SRCDIET),$(call SrcDiet,$(1)$(LUCI_LIBRARYDIR)/),true); \
		$(call SubstituteVersion,$(1)$(LUCI_LIBRARYDIR)/); \
	fi
	# ---- 静态资源 ----
	if [ -d "$(2)/htdocs" ]; then \
		$(INSTALL_DIR) $(1)$(HTDOCS)/luci-static/mmdvm; \
		cp -pR $(2)/htdocs/. $(1)$(HTDOCS)/luci-static/mmdvm/; \
		$(if $(CONFIG_LUCI_JSMIN),  $(call JsMin,  $(1)$(HTDOCS)/),true); \
		$(if $(CONFIG_LUCI_CSSTIDY),$(call CssTidy,$(1)$(HTDOCS)/),true); \
	fi
	# ---- root 覆盖（init.d / uci-defaults / sbin）----
	if [ -d "$(2)/root" ]; then \
		cp -pR $(2)/root/. $(1)/; \
		$(if $(CONFIG_LUCI_SRCDIET),$(call SrcDiet,$(1)/usr/lib/lua/),true); \
		if [ -f $(1)/usr/sbin/dmrid.lua ]; then \
			mv -f $(1)/usr/sbin/dmrid.lua $(1)/usr/sbin/dmrid; \
			chmod 750 $(1)/usr/sbin/dmrid; \
		fi; \
	fi
	# ---- 可选 C helper（原来这里会炸）----
	if [ -d "$(2)/src" ] && [ -d "$(PKG_INSTALL_DIR)" ]; then \
		cp -pR $(PKG_INSTALL_DIR)/. $(1)/; \
	fi
endef

define Package/mmdvm-luci/install
	$(call Package/common/install,$(1),luci-app-mmdvm)
	$(if $(CONFIG_LUCI_THEME_BOOTSTRAP_MMDVM),\
		$(call Package/common/install,$(1),luci-theme-bootstrap-mmdvm),\
		true)
endef

# ★ 不依赖 postinst：用 uci-defaults，开机执行一次自删（apk --no-scripts 也安全）
define Package/mmdvm-luci/install
	$(call Package/common/install,$(1),luci-app-mmdvm)
	$(if $(CONFIG_LUCI_THEME_BOOTSTRAP_MMDVM),$(call Package/common/install,$(1),luci-theme-bootstrap-mmdvm),true)
	$(INSTALL_DIR) $(1)/etc/uci-defaults
	cat > $(1)/etc/uci-defaults/90-mmdvm-luci <<'EOF'
#!/bin/sh
[ -x /etc/init.d/dmrid ] && /etc/init.d/dmrid enable 2>/dev/null
[ -x /etc/init.d/mmdvm-host ] && /etc/init.d/mmdvm-host enable 2>/dev/null
# 首次装完给 luci 清一次缓存
rm -f /tmp/luci-indexcache* 2>/dev/null
exit 0
EOF
	chmod 0755 $(1)/etc/uci-defaults/90-mmdvm-luci
endef

define Package/mmdvm-luci/postinst
#!/bin/sh
# apk --no-scripts 会跳过这里；真机上手装 apk 时兜一下
[ -n "$${IPKG_INSTROOT}" ] || { \
	[ -x /etc/init.d/dmrid ] && /etc/init.d/dmrid enable || :; \
	rm -f /tmp/luci-indexcache* 2>/dev/null; \
}
exit 0
endef

define Package/mmdvm-luci/prerm
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] || { \
	[ -x /etc/init.d/dmrid ] && /etc/init.d/dmrid disable || :; \
	rm -f /tmp/luci-indexcache* 2>/dev/null; \
}
exit 0
endef

# ---------------- 语言包 ----------------
define LuciTranslation
	define Package/$(PKG_NAME)-i18n-$(1)
		SECTION:=luci
		CATEGORY:=MMDVM
		TITLE:=$(PKG_NAME) - $(1) translation
		HIDDEN:=1
		PKGARCH:=all
		DEPENDS:=+$(PKG_NAME)
	endef

	define Package/$(PKG_NAME)-i18n-$(1)/description
		Translation for mmdvm-luci ($(LUCI_LANG.$(1))).
	endef

	define Package/$(PKG_NAME)-i18n-$(1)/install
		$$(INSTALL_DIR) $$(1)/etc/uci-defaults
		printf '#!/bin/sh\nuci -q get luci.languages.$(subst -,_,$(1)) >/dev/null || { uci set luci.languages.$(subst -,_,$(1))='$(LUCI_LANG.$(1))'; uci commit luci; }\nexit 0\n' \
			> $$(1)/etc/uci-defaults/99-$(PKG_NAME)-i18n-$(1)
		chmod 0755 $$(1)/etc/uci-defaults/99-$(PKG_NAME)-i18n-$(1)
		$$(INSTALL_DIR) $$(1)$$(LUCI_LIBRARYDIR)/i18n
		for po in $(wildcard ./luci-app-mmdvm/po/$(1)/*.po); do \
			[ -f "$$$$po" ] && po2lmo "$$$$
po" \
				$$(1)$$(LUCI_LIBRARYDIR)/i18n/$$($$(basename $$$$po .po)).$(1).lmo; \
		done
	endef

	LUCI_BUILD_PACKAGES += $(PKG_NAME)-i18n-$(1)
endef

LUCI_BUILD_PACKAGES := $(PKG_NAME)
$(foreach lang,$(LUCI_LANGUAGES),$(eval $(call LuciTranslation,$(lang))))
$(foreach pkg,$(LUCI_BUILD_PACKAGES),$(eval $(call BuildPackage,$(pkg))))
