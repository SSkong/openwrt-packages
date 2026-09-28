---
AIGC:
  ContentProducer: '001191110102MAD55U9H0F10002'
  ContentPropagator: '001191110102MAD55U9H0F10002'
  Label: '1'
  ProduceID: 'aec5e1bd-ca7b-47ca-8f0f-637766f6e7ff'
  PropagateID: 'aec5e1bd-ca7b-47ca-8f0f-637766f6e7ff'
  ReservedCode1: 'a5baf874-0992-421e-a9e9-297f1adfd757'
  ReservedCode2: 'a5baf874-0992-421e-a9e9-297f1adfd757'
---

# openwrt-packages

BPI-R4 OpenWrt 固件编译用的第三方包镜像仓库。

所有第三方 LuCI 应用、主题、工具包以 Git submodule 形式聚合在此仓库中，
编译时只需 clone 本仓库即可获取全部自定义包，无需逐个 clone 上游仓库。

## 结构

```
openwrt-packages/
├── luci-app-*/           # 各 LuCI 应用（submodule）
├── luci-theme-*/         # LuCI 主题（submodule）
├── openwrt-*/            # OpenWrt 底层包（submodule）
├── OpenWrt-Add/          # QiuSimons/OpenWrt-Add（submodule，luci-app-ap-modem 源）
├── luci-app-ap-modem/    # 从 OpenWrt-Add 提取的子目录（非 submodule）
├── luci-app-model-gateway/  # 预编译 Makefile + .lmo（非 submodule，本地维护）
├── node/                 # sbwml 预编译 Node.js（submodule，分支 packages-25.12）
└── .github/workflows/    # 定时同步工作流
```

## 同步机制

- 每周一自动同步所有 submodule 到上游最新版本
- 可手动触发（Actions → Sync Submodules → Run workflow）
- fancontrol 锁定在 commit 7655e6d（上游 HEAD 有破坏性变更）
- luci-app-ap-modem 每次同步时从 OpenWrt-Add 重新提取

## 使用方式

在 OpenWrt 编译仓库的 diy-part1.sh 中：

```bash
git clone --depth 1 https://github.com/SSkong/openwrt-packages.git /tmp/packages
cp -a /tmp/packages/* package/custom/
rm -rf /tmp/packages
```