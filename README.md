# releases

`top-android` cannon 的 OTA 元数据仓库，给系统自带的「系统更新」读。

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
