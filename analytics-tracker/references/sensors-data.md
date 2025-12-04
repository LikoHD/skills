# 神策数据 (Sensors Data) 埋点参考

## 系统概述

神策数据是国内领先的数据分析平台,提供用户行为分析和数据驱动决策支持。

## 请求特征

### 收集端点
神策数据通常使用客户自定义的收集域名:

```
https://[client-domain]/sa
https://[client-domain]/sa.gif
https://sa.example.com/sa?project=default
```

### 常见域名模式
- `sa.[company].com`
- `sensors.[company].com`
- `data.[company].com/sa`
- CDN 加速域名

## 数据格式

### 请求方法
1. **POST 请求** - 发送 JSON 数据
2. **GET 请求** - URL 参数携带 Base64 编码数据

### 数据结构

#### 基础事件格式
```json
[
  {
    "distinct_id": "user123",
    "time": 1638360000000,
    "type": "track",
    "event": "PageView",
    "properties": {
      "page_url": "https://example.com/home",
      "page_title": "首页",
      "$os": "Windows",
      "$browser": "Chrome",
      "$screen_width": 1920,
      "$screen_height": 1080
    }
  }
]
```

#### 字段说明

**必需字段:**
- `distinct_id` - 用户唯一标识符
- `time` - 事件发生时间(毫秒时间戳)
- `type` - 事件类型
- `event` - 事件名称(当 type 为 track 时)

**事件类型 (type):**
- `track` - 行为事件
- `track_signup` - 注册事件
- `profile_set` - 用户属性设置
- `profile_set_once` - 用户属性首次设置
- `profile_increment` - 用户属性增量
- `profile_append` - 用户属性追加
- `profile_unset` - 删除用户属性
- `profile_delete` - 删除用户

**预置属性 (以 $ 开头):**
- `$os` - 操作系统
- `$browser` - 浏览器
- `$browser_version` - 浏览器版本
- `$screen_width` / `$screen_height` - 屏幕分辨率
- `$referrer` - 来源页面
- `$url` - 当前页面 URL
- `$title` - 页面标题
- `$device_id` - 设备 ID

## SDK 实现

### Web SDK 初始化
```javascript
// 加载 SDK
<script src="https://cdn.example.com/sensorsdata.min.js"></script>

// 初始化
sensors.init({
  server_url: 'https://sa.example.com/sa?project=default',
  heatmap: {
    // 热力图配置
    clickmap: 'default',
    scroll_notice_map: 'default'
  },
  show_log: true
});

// 快速注册页面浏览事件
sensors.quick('autoTrack');
```

### 发送事件
```javascript
// 追踪自定义事件
sensors.track('ButtonClick', {
  button_name: '立即购买',
  product_id: 'SKU123',
  price: 99.99
});

// 设置用户属性
sensors.setProfile({
  $name: '张三',
  $email: 'zhangsan@example.com',
  membership_level: 'VIP'
});

// 登录用户标识
sensors.login('user_id_12345');
```

## URL 参数编码

神策数据使用 Base64 + URL 编码传输数据:

### 编码示例
```javascript
// 原始数据
const data = [{
  distinct_id: "user123",
  time: Date.now(),
  type: "track",
  event: "PageView",
  properties: {
    page_url: window.location.href
  }
}];

// Base64 编码
const base64Data = btoa(JSON.stringify(data));

// URL 编码
const urlEncoded = encodeURIComponent(base64Data);

// 最终请求
// https://sa.example.com/sa?project=default&data=eyJkaXN0aW5jdF9pZCI...
```

### 解码分析
```javascript
// 从 URL 中提取 data 参数
const params = new URLSearchParams(window.location.search);
const encodedData = params.get('data');

// 解码
const decodedData = JSON.parse(atob(decodeURIComponent(encodedData)));
console.log(decodedData);
```

## 常见事件类型

### 电商场景
```javascript
// 商品浏览
sensors.track('ViewProduct', {
  product_id: 'SKU123',
  product_name: 'iPhone 15 Pro',
  category: '手机数码',
  price: 7999
});

// 加入购物车
sensors.track('AddToCart', {
  product_id: 'SKU123',
  quantity: 1
});

// 订单提交
sensors.track('SubmitOrder', {
  order_id: 'ORDER20231201001',
  amount: 7999,
  payment_method: '支付宝'
});
```

### 内容平台场景
```javascript
// 文章阅读
sensors.track('ViewArticle', {
  article_id: 'A123',
  article_title: '如何使用神策数据',
  category: '技术教程',
  reading_duration: 120 // 秒
});

// 视频播放
sensors.track('PlayVideo', {
  video_id: 'V456',
  video_title: '产品介绍',
  play_position: 30 // 秒
});
```

## 用户身份管理

### 匿名 ID vs 登录 ID

神策支持三种 ID 关联策略:

1. **登录前: 使用匿名 ID**
```javascript
// SDK 自动生成匿名 ID
sensors.track('PageView', {});
// distinct_id: "anonymous_id_xxxx"
```

2. **登录时: 绑定真实 ID**
```javascript
sensors.login('user_12345');
// 此后事件的 distinct_id 为 "user_12345"
```

3. **ID 关联**
```javascript
// 神策后台会自动关联登录前后的行为
// 匿名 ID -> 登录 ID 的映射
```

## 检查要点

### 1. SDK 加载检查
```javascript
// 检查 SDK 是否正确加载
if (typeof sensors !== 'undefined') {
  console.log('神策 SDK 已加载');
}
```

### 2. 请求验证
在 Network 面板中查找:
- 请求 URL 包含 `/sa` 或 `sa.gif`
- 查看 `data` 参数的内容
- 检查响应状态码 (200 OK)

### 3. 数据完整性
- 必需字段是否存在
- 时间戳是否准确
- 用户标识是否正确
- 自定义属性是否完整

### 4. 预置属性自动采集
- 浏览器信息
- 设备信息
- 页面 URL
- Referrer

## 高级功能

### 1. 全埋点(无埋点)
```javascript
sensors.quick('autoTrack', {
  // 自动采集所有元素点击
  element_selector: 'button, a, input',
  // 自动采集页面停留时长
  max_duration: 432000 // 5天
});
```

### 2. 可视化埋点
- 通过神策分析平台的可视化工具圈选元素
- 无需修改代码即可添加事件跟踪
- 适合营销人员和产品经理使用

### 3. 热力图
```javascript
sensors.init({
  heatmap: {
    clickmap: 'default', // 点击热力图
    scroll_notice_map: 'default', // 滚动热力图
    collect_tags: {
      div: true,
      button: true
    }
  }
});
```

### 4. Session 管理
```javascript
// 自动管理 session
// 默认 30 分钟无操作后 session 过期
sensors.track('EventName', {
  $event_session_id: 'session_xxx' // 自动添加
});
```

## 数据调试

### 1. 使用 Debug 模式
```javascript
sensors.init({
  server_url: 'https://sa.example.com/sa?project=default',
  show_log: true, // 在控制台显示日志
  name: 'sa', // SDK 实例名称
  is_track_single_page: true, // SPA 页面跟踪
  use_client_time: true, // 使用客户端时间
  send_type: 'beacon' // 使用 sendBeacon API
});
```

### 2. 验证工具
- 神策提供 Chrome 插件 "神策助手"
- 实时查看上报的事件数据
- 验证数据格式是否正确

## 性能优化

### 批量发送
```javascript
sensors.init({
  batch_send: {
    enable: true,
    size: 20, // 批量发送阈值
    timeout: 6000 // 批量发送超时(毫秒)
  }
});
```

### 采样配置
```javascript
// 对高频事件进行采样
sensors.track('HighFrequencyEvent', {
  // 只采集 10% 的数据
  $sampling_rate: 0.1
});
```

## 常见问题

### 1. 数据未上报
- 检查网络请求是否被拦截
- 确认 server_url 配置正确
- 查看浏览器控制台错误信息

### 2. 用户标识混乱
- 确保在登录时调用 `sensors.login()`
- 避免频繁切换 distinct_id
- 使用 cookie 持久化用户标识

### 3. 跨域问题
- 配置 CORS 允许埋点域名
- 使用第一方 Cookie
- 考虑使用代理转发

### 4. 时间戳问题
- 使用服务端时间 vs 客户端时间
- 注意时区差异
- 毫秒级时间戳格式
