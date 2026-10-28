# releases

`top-android` cannon 的 OTA 元数据仓库，给系统自带的「系统更新」读。

## 最新版本

**[v26.10.28](https://github.com/top-android/releases/releases/latest)** ·
`lineage-21.0-20261028-UNOFFICIAL-cannon.zip` ·
md5 `c6d46279fe8f42c0f90df9cfbb578a64`

## 刷机

### 日常升级（已经在同一签名版本上）

不清数据，两种都行。

**A. 系统更新** —— 设置 → 系统 → 系统更新 → 检查 → 下载 → 安装

**B. sideload**

```powershell
adb reboot sideload
adb sideload lineage-21.0-XXXXXXXX-UNOFFICIAL-cannon.zip
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
3. `adb sideload lineage-21.0-XXXXXXXX-UNOFFICIAL-cannon.zip`

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
`https://download.lineageos.org/api/v1/{device}/{type}/{incr}`，
返回的就是这种 `{"response":[...]}` 结构，所以换成静态 JSON 完全兼容。

## 发布一版新 ROM 的流程

1. 把 `lineage-21.0-<日期>-UNOFFICIAL-cannon.zip` 传到一个 GitHub Release：
   `https://github.com/top-android/releases/releases/tag/<tag>`
2. 改 `cannon.json`，写一条：

```json
{
  "response": [
    {
      "datetime": 1793200000,
      "filename": "lineage-21.0-20261027-UNOFFICIAL-cannon.zip",
      "id": "任意唯一字符串",
      "romtype": "unofficial",
      "size": 1182507966,
      "url": "https://github.com/top-android/releases/releases/download/20261027/lineage-21.0-20261027-UNOFFICIAL-cannon.zip",
      "version": "21.0"
    }
  ]
}
```

3. `git commit && git push`

Updater 只认 `response` 里 `datetime` 最大的那条，所以每次覆盖写一条即可；
想留历史就多写几条，它自己挑最新的。

## 注意

- `size` 和 `url` 必须对得上，Updater 下载完会校验大小。
- jsDelivr 对分支内容有缓存（最长 12 小时）。想立刻生效可以改用 tag：
  `https://cdn.jsdelivr.net/gh/top-android/releases@v20261027/cannon.json`
- `raw.githubusercontent.com` 在国内经常连不上，不建议直接用。
