---
AIGC:
  ContentProducer: '001191110102MAD55U9H0F10002'
  ContentPropagator: '001191110102MAD55U9H0F10002'
  Label: '1'
  ProduceID: '12daa0f0-37ff-4506-b751-1200adc67741'
  PropagateID: '12daa0f0-37ff-4506-b751-1200adc67741'
  ReservedCode1: 'b1c987a3-6204-4566-8668-71156a656cd5'
  ReservedCode2: 'b1c987a3-6204-4566-8668-71156a656cd5'
---

# openwrt-packages

BPI-R4 OpenWrt 固件编译用的第三方包镜像仓库。

所有第三方 LuCI 应用、主题、工具包、代理 feeds 以 Git submodule 形式聚合在此仓库中，
编译时只需 clone 本仓库即可获取全部依赖，无需逐个 clone 上游仓库。

## 结构

```
openwrt-packages/
├── openwrt-passwall-packages/  # 代理核心 feeds（submodule，main）
├── openwrt-passwall/           # luci-app-passwall feeds（submodule，main）
├── helloworld/                 # ssr-plus + 额外核心 feeds（submodule，dev）
├── luci-app-*/                 # 各 LuCI 应用（submodule）
├── luci-theme-*/               # LuCI 主题（submodule）
├── openwrt-*/                  # OpenWrt 底层包（submodule）
├── luci-app-ap-modem/          # 静态目录（锁定版本，不参与同步）
├── luci-app-model-gateway/     # 预编译 Makefile + .lmo（本地维护）
├── node/                       # sbwml 预编译 Node.js（submodule，packages-25.12）
└── .github/workflows/          # 每天自动同步工作流
```

## 同步机制

- **每天**自动同步所有 submodule 到上游最新版本（UTC 06:00 / 北京 14:00）
- 可手动触发（Actions → Sync Submodules → Run workflow）
- fancontrol 锁定在 commit 7655e6d（上游 HEAD 有破坏性变更）
- luci-app-ap-modem 与 luci-app-model-gateway 为静态目录，不参与同步

## 使用方式

在 OpenWrt 编译仓库的 diy-part1.sh 中：

```bash
# 1. clone monorepo（含全部 submodule）
git clone --depth 1 --recurse-submodules \
  https://github.com/SSkong/openwrt-packages.git /tmp/openwrt-packages

# 2. 复制 custom 包到 package/custom/（排除 feeds 仓库）
for d in /tmp/openwrt-packages/*/; do
  name=$(basename "$d")
  case "$name" in .github|openwrt-passwall-packages|openwrt-passwall|helloworld) continue;; esac
  mkdir -p "package/custom/$name"
  cp -a "$d". "package/custom/$name/"
  rm -rf "package/custom/$name/.git"
done

# 3. feeds 注入指向 monorepo 本地路径（src-link，无需再次 clone）
echo 'src-link passwall_packages /tmp/openwrt-packages/openwrt-passwall-packages' >> feeds.conf.default
echo 'src-link passwall /tmp/openwrt-packages/openwrt-passwall' >> feeds.conf.default
echo 'src-link helloworld /tmp/openwrt-packages/helloworld' >> feeds.conf.default
```