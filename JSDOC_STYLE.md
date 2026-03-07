## JSDoc 注释规范风格（本项目约定）

> 用于本项目 Vue 组件/逻辑代码的注释风格总结

### 1. 通用格式

- 使用块级注释：`/** ... */`，紧贴在被注释目标的上一行。
- 业务文案统一用中文，语气简洁，以功能/作用开头。
- 标签常用顺序：`@description` → `@property` / `@param` → `@returns` / `@event` → 其他。

示例：

```ts
/**
 * @description 获取物料总价的展示文案（两位小数）
 * @param {Record<string, any>} item 物料数据
 * @returns {string} 用于展示的价格字符串
 */
const resolvePrice = (item: Record<string, any>) => {
  // ...
}
```

### 2. 组件 props 注释

- 写在 `defineProps` 前一行。
- 用 `@description` 简要说明组件用途。
- 使用多行 `@property` 描述每个 prop：类型、是否可选、业务含义。
- 类型写法与 TS 基本一致，例如：`{number | string}`、`{Array<Record<string, any>>}`。

示例：

```ts
/**
 * @description 商品信息与配件选择卡片入参
 * @property {Array<Record<string, any>>} [goods] 订单商品列表
 * @property {number | string} [differencePrice] 补差价金额
 * @property {number | string} [additionalPrice] 加项合计金额
 */
const props = defineProps<{
  goods?: Array<Record<string, any>>
  differencePrice?: number | string
  additionalPrice?: number | string
}>()
```

### 3. 组件事件（emit）注释

- 写在 `defineEmits` 前一行。
- `@description` 说明此组件对外抛出的事件集合。
- 每个事件使用 `@event`，名称为事件名，后面说明触发时机和含义，如有参数可在说明里写清。

示例：

```ts
/**
 * @description 组件向外抛出的事件
 * @event refresh 需要刷新订单配件列表时触发
 */
const emit = defineEmits<{
  (event: 'refresh'): void
}>()
```

### 4. 计算属性 / 状态变量注释

- 对业务含义不够直观的 `computed` / `ref` 添加 `@description`。
- 侧重“这个字段代表什么”而不是实现细节。

示例：

```ts
/**
 * @description 已选择的物料总金额的展示文案（两位小数）
 */
const totalPriceDisplay = computed(() => totalPrice.value.toFixed(2))
```

### 5. 函数方法注释

- 所有对外暴露或逻辑复杂的函数需写完整注释。
- 标签使用：
  - `@description`：一句话说明做什么、在什么场景下调用。
  - `@param {Type} name 描述`：每个入参一行，类型写在花括号里。
  - `@returns {Type} 描述`：返回值类型和含义，没有返回值可省略或使用 `@returns {void}`。

示例：

```ts
/**
 * @description 数量选择器变更时更新对应物料数量
 * @param {StaffMaterialItem} material 物料数据
 * @param {number} value 最新数量
 */
const handleQuantityChange = (material: StaffMaterialItem, value: number) => {
  // ...
}
```

### 6. Options API 组件（如 demo.vue）

- 在 `export default` 前写组件级注释，标明用途与关键属性。
- 使用 `@component` 标识组件。
- 可用 `@property` 描述 `data` 中的核心字段。
- `methods` 内的方法同样用函数注释格式。

示例：

```js
/**
 * @description SwipeAction 示例列表组件
 * @component
 * @property {Array<Object>} list 列表数据，包含 id、title、images、show 等字段
 */
export default {
  // ...
  methods: {
    /**
     * @description 处理滑动操作按钮点击事件（收藏 / 删除）
     * @param {number} index 列表项索引
     * @param {number} index1 按钮索引，1 为删除，其它为收藏
     */
    click(index, index1) {
      // ...
    }
  }
}
```

### 7. 风格小结

- 注释语言统一使用中文，偏业务描述。
- 类型尽量与 TS 保持一致，避免过于随意的 `any`。
- 只对“有业务意义”的变量/函数写注释，简单中间变量可以不写。
- 优先保证：组件入参（props）、事件（emit）、对外方法、复杂计算属性有清晰注释。
- 所有对外暴露的请求/响应接口类型（如 `interface`、`type`、参数对象）应在类型说明后，对每个字段使用独立的文档注释（`/** … */`），明确一行描述字段含义。可参考下方示例，避免仅靠文字说明某一类型即可理解。

### 示例：分字段注释的请求参数接口

```ts
/**
 * @description 逆地址解析（Gcoder）请求参数
 */
export interface TencentReverseGeocoderParams {
  /**
   * 腾讯位置服务开发密钥
   */
  key: string
  /**
   * GCJ02 坐标，格式为 "lat,lng"
   */
  location: string
  /**
   * 行政区划吸附半径，单位米
   */
  radius?: number
  /**
   * 是否返回周边普通地点/POI，1 为返回
   */
  get_poi?: 0 | 1
}
```

示例中的注释风格即是我们希望在类型声明中遵循的方式：每个属性前都用 `/** ... */` 解释字段含义，方便后续阅读和自动提示。
