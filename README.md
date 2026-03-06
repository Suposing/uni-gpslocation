# uni-gpslocation（自实现）

本插件用于在项目内替代同名插件的使用方式，按插件市场页面的“使用文档”实现相同的 API 形态：

- `isProviderEnabled`
- `openLocSetting`
- `requestBackgroundLocPer`
- `onStartLoc` / `onStartLocs`
- `getLastLocations`
- `stop`

说明：

- Android：支持前台/后台定位（后台模式使用前台服务 + 常驻通知）。
- iOS：支持持续定位与后台定位开关（需 Info.plist 权限与后台模式配置）。
- Web/鸿蒙：提供占位实现（返回不支持）。

## 使用示例（与插件市场文档一致）

### uni-app x

```ts
import {
	isProviderEnabled,
	openLocSetting,
	requestBackgroundLocPer,
	onStartLoc,
	onStartLocs,
	stop,
	getLastLocations,
	LocData,
	LoctionData,
} from "@/uni_modules/uni-gpslocation"

requestBackgroundLocPer((res) => {
	console.log('后台定位权限', res)
})

onStartLoc({
	gps: true,
	backgroud: true,
	title: '后台定位中',
	content: 'GPS定位正在运行…',
	resIcon: 'ic_loc',
	time: 1000,
	distance: 1,
	onlyOnce: false
} as LocData, (location: LoctionData) => {
	console.log('定位', location)
})

stop(true, (res) => {
	console.log('停止定位', res)
})
```

### uni-app

```js
import {
	isProviderEnabled,
	openLocSetting,
	requestBackgroundLocPer,
	onStartLoc,
	onStartLocs,
	stop,
	getLastLocations
} from "@/uni_modules/uni-gpslocation"

requestBackgroundLocPer((res) => {
	console.log('后台定位权限', res)
})

onStartLoc({
	gps: true,
	backgroud: true,
	title: '后台定位中',
	content: 'GPS定位正在运行…',
	resIcon: 'ic_loc',
	time: 1000,
	distance: 1,
	onlyOnce: false
}, (location) => {
	console.log('定位', location)
})
```

## 数据字段说明

`province/city/district/address`：本实现未做逆地理编码，默认返回空字符串（与原插件“纯 GPS 不依赖三方 SDK”的定位能力一致）。
