# releases

`top-android` cannon 的 OTA 元数据仓库，给系统自带的「系统更新」读。

## 最新版本

**[v26.11.22](https://github.com/top-android/releases/releases/tag/v26.11.22)** ·（Android 17 / LineageOS 24.0）·
`lineage-24.0-20261122-UNOFFICIAL-cannon.zip` ·
size `1358908579`（1.36 GB）·
sha256 `8bd310206f94eb1e9d7e4db8fb31d76a8d33cfcce1cad52987ae15f5420cf2dc`

> 2026-11-22 22:44 重编（同一 tag，包内容已换）：
> ① `libTEECommon.so` 回退 13.0.3——14.0.6 版会让 beanpod keymaster 在 `Initialize()` 崩、`/data` 解不开、卡 MI logo（Magisk 模块测不出来，keymaster 是 `early_hal`，比模块挂载早）；
> ② `lineage.updater.uri` 移到 `/system/build.prop`（原来放 vendor 会被 init 按 SELinux 丢弃），系统更新可正常检查。
> 已经刷过旧包的，可以只 fastbootd 刷 `vendor-2026.11.22-teefix.zip` 里的 `vendor.img` 救回来，但 OTA 修复要整包。

本次内核无改动（与 2026.11.21 完全一致，仅编译时间戳不同），未单独提供 `boot.img` 与 KSU 内核包，可直接沿用上一版的 `boot-20261121.img` / `SusfsNew.zip`。

## 刷机

### 日常升级（已经在同一签名版本上）

不清数据，两种都行。

**A. 系统更新** —— 设置 → 系统 → 系统更新 → 检查 → 下载 → 安装

**B. sideload**

```powershell
adb reboot sideload
adb sideload lineage-23.2-XXXXXXXX-UNOFFICIAL-cannon-user.zip
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
3. `adb sideload lineage-23.2-XXXXXXXX-UNOFFICIAL-cannon-user.zip`

包里带着 boot / dtbo / vbmeta / system / vendor / product / system_ext，recovery 会一起写进去。

### 从 testkey 老版本迁移（仅一次）

20261027 及更早是 AOSP testkey 签名，和现在的自签 key 不兼容，sideload 会验签失败。必须全量 fastboot + `-w`：

```powershell
fastboot flash boot boot.img
fastboot flash recovery recovery.img
fastboot flash dtbo dtbo.img
fastboot flash vbmeta vbmeta.img
fastboot flash vbmeta_system vbmeta_system.img
fastboot flash vbmeta_vendor vbmeta_vendor.img
fastboot -w
fastboot reboot fastboot
fastboot flash system system.img
fastboot flash vendor vendor.img
fastboot flash product product.img
fastboot flash system_ext system_ext.img
fastboot reboot
```

boot / recovery / dtbo / vbmeta 是物理分区（bootloader 刷），system / vendor / product / system_ext 在 super 里是逻辑分区（必须 fastbootd）。

---

## 地址

系统通过 `lineage.updater.uri` 读取（`device/xiaomi/cannon/device.mk` 里设置）：

    https://cdn.jsdelivr.net/gh/top-android/releases@main/{device}.json

`{device}` 会被替换成 `ro.product.device`，本机是 `cannon`，所以实际请求
`https://cdn.jsdelivr.net/gh/top-android/releases@main/cannon.json`。

LineageOS 的 Updater 默认地址是
`https://download.lineageos.org/api/v1/{device}/{type}/{incr}`。

**注意：23.x 起 Updater 被重写成 Kotlin 实现，返回结构变了** —— 不再是
`{"response":[...]}`，而是**顶层数组**，每条带 `files[]`，见下面的发布流程。
旧格式（`response` 对象）在 23.x 上会直接解析失败，界面显示「检查更新失败」。
21/22 及更早的旧 Updater 只认 `response` 对象，这批设备请手动 sideload。

## 发布一版新 ROM 的流程

1. 把 `lineage-24.0-<日期>-UNOFFICIAL-cannon.zip` 传到一个 GitHub Release：
   `https://github.com/top-android/releases/releases/tag/<tag>`
2. 改 `cannon.json`，顶层写成数组：

```json
[
  {
    "datetime": 1795263199,
    "type": "unofficial",
    "version": "24.0",
    "files": [
      {
        "filename": "lineage-24.0-20261121-UNOFFICIAL-cannon.zip",
        "sha256": "73326fdcbc54dcaec3ec0e243eb65deb2622024c4aa6c4ae0d4859f370e3a9bb",
        "size": 1358491184,
        "url": "https://github.com/top-android/releases/releases/download/v26.11.21/lineage-24.0-20261121-UNOFFICIAL-cannon.zip",
        "os_patch_level": "2026-11-01",
        "os_sdk_level": 37
      }
    ]
  }
]
```

3. `git commit && git push`，然后刷新 jsDelivr 缓存（见「注意」）。

字段要求（`NetworkUpdate.kt` / `UpdatesRepository.kt` 里的硬性过滤）：

- `files[]` 每项**必填** `filename` / `sha256` / `size` / `url`；`sha256` 同时被当作下载 ID。
- `os_sdk_level` 可省，但省了会被当成 0，`< 设备当前 SDK` 直接丢弃 —— 跨版本时必须写对。
- `type` 必须等于设备的 `ro.lineage.releasetype`（`UNOFFICIAL`，比较忽略大小写）。
- `datetime` 必须**大于**设备的 `ro.build.date.utc`，否则视为「不比当前新」。
- 不需要 `id` / `romtype` 字段了（那是旧格式的）。

## 注意

- `size` 和 `url` 必须对得上，Updater 下载完会校验大小。
- jsDelivr 对分支内容有缓存（最长 12 小时）。想立刻生效可以改用 tag：
  `https://cdn.jsdelivr.net/gh/top-android/releases@v26.11.2.1/cannon.json`
- `raw.githubusercontent.com` 在国内经常连不上，不建议直接用。
