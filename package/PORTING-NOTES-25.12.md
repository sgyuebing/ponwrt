# MMDVM Suite 移植到 OpenWrt 25.12.4 记录

目标平台：x86_64（x86/64，gcc 14.3.0 musl），SDK 构建。
构建根目录：`sdk/openwrt-sdk-25.12.4-x86-64_gcc-14.3.0_musl.Linux-x86_64/`

## 一、已完成并编译通过的包（产物见 build-apks/）

| 包 | 版本 | 状态 |
|---|---|---|
| mmdvm-cal | 1.3.0 | ✅ 通过 |
| p25-clients | 1.3.0-r2 | ✅ 通过 |
| ysf-clients | 1.2.0-r2 | ✅ 通过 |
| nxdn-clients | 1.3.0-r1 | ✅ 通过 |
| dapnet-gateway | 1.3.0 | ✅ 通过（已打补丁） |
| libwxwidgets | 3.0-r6 | ✅ 通过 |
| libmmdvm | 1.2.0-r1 | ✅ 通过（链接 liblua.so.5.1.5） |

## 二、已做的源码/构建修复

1. **dapnet-gateway/patches/002-cstdint.patch（新增）**
   GCC 14 不再间接暴露 `<cstdint>`，`Conf.cpp` 使用 `uint32_t` 报
   `'uint32_t' was not declared in this scope`。补丁在 Conf.cpp 增加
   `#include <cstdint>`。

2. **ircddb-gateway/Makefile：更新 PKG_MIRROR_HASH**
   旧哈希是老版本 buildroot 打包生成的；25.12 buildroot 重新 git archive
   后哈希变化导致校验失败（expected 4e1d1c…，got e5f30c…）。
   已将 `PKG_MIRROR_HASH` 更新为当前构建根重打包得到的值
   `e5f30c6cf166aaafdcfe52417bc1f5349ab6a3724692337ac955d8a08b8cbcc7`。

## 三、尚未完成 / 待处理项（按顺序继续）

### 1. ircddb-gateway（编译中断前未出现致命错误）
哈希修复后已进入 C++ 编译阶段（Common / ircDDB / ircDDBGateway / TimeServer），
继续执行即可：

```
make package/ircddb-gateway/compile V=s -j2
```

若 GCC 14 报缺类型/缺头文件，参照 dapnet 的做法在 `ircddb-gateway/patches/`
补 `<cstdint>` / `<cstdio>` 等头文件补丁。

### 2. mmdvm-host（尚未编译，整套套件的核心）
依赖 ysf/p25/nxdn/dapnet/ircddb。继续：

```
make package/mmdvm-host/compile V=s -j2
```

预判风险：老 g4klx 系代码在 GCC 14 下可能出现
`uint32_t/uint16_t 未声明`、隐式 `int` 转换警告升级为错误等；
按报错在 `mmdvm-host/patches/` 下补头文件补丁。
另外 `TARGET_CFLAGS += -std=c++0x` 可按需改为 `-std=gnu++11`。

### 3. ysfbmdirect（Makefile 需要重写）
- 文件注释/描述为 GBK 乱码；
- `PKG_SOURCE_VERSION:=HEAD`、`PKG_MIRROR_HASH:=skip`，25.12 下载规范下不可靠；
- 缺少标准的版本/源码哈希定义。
应改为与其它包一致的固定 commit + 校验哈希写法（源码仓：
https://github.com/stefanolande/YSFBMDirect ）。

### 4. mmdvm-luci（旧版 Lua LuCI 封装，建议按现代 luci.mk 精简）
- 顶层 `mmdvm-luci/Makefile` 手工实现了 SrcDiet/JsMin/CssTidy/i18n 逻辑，
  25.12 的 `feeds/luci/luci.mk` 已自动处理 luasrc/htdocs/root/ 与压缩、翻译；
  建议删掉顶层自定义编译逻辑，只保留两个子包（luci-app-mmdvm、
  luci-theme-bootstrap-mmdvm）走 luci.mk。
- 依赖需复核：旧声明 `+libubus-lua +libubox-lua +luci-compat`；
  25.12 下 nixio/jsonc/sys/util 等 Lua 运行时由 `luci-lua-runtime` 提供，
  建议应用依赖改为 `+luci-lua-runtime +luci-compat +libubus-lua +libubox-lua`。
- 应用为旧版 Lua 控制器（`luasrc/controller` + CBI），运行期依赖
  `luci-compat`；需在真实设备上验证菜单、CBI 表单与 dmrid.lua（ubus/uloop）。
- `root/` 目录（dmrid 服务、acl json、uci-defaults）由 luci.mk 自动安装，
  迁移时不要遗漏。

### 5. 小项
- `dapnet-gateway`、`mmdvm-cal` 的 `PKG_RELEASE:=0` 建议改为 `1`（规范 r≥1）。
- 各包 `wget http://42.194.130.14/...` 等更新脚本为运行期行为，与编译无关，
  上线前需核对数据源可用性。

## 四、复现步骤

```
# 1. 解压 25.12.4 x86/64 SDK
# 2. feeds.conf 增加本地源：
src-link mmdvm <本 feed 所在绝对路径>
# 3.
./scripts/feeds update -a
./scripts/feeds install -a
make defconfig
# 4. 逐包编译（示例）
make package/p25-clients/compile V=s -j2
make package/ircddb-gateway/compile V=s -j2
make package/mmdvm-host/compile V=s -j2
```

产物目录：`bin/packages/x86_64/mmdvm/`（25.12 为 .apk 格式，非旧版 .ipk）。
