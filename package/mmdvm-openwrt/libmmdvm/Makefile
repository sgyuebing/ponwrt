#
# 聚合 Makefile —— 放在 mmdvm-openwrt/ 根，方便整个 feed 一键操作
# 用法（在 OpenWrt 顶层目录里）：
#   make -C package/mmdvm-openwrt libmmdvm.compile
#   make -C package/mmdvm-openwrt libmmdvm.clean
#   make -C package/mmdvm-openwrt libmmdvm.install
#   make -C package/mmdvm-openwrt tabcheck      ← 发版前必跑
#   make -C package/mmdvm-openwrt hashfill      ← 首编后回填 PKG_HASH
#

PKG_SUBDIR := libmmdvm
PKG_MAKE   := $(MAKE) -C $(PKG_SUBDIR)

.PHONY: all libmmdvm.compile libmmdvm.clean libmmdvm.install libmmdvm.prepare tabcheck hashfill help

all: libmmdvm.compile

libmmdvm.prepare:
	$(PKG_MAKE) prepare V=s

libmmdvm.compile:
	$(PKG_MAKE) compile V=s -j1

libmmdvm.clean:
	$(PKG_MAKE) clean

libmmdvm.install:
	$(PKG_MAKE) install V=s

# missing-separator 复发探测器：任何行首空格都算失败
tabcheck:
	@echo "→ 扫描 $(PKG_SUBDIR)/Makefile 行首空格 ..."
	@if grep -nP '^ ' $(PKG_SUBDIR)/Makefile; then \
		echo "❌ 发现行首空格（上为行号），missing separator 必炸"; \
		echo "   一键修: sed -i 's/^    /\t/' $(PKG_SUBDIR)/Makefile"; \
		exit 1; \
	else \
		echo "✅ Tab 干净，可以编"; \
	fi
	@echo "→ 扫描 patches/*.patch 是否带 CRLF ..."
	@if file $(PKG_SUBDIR)/patches/*.patch | grep -q CRLF; then \
		echo "❌ patch 带 Windows 换行，patch -p1 会糊"; \
		exit 1; \
	else \
		echo "✅ patch 换行干净"; \
	fi

# 首编后回填 PKG_HASH
hashfill:
	@SRC=$$(ls -1 dl/MMDVMHost-9444c0d*.tar.xz 2>/dev/null | head -1); \
	if [ -z "$$SRC" ]; then \
		echo "❌ dl/ 里还没源码，先跑 libmmdvm.prepare"; exit 1; \
	fi; \
	H=$$(mkhash sha256 "$$SRC"); \
	echo "→ sha256 = $$H"; \
	sed -i "s|^PKG_HASH:=skip|PKG_HASH:=$$H|" $(PKG_SUBDIR)/Makefile; \
	grep -n '^PKG_HASH:=' $(PKG_SUBDIR)/Makefile

help:
	@sed -n '2,12p' $(MAKEFILE_LIST)

.DEFAULT_GOAL := help
