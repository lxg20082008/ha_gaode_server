# Ha Gaode Server

[![hacs_badge](https://img.shields.io/badge/HACS-Default-41BDF5.svg)](https://github.com/hacs/integration)

把 `device_tracker` 的 **WGS-84** 坐标转成 **GCJ-02**（高德火星坐标），再和 GCJ-02 的 zone 算距离、判定 home/not_home，解决国内 HA 的两类问题：

- **「手机回家却判离家」**：zone 用高德 GCJ-02，companion 上报 WGS-84，原生判定直接算会误判。
- **「地图 marker 偏移约 500 米」**：原生地图底图换成高德瓦片（GCJ-02）后，marker 仍是 WGS-84。

## 前端搭配

本组件是「**坐标火星化**」层，前端二选一：

| 结构 | 前端 | 本组件配置 |
| --- | --- | --- |
| **A（推荐，快）** | 原生地图 + 高德瓦片（[amap_ha](https://github.com/lxg20082008/amap_ha) + [amap_proxy](https://github.com/lxg20082008/amap_proxy)） | `change_gpslogger_coord: true`（默认），把 GCJ-02 写回 `latitude`/`longitude` |
| **B** | [ha_gaode](https://github.com/d380025303/ha_gaode)（dx-gaode-map-card） | 卡片读 `gcj02_*` 属性，无需写回 |

> A 用原生 Leaflet，快、功能全；B 用高德 JS SDK，较慢。

## 更新

- v5.1.2
  - 新增 `change_gpslogger_coord`（默认 `true`）：把 GCJ-02 写回 `latitude`/`longitude`，让原生地图 marker 与高德瓦片对齐
- v5.1.1
  - 修复人员跟踪操作导致数据丢失问题
- v5.1
  - 增加配置可使某些 device_tracker 不做距离计算
- v5.0
  - 支持多边形范围计算
- v4.1
  - 增加配置可在 device_tracker 上报后将数据通过 POST 请求推送
- v4.0
  - 增加配置可使某些 device_tracker 不自动做高德坐标转换
- v3.0
  - GPSLogger 可存储到数据库（实现根据时间绘制路径轨迹功能）

## 安装

### 手动安装

1. 下载 `custom_components/ha_gaode_server` 下的所有文件。
2. 复制到 `<config>/custom_components/`（包含 `ha_gaode_server` 文件夹）。
3. 重启 Home Assistant。
4. 修改 `configuration.yaml`，增加下方配置。

## 配置

```yaml
ha_gaode_server:
  # 高德 Server Key：需要是「Web服务」类型，与 Web 端（JS API）的 key 区分开
  gaode_server_key: 你的高德serverkey
  # 是否同步修改 device_tracker 的 state
  change_gpslogger_state: true
  # 是否把 GCJ-02 写回 latitude/longitude（原生地图 marker 对齐高德瓦片）
  change_gpslogger_coord: true
  # SQLite 数据库，默认存储在 config/dx_db.db
  db_url: dx_db.db
  # 不做高德转换的 device_tracker 列表
  ignore_transform_device_trackers:
    - device_tracker.XXXXX
  # 不做距离计算的 device_tracker 列表
  ignore_distance_device_trackers:
    - device_tracker.XXXXX
  # device_tracker 上报后 POST 请求地址
  push_device_trackers_post:
```

## 增强属性

device_tracker 获得以下增强属性：

| 属性 | 说明 |
| --- | --- |
| `gcj02_longitude` / `gcj02_latitude` | 高德 GCJ-02 经纬度 |
| `dx_state` | 设备当前状态：`home`（在 home 范围内）/ `zone.XXX`（在 zone.XXX 范围内）/ `not_home`（不在任何范围内） |
| `dx_pre_state` | 设备前一个状态，取值同 `dx_state` |
| `dx_distance` | 进入范围内时距中心距离；不在范围内为 `-1` |
| `dx_record_datetime` | device_tracker 上报时间 |

## 兼容性

本 fork 已适配新版 Home Assistant（2024+）：

- `homeassistant.core.Config` → `homeassistant.helpers.typing.ConfigType`（新版移除了 `Config`）。
- `zone.py` 同步文件 IO → `hass.async_add_executor_job`（消除 blocking call）。
