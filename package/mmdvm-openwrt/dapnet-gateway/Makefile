#
# Copyright (C) 2026 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=dapnet-gateway
PKG_VERSION:=1.3.0
PKG_RELEASE:=2

PKG_SOURCE_PROTO:=git
PKG_SOURCE_URL:=https://github.com/sgyuebing/DAPNETGateway.git
PKG_SOURCE_VERSION:=631297e
PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION)-$(PKG_SOURCE_VERSION).tar.xz
PKG_SOURCE_SUBDIR:=$(PKG_NAME)-$(PKG_VERSION)
PKG_HASH:=696ce0c02a60442910b9dcf13def22e4167adcaa8a77637a3ac39c85d23c71e0

PKG_LICENSE:=GPL-2.0-only
PKG_LICENSE_FILES:=LICENCE
PKG_MAINTAINER:=YOUR_NAME <your@email.com>

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

# ==========================================================================
# 替代原来的 patches/001-add-missing-cstdint.patch
# 直接在解包后的源码顶部补 #include <cstdint>，幂等、不进 quilt、不会 FAILED
# ==========================================================================
define Build/Prepare
	$(call Build/Prepare/Default)
	@for f in Conf.h Conf.cpp DAPNETNetwork.h DAPNETNetwork.cpp \
	          StopWatch.h Timer.h Version.h Gateway.cpp Logger.cpp \
	          RSSIInterpolator.h MMDVMNetwork.h UDPSocket.h; do \
		if [ -f "$(PKG_BUILD_DIR)/$$$$f" ]; then \
			if ! grep -qE '^#include <(cstdint|stdint.h)>' "$(PKG_BUILD_DIR)/$$$$f"; then \
				sed -i '1i #include <cstdint>' "$(PKG_BUILD_DIR)/$$$$f"; \
				echo "  [cstdint] injected -> $$$$f"; \
			else \
				echo "  [cstdint] already present, skip $$$$f"; \
			fi; \
		fi; \
	done
	@# 上游 Makefile 若写死 CXXFLAGS，这里就地松绑，避免我们的 flags 被吞
	@if grep -qE '^CXXFLAGS\s*[:?]?=' "$(PKG_BUILD_DIR)/Makefile"; then \
		sed -i -E 's|^CXXFLAGS\s*:=|CXXFLAGS +=|; s|^CXXFLAGS\s*\?=|CXXFLAGS +=|' \
			"$(PKG_BUILD_DIR)/Makefile"; \
		echo "  [makefile] CXXFLAGS := -> +="; \
	fi
	@# 顺手把 -Werror 摘了（gcc 14 告警多）
	@sed -i -E 's/-Werror(\b[^a-z]|$)/ /g' "$(PKG_BUILD_DIR)/Makefile" || true
endef

# ---- 编译参数：把 OpenWrt cross toolchain 灌进上游纯 Make ----
define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) \
		$(TARGET_CONFIGURE_OPTS) \
		CC="$(TARGET_CC)" \
		CXX="$(TARGET_CXX)" \
		LD="$(TARGET_CXX)" \
		AR="$(TARGET_AR)" \
		RANLIB="$(TARGET_RANLIB)" \
		CFLAGS="$(TARGET_CFLAGS) $(TARGET_CPPFLAGS) -include stdint.h" \
		CXXFLAGS="$(TARGET_CXXFLAGS) $(TARGET_CPPFLAGS) \
			-std=gnu++98 \
			-include cstdint -include cstring \
			-fno-sized-deallocation \
			-Wno-error -Wno-error=deprecated-declarations \
			-Wno-error=unused-parameter -Wno-error=unused-variable" \
		LDFLAGS="$(TARGET_LDFLAGS)" \
		LIBS="-lpthread -lrt"
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
