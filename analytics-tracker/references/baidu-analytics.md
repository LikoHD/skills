# 百度统计 (Baidu Analytics) 埋点参考

## 系统概述

百度统计是百度推出的免费网站流量分析工具,在中国市场拥有广泛使用。

## 请求特征

### 收集端点

```
https://hm.baidu.com/hm.js?[站点ID]
https://hm.baidu.com/hm.gif
https://hmcdn.baidu.com/
```

### URL 参数特征
- `hm` - 站点 ID (16位十六进制字符串)
- `cc=1` - Cookie 开启
- `ck=1` - Cookie 可用
- `cl=24` - 颜色深度
- `ds=[width]x[height]` - 屏幕分辨率
- `vl=[width]x[height]` - 可视区域大小
- `ep=页面标题` - Entry Page 标题
- `et=0` - 事件类型 (0: PV, 1: 事件, 2: 下载, 3: 外链点击)
- `rnd=[random]` - 随机数,防止缓存

## 站点 ID 识别

百度统计的站点 ID 通常在 JS 加载 URL 中:

```javascript
// 站点 ID 为 16 位十六进制
https://hm.baidu.com/hm.js?1234567890abcdef
```

可以在页面源码中搜索:
```html
<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?1234567890abcdef";
  var s = document.getElementsByTagName("script")[0];
  s.parentNode.insertBefore(hm, s);
})();
</script>
```

## 数据格式

### PV 请求示例

```
https://hm.baidu.com/hm.gif?
  hm=1234567890abcdef&
  cc=1&
  ck=1&
  cl=24&
  ds=1920x1080&
  vl=1920x969&
  ep=首页&
  et=0&
  ja=0&
  ln=zh-cn&
  lo=0&
  rnd=123456789&
  si=1234567890abcdef&
  v=1.2.3&
  lv=1&
  sn=1&
  r=0&
  ww=1920&
  u=https://example.com/
```

### 参数详解

**基础参数:**
- `cc` - Cookie 是否开启 (1: 开启, 0: 关闭)
- `ck` - Cookie 是否可用
- `cl` - 颜色深度 (位数)
- `ds` - 屏幕分辨率 (宽x高)
- `vl` - 浏览器可视区域大小
- `ep` - 页面标题 (URL 编码)
- `u` - 页面 URL
- `r` - Referrer URL 的 hash 值

**事件参数 (et=1 时):**
- `ct` - 事件类别 (Category)
- `ca` - 事件操作 (Action)
- `cl` - 事件标签 (Label)
- `cv` - 事件值 (Value)

**会话参数:**
- `sn` - 会话序号
- `lv` - 访问次数
- `v` - 统计代码版本
- `rnd` - 随机数

## 代码实现

### 标准初始化代码

```javascript
// 百度统计异步加载代码
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?1234567890abcdef"; // 替换为你的站点ID
  var s = document.getElementsByTagName("script")[0];
  s.parentNode.insertBefore(hm, s);
})();
```

### 页面浏览跟踪 (PV)

```javascript
// 页面加载时自动发送 PV
// 无需手动调用,SDK 会自动处理

// SPA 应用手动发送 PV
_hmt.push(['_trackPageview', '/virtual-page']);
```

### 事件跟踪

```javascript
// 事件跟踪格式
_hmt.push(['_trackEvent', category, action, opt_label, opt_value]);

// 示例: 按钮点击
_hmt.push(['_trackEvent', '按钮', '点击', '立即购买', 1]);

// 示例: 视频播放
_hmt.push(['_trackEvent', '视频', '播放', '产品介绍视频', 30]);

// 示例: 表单提交
_hmt.push(['_trackEvent', '表单', '提交', '注册表单']);
```

### 自定义变量

```javascript
// 页面级自定义变量
_hmt.push(['_setCustomVar', index, name, value, scope]);

// 示例: 用户类型
_hmt.push(['_setCustomVar', 1, 'user_type', 'VIP', 3]);
// scope: 1=访问者级, 2=会话级, 3=页面级

// 示例: 会员等级
_hmt.push(['_setCustomVar', 2, 'membership', 'gold', 1]);
```

### 电商跟踪

```javascript
// 商品详情页
_hmt.push(['_addItem',
  'orderID',      // 订单号
  'SKU123',       // 商品SKU
  'iPhone 15',    // 商品名称
  '手机',         // 分类
  7999,           // 单价
  1               // 数量
]);

// 订单完成
_hmt.push(['_addTrans',
  'ORDER12345',   // 订单号
  '',             // 店铺名称
  7999,           // 订单总额
  0,              // 税费
  0,              // 运费
  'Beijing',      // 城市
  'Beijing',      // 省份
  'China'         // 国家
]);

// 发送电商数据
_hmt.push(['_trackTrans']);
```

## 高级功能

### 1. 跨域跟踪

```javascript
// 设置子域名共享 Cookie
_hmt.push(['_setDomainName', '.example.com']);

// 跨域链接标记
_hmt.push(['_addOrganic', 'baidu', 'word']);
```

### 2. 搜索词跟踪

```javascript
// 站内搜索跟踪
_hmt.push(['_trackEvent', '站内搜索', '搜索', '关键词: iPhone']);
```

### 3. 下载跟踪

```javascript
// 跟踪文件下载
document.querySelectorAll('a[download]').forEach(link => {
  link.addEventListener('click', function() {
    _hmt.push(['_trackEvent', '下载', '点击', this.href]);
  });
});
```

### 4. 外链点击跟踪

```javascript
// 跟踪外部链接点击
document.querySelectorAll('a[href^="http"]').forEach(link => {
  if (!link.href.includes(window.location.hostname)) {
    link.addEventListener('click', function() {
      _hmt.push(['_trackEvent', '外链', '点击', this.href]);
    });
  }
});
```

## 热力图

百度统计提供点击热力图和链接点击图功能:

```javascript
// 热力图会自动采集,无需额外代码
// 在百度统计后台启用热力图功能即可

// 如需排除某些元素不参与热力图统计
<div class="no-heatmap">此区域不统计</div>
```

## Session 管理

百度统计使用 Cookie 管理会话:

### Cookie 说明
- `HMACCOUNT` - 账号 Cookie
- `Hm_lvt_[站点ID]` - 访问时间列表 (最多记录4次)
- `Hm_lpvt_[站点ID]` - 最后一次访问时间

### Cookie 示例
```
Hm_lvt_1234567890abcdef=1638360000,1638361000,1638362000,1638363000
Hm_lpvt_1234567890abcdef=1638363000
```

### Session 超时
- 默认 30 分钟无活动则 Session 结束
- 跨日访问会创建新 Session

## 检查要点

### 1. 代码安装检查
```javascript
// 检查百度统计是否加载
if (typeof _hmt !== 'undefined') {
  console.log('百度统计已加载');
}

// 检查站点 ID
var scripts = document.getElementsByTagName('script');
for (var i = 0; i < scripts.length; i++) {
  if (scripts[i].src.includes('hm.baidu.com/hm.js')) {
    console.log('站点 ID:', scripts[i].src.split('?')[1]);
  }
}
```

### 2. 网络请求验证
在 Network 面板中:
- 查找 `hm.baidu.com/hm.gif` 请求
- 检查请求参数是否完整
- 验证响应状态码 (204 No Content 或 200 OK)

### 3. 事件触发验证
```javascript
// 监听 _hmt 队列
var originalPush = _hmt.push;
_hmt.push = function() {
  console.log('百度统计事件:', arguments);
  return originalPush.apply(this, arguments);
};
```

### 4. Cookie 验证
```javascript
// 检查百度统计 Cookie
document.cookie.split(';').forEach(cookie => {
  if (cookie.includes('Hm_')) {
    console.log('百度统计 Cookie:', cookie.trim());
  }
});
```

## 隐私合规

### GDPR/CCPA 合规
百度统计默认不支持 GDPR 用户同意机制,需要自行实现:

```javascript
// 用户同意后再加载百度统计
function loadBaiduAnalytics() {
  if (userConsent) {
    var hm = document.createElement("script");
    hm.src = "https://hm.baidu.com/hm.js?1234567890abcdef";
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(hm, s);
  }
}

// 用户点击同意后调用
document.querySelector('#consent-btn').addEventListener('click', loadBaiduAnalytics);
```

### IP 匿名化
百度统计不提供 IP 匿名化功能,IP 地址会被记录用于地理位置分析。

## 性能影响

### 加载方式
- 异步加载,不阻塞页面渲染
- 建议放在 `</body>` 前

### 性能优化
```javascript
// 延迟加载统计代码
window.addEventListener('load', function() {
  setTimeout(function() {
    // 加载百度统计
  }, 1000);
});
```

## 常见问题

### 1. PV 统计不准确
- 检查是否有重复的统计代码
- SPA 应用需手动触发虚拟 PV
- 排查 iframe 中的重复统计

### 2. 事件未上报
- 检查 _hmt 变量是否存在
- 确认事件调用时机(是否在 SDK 加载前)
- 查看 Network 面板是否有请求失败

### 3. 跨域跟踪失效
- 确认 Cookie 域设置正确
- 检查浏览器的第三方 Cookie 设置
- Safari 可能会阻止跨域 Cookie

### 4. 实时数据延迟
- 实时数据: 约 10-20 秒延迟
- 标准报告: 4-5 小时更新
- 趋势分析: 天级更新

## 百度统计后台功能

### 可查看的数据
- 实时访客
- 流量来源分析
- 访问页面排行
- 搜索词分析
- 访客属性(地域、系统、浏览器)
- 事件跟踪报告
- 热力图

### 目标转化设置
在后台可设置转化目标:
- URL 目标(访问特定页面)
- 事件目标(触发特定事件)
- 持续时间目标(访问时长)
- 页面浏览数目标(浏览页面数)

## 对比其他平台

| 功能 | 百度统计 | Google Analytics | 神策数据 |
|------|---------|------------------|---------|
| 实时数据 | 支持(延迟10-20秒) | 支持(延迟几秒) | 支持(延迟几秒) |
| 免费版限额 | 无限制 | 1000万次/月 | 根据套餐 |
| 热力图 | 内置 | 需第三方 | 支持 |
| 电商跟踪 | 基础支持 | 完善 | 完善 |
| 用户细分 | 基础 | 强大 | 强大 |
| 自定义事件 | 支持 | 支持 | 支持 |
| 隐私合规 | 基础 | 较完善 | 较完善 |

## 调试工具

### 1. Chrome DevTools
- Network 面板过滤 `hm.baidu.com`
- Console 监听 `_hmt` 队列

### 2. 百度统计助手 (Chrome 插件)
- 实时查看事件上报
- 验证代码安装
- 查看 Cookie 信息

### 3. Charles/Fiddler
- 抓取 `hm.gif` 请求
- 解析 URL 参数
- 验证数据完整性

## 最佳实践

1. **代码部署**: 放在 `</body>` 前,异步加载
2. **SPA 应用**: 路由变化时手动调用 `_trackPageview`
3. **事件命名**: 使用清晰的类别、操作、标签
4. **性能监控**: 避免高频事件(如鼠标移动、滚动)
5. **测试验证**: 使用实时访客功能验证埋点
6. **数据治理**: 建立事件命名规范和文档
