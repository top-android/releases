# releases

`top-android` cannon 的 OTA 元数据仓库，给系统自带的「系统更新」读。

## 最新版本

**[v26.11.23](https://github.com/top-android/releases/releases/tag/v26.11.23)** ·（Android 17 / LineageOS 24.0）·
`lineage-24.0-20261123-UNOFFICIAL-cannon.zip` ·
size `1358901223`（1.36 GB）·
sha256 `1b9eaa2b17bf00ad9d038c080cbb59b298c021336e385eabe96861bd7e72d8f9`

> 2026-11-23 重编：唯一改动是蓝牙修复 —— `audio.bluetooth.default` 在 `adev_open()` 里注册 AIDL
> `IBluetoothAudioProviderFactory/default` 并先起 binder 线程池，修掉蓝牙卡「正在开启」、搜不到设备。
> 其余与 2026.11.22 一致（含 `libTEECommon` 回退 13.0.3、`lineage.updater.uri` 修复）。
> 之前装过 `magisk-cannon-btaudio-fix` 模块的，这版已内置，刷完删掉模块再重启。

本次内核无改动（与 2026.11.21 完全一致，仅编译时间戳不同），未单独提供 `boot.img` 与 KSU 内核包，可直接沿用上一版的 `boot-20261121.img` / `SusfsNew.zip`。

## 刷机

### 日常升级（已经在同一签名版本上）

不清数据，两种都行。

**A. 系统更新** —— 设置 → 系统 → 系统更新 → 检查 → 下载 → 安装

**B. sideload**

```powershell
adb reboot sideload
adb sideload lineage-24.0-XXXXXXXX-UNOFFICIAL-cannon.zip
```

### 首次刷机（从 MIUI 或别的 ROM 过来）

只刷 recovery，其余由 OTA 包自己写：

```powershell
fastboot flash recovery recovery.img
fastboot reboot recovery
```

recovery 里：

1. **Factory reset → Format data**（从 MIUI 过来必须清，加密格式不兼容）
2. **Apply update → Apply from ADB**
3. `adb sideload lineage-24.0-XXXXXXXX-UNOFFICIAL-cannon.zip`

包里带着 boot / dtbo / vbmeta / system / vendor / product / system_ext，recovery 会一起写进去。
