# sup-gpslocation

`sup-gpslocation` 是一个基于 UTS 的前后台 GPS 定位插件，面向 `uni-app x` / `uni_modules` 场景使用。

当前实现重点覆盖：

- Android 前台定位、后台定位、息屏定位
- iOS 前台定位、后台持续定位
- 获取最近一次定位结果
- 打开系统定位设置页

## 支持平台

| 平台 | 支持情况 | 说明 |
| --- | --- | --- |
| Android | 支持 | 最低 `minSdkVersion = 21`，后台定位通过前台服务实现 |
| iOS | 支持 | 后台定位依赖 `Always` 权限和 `location` 后台模式 |
| Harmony | 未实现 | 当前为占位实现，接口返回 `false` 或空定位数据 |
| Web | 未实现 | 当前为占位实现，接口返回 `false` 或空定位数据 |

## 功能说明

插件当前提供以下能力：

- 检查系统定位服务是否开启
- 跳转到定位相关设置页面
- 检查应用通知状态
- 申请应用通知权限
- 跳转到应用通知设置页
- 申请后台定位相关权限
- 开始持续定位
- 开始单次定位
- 按需输出 `wgs84` / `gcj02` 坐标
- Android 输出 GNSS 信号等级、卫星数量和平均 C/N0
- 获取缓存或系统最近一次定位
- 停止定位

当前实现也有这些边界：

- 不包含逆地理编码，`province`、`city`、`district`、`address` 目前默认为空字符串
- 回调形式为 `callback`，不返回 `Promise`
- iOS 端 `gps` 参数仅作为“更高精度”提示使用，没有 Android 那种显式 GPS / Network provider 选择
- GNSS 信号字段当前只在 Android 尽量返回有效值，iOS / Harmony / Web 默认返回不可用值

## 自动注入配置

### Android

插件内已包含以下权限与服务声明：

- `android.permission.ACCESS_FINE_LOCATION`
- `android.permission.ACCESS_COARSE_LOCATION`
- `android.permission.ACCESS_BACKGROUND_LOCATION`
- `android.permission.FOREGROUND_SERVICE`
- `android.permission.FOREGROUND_SERVICE_LOCATION`
- `android.permission.WAKE_LOCK`
- `android.permission.POST_NOTIFICATIONS`

同时注册了前台服务：

- `uts.sdk.modules.uniGpslocation.GpsLocationService`

### iOS

插件内已包含以下 `Info.plist` 配置：

- `NSLocationWhenInUseUsageDescription`
- `NSLocationAlwaysAndWhenInUseUsageDescription`
- `UIBackgroundModes -> location`

## 导入方式

```ts
import {
	isProviderEnabled,
	isNotificationPermissionAuthorized,
	openLocSetting,
	openNotificationSetting,
	requestNotificationPermission,
	requestBackgroundLocPer,
	onStartLocs,
	getLastLocations,
	stop,
	type LocData,
	type LocationQueryOptions,
	type LocationData,
} from "@/uni_modules/sup-gpslocation"
```

## 快速示例

```ts
import {
	onStartLocs,
	stop,
	requestNotificationPermission,
	requestBackgroundLocPer,
	getLastLocations,
	type LocData,
	type LocationQueryOptions,
	type LocationData,
} from "@/uni_modules/sup-gpslocation"

const options: LocData = {
	gps: true,
	coordType: "gcj02",
	background: true,
	onlyOnce: false,
	time: 3000,
	distance: 1,
	title: "后台定位测试中",
	content: "请保持应用运行以观察回调",
}

requestNotificationPermission((granted:boolean) => {
	console.log("应用通知是否已就绪", granted)
})

requestBackgroundLocPer((granted:boolean) => {
	console.log("后台权限是否已就绪", granted)
})

onStartLocs(options, (res:LocationData) => {
	console.log("定位回调", res.lat, res.lng, res.accuracy, res.signalLevel, res.satelliteCount, res.cn0DbHzAvg)
})

const queryOptions: LocationQueryOptions = {
	coordType: "gcj02",
}

getLastLocations(queryOptions, (res:LocationData) => {
	console.log("最近位置", res)
})

stop(true, (ok:boolean) => {
	console.log("停止定位", ok)
})
```

## API

### `isProviderEnabled(cb)`

检查系统定位服务是否开启。

```ts
isProviderEnabled((enabled:boolean) => {
	console.log(enabled)
})
```

说明：

- Android 检查 `GPS_PROVIDER` 或 `NETWORK_PROVIDER` 是否可用
- iOS 检查系统定位服务是否开启
- 这不是“应用定位权限是否已授权”的判断

### `openLocSetting(cb)`

打开定位设置页。

```ts
openLocSetting((opened:boolean) => {
	console.log(opened)
})
```

说明：

- Android 打开系统定位设置页
- iOS 打开当前 App 的系统设置页

### `openNotificationSetting(cb)`

打开应用通知设置页。

```ts
openNotificationSetting((opened:boolean) => {
	console.log(opened)
})
```

说明：

- Android 优先打开应用通知设置页，并尽量定位到插件使用的通知渠道
- iOS 当前打开 App 设置页

### `isNotificationPermissionAuthorized(cb)`

检查应用通知是否已就绪。

```ts
isNotificationPermissionAuthorized((granted:boolean) => {
	console.log(granted)
})
```

说明：

- Android 会同时检查应用通知总开关、Android 13+ `POST_NOTIFICATIONS` 权限，以及插件定位通知渠道是否被关闭
- 若返回 `false`，前台服务通知可能不会显示
- iOS 当前按“无需额外前台服务通知授权”处理

### `requestNotificationPermission(cb)`

申请应用通知权限。

```ts
requestNotificationPermission((granted:boolean) => {
	console.log(granted)
})
```

说明：

- Android 13+ 会尝试申请 `POST_NOTIFICATIONS`
- 若系统仅支持手动开启通知，或应用通知/通知渠道已被系统关闭，回调会返回当前状态，通常需要再调用 `openNotificationSetting`
- iOS 当前按“无需额外前台服务通知授权”处理

### `requestBackgroundLocPer(cb)`

请求后台定位相关权限。

```ts
requestBackgroundLocPer((granted:boolean) => {
	console.log(granted)
})
```

说明：

- Android 会先申请前台定位权限
- Android 13+ 会尝试申请通知权限，用于前台服务通知
- Android 10 会直接申请 `ACCESS_BACKGROUND_LOCATION`
- Android 11 及以上通常会跳到应用详情设置页，用户需手动开启“始终允许”，此时回调大概率先返回 `false`
- iOS 会触发 `requestAlwaysAuthorization()`；如果当前还没拿到 `Always` 权限，回调不会直接返回成功

### `onStartLocs(data, cb)`

开始定位。

```ts
const data: LocData = {
	gps: true,
	coordType: "gcj02",
	background: false,
	time: 5000,
	distance: 0,
	onlyOnce: false,
}

onStartLocs(data, (res:LocationData) => {
	console.log(res)
})
```

补充说明：

- `onStartLocs` 会真正启动原生定位监听，持续采集新的位置结果
- 适合实时定位、持续轨迹跟踪、定期上报经纬度等业务场景
- 如果业务需要通过 WebSocket 或 HTTP 持续发送最新经纬度，应该优先使用这个接口

### `getLastLocations(options, cb)`

获取最近一次定位。

```ts
const options: LocationQueryOptions = {
	coordType: "gcj02",
}

getLastLocations(options, (res:LocationData) => {
	console.log(res)
})
```

说明：

- 优先返回插件内部缓存的最后一次定位
- 若缓存为空，Android 会读取系统 `LastKnownLocation`
- 若缓存为空，iOS 会读取 `CLLocationManager.location`
- `coordType` 支持 `gcj02` / `wgs84`，默认 `gcj02`
- `getLastLocations` 不会主动重新发起一次新的定位请求，只是读取“最近已经存在的位置结果”
- 更适合页面初始化时快速展示最近点、调试查看最近位置或作为轻量兜底查询
- 没有可用定位时返回空结构，数值字段为 `0`

### `stop(removeNotif, cb)`

停止定位。

```ts
stop(true, (ok:boolean) => {
	console.log(ok)
})
```

说明：

- Android 建议传 `true`
- iOS 当前会忽略 `removeNotif`

## `LocData` 参数

| 字段 | 类型 | 默认含义 | 说明 |
| --- | --- | --- | --- |
| `gps` | `boolean` | `false` | Android 优先使用 GPS 高精度；iOS 仅作为更高精度配置参考 |
| `coordType` | `string` | `gcj02` | 输出坐标系，支持 `gcj02` / `wgs84` |
| `background` | `boolean` | `false` | 是否启用后台定位 |
| `title` | `string` | `后台定位` | Android 前台服务通知标题 |
| `content` | `string` | `正在定位…` | Android 前台服务通知内容 |
| `resIcon` | `string` | 应用图标 | Android 通知小图标，填 `drawable` 资源名，不带扩展名 |
| `time` | `number` | `0` | 定位时间间隔，单位毫秒；Android 发起定位请求时最小按 `1000` 处理 |
| `distance` | `number` | `0` | 最小位移间隔，单位米 |
| `onlyOnce` | `boolean` | `false` | 是否只定位一次 |

补充说明：

- Android 中，`time` 既参与原生请求间隔，也用于回调节流
- iOS 中，`time` 主要用于回调节流，`distance` 会映射到 `distanceFilter`
- 原生定位结果按 `wgs84` 处理，对外可按 `coordType` 输出为 `gcj02`

## `LocationQueryOptions` 参数

| 字段 | 类型 | 默认含义 | 说明 |
| --- | --- | --- | --- |
| `coordType` | `string` | `gcj02` | 最近定位输出坐标系，支持 `gcj02` / `wgs84` |

补充说明：

- 中国大陆外不会做 `wgs84 -> gcj02` 偏移，直接返回原始坐标

## `LocationData` 返回值

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `lat` | `number` | 纬度 |
| `lng` | `number` | 经度 |
| `speed` | `number` | 速度 |
| `altitude` | `number` | 海拔 |
| `bearing` | `number` | 方向 |
| `accuracy` | `number` | 精度 |
| `locationType` | `number` | `1 = GPS`，`2 = Network/Other`，`0 = 空结果/未实现` |
| `signalLevel` | `number` | GNSS 信号等级，`-1 = 不可用`，`0-4 = 由弱到强` |
| `satelliteCount` | `number` | 当前可见卫星数量 |
| `usedInFixCount` | `number` | 当前参与定位解算的卫星数量 |
| `cn0DbHzAvg` | `number` | 当前参与解算卫星的平均载噪比，单位 `dB-Hz` |
| `province` | `string` | 当前未实现，默认空字符串 |
| `city` | `string` | 当前未实现，默认空字符串 |
| `district` | `string` | 当前未实现，默认空字符串 |
| `address` | `string` | 当前未实现，默认空字符串 |

## 平台差异

### Android

- 后台定位依赖前台服务通知
- `background: true` 时会启动 `GpsLocationService`
- 通知标题、内容、小图标都可以配置
- `signalLevel`、`satelliteCount`、`usedInFixCount`、`cn0DbHzAvg` 会尽量基于 GNSS 状态回调输出
- GNSS 信号字段更适合在 `GPS_PROVIDER` 生效、室外开阔场景观察
- 应用通知总开关或定位通知渠道被关闭时，前台服务通知可能不会显示
- 如果系统定位开关未开，即使权限通过也无法拿到有效位置

### iOS

- 后台定位依赖 `Always` 权限和 `UIBackgroundModes.location`
- 如果未先拿到后台权限，`background: true` 不代表立刻具备后台持续定位能力
- `openLocSetting` 实际打开的是 App 设置页
- 信号字段当前默认返回不可用值，不承诺真实卫星信号强度
- 通知相关接口当前按“无需额外前台服务通知授权”处理

### Harmony / Web

- 目前只是占位实现
- 调用后通常返回 `false` 或空定位结构，信号字段默认不可用

## 接入建议

推荐调用顺序：

1. 先调用 `isProviderEnabled`
2. 未开启时调用 `openLocSetting`
3. 需要后台定位时先调用 `requestBackgroundLocPer`
4. 后台持续定位前调用 `isNotificationPermissionAuthorized` / `requestNotificationPermission`
5. 通知未就绪时调用 `openNotificationSetting`
6. 再调用 `onStartLocs`
7. 页面销毁或业务结束时调用 `stop(true, ...)`

接口选型建议：

- 需要持续拿到新位置、实时上报经纬度：使用 `onStartLocs`
- 只需要读取最近已有位置做展示或兜底：使用 `getLastLocations`

## 测试建议

- Android 室内优先用 `gps: false` 做基础验证，室外再测 `gps: true`
- Android 如果点击“开始持续定位”后没有看到前台通知，先检查应用通知总开关、通知权限与定位通知渠道
- Android 若要验证信号强度，建议室外开阔环境、开启 `gps: true`，并重点关注 `signalLevel`、`satelliteCount`、`usedInFixCount`、`cn0DbHzAvg`
- iOS 后台定位请重点验证锁屏、切后台、再次唤醒场景
- 若只验证首包定位，建议使用 `onlyOnce: true`
