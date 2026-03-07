# uni-gpslocation

`uni-gpslocation` 是一个基于 UTS 的前后台 GPS 定位插件，面向 `uni-app x` / `uni_modules` 场景使用。

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
- 申请后台定位相关权限
- 开始持续定位
- 开始单次定位
- 获取缓存或系统最近一次定位
- 停止定位

当前实现也有这些边界：

- 不包含逆地理编码，`province`、`city`、`district`、`address` 目前默认为空字符串
- 回调形式为 `callback`，不返回 `Promise`
- iOS 端 `gps` 参数仅作为“更高精度”提示使用，没有 Android 那种显式 GPS / Network provider 选择
- `onStartLocs` 只是 `onStartLoc` 的别名
- API 中的 `backgroud`、`LoctionData` 为当前实际导出命名，文档按现状保留

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
	openLocSetting,
	requestBackgroundLocPer,
	onStartLoc,
	onStartLocs,
	getLastLocations,
	stop,
	type LocData,
	type LoctionData,
} from "@/uni_modules/uni-gpslocation"
```

## 快速示例

```ts
import {
	onStartLocs,
	stop,
	requestBackgroundLocPer,
	getLastLocations,
	type LocData,
	type LoctionData,
} from "@/uni_modules/uni-gpslocation"

const options: LocData = {
	gps: true,
	backgroud: true,
	onlyOnce: false,
	time: 3000,
	distance: 1,
	title: "后台定位测试中",
	content: "请保持应用运行以观察回调",
}

requestBackgroundLocPer((granted:boolean) => {
	console.log("后台权限是否已就绪", granted)
})

onStartLocs(options, (res:LoctionData) => {
	console.log("定位回调", res.lat, res.lng, res.accuracy)
})

getLastLocations((res:LoctionData) => {
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

### `onStartLoc(data, cb)`

开始定位。

```ts
const data: LocData = {
	gps: true,
	backgroud: false,
	time: 5000,
	distance: 0,
	onlyOnce: false,
}

onStartLoc(data, (res:LoctionData) => {
	console.log(res)
})
```

### `onStartLocs(data, cb)`

与 `onStartLoc` 等价，方便按现有项目习惯调用。

### `getLastLocations(cb)`

获取最近一次定位。

```ts
getLastLocations((res:LoctionData) => {
	console.log(res)
})
```

说明：

- 优先返回插件内部缓存的最后一次定位
- 若缓存为空，Android 会读取系统 `LastKnownLocation`
- 若缓存为空，iOS 会读取 `CLLocationManager.location`
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
| `backgroud` | `boolean` | `false` | 是否启用后台定位 |
| `title` | `string` | `后台定位` | Android 前台服务通知标题 |
| `content` | `string` | `正在定位…` | Android 前台服务通知内容 |
| `resIcon` | `string` | 应用图标 | Android 通知小图标，填 `drawable` 资源名，不带扩展名 |
| `time` | `number` | `0` | 定位时间间隔，单位毫秒；Android 发起定位请求时最小按 `1000` 处理 |
| `distance` | `number` | `0` | 最小位移间隔，单位米 |
| `onlyOnce` | `boolean` | `false` | 是否只定位一次 |

补充说明：

- Android 中，`time` 既参与原生请求间隔，也用于回调节流
- iOS 中，`time` 主要用于回调节流，`distance` 会映射到 `distanceFilter`

## `LoctionData` 返回值

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `lat` | `number` | 纬度 |
| `lng` | `number` | 经度 |
| `speed` | `number` | 速度 |
| `altitude` | `number` | 海拔 |
| `bearing` | `number` | 方向 |
| `accuracy` | `number` | 精度 |
| `locationType` | `number` | `1 = GPS`，`2 = Network/Other`，`0 = 空结果/未实现` |
| `province` | `string` | 当前未实现，默认空字符串 |
| `city` | `string` | 当前未实现，默认空字符串 |
| `district` | `string` | 当前未实现，默认空字符串 |
| `address` | `string` | 当前未实现，默认空字符串 |

## 平台差异

### Android

- 后台定位依赖前台服务通知
- `backgroud: true` 时会启动 `GpsLocationService`
- 通知标题、内容、小图标都可以配置
- 如果系统定位开关未开，即使权限通过也无法拿到有效位置

### iOS

- 后台定位依赖 `Always` 权限和 `UIBackgroundModes.location`
- 如果未先拿到后台权限，`backgroud: true` 不代表立刻具备后台持续定位能力
- `openLocSetting` 实际打开的是 App 设置页

### Harmony / Web

- 目前只是占位实现
- 调用后通常返回 `false` 或空定位结构

## 接入建议

推荐调用顺序：

1. 先调用 `isProviderEnabled`
2. 未开启时调用 `openLocSetting`
3. 需要后台定位时先调用 `requestBackgroundLocPer`
4. 再调用 `onStartLoc` / `onStartLocs`
5. 页面销毁或业务结束时调用 `stop(true, ...)`

## 测试建议

- Android 室内优先用 `gps: false` 做基础验证，室外再测 `gps: true`
- iOS 后台定位请重点验证锁屏、切后台、再次唤醒场景
- 若只验证首包定位，建议使用 `onlyOnce: true`
