# 字节跳动 Finder/Ranger 埋点参考

## 系统概述

Finder (也称 Ranger、AppLog、火山引擎应用性能监控) 是字节跳动内部的数据采集和分析平台,现作为火山引擎的商业化产品对外提供服务。

## 请求特征

### 收集端点

**字节内部域名:**
```
https://mon.zijieapi.com/monitor_browser/collect/
https://mon.snssdk.com/monitor/collect/
https://i.snssdk.com/log/
https://mcs.volceapplog.com/
```

**火山引擎商业化域名:**
```
https://mcs.volceapplog.com/list
https://mcs-va.volceapplog.com/list (海外)
```

### URL 参数特征
- `aid` - App ID (应用标识)
- `device_id` - 设备 ID
- `ev_type` - 事件类型 (如 `event`, `page`, `duration`)
- `app_version` - 应用版本
- `os_name` - 操作系统名称
- `os_version` - 操作系统版本

## 数据格式

### 请求格式
Finder 支持多种数据传输格式:

1. **JSON 格式** (常用于 Web)
2. **Protobuf 格式** (常用于 App,性能更优)

### JSON 数据结构示例

```json
{
  "events": [
    {
      "event": "page_view",
      "params": {
        "page_url": "https://example.com/home",
        "page_title": "首页",
        "referrer": "https://google.com"
      },
      "local_time_ms": 1638360000000
    },
    {
      "event": "button_click",
      "params": {
        "element_id": "btn-submit",
        "element_text": "提交",
        "page_url": "https://example.com/form"
      },
      "local_time_ms": 1638360005000
    }
  ],
  "user": {
    "user_unique_id": "user_12345",
    "user_is_login": true,
    "user_is_new": false
  },
  "header": {
    "app_id": 123456,
    "device_id": "device_abc123",
    "platform": "web",
    "os_name": "windows",
    "os_version": "10",
    "browser": "chrome",
    "browser_version": "96.0"
  }
}
```

### 核心字段说明

#### Header (公共头部)
- `app_id` - 应用 ID (必需)
- `device_id` - 设备唯一标识 (必需)
- `platform` - 平台 (web, android, ios)
- `os_name` / `os_version` - 操作系统信息
- `browser` / `browser_version` - 浏览器信息
- `network_type` - 网络类型 (wifi, 4g, 5g)
- `app_version` - 应用版本

#### User (用户信息)
- `user_unique_id` - 用户唯一 ID
- `user_is_login` - 是否登录
- `user_is_new` - 是否新用户
- 自定义用户属性

#### Events (事件列表)
- `event` - 事件名称
- `params` - 事件参数(键值对)
- `local_time_ms` - 客户端时间戳(毫秒)

## SDK 实现

### Web SDK 初始化

```javascript
// 引入 SDK
import Finder from '@datarangers/sdk-javascript';

// 初始化
const finder = new Finder({
  app_id: 123456, // 应用 ID
  channel: 'cn', // 数据上报渠道 (cn: 国内, sg: 海外)
  log: true, // 开启日志
  enable_ab_test: true, // 启用 AB 测试
  auto_track: true // 启用全埋点
});

// 配置用户信息
finder.config({
  user_unique_id: 'user_12345'
});

// 启动 SDK
finder.start();
```

### 发送事件

```javascript
// 发送自定义事件
finder.event('button_click', {
  button_name: '立即购买',
  product_id: 'SKU123',
  price: 99.99,
  category: '手机数码'
});

// 发送页面浏览事件
finder.event('page_view', {
  page_url: window.location.href,
  page_title: document.title,
  referrer: document.referrer
});
```

### 设置用户属性

```javascript
// 设置用户属性
finder.profileSet({
  user_name: '张三',
  age: 25,
  gender: 'male',
  membership_level: 'VIP'
});

// 增量设置(只更新特定字段)
finder.profileSetOnce({
  first_visit_time: Date.now()
});

// 数值增量
finder.profileIncrement({
  total_purchase_count: 1,
  total_amount: 99.99
});
```

## 全埋点 (Auto Track)

### 自动采集的事件

启用 `auto_track: true` 后,SDK 会自动采集:

1. **页面浏览** (`predefine_pageview`)
```javascript
{
  event: 'predefine_pageview',
  params: {
    url: 'https://example.com/home',
    title: '首页',
    referrer: 'https://google.com'
  }
}
```

2. **页面停留时长** (`predefine_pagestay`)
```javascript
{
  event: 'predefine_pagestay',
  params: {
    url: 'https://example.com/home',
    duration: 30000 // 毫秒
  }
}
```

3. **元素点击** (`bav2b_page`) - 需要额外配置
```javascript
finder.config({
  auto_track: {
    click: true, // 启用点击采集
    exposure: true // 启用曝光采集
  }
});
```

## AB 测试集成

Finder 内置 AB 测试能力:

```javascript
// 获取 AB 测试变量
finder.getVar('button_color', 'blue', (value) => {
  console.log('按钮颜色:', value);
  // 根据实验分组设置按钮颜色
  document.querySelector('#btn').style.backgroundColor = value;
});

// 上报 AB 测试曝光
finder.event('ab_test_exposure', {
  experiment_id: 'exp_001',
  variant_id: 'variant_a'
});
```

## 性能监控

### 页面性能指标

```javascript
// 自动采集页面性能
finder.config({
  enable_performance: true
});

// 手动上报性能指标
finder.event('page_performance', {
  page_url: window.location.href,
  load_time: performance.timing.loadEventEnd - performance.timing.navigationStart,
  dom_ready_time: performance.timing.domContentLoadedEventEnd - performance.timing.navigationStart,
  first_paint: performance.getEntriesByType('paint')[0]?.startTime
});
```

### 资源加载性能

```javascript
// 监控资源加载
const resources = performance.getEntriesByType('resource');
resources.forEach(resource => {
  finder.event('resource_load', {
    resource_url: resource.name,
    resource_type: resource.initiatorType,
    duration: resource.duration,
    size: resource.transferSize
  });
});
```

### 错误监控

```javascript
// 自动采集 JS 错误
window.addEventListener('error', (event) => {
  finder.event('js_error', {
    error_message: event.message,
    error_stack: event.error?.stack,
    page_url: window.location.href,
    line_number: event.lineno,
    column_number: event.colno
  });
});

// 采集 Promise 未捕获的异常
window.addEventListener('unhandledrejection', (event) => {
  finder.event('promise_error', {
    error_message: event.reason?.message || event.reason,
    page_url: window.location.href
  });
});
```

## 数据上报策略

### 批量上报
```javascript
finder.config({
  batch: {
    enable: true,
    size: 10, // 达到 10 条事件时触发上报
    timeout: 5000 // 5秒超时自动上报
  }
});
```

### 上报方式
1. **sendBeacon API** - 页面卸载时可靠上报
2. **XHR/Fetch** - 常规上报
3. **Image** - 兼容方案

```javascript
finder.config({
  send_type: 'beacon' // 'beacon' | 'xhr' | 'image'
});
```

## Protobuf 格式

对于 App 端,Finder 使用 Protobuf 压缩数据:

### Proto 定义示例
```protobuf
message Event {
  string event = 1;
  map<string, string> params = 2;
  int64 local_time_ms = 3;
}

message LogRequest {
  repeated Event events = 1;
  Header header = 2;
  User user = 3;
}
```

### 解析 Protobuf 数据
需要获取 `.proto` 文件或使用反向工程工具:

```javascript
// 使用 protobuf.js 解析
const protobuf = require('protobufjs');

protobuf.load('finder.proto', (err, root) => {
  const LogRequest = root.lookupType('LogRequest');
  const buffer = /* Protobuf 二进制数据 */;
  const message = LogRequest.decode(buffer);
  console.log(message);
});
```

## 检查要点

### 1. SDK 加载检查
```javascript
// 检查 Finder SDK 是否加载
if (window.collectEvent) {
  console.log('Finder SDK 已加载');
}
```

### 2. 网络请求验证
在 DevTools Network 面板中:
- 查找 `mon.zijieapi.com` 或 `mcs.volceapplog.com` 域名
- 检查请求 payload (可能是 JSON 或 Protobuf)
- 验证响应状态码 (200 OK)

### 3. 数据完整性
- `app_id` 和 `device_id` 是否正确
- 事件名称和参数是否符合预期
- 时间戳是否准确
- 用户标识是否一致

### 4. 预置事件检查
- 页面浏览事件是否触发
- 页面停留时长是否准确
- 用户登录状态是否正确

## 调试工具

### 1. 控制台日志
```javascript
finder.config({
  log: true, // 开启日志
  log_level: 'info' // 日志级别: debug, info, warn, error
});
```

### 2. Chrome DevTools 过滤
```
domain:*volceapplog.com
domain:*zijieapi.com
method:POST
```

### 3. Charles/Fiddler 抓包
- 设置域名过滤
- 解析 JSON/Protobuf 数据
- 验证请求频率

## 常见问题

### 1. 跨域问题
Finder 的收集域名已配置 CORS,通常不会有跨域问题。如有问题:
- 检查浏览器控制台错误
- 确认使用官方 SDK 而非自定义请求

### 2. 数据延迟
- 实时数据: 几秒到几分钟
- 分析报表: 5-10 分钟
- 大数据分析: 可能有小时级延迟

### 3. device_id 持久化
- Web 端使用 Cookie 或 LocalStorage
- App 端使用设备指纹或 IDFV/IDFA
- 注意隐私政策和用户授权

### 4. 事件丢失
- 检查批量上报配置
- 确认网络状况
- 验证 SDK 初始化时机

## 火山引擎商业化版本

如果使用火山引擎的 AppLog 产品:

```javascript
// 使用火山引擎 CDN
import RangersApplog from '@datarangers/sdk-javascript';

const ranger = new RangersApplog({
  app_id: YOUR_APP_ID,
  channel: 'cn', // cn 或 sg
  log: true
});

ranger.start();
```

官方文档: https://www.volcengine.com/docs/6285/65675

## 最佳实践

1. **事件命名规范**: 使用 snake_case,如 `button_click`, `page_view`
2. **参数设计**: 保持简洁,避免嵌套过深
3. **批量上报**: 减少网络请求次数
4. **采样策略**: 对高频事件进行采样(如滚动、鼠标移动)
5. **性能优化**: 异步加载 SDK,避免阻塞页面渲染
6. **隐私合规**: 获取用户同意,支持数据删除请求
