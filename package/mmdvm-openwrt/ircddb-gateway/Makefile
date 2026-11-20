cd /workdir/openwrt/package/mmdvm-openwrt/ircddb-gateway
cp Makefile Makefile.bak.$(date +%F-%H%M)
mkdir -p files patches

cat > Makefile <<'MAKEFILE_EOF'
# ircddb-gateway for OpenWrt 24.10 / gcc-14 / musl-aarch64
# (c) 2019-2020 Michael BD7MQB, rewritten 2026-06
#
# 与原版差异（关键 6 处）：
#  1) PKG_MIRROR_HASH 重测（git-archive tarball 跨 bzip2 版本必变）
#  2) PKG_RELEASE 去重，版本递增
#  3) DEPENDS 补齐 libconfuse/libgps/openssl/zlib；libwxwidgets 名自动探测
#  4) TARGET_LIBS 去掉伪 "-Wl,-lxxx" 前缀
#  5) CXXFLAGS/LDFLAGS 一律 += ，不冲掉 --sysroot/-mcpu
#  6) 不调 MakefileGUI（本 feed wx 是 --disable-gui，只有 wxBase）
#
# 语法铁律（改这份文件前必读）：
#   * include / PKG_*:= / define / endef / $(eval  -> 行首【零缩进】
#   * define ... endef 【内部】                     -> 行首【2 个空格】，禁止 Tab
#   * 规则命令体（Build/Compile、install 里的 shell）-> 行首【1 个真 Tab】
#   * 注释里【不要】出现 ─ │ └ 等 box-drawing 字符，用 # 和 - 代替

include $(TOPDIR)/rules.mk

PKG_NAME:=ircddb-gateway
PKG_VERSION:=1.3.1
PKG_RELEASE:=2

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/lazywalker/ircDDBGateway.git
PKG_SOURCE_VERSION:=380c094
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
# 本机 bzip2-1.0.8 + scripts/dl_github_archive.py 复现值。
# 若 scripts/download.pl 报 MIRROR_HASH mismatch，临时改成 PKG_MIRROR_HASH:=skip
# 或重跑: make package/mmdvm-openwrt/ircddb-gateway/download V=s; sha256sum dl/*.tar.bz2
PKG_MIRROR_HASH:=e5f30c6cf166aaafdcfe52417bc1f5349ab6a3724692337ac955d8a08b8cbcc7

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=COPYING.txt
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

PKG_CHECK_FORMAT_SECURITY:=0
PKG_USE_CCACHE:=1

include $(INCLUDE_DIR)/package.mk

# ---- OpenDV 兼容路径 -------------------------------------------------------
IRCDD_CONFDIR := /etc
IRCDD_DATADIR := /etc/mmdvm/ircddbgateway
IRCDD_LOGDIR  := /var/log/ircddbgateway
IRCDD_RUNDIR  := /var/run

# ---- wxBase 包名自动探测（避免 no such package 重演）-----------------------
# 优先级：libwxwidgets > wxbase3 > libwxbase3 > 空(只链 -lwx_baseu*)
WX_PKG := $(firstword \
	$(foreach c,libwxwidgets wxbase3 libwxbase3 libwxbase,\
		$(if $(wildcard $(STAGING_DIR)/usr/lib/pkgconfig/$(c).pc \
		               $(STAGING_DIR)/usr/include/wx-3.*/wx/wxchar.h),$(c))) \
)
ifeq ($(strip $(WX_PKG)),)
  WX_PKG := libwxwidgets
endif

define Package/$(PKG_NAME)
  SECTION:=net
  CATEGORY:=Ham Radio
  SUBMENU:=MMDVM
  TITLE:=ircDDB Gateway for D-Star (ircddbgatewayd, OpenDV layout)
  URL:=https://github.com/lazywalker/ircDDBGateway
  DEPENDS:=+libstdcpp +libconfuse +libgps +libopenssl +zlib +libpthread +ca-bundle @(wxUSE||1) $(WX_PKG:%=>%)
endef

define Package/$(PKG_NAME)/description
  ircDDB Gateway for D-Star: callsign routing via ircDDB, plus
  D-Plus(REF) / DExtra(XRF) / DCS / XLX reflectors, CCS7 routing,
  D-PRS to APRS-IS, StarNet, D-RATS and DTMF/UR control.

  Built with TARGET=opendv -> drop-in for DL5DI OpenDV layouts:
    daemon  /usr/sbin/ircddbgatewayd
    config  /etc/ircddbgateway
    data    /etc/mmdvm/ircddbgateway
    service /etc/init.d/ircddbgatewayd

  GUI (ircDDBGatewayConfig) is NOT built: this feed ships wxBase only
  (--disable-gui, no GTK/X11). Edit /etc/ircddbgateway by hand or LuCI.
endef

define Package/$(PKG_NAME)/conffiles
/etc/ircddbgateway
endef

# ---- 编译参数：一律 += -----------------------------------------------------
TARGET_CXXFLAGS += \
	-O2 -pipe \
	-std=gnu++11 \
	-D_GNU_SOURCE \
	-DwxDEBUG_LEVEL=0 \
	-DwxUSE_SELECT_DISPATCHER=1 \
	-DwxHAS_INOTIFY=1 \
	'-DLOG_DIR="$(IRCDD_LOGDIR)"' \
	'-DCONF_DIR="$(IRCDD_CONFDIR)"' \
	'-DDATA_DIR="$(IRCDD_DATADIR)"' \
	-Wno-error -Wno-deprecated -Wno-narrowing \
	-Wno-unused-result -Wno-sign-compare -Wno-class-memaccess \
	-Wno-missing-field-initializers \
	-fpermissive \
	-include cstring -include cstdlib -include unistd.h

TARGET_CPPFLAGS += \
	-I$(STAGING_DIR)/usr/include \
	-I$(STAGING_DIR)/usr/include/wx-3.0 \
	-I$(STAGING_DIR)/usr/include/wx-3.2

# 不重复加 -znow/-zrelro：24.10 的 hardening 已开，重复会与 RELR/IBT 冲突
TARGET_LDFLAGS += -L$(STAGING_DIR)/usr/lib

# wx libs：延迟到 Build/Compile 里用 shell 取，不在 parse 阶段 $(shell)
# 兜底顺序：wx-config -> 3.2 -> 3.0 -> 裸名
define GetWxLibs
	$(if $(wildcard $(STAGING_DIR)/usr/bin/wx-config),\
		$(shell $(STAGING_DIR)/usr/bin/wx-config --libs base,net,xml 2>/dev/null),\
		$(if $(wildcard $(STAGING_DIR)/usr/lib/libwx_baseu-3.2.so* $(STAGING_DIR)/usr/lib/libwx_baseu-3.2.a),\
			-lwx_baseu-3.2 -lwx_baseu_net-3.2 -lwx_baseu_xml-3.2,\
			-lwx_baseu-3.0 -lwx_baseu_net-3.0 -lwx_baseu_xml-3.0))
endef

IRCDD_EXTRA_LIBS := -lconfuse -lgps -lssl -lcrypto -lz -lpthread -lm

# ---- Prepare：物理摘掉 GUI -------------------------------------------------
define Build/Prepare
	$(call Build/Prepare/Default)
	rm -rf  $(PKG_BUILD_DIR)/ircDDBGatewayConfig \
	        $(PKG_BUILD_DIR)/GUICommon \
	        $(PKG_BUILD_DIR)/MakefileGUI \
	        $(PKG_BUILD_DIR)/debian \
	        $(PKG_BUILD_DIR)/*.nsi \
	        $(PKG_BUILD_DIR)/ircDDB
endef

# ---- Compile：显式传参 + TARGET=opendv + all 兜底 --------------------------
define Build/Compile
	+$(MAKE) $(PKG_JOBS) -C $(PKG_BUILD_DIR) \
		CC="$(TARGET_CC)" \
		CXX="$(TARGET_CXX)" \
		LD="$(TARGET_CXX)" \
		AR="$(TARGET_AR)" \
		RANLIB="$(TARGET_RANLIB)" \
		STRIP="$(TARGET_STRIP)" \
		CXXFLAGS="$(TARGET_CXXFLAGS) $(TARGET_CPPFLAGS)" \
		CPPFLAGS="$(TARGET_CPPFLAGS)" \
		LDFLAGS="$(TARGET_LDFLAGS)" \
		LIBS="$(call GetWxLibs) $(IRCDD_EXTRA_LIBS)" \
		TARGET=opendv \
		WX_CONFIG="$(STAGING_DIR)/usr/bin/wx-config" \
		all
endef

# ---- Install：手工装，不依赖上游 install 目标 ------------------------------
define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ircDDBGateway/ircddbgatewayd $(1)/usr/sbin/ircddbgatewayd
	$(STRIP) $(1)/usr/sbin/ircddbgatewayd

	$(INSTALL_DIR) $(1)/usr/bin
	for t in TimeServer/timeserverd \
	         RemoteControl/remotecontrol \
	         TimerControl/timercontrol \
	         TextTransmit/texttransmit \
	         VoiceTransmit/voicetransmit \
	         APRSTransmit/aprstransmit \
	         StarNetServer/starnetserver ; do \
		n=$$(basename $$t) ; \
		if [ -x "$(PKG_BUILD_DIR)/$$t" ]; then \
			$(INSTALL_BIN) "$(PKG_BUILD_DIR)/$$t" "$(1)/usr/bin/$$n" ; \
			$(STRIP) "$(1)/usr/bin/$$n" ; \
		else \
			echo "ircddb-gateway: skip $$n (not built)" ; \
		fi ; \
	done

	$(INSTALL_DIR) $(1)$(IRCDD_DATADIR)
	$(CP) $(PKG_BUILD_DIR)/Data/* $(1)$(IRCDD_DATADIR)/ 2>/dev/null || true
	$(CP) $(PKG_BUILD_DIR)/Data/.[!.]* $(1)$(IRCDD_DATADIR)/ 2>/dev/null || true

	$(INSTALL_DIR) $(1)/etc
	if [ -f $(PKG_BUILD_DIR)/ircDDBGateway/ircddbgateway ]; then \
		$(INSTALL_CONF) $(PKG_BUILD_DIR)/ircDDBGateway/ircddbgateway $(1)/etc/ircddbgateway ; \
	elif [ -f $(CURDIR)/files/ircddbgateway.example ]; then \
		$(INSTALL_CONF) $(CURDIR)/files/ircddbgateway.example $(1)/etc/ircddbgateway ; \
	else \
		: > $(1)/etc/ircddbgateway ; \
	fi

	$(INSTALL_DIR) $(1)/etc/init.d
	if [ -f $(CURDIR)/files/ircddbgatewayd.init ]; then \
		$(INSTALL_BIN) $(CURDIR)/files/ircddbgatewayd.init $(1)/etc/init.d/ircddbgatewayd ; \
	else \
		echo "ircddb-gateway: WARN files/ircddbgatewayd.init missing, installing stub" ; \
		printf '#!/bin/sh /etc/rc.common\nSTART=99\nstart(){ :; }\n' > $(1)/etc/init.d/ircddbgatewayd ; \
		chmod 0755 $(1)/etc/init.d/ircddbgatewayd ; \
	fi

	if [ -d $(1)/lib/systemd/system ] || grep -qs '^CONFIG_USE_SYSTEMD=y' $(TOPDIR)/.config ; then \
		$(INSTALL_DIR) $(1)/lib/systemd/system ; \
		[ -f $(CURDIR)/files/ircddbgatewayd.service ] && \
			$(INSTALL_DATA) $(CURDIR)/files/ircddbgatewayd.service $(1)/lib/systemd/system/ircddbgatewayd.service ; \
	fi

	$(INSTALL_DIR) $(1)$(IRCDD_LOGDIR) $(1)$(IRCDD_RUNDIR)
endef

# ---- postinst/prerm：纯 ASCII，$$ 转义，不用 make 变量插 shell -------------
define Package/$(PKG_NAME)/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] && exit 0
[ -e /etc/ircddbgateway.conf ] || ln -sf /etc/ircddbgateway /etc/ircddbgateway.conf 2>/dev/null
mkdir -p /var/log/ircddbgateway /var/lib/ircddbgateway
if command -v systemctl >/dev/null 2>&1 && [ -d /run/systemd/system ]; then
	systemctl enable ircddbgatewayd.service >/dev/null 2>&1 || true
else
	/etc/init.d/ircddbgatewayd enable >/dev/null 2>&1 || true
fi
echo "ircddb-gateway: edit /etc/ircddbgateway (callsign/suffix/reflector), then /etc/init.d/ircddbgatewayd start"
exit 0
endef

define Package/$(PKG_NAME)/prerm
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] && exit 0
if command -v systemctl >/dev/null 2>&1 && [ -d /run/systemd/system ]; then
	systemctl disable ircddbgatewayd.service >/dev/null 2>&1 || true
fi
/etc/init.d/ircddbgatewayd stop >/dev/null 2>&1 || true
exit 0
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
MAKEFILE_EOF
