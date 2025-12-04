---
name: analytics-tracker
description: Analyzes website analytics and tracking implementations including Google Analytics, Baidu Analytics, Sensors Data, ByteDance Finder, and other tracking pixels. Use when analyzing埋点 (tracking events), monitoring network requests, or reviewing analytics setup.
---

# Analytics Tracker - 网站埋点分析器

这个 Skill 用于分析网站的埋点实现,帮助你理解网站如何收集用户行为数据。

## 功能说明

当用户提供一个网站 URL 时,这个 Skill 会指导如何:

1. **监控网络请求** - 使用浏览器开发者工具或代理工具捕获请求
2. **识别埋点系统** - 检测常见的分析平台
3. **分析埋点设计** - 评估数据收集的完整性和质量
4. **生成分析报告** - 提供结构化的埋点分析结果

## 支持的埋点系统

### 国际主流平台
- **Google Analytics (GA)** - GA4 和 Universal Analytics
- **Google Tag Manager (GTM)**
- **Facebook Pixel**
- **Mixpanel**
- **Amplitude**
- **Segment**

### 国内主流平台
- **百度统计** (Baidu Analytics)
- **神策数据** (Sensors Data)
- **字节跳动 Finder/Ranger**
- **友盟** (Umeng)
- **腾讯分析** (Tencent Analytics)
- **热云数据** (Trackingio)

## 分析流程

### 步骤 1: 获取网络请求

使用以下方法之一捕获网站的网络请求:

**方法 A: Chrome DevTools**
```javascript
// 在浏览器控制台运行,记录所有 XHR/Fetch 请求
const originalFetch = window.fetch;
const originalXHR = window.XMLHttpRequest.prototype.open;

window.fetch = function(...args) {
  console.log('Fetch Request:', args[0]);
  return originalFetch.apply(this, args);
};

window.XMLHttpRequest.prototype.open = function(method, url) {
  console.log('XHR Request:', method, url);
  return originalXHR.apply(this, arguments);
};
```

**方法 B: 使用 HAR 文件**
1. 打开 Chrome DevTools → Network 标签
2. 访问目标网站并进行交互
3. 右键点击任意请求 → "Save all as HAR with content"
4. 将 HAR 文件提供给分析

**方法 C: 使用命令行工具**
```bash
# 使用 curl 捕获请求头
curl -I "https://example.com"

# 使用 mitmproxy 拦截 HTTPS 请求
mitmproxy --mode reverse:https://example.com
```

### 步骤 2: 识别埋点请求

根据请求特征识别不同的埋点系统:

#### Google Analytics 特征
- **域名**: `google-analytics.com`, `analytics.google.com`
- **路径**: `/collect`, `/g/collect`, `/j/collect`
- **参数**: `tid` (Tracking ID), `cid` (Client ID), `t` (Hit Type)
- **示例**: `https://www.google-analytics.com/collect?v=1&tid=UA-XXXXX-Y`

#### 百度统计特征
- **域名**: `hm.baidu.com`, `hmcdn.baidu.com`
- **路径**: `/hm.js`, `/hm.gif`
- **参数**: `hm` (站点ID), `cc` (字符集), `et` (事件类型)

#### 神策数据特征 ⭐ 重要
- **域名**:
  - 通常是**客户自定义域名**（如 `api.saasexch.com`）
  - 也可能是神策官方域名（如 `sensors-data.com`）
  - ⚠️ **关键**: 不要只依赖域名，重点看路径和参数！

- **路径特征**（最可靠的识别方式）:
  - `/sa.gif` ✅ **核心特征**
  - `/sa`
  - `/sa.gif?project=xxx&data=xxx` ✅ **标准格式**
  - 可能在各种路径下，如 `/bapi/fe/usd/sa.gif`

- **参数特征**（必备）:
  - `project=<项目名称>` ✅ **必有参数**
  - `data=<Base64编码的数据>` ✅ **必有参数**
  - `ext=crc%3D<校验码>` (可选)

- **数据编码特征**:
  - `data` 参数是 **Base64 + URL Encoding** 双重编码
  - 解码后是包含 `distinct_id`, `event`, `properties` 的 JSON

- **识别优先级**:
  1. **最高优先级**: URL 包含 `sa.gif`
  2. **次优先级**: 查询参数包含 `project=` 和 `data=`
  3. **辅助特征**: 域名包含 `sa` 或 `sensors`

- **示例 URL**:
  ```
  https://api.saasexch.com/bapi/fe/usd/sa.gif?project=binance&data=eyJpZGVudGl0aWVzIjp7...
  https://sa.example.com/sa.gif?project=myapp&data=eyJ0eXBlIjoidHJhY2siLCJldmVudCI6...
  ```

#### 字节 Finder/Ranger 特征 ⭐ 重要
- **域名**:
  - `mon.zijieapi.com`
  - `mon.snssdk.com`
  - `i.snssdk.com`
  - `mcs.volceapplog.com`
  - `toblog.ctobsnssdk.com`
  - 或其他字节跳动相关域名

- **路径特征**（最可靠的识别方式）:
  - `/monitor_browser/collect/` ✅ **浏览器端核心路径**
  - `/monitor/collect/`
  - `/service/2/collect/` ✅ **常见格式**
  - 可能包含版本号，如 `/v1/collect/`

- **参数特征**（必备）:
  - `aid=<App ID>` ✅ **必有参数** - 应用标识
  - `list=<事件数组>` ✅ **核心特征** - 批量事件列表
  - `ev_type=<事件类型>` (可选)
  - `device_id=<设备ID>` (常见)
  - `user_unique_id=<用户ID>` (常见)

- **数据格式特征**:
  - **方法**: POST (主要) 或 GET
  - **编码**: 可能使用 Protobuf (二进制) 或 JSON
  - **批量发送**: `list` 参数包含多个事件的数组
  - **压缩**: 可能使用 gzip 压缩

- **`list` 参数结构**:
  ```json
  {
    "list": [
      {
        "event": "page_view",
        "params": "{\"url\":\"https://example.com\"}",
        "local_time_ms": 1638360000000
      },
      {
        "event": "click",
        "params": "{\"element_id\":\"btn-submit\"}",
        "local_time_ms": 1638360001000
      }
    ]
  }
  ```

- **识别优先级**:
  1. **最高优先级**: 查询参数包含 `list=` ✅
  2. **次优先级**: 路径包含 `/collect/` + 域名包含字节特征
  3. **辅助特征**: 参数包含 `aid=` 和 `device_id=`

- **示例 URL**:
  ```
  POST https://mcs.volceapplog.com/service/2/collect/?aid=123456&list=[...]
  POST https://mon.snssdk.com/monitor/collect/?aid=654321&device_id=xxx&list=[...]
  ```

- **特殊说明**:
  - 字节跳动埋点系统通常**批量发送**事件，提高性能
  - `list` 参数可能包含**多个事件**，需要解析数组
  - 如果使用 Protobuf 编码，需要对应的 `.proto` 文件才能解码

### 步骤 3: 分析埋点质量

评估以下维度:

#### 1. 覆盖完整性
- [ ] 页面浏览 (Page View) 跟踪
- [ ] 用户行为事件 (Click, Submit, Scroll)
- [ ] 转化目标跟踪
- [ ] 异常错误跟踪
- [ ] 性能指标跟踪

#### 2. 埋点设计
- [ ] 主要参数结构
- [ ] 事件命名规范
- [ ] 事件属性设计
- [ ] 事件维度设计
- [ ] 事件关联设计



#### 2. 数据质量
- [ ] 用户标识是否唯一且持久 (User ID/Device ID)
- [ ] 事件属性是否完整 (Properties)
- [ ] 时间戳是否准确
- [ ] 是否有数据验证和去重机制

#### 3. 隐私合规
- [ ] 是否展示隐私政策
- [ ] 是否提供用户同意机制 (Cookie Banner)
- [ ] 敏感数据是否脱敏
- [ ] 是否符合 GDPR/CCPA/个人信息保护法

#### 4. 技术实现
- [ ] 加载方式 (同步/异步)
- [ ] 错误处理机制
- [ ] 性能影响评估
- [ ] 采样策略

### 步骤 4: 生成分析报告

使用 `report-template.md` 生成结构化报告。

## 常见埋点模式

### 1. 直接请求模式
```javascript
// 发送 1x1 像素 GIF 请求
const img = new Image();
img.src = 'https://analytics.example.com/track.gif?event=click&page=/home';
```

### 2. Beacon API 模式
```javascript
// 使用 Navigator.sendBeacon 发送埋点
navigator.sendBeacon('https://analytics.example.com/collect', JSON.stringify({
  event: 'page_view',
  properties: { page: '/home' }
}));
```

### 3. Fetch/XHR 模式
```javascript
// 使用 Fetch API 发送埋点
fetch('https://analytics.example.com/api/track', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ event: 'click', element_id: 'btn-submit' })
});
```

### 4. Script Tag 模式
```html
<!-- 异步加载跟踪脚本 -->
<script async src="https://analytics.example.com/sdk.js"></script>
```

## 解码埋点数据

### Base64 解码
许多埋点系统使用 Base64 编码传输数据:

```javascript
// 解码神策数据
const encodedData = 'eyJldmVudCI6ImNsaWNrIiwicHJvcGVydGllcyI6eyJidXR0b25faWQiOiJzdWJtaXQifX0=';
const decodedData = atob(encodedData);
console.log(JSON.parse(decodedData));
```

### URL 解码
```javascript
const params = new URLSearchParams(window.location.search);
const eventData = decodeURIComponent(params.get('data'));
```

### Protobuf 解码
对于使用 Protobuf 的系统(如字节 Finder),需要:
1. 获取 .proto 定义文件
2. 使用 protobuf.js 或其他工具解码
3. 分析解码后的结构化数据

**字节 Finder Protobuf 解码示例**:
```javascript
// 需要先安装 protobufjs: npm install protobufjs
const protobuf = require('protobufjs');

// 加载 .proto 文件
protobuf.load('finder.proto', (err, root) => {
  if (err) throw err;

  // 获取消息类型
  const EventMessage = root.lookupType('finder.Event');

  // 解码二进制数据
  const buffer = Buffer.from(binaryData, 'base64');
  const message = EventMessage.decode(buffer);
  const object = EventMessage.toObject(message);

  console.log(JSON.stringify(object, null, 2));
});
```

### JSON 数组解码
字节 Finder 的 `list` 参数通常是 URL 编码的 JSON 数组:

```javascript
// 假设从 URL 中提取到了 list 参数
const listParam = decodeURIComponent(urlParams.get('list'));
const events = JSON.parse(listParam);

events.forEach(event => {
  console.log('Event:', event.event);
  console.log('Params:', JSON.parse(event.params));
  console.log('Time:', new Date(event.local_time_ms));
});
```

## 最佳实践建议

基于分析结果,提供以下方面的建议:

1. **事件命名规范** - 使用清晰、一致的命名约定
2. **属性设计** - 包含足够的上下文信息
3. **数据治理** - 建立数据字典和文档
4. **性能优化** - 批量发送、异步加载、采样策略
5. **隐私保护** - 数据脱敏、用户同意、数据保留政策

## 工具推荐

- **Tag Assistant** (Chrome插件) - Google 标签助手
- **Omnibug** (Chrome插件) - 多平台埋点调试
- **Charles/Fiddler** - HTTP 代理调试工具
- **Wireshark** - 网络包分析
- **神策埋点验证工具** - 神策官方提供

## 常见问题排查

### 埋点未触发
- 检查脚本是否正确加载
- 验证事件监听器是否绑定
- 确认网络请求是否被拦截

### 数据不准确
- 检查时区设置
- 验证用户标识逻辑
- 排查重复上报问题

### 性能影响
- 检查脚本加载方式(同步→异步)
- 实施批量发送策略
- 考虑采样机制

## 参考文档

详细的埋点系统规范请查看:
- `references/google-analytics.md`
- `references/sensors-data.md`
- `references/finder-tracker.md`
- `references/baidu-analytics.md`

## 使用示例

**用户输入:**
```
请分析 https://www.example.com 的埋点实现
```

**Skill 工作流程:**
1. 引导用户捕获网络请求(提供 DevTools 脚本或 HAR 导出说明)
2. 识别请求中的埋点系统特征
3. 分析埋点数据结构和传输内容
4. 评估埋点质量(完整性、准确性、合规性)
5. 生成分析报告,包含发现的问题和改进建议

---

**注意事项:**
- 埋点分析需要实际访问网站并捕获网络流量
- 某些埋点数据可能经过加密或混淆
- 分析时请遵守网站的服务条款和隐私政策

---

## ⚡ 快速识别检查清单

### 1️⃣ 打开 DevTools Network 标签
```
Chrome DevTools → Network → XHR/Fetch
```

### 2️⃣ 刷新页面并搜索关键特征

| 埋点系统 | 搜索关键词 | 必有特征 |
|---------|-----------|---------|
| **神策数据** | `sa.gif` | URL 包含 `sa.gif` + `project=` + `data=` |
| **字节 Finder** | `list=` 或 `/collect/` | 参数包含 `list=` + `aid=` |
| **百度统计** | `hm.gif` 或 `hm.baidu.com` | URL 包含 `hm.gif` 或域名 `hm.baidu.com` |
| **Google Analytics** | `/collect` | URL 包含 `/collect` + `tid=` + `cid=` |
| **Sentry** | `sentry.io` | 域名包含 `sentry.io` |
| **Mixpanel** | `mixpanel.com` | 域名包含 `mixpanel.com` + `data=` |
| **Amplitude** | `amplitude.com` | 域名包含 `amplitude.com` |
| **Facebook Pixel** | `facebook.com/tr` | URL 包含 `facebook.com/tr` |

### 3️⃣ 检查请求详情

**查看以下信息**:
- [ ] Request URL (完整 URL)
- [ ] Request Method (GET/POST)
- [ ] Query String Parameters (查询参数)
- [ ] Request Payload (POST 请求体)
- [ ] Response (响应内容)

### 4️⃣ 解码数据

根据编码方式解码：
- **Base64**: 使用 `atob()` 或在线工具
- **URL Encoding**: 使用 `decodeURIComponent()`
- **Protobuf**: 需要 .proto 文件和 protobufjs
- **gzip**: 使用 pako 库或命令行 `gunzip`

### 5️⃣ 模拟用户交互

触发更多埋点：
- [ ] 点击按钮
- [ ] 滚动页面
- [ ] 填写表单
- [ ] 离开页面（关闭标签页）
- [ ] 等待 5-10 秒（延迟触发）

---

## 🔥 常见陷阱与解决方案

### 陷阱 1: 只看域名，忽略路径
❌ **错误**: `api.example.com` → 不是埋点
✅ **正确**: `api.example.com/sa.gif` → 神策数据！

### 陷阱 2: 过滤关键词太宽泛
❌ **错误**: 搜索 `log` → 匹配到几百个请求
✅ **正确**: 搜索 `sa.gif`、`list=`、`/collect` → 精确匹配

### 陷阱 3: 只用自动化工具
❌ **错误**: Puppeteer 没抓到 → 认为没有埋点
✅ **正确**: Puppeteer + 手动 DevTools → 交叉验证

### 陷阱 4: 忽略批量请求
❌ **错误**: 只看单个事件请求
✅ **正确**: 注意 `list` 参数的批量事件数组

### 陷阱 5: 不解码数据
❌ **错误**: 看到 Base64 就放弃
✅ **正确**: 解码后才能看到真实数据结构

---

## 📚 核心识别规则总结

### 规则 1: 路径优先于域名
```
优先级: 路径特征 > 参数特征 > 域名特征
```

### 规则 2: 精确匹配优先于模糊匹配
```javascript
// ✅ 好
if (url.includes('sa.gif')) { ... }

// ❌ 差
if (url.includes('sa')) { ... }  // 会误匹配 'usage', 'message' 等
```

### 规则 3: 必备参数是核心特征
```
神策: project= + data=
字节: list= + aid=
GA: tid= + cid=
```

### 规则 4: 解码才是真相
```
Base64 编码 → 解码 → JSON 解析 → 看到真实数据
```

---

## 🎯 实战技巧

### 技巧 1: 使用 DevTools 的 Filter
```
在 Network 标签的 Filter 输入框中输入:
- method:POST  (只看 POST 请求)
- domain:*analytics*  (域名包含 analytics)
- sa.gif  (URL 包含 sa.gif)
```

### 技巧 2: 复制为 cURL 命令
```
右键请求 → Copy → Copy as cURL
可以在终端重放请求，方便调试
```

### 技巧 3: 保存为 HAR 文件
```
右键 Network 标签 → Save all as HAR with content
可以离线分析，或用工具批量处理
```

### 技巧 4: 使用浏览器扩展
```
- Omnibug: 实时显示埋点数据
- Tag Assistant: Google 官方工具
- 神策埋点验证工具: 神策官方扩展
```

### 技巧 5: 编写自动化脚本
```javascript
// Puppeteer 监控脚本，按系统分类存储
const trackers = {
  sensorsData: [],
  byteFinder: [],
  sentry: []
};

// 精确识别函数
function identifyTracker(url) {
  if (url.includes('sa.gif')) return 'sensorsData';
  if (url.includes('list=')) return 'byteFinder';
  // ...
}
```

---

## 🚨 重要提醒

1. **隐私与合规**
   - 分析埋点时要遵守隐私法规
   - 不要泄露用户敏感信息
   - 仅用于技术学习和优化目的

2. **反爬虫检测**
   - Headless 浏览器可能被识别
   - 某些埋点会检测自动化环境
   - 建议结合真实浏览器测试

3. **数据保护**
   - 不要公开分享埋点数据
   - 不要逆向破解加密算法
   - 尊重网站的服务条款

---

**Skill 版本**: 2.0
**最后更新**: 2025-12-03
**更新内容**: 补充神策数据和字节 Finder 的详细识别规则
