# Google Analytics 埋点参考

## 版本识别

### Universal Analytics (UA)
- **域名**: `www.google-analytics.com`
- **Tracking ID 格式**: `UA-XXXXXXX-Y`
- **状态**: 已于 2023 年 7 月停止处理数据

### Google Analytics 4 (GA4)
- **域名**: `www.google-analytics.com`, `analytics.google.com`
- **Measurement ID 格式**: `G-XXXXXXXXXX`
- **状态**: 当前主流版本

## 请求特征

### UA 收集端点
```
https://www.google-analytics.com/collect
```

**常见参数:**
- `v=1` - 协议版本
- `tid=UA-XXXXX-Y` - Tracking ID
- `cid=XXXX` - Client ID (匿名客户端标识符)
- `t=pageview` - Hit Type (pageview, event, transaction, etc.)
- `dl=https://example.com` - Document Location
- `dt=Page Title` - Document Title
- `dr=https://referrer.com` - Document Referrer

**事件跟踪参数:**
- `ec=category` - Event Category
- `ea=action` - Event Action
- `el=label` - Event Label
- `ev=value` - Event Value

### GA4 收集端点
```
https://www.google-analytics.com/g/collect
https://analytics.google.com/g/collect
```

**常见参数:**
- `v=2` - 协议版本
- `tid=G-XXXXXXXXXX` - Measurement ID
- `cid=XXXX.XXXX` - Client ID
- `_p=XXXXX` - 页面加载随机数
- `en=page_view` - Event Name
- `ep.page_title=Title` - Event Parameters

**GA4 使用 Measurement Protocol v2:**
- 事件驱动模型(所有数据都是事件)
- 使用 `en` 参数指定事件名称
- 使用 `ep.*` 前缀传递事件参数
- 使用 `up.*` 前缀传递用户属性

## 代码实现示例

### Universal Analytics
```javascript
// 加载 GA 脚本
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

// 初始化跟踪器
ga('create', 'UA-XXXXX-Y', 'auto');

// 发送页面浏览
ga('send', 'pageview');

// 发送事件
ga('send', 'event', 'Category', 'Action', 'Label', 123);
```

### Google Analytics 4
```javascript
// 使用 gtag.js
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');

  // 发送自定义事件
  gtag('event', 'button_click', {
    'event_category': 'engagement',
    'event_label': 'submit_form',
    'value': 1
  });
</script>
```

## 关键指标

### 页面浏览 (Page View)
- 跟踪用户访问的页面
- 包含页面 URL、标题、referrer

### 事件 (Event)
- 用户交互行为(点击、提交、视频播放等)
- UA: 类别、操作、标签、值
- GA4: 事件名称 + 自定义参数

### 电商跟踪 (E-commerce)
- 产品展示、点击、加入购物车
- 结账流程、交易完成
- GA4 支持增强型电商测量

### 用户标识
- Client ID: 浏览器级别的匿名标识
- User ID: 跨设备的已登录用户标识

## 数据层 (Data Layer)

Google Tag Manager 使用 dataLayer 对象:

```javascript
// 定义 dataLayer
window.dataLayer = window.dataLayer || [];

// 推送事件到 dataLayer
dataLayer.push({
  'event': 'purchase',
  'ecommerce': {
    'transaction_id': 'T12345',
    'value': 99.99,
    'currency': 'USD',
    'items': [{
      'item_name': 'Product Name',
      'item_id': 'SKU123',
      'price': 99.99,
      'quantity': 1
    }]
  }
});
```

## 检查要点

1. **Tracking ID 配置**: 确认使用正确的 ID
2. **页面浏览跟踪**: 每个页面是否正确触发
3. **事件跟踪**: 关键交互是否被捕获
4. **电商跟踪**: 交易数据是否完整
5. **跨域跟踪**: 多域名站点是否正确配置
6. **隐私设置**: IP 匿名化、Cookie 设置
7. **过滤器**: 是否排除内部流量

## 常见问题

### 数据延迟
- GA 实时报告: 几秒到几分钟
- 标准报告: 24-48 小时

### 采样问题
- 免费版 GA 在高流量时会采样
- GA4 标准属性: 1000万事件/月
- 解决方案: 缩小日期范围或升级到 GA360

### Cookie 依赖
- UA 依赖第三方 Cookie
- GA4 使用第一方 Cookie
- 隐私模式下可能失效
