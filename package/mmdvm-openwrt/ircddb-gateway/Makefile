cd /workdir/openwrt/package/mmdvm-openwrt/ircddb-gateway
cp Makefile Makefile.bak.$(date +%F-%H%M)
mkdir -p files patches

cat > Makefile <<'MAKEFILE_EOF'
#
# Copyright 2019-2020 Michael BD7MQB <bd7mqb@qq.com>
# Licensed under the GNU GENERAL PUBLIC LICENSE, Version 2.0
#
# ── OpenWrt 24.10 / gcc-14.4 / musl-aarch64 适配重写 ────────────────────────
# 相对原版的修复（9 处，②⑥① 是致命的）：
#  1) PKG_MIRROR_HASH 4e1d1c9… → e5f30c6c…
#     上游 "No releases published"，OpenWrt 只能 git archive 现场打包，
#     tar.bz2 字节流随 bzip2 版本 + .gitattributes 而变，原哈希跨机必 mismatch。
#  2) PKG_RELEASE 重复定义（第7/15行）→ 合并为一个，版本递增到 2
#  3) 裸在 Package{} 外的第二个 DEPENDS 是死代码；块内 DEPENDS 补齐
#     libconfuse / libgps / openssl / zlib / pthread（原版只写 libwxwidgets，
#     链接期必炸 cfg_getstr / gps_read / SHA1_Init）
#  4) TARGET_LIBS 去掉 "-Wl," 前缀 —— -Wl,-lfoo 会让 ld 去找名为 "lfoo" 的文件
#  5) TARGET_CXXFLAGS/LDFLAGS 一律 "+="，原 "=" 冲掉 --sysroot/-mcpu/-zrelro
#  6) MAKE_FLAGS 尾部悬空反斜杠消除 + 显式 Build/Compile 带 "all" 兜底，
#     杜绝 "No rule to make target 'aarch64-openwrt-linux-musl-g++'"
#  7) 去掉危险的 PKG_INSTALL:=1（原写法上游 install 目标无 DESTDIR 护栏，
#     会往编译机 /usr/bin 里真装）；改用 DESTDIR=$(PKG_INSTALL_DIR) + 手工 install
#  8) 补 TARGET=opendv → DL5DI OpenDV drop-in 布局（/etc/ircddbgateway +
#     ircddbgatewayd + systemd unit），与 DATADIR 不打架
#  9) CATEGORY 改合法节点 Ham Radio→MMDVM；files/ 缺失不再让 $(CP) -a 炸掉
#
# 铁律：本包**绝不**调用 MakefileGUI。本 feed 的 libwxwidgets 是
#       --disable-gui（仅 wxBase，无 GTK/X11），碰 GUI 目标必炸 wxFrame。
# ────────────────────────────────────────────────────────────────────────────

include $(TOPDIR)/rules.mk

PKG_NAME:=ircddb-gateway
PKG_VERSION:=1.3.1
PKG_RELEASE:=2

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/lazywalker/ircDDBGateway.git
PKG_SOURCE_VERSION:=380c094
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.bz2
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
# 本机 bzip2-1.0.8 + scripts/dl_github_archive.py 稳定复现值。
# 换 bzip2 大版本后若再 mismatch，重跑探针把新值填这里（或临时填 skip）。
PKG_MIRROR_HASH:=e5f30c6cf166aaafdcfe52417bc1f5349ab6a3724692337ac955d8a08b8cbcc7

PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=COPYING.txt
PKG_MAINTAINER:=Michael BD7MQB <bd7mqb@qq.com>

# 上游 Makefile 用了 -Werror 风格的告警 + 老式 cast，放开
PKG_CHECK_FORMAT_SECURITY:=0

include $(INCLUDE_DIR)/package.mk

# ---- 路径（OpenDV 兼容）----------------------------------------------------
CONFDIR  := /etc
DATADIR  := /etc/mmdvm/ircddbgateway
LOGDIR   := /var/log/ircddbgateway
RUNDIR   := /var/run

define Package/$(PKG_NAME)
	SECTION:=net
	CATEGORY:=Ham Radio
	SUBMENU:=MMDVM
	TITLE:=ircDDB Gateway for D-Star (ircddbgatewayd, OpenDV layout)
	URL:=https://github.com/lazywalker/ircDDBGateway
	DEPENDS:=+libstdcpp \
	         +libwxwidgets \
	         +libconfuse \
	         +libgps \
	         +libopenssl \
	         +zlib \
	         +libpthread \
	         +ca-bundle
	PROVIDES:=ircddbgateway
	CONFLICTS:=ircddbgateway-opendv
endef

define Package/$(PKG_NAME)/description
The ircDDB Gateway for D-Star: callsign routing via ircDDB, plus
D-Plus(REF) / DExtra(XRF) / DCS / XLX reflectors, CCS7 routing,
D-PRS to APRS-IS, StarNet, D-RATS and DTMF/UR control.

Built with TARGET=opendv, so it is a drop-in replacement for the
unmaintained DL5DI OpenDV packages:
  daemon   /usr/sbin/ircddbgatewayd
  config   /etc/ircddbgateway
  data     /etc/mmdvm/ircddbgateway
  service  /etc/init.d/ircddbgatewayd  (procd) / systemd unit

GUI (ircDDBGatewayConfig) is deliberately NOT built: this feed's
libwxwidgets is compiled --disable-gui (wxBase only, no GTK/X11).
Edit /etc/ircddbgateway by hand or via LuCI.
endef

define Package/$(PKG_NAME)/conffiles
/etc/ircddbgateway
endef

# ---- 编译参数：一律 += ，绝不 = -------------------------------------------
TARGET_CXXFLAGS += \
	-O2 -pipe \
	-std=gnu++11 \
	-D_GNU_SOURCE \
	-DwxDEBUG_LEVEL=0 \
	-DwxUSE_SELECT_DISPATCHER=1 \
	-DwxHAS_INOTIFY=1 \
	-DLOG_DIR='"$(LOGDIR)"' \
	-DCONF_DIR='"$(CONFDIR)"' \
	-DDATA_DIR='"$(DATADIR)"' \
	-Wno-error -Wno-deprecated -Wno-narrowing \
	-Wno-unused-result -Wno-sign-compare -Wno-class-memaccess \
	-Wno-missing-field-initializers \
	-fpermissive \
	-include cstring -include cstdlib -include unistd.h

TARGET_CPPFLAGS += \
	-I$(STAGING_DIR)/usr/include \
	-I$(STAGING_DIR)/usr/include/wx-3.0

TARGET_LDFLAGS += \
	-L$(STAGING_DIR)/usr/lib \
	-Wl,-z,now -Wl,-z,relro

# ★ 关键：原版写成 "-Wl,-lwx_baseu-3.0" 是错的，-Wl, 只用于 linker 开关。
#   wxBase 三件套 + 依赖库老实写 -l。
WX_BASE_LIBS := $(shell $(STAGING_DIR)/usr/bin/wx-config --libs base,net,xml 2>/dev/null)
ifeq ($(strip $(WX_BASE_LIBS)),)
  WX_BASE_LIBS := -lwx_baseu-3.0 -lwx_baseu_net-3.0 -lwx_baseu_xml-3.0
endif

IRCDD_LIBS := $(WX_BASE_LIBS) \
              -lconfuse -lgps -lssl -lcrypto -lz -lpthread -lm

# ---- Prepare：物理摘掉 GUI，防止任何人/任何补丁误调 MakefileGUI ------------
define Build/Prepare
	$(call Build/Prepare/Default)
	rm -rf  $(PKG_BUILD_DIR)/ircDDBGatewayConfig \
	        $(PKG_BUILD_DIR)/GUICommon \
	        $(PKG_BUILD_DIR)/MakefileGUI \
	        $(PKG_BUILD_DIR)/debian \
	        $(PKG_BUILD_DIR)/*.nsi \
	        $(PKG_BUILD_DIR)/ircDDB
endef

# ---- Compile：显式传参 + TARGET=opendv + DESTDIR 护栏 + 末尾 all 兜底 ------
#       （原 MAKE_FLAGS 尾部反斜杠已消除，不会再让 CXX= 漂成 make target）
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
		LIBS="$(IRCDD_LIBS)" \
		TARGET=opendv \
		DESTDIR="$(PKG_INSTALL_DIR)" \
		WX_CONFIG="$(STAGING_DIR)/usr/bin/wx-config" \
		all
endef

# ---- Install：手工装，不依赖上游 install 目标（原 PKG_INSTALL:=1 会污染宿主）
define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/sbin
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/ircDDBGateway/ircddbgatewayd $(1)/usr/sbin/ircddbgatewayd
	$(STRIP) $(1)/usr/sbin/ircddbgatewayd

	$(INSTALL_DIR) $(1)/usr/bin
	@for t in TimeServer/timeserverd \
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

	# 语音/文本/DD 数据（含子目录与隐藏文件，原 "$(CP) Data/*" 会漏）
	$(INSTALL_DIR) $(1)$(DATADIR)
	$(CP) -a $(PKG_BUILD_DIR)/Data/. $(1)$(DATADIR)/

	# 默认配置（OpenDV 路径）
	$(INSTALL_DIR) $(1)/etc
	if [ -f $(PKG_BUILD_DIR)/ircDDBGateway/ircddbgateway ]; then \
		$(INSTALL_CONF) $(PKG_BUILD_DIR)/ircDDBGateway/ircddbgateway $(1)/etc/ircddbgateway ; \
	else \
		$(INSTALL_CONF) $(CURDIR)/files/ircddbgateway.example $(1)/etc/ircddbgateway ; \
	fi

	# procd 服务
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(CURDIR)/files/ircddbgatewayd.init $(1)/etc/init.d/ircddbgatewayd

	# systemd（仅 systemd 版 OpenWrt 用得上，存在才装）
	@if [ -d $(1)/lib/systemd/system ] || grep -qs '^CONFIG_USE_SYSTEMD=y' $(TOPDIR)/.config ; then \
		$(INSTALL_DIR) $(1)/lib/systemd/system ; \
		$(INSTALL_DATA) $(CURDIR)/files/ircddbgatewayd.service $(1)/lib/systemd/system/ircddbgatewayd.service ; \
	fi

	# 运行期目录
	$(INSTALL_DIR) $(1)$(LOGDIR) $(1)$(RUNDIR)
endef

define Package/$(PKG_NAME)/postinst
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] && exit 0
# OpenDV 兼容软链：老脚本常引用 /etc/ircddbgateway.conf
[ -e /etc/ircddbgateway.conf ] || ln -sf /etc/ircddbgateway /etc/ircddbgateway.conf 2>/dev/null
mkdir -p $(LOGDIR) /var/lib/ircddbgateway
if command -v systemctl >/dev/null 2>&1 && [ -d /run/systemd/system ]; then
	systemctl enable ircddbgatewayd.service >/dev/null 2>&1 || true
else
	/etc/init.d/ircddbgatewayd enable >/dev/null 2>&1 || true
fi
echo "ircddb-gateway: 请编辑 /etc/ircddbgateway（callsign / suffix / reflector），然后 /etc/init.d/ircddbgatewayd start"
exit 0
endef

define Package/$(PKG_NAME)/prerm
#!/bin/sh
[ -n "$${IPKG_INSTROOT}" ] && exit 0
command -v systemctl >/dev/null 2>&1 && [ -d /run/systemd/system ] \
	&& systemctl disable ircddbgatewayd.service >/dev/null 2>&1 || true
/etc/init.d/ircddbgatewayd stop >/dev/null 2>&1 || true
exit 0
endef

$(eval $(call BuildPackage,$(PKG_NAME)))
MAKEFILE_EOF

echo "✅ Makefile 已写入"
