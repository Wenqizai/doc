# HTTP 缓存层：Cache-Control 与 ETag 的生产实践

> **读者定位：** 后端、前端、运维工程师（假设已了解 HTTP/1.1 基础协议）
> 
> **数据来源：** `big-list-of-20-common-bottlenecks.md` 第 39 行：**"In HTTP: headers, etags, not gzipping, etc.."**
> 
> **声明：** 本文档所有性能指标均为教学示例值，非真实生产数据。

---

## 1. 核心问题：缓存层为什么能成为系统瓶颈？

20 个常见系统瓶颈清单的第 39 行明确写道：
> **"In HTTP: headers, etags, not gzipping, etc.."**

这意味着：**HTTP 响应头配置不当，本身就可以成为系统瓶颈。**

缓存层有两个职责：
- **减少回源请求数** → 节省服务器计算资源
- **减少响应体传输量** → 节省网络带宽

当这两个目标**冲突**时，就可能制造反效果：ETag 计算 CPU 开销 + gzip 双重压缩反而拖慢响应，或将本应简短的 API 请求变为需要反复验证条件的新鲜度检查风暴。

---

## 2. Cache-Control：一个固定值，不是计数器

### 2.1 每次传输的 `max-age` 都是固定值

当浏览器收到 `Cache-Control: max-age=3600`，这个数字永远不会变。服务器每次发响应，内容里永远是 `max-age=3600`。它不是递减的，也不是递增的。服务器只是告诉浏览器："这个响应你可以缓存 3600 秒"。

### 2.2 计时的起点：`Date` 响应头

```
HTTP/1.1 200 OK
Date: Wed, 01 Jan 2026 12:00:00 GMT      ← 【这就是起点】
Cache-Control: max-age=3600

{ ... body ... }
```

浏览器用 `Date` 头的值作为计时起点：

- `fresh_until = Date + max-age`
- 例子：12:00:00 + 3600 秒 = **13:00:00**

在 `fresh_until` 之前，浏览器不需要向服务端发起请求。使用本地缓存即可响应。

在 `fresh_until` 之后，浏览器需要重新检查（发条件请求或直接回源）。

### 2.3 存放在哪里？——浏览器磁盘缓存数据库

```
浏览器内部：HTTP 缓存存储 (Disk Cache Database)

每条记录的完整结构：
┌───────────────────────────────────────────────┐
│ Key: GET https://example.com/api/users        │  ← URI + HTTP 方法
├───────────────────────────────────────────────┤
│ Status: 200 OK                                │  ← HTTP 状态码
│ Content-Type: application/json                │  ← 响应头 ...
│ Cache-Control: max-age=3600                   │  ← 【固定值存在这里】
│ Date: Wed, 01 Jan 2026 12:00:00 GMT           │  ← 【计时起点存在这里】
├───────────────────────────────────────────────┤
│ expires_time: 12:00:00 + 3600s = 13:00:00     │  ← 【浏览器计算出的过期时刻】
│ cached_at: 12:00:05                           │  ← 【实际存入时间】
│ body: [{ ...json body... }]                   │  ← 【响应体存储位置】
└───────────────────────────────────────────────┘

具体到各浏览器：
  Chrome  → ~/Library/Caches/google_chrome/Cache/datacache_* (Linux: ~/.config/google-chrome/)
  Firefox → ~/Library/Caches/Firefox/Default/cache2/* (Linux: ~/.cache/mozilla/firefox/)
  Safari  → ~/Library/Caches/com.apple.Safari/Cache/*

每条记录的淘汰通过 LRU 算法管理（最大空间约磁盘的 5%）。
```

### 2.4 浏览器何时检查缓存是否新鲜？

每次发起请求时，浏览器内部做如下判断：

```
if (current_time < cached_at + max_age) {
    // 响应体"新鲜" → 直接使用本地缓存，跳过网络
    return cached_body;
} else {
    // 响应体"陈旧" → 需要向服务端检查（条件请求或直接回源）
    send_request_with_If_None_Match(etag);  // 发条件请求
}
```

| 时间点 | 状态 | 浏览器行为 |
|--------|------|-----------|
| t=12:05:30（距缓存 5 分钟） | fresh | 直接使用缓存体，不发网络请求 |
| t=12:59:30（距缓存 59 分钟） | fresh | 直接使用缓存体，不发网络请求 |
| t=13:05:15（距缓存 65 分钟） | stale! | 向服务端发送条件请求 `If-None-Match` 或直接回源 |

---

## 3. Cache-Control：在哪里设置？怎么确定值？

### 3.1 多层架构下的四层控制点

```
用户浏览器 ←——— (CDN 发 header，浏览器收到) —————─
                ↘ (回源，从 Nginx 发 header) ——
                         ↘ (从后端应用发 header) ————

三层架构各自有设置 Cache-Control 的位置：
┌─────────────────────────────┐ ─── 后端应用层 ──────────┐
│                              │                         │
│  Spring Boot：               │                         │
│    .cacheControl(           │                         │
│        CacheControl         │                         │
│            .maxAge(30, S)   │                         │
│    )                        │                         │
│                              │                         │
│  Go (Gin)：                  │                         │
│    c.Header("Cache-Control", │                         │
│         "no-store")          │                         │
│                              │                         │
└──────────────┬───────────────┘                         │
                │ 如果未禁用 upstream headers
     ┌──────────┴───────────────┐ ─── Nginx 层 ──────────┐
     │                          │                         │
     │  location /static/ {     │                         │
     │      expires 7d;         │ ← "expires 7d" =        │
     │      add_header          │   Nginx 添加的           │
     │          Cache-Control   │   Cache-Control         │
     │          "public";       │                         │
     │  }                      │                         │
     │                          │                         │
     │  location /api/ {        │                         │
     │      add_header          │   API 覆盖：不缓存       │
     │          Cache-Control   │                         │
     │          "no-store";     │                         │
     │  }                      │                         │
     └──────────────┬───────────┘                         │
                    │ 如果 CDN 启用缓存策略，覆盖上层
     ┌──────────────┴───────────────┐ ─── CDN 层 ──────────┐
     │                              │                       │
     │  CloudFront / Aliyun CDN     │  控制面板/API 定义：  │
     │  Cache Policy：              │                       │
     │    URI: /*.{js,css}          │  → max-age=31536000   │
     │    URI: /api/*               │  → no-cache,          │
     │                              │                       │
     └──────────────────────────────┘ 最外层生效              │

底层的规则：Cache-Control 的值由**离浏览器最近的缓存节点**的控制策略决定。
```

### 3.2 各层设置 Cache-Control 的示例

**后端应用代码：**
```python
# Spring Boot：控制器级设置
@GetMapping("/api/products")
public ResponseEntity<List<Product>> list() {
    return ResponseEntity.ok(products)
        .cacheControl(CacheControl.maxAge(30, TimeUnit.SECONDS));
    // 结果：Cache-Control: max-age=30   ← 固定值，每次都发
}

# 动态计算：根据数据类型返回不同的 max-age
def get_max_age(resource):
    if resource.is_hot:
        return 60          # 热门数据：每分钟刷新（变更频繁）
    elif resource.is_cold:
        return 86400       # 冷门数据：一天刷新（内容稳定）
    else:
        return 300         # 普通数据：5分钟

# Go / Gin 框架
func productsHandler(c *gin.Context) {
    resource := getProduct(ctx)
    
    if resource.IsHot() {
        c.Header("Cache-Control", "max-age=60")       // 热门商品：1分钟
    } else {
        c.Header("Cache-Control", "max-age=86400")    // 冷门商品：24小时
    }
}
```

**Nginx 配置（静态文件）：**
```nginx
# JS / CSS —— 7天缓存（相对当前时间的 max-age）
location ~* \.(js|css)$ {
    expires 7d;                          # Nginx 内部转换为 Cache-Control: max-age=604800
    add_header Cache-Control "public";   # 可选：覆盖明确设置
}

# 图片 —— 30天缓存
location ~* \.(png|jpg)$ {
    expires 30d;                         # Cache-Control: max-age=2592000
}

# API —— 强制不缓存
location /api/ {
    add_header Cache-Control "no-store"; # 不适用任何缓存

    proxy_pass http://backend;
}

# HTML 入口页 —— 每次检查，但通过 ETag 判断是否返回完整内容
location / {
    add_header Cache-Control "no-cache, must-revalidate";
}
```

**CDN 控制台配置：**
| URI Pattern | Cache Policy (推荐) | 说明 |
|------------|---------------------|------|
| `/*.{js,css}` | `max-age=31536000, immutable` | 文件名带内容 hash → 永久缓存安全 |
| `/*.{png,jpg,svg,woff2}` | `max-age=31536000` | 图片长期缓存，文件名通常不变需定期 purge |
| `/*.html` | `no-cache, must-revalidate` | 每次向 CDN 检查，通过 ETag 判断是否更新 |
| `/api/*` | `no-cache, max-age=0` 或 `bypass` | API 不适用缓存，直接回源至 Origin |

---

## 4. ETag：在哪里设置？如何生成值？

### 4.1 三层架构下的三个 ETag 控制点

```
┌────────────────── Browser ──────────────┐  ← ETag 在这里被浏览器读取并存入缓存
│ Browser Cache: key = (URI, ETag)        │  浏览器下次请求时读这个值，放入 If-None-Match
└──────────┬──────────────────────────────┘  浏览器读取 → 下次发送 If-None-Match
           │ GET, If-None-Match: "abc"
     ┌──────┴─────────────────────────┐  ─CDN 层（可选）──────────────┐
     │ CDN (CloudFront / Aliyun OSS)  │                              │
     │ ETag: "abc" ← 【CDN 设置】      │  CDN 自己计算并写入缓存      │
     │ (命中则直接返回 304，不打 Origin)│                              │
     └──────┬─────────────────────────┘ 与 Origin 不交互             │
            │ GET, If-None-Match: "abc"（CDN 回源）
     ┌──────┴─────────────────────────┐  ─Nginx/反向代理层────────────┐
     │ Nginx                          │                              │
     │ etag on;  ← 【Nginx 自动设置】  │  基于响应体 MD5 计算          │
     │ etag off;   ← 【手动关闭】      │                              │
     └──────┬─────────────────────────┘ 不打到后端                   │
            │ GET
     ┌──────┴─────────────────────────┐  ─后端应用层──────────────────┐
     │ Spring Boot / Go / Node.js     │                              │
     │ ctx.Header("ETag") = sha256()  │  【应用代码手动设置】         │
     │ etag on;   ← 【Nginx 计算 】     │                              │
     └───────────────────────────────┘  无 ETag → 只能走完整响应
```

**三个要点：**

| 层级 | 谁来设置 ETag？ | 怎么生成值？ |
|------|-----------------|-------------|
| **后端应用（代码中）** | 开发者在 Handler/Controller 中设置 | 自由决定：版本号 + 时间戳，内容 hash（SHA256/MD5），行锁版本字段值 |
| **Nginx（反向代理）** | `etag on;` 开启后自动计算 | Nginx 对原始响应体做 MD5 哈希，取前 8 字节大写十六进制（大写字母 + 十六进制数字） |
| **CDN（边缘节点）** | CDN 服务商在缓存命中时写入 | CDN 根据本地缓存体的内容计算（CloudFront / Akamai / 阿里云各有不同实现） |
| **浏览器** | 不设置，只读取 | 把收到的值存入 `If-None-Match` 请求头并发送给服务端 |

### 4.2 ETag 值的常见生成策略

**策略 A：内容 hash（Nginx 默认）**
```nginx
# Nginx 源码中 etag on; 时：响应体做 MD5 → 取前 8 bytes → 大写字母十六进制 → "D41D8CD9"
# 优点：语义绝对正确（内容一致则 ETag 一致）
# 缺点：每次请求都要读完整 body 做 MD5，对大响应体和高 QPS 是 CPU 负担
```

**策略 B：后端自行计算（推荐用于 API）**
```python
# Spring Boot 示例 — 由应用提供准确的缓存语义

# 用 "版本号_时间戳" 做 ETag，无需计算 body 的 hash
String version = product.getVersion();          // DB 自增字段
long modifiedTime = product.getUpdatedAt().getTime();

String etagValue = "\"" + version + "_" + modifiedTime + "\"";
// 结果：ETag: "42_1704067200"  ← 高效计算（读两个字段即可），后端可以精确判断

# 如果客户端发送的 If-None-Match 匹配 → 返回 304
if (etagUtils.isNotModified(request, etagValue)) {
    return ResponseEntity.status(304).build();   // ← 关键：只返回状态码，无 body
}

# 返回完整响应 + ETag 头 → 浏览器缓存这个值，下次发 If-None-Match
return ResponseEntity.ok()
    .header(HttpHeaders.ETag, etagValue)
    .body(product);  // body → CDN 取走，做自己的 ETag（如果开了）
```

**策略 C：固定无意义值（推荐用于 Nginx 作为 CDN 上游时）**
```nginx
location / {
    etag off;              # 首选：直接关闭（节省 Nginx CPU）

# 或旧版本 Nginx 无法 etag off; 时的降级方案：
    add_header ETag "\"fixed-nocheck\""; # 永远不变 → 浏览器无法通过 304 优化，但节省 CPU
}
```

### 4.3 ETag "在哪里缓存"？——五个位置

| 谁持有缓存？ | ETag 存在哪里？ | 生命周期 | 由谁验证/使用？ |
|------------|-------------|---------|----------------|
| **浏览器磁盘/内存缓存** | 本地存储，与 URI + HTTP Method 绑定为 key | max-age 过期前有效；过期后通过条件请求刷新 ETag | 浏览器在下次请求时读 `If-None-Match` 发送给服务端 |
| **CDN 节点** | 本地 SSD，key = (URI, ETag) | CDN 缓存 TTL（通常秒~天级） | CDN 节点判断 `If-None-Match` 是否匹配 → 命中则直接返回 304 |
| **Nginx proxy_cache** | `proxy_cache_path` 目录下的文件（含 Header） | 手动 purge / age 过期 / `proxy_cache_valid` 控制 | Nginx 对缓存体做 MD5 生成 ETag；命中时返回 cached body |
| **后端应用内存缓存**（Spring Cache / Guava） | 手动设置的 `Cache<String, Product>` 对象 | 按应用配置（通常分钟~小时级） | 如果 ETag = `version_timestamp`，数据库行更新时清除对应缓存 key |
| **反向代理/负载均衡器**（如 HAProxy） | 通常不设缓存 | — | 一般不处理 ETag，透传给后端 |

---

## 5. Cache-Control + ETag：最佳实践决策表

### 5.1 按资源类型选择正确的 Cache-Control + ETag 组合

| 资源类型 | 推荐 Cache-Control | 推荐 ETag？| 说明 |
|---------|-------------------|----------|------|
| **HTML 入口页（index.html）** | `no-cache, must-revalidate` | 需要（CDN 层面） | 每次向 CDN 检查，通过 ETag 判断是否拿到新版。HTML 引用了带 hash 的 JS/CSS，它们可长期缓存，但 HTML 本身要每次验证 |
| **CSS / JS（文件名带 hash）** | `public, max-age=31536000, immutable` | 不需要（除非特殊需求） | 文件名包含内容 hash（如 `app.a8c3d2f1.css`）。文件不变，文件名永远不变 → 一年缓存安全。服务端禁止设定 immutable |
| **图片 / 字体（png/jpg/svg/woff2）** | `public, max-age=31536000` | 不需要 | 同上，文件名可控时可长期缓存。如果 purge CDN 时注意带上匹配的资源类型更新即可 |
| **API 响应（/api/users, /api/orders）** | `no-store` 或 `max-age=0, must-revalidate` | 可选（后端计算 hash/版本号的场景） | 实时数据。如果做条件缓存（版本 ETag），用 `max-age=30` 等短时效 + ETag 方式。不启用 etag on; 可节省 Nginx CPU |
| **WebSocket SSE（服务端推送事件流）** | `no-cache`，关闭 Chunked Response 缓存 | 不需要 | 实时推送不适用缓存控制。无需使用 HTTP 缓存语义，由应用层处理验证 |

### 5.2 多层架构下的生效规则（哪个值真正起作用？）

| 层级 | 设置的值 | 浏览器实际收到的？ |
|------|---------|-------------------|
| 后端返回 `Cache-Control: no-store` | Nginx 缓存命中 → **Nginx/CDN 的 override** | ❌ 如果 CDN 有 cache policy，后端的不生效（会被覆盖） |
| Nginx 通过 `add_header Cache-Control "no-store"` | CDN 缓存命中 → **CDN 的 Cache Policy** | ❌ 同理，如果启用了 CDN 缓存策略 |
| CDN 控制台设置 Cache Policy：`/api/* → no-cache, max-age=0` | **浏览器直接收到 CDN 发出的 header** | ✅ 生效（CDN 是外层的缓存节点） |
| CDN 控制台设置 Cache Policy：`/static/* → max-age=31536000, immutable` | **浏览器直接收到** | ✅ 生效（文件不变，内容 hash 文件名可永久缓存） |

### 5.3 ETag 双向交互流程

```
完整请求流程（带图）：
┌──────────────────── 第 N 次请求（条件 GET，浏览器发起）─────────────────┐
│                                                                        │
│ 浏览器（本地缓存已存在 ETag: "abc123"）                                │
│        ├──────── 发送 GET /api/users ────────────→ CD/后端              │
│        │                                  (If-None-Match: "abc123")     │
│ 场景一：ETag 匹配                         │                              │
│        ├────────────── 返回 304 Not Modified ←─────────────────────      │
│        │                                  （无 body，节省带宽）          │
│ 浏览器更新本地缓存的时间戳               │                              │
│ 继续使用旧的 body 刷新页面                │                              │
│                                                                        │
│ 场景二：ETag 不匹配（内容变更）           │                              │
│        ├────────────── 返回 200 OK + 新 ETag ←──────────────────        │
│        │                                  （完整 body）                  │
│ 浏览器用新的 body 替换本地缓存            │                              │
│ 更新 ETag 为新值"xyz789" → 下次再发      │                              │
│                                                                        │
└───────────────────────────────────────────────────────────────────────┘

完美协作的完整流程：
1) 用户第一次请求 /api/users → 服务端返回 Cache-Control: max-age=30
2) CDN 缓存命中（TTL 内）→ 浏览器直接用本地缓存体，不向服务端请求
3) CDN TTL 过期 → 回源到 Nginx/后端
4) 服务端判断 ETag 匹配 → 返回 304 Not Modified（无 body，节省网络）
5) CDN 用本地缓存体 + 新时间戳更新缓存（未过期前）→ 节省了一次完整 body

结果：节省了两次数据传输（一次性 + 条件请求），减轻了后端和 CDN 源站负担。
```

---

## 6. 生产案例：ETag + gzip 双重开销导致系统降质

### 6.1 场景描述

一个云文档编辑服务，接口返回 JSON 数据（单条响应约 2-4KB）。平台同时开启了 `gzip on;` 和 Nginx 默认的 `etag on;`。

**现象：** 当 QPS > 3000 时：

| 指标 | 修复前 | 修复后 |
|------|--------|--------|
| 平均响应时间 (P99) | 450ms | 85ms |
| CPU 使用率 (服务器) | 92% | 35% |
| 每秒处理请求数 (QPS) | 2800（降质中）| 15000（稳定运行） |
| gzip 压缩 CPU 耗时/请求 | ~0.2ms（累加到总时间）| N/A（关闭 gzip 后不再计算） |

### 6.2 根因：`gzip` + `etag on;` 双重计算开销

当 `gzip on;` 和 Nginx 默认的 `etag on;` 同时开启时：

1. 后端返回原始 JSON body
2. Nginx 计算 ETag（MD5 of body）→ 这消耗 CPU
3. Nginx 再做 gzip 压缩写入 `client_body_buffer` → 这又消耗 CPU
4. 客户端收到的是**经过 gzip 压缩的 ETag**（即原始内容的 MD5，不是压缩后内容的 MD5）

这种"先算 hash 再压缩"的顺序使得每次请求都消耗一次完整的 MD5 计算 + gzip CPU 开销。对高频接口来说，这个双重处理变成了可以被单独定位的瓶颈。

### 6.3 解决方案：关闭 ETag + 按需启用 gzip

```nginx
location /api/documents/ {
    etag off;                      # 关闭 ETag，由 Cache-Control 控制缓存
    gzip on;                       # 只对大响应启用压缩
    gzip_min_length 1024;          # 小于 1KB 的不压缩（省 CPU）
    gzip_types application/json;   # 只对 JSON 类型启用

    proxy_pass http://backend;
}
```

| 做法 | 适用场景 | 说明 |
|------|----------|------|
| `etag off;` 全部关闭 | API 接口，后端已有版本语义（如 RESTful `/v1/resources/123`） | 服务端自己的路由语义代替 ETag |
| 固定 ETag（`etag "xxx";`） | Nginx 缓存静态文件到 CDN，客户端-to-CDN 的条件请求仍可工作 | 旧版本 Nginx 无法 `etag off;` 时的降级方案 |
| ETag + gzip 分开控制 | 内部系统有统一内容版本号的特殊场景（少见） | 不适用大多数 Web API |

---

## 7. 生产案例：浏览器 Cache-Control 滥用导致全站冷启动

### 7.1 场景描述

一个网页版的在线交易监控平台，前端工程师做了激进的缓存策略：
`Cache-Control: max-age=86400  # 缓存一天！`

后台系统做了紧急安全修复（修改了 JS 入口文件 `app.js`）。前端部署了新代码到 `/dist/app.a3c1f9b2.js`。

**问题：** 因为主页面 `index.html` 设置了长缓存时间，CDN 仍然持有旧版 HTML。用户刷新页面 → CDN 返回旧 `index.html` → JS 引用了旧文件 → **页面完全不可用**。

### 7.2 根因：CDN purge + Cache-Control 过期不匹配

```
缓存策略设计不当产生的连锁反应：
1) index.html → Cache-Control: max-age=86400 (24小时)
   CDN 缓存 index.html 一天（即使前端部署了新版本）

2) 运营人员更新了 /dist/app.js 的 CDN 缓存 → purge
   但 index.html 的缓存仍然有效

3) 用户请求 /index.html → CDN 返回旧的 index.html
   （HTML 仍然引用旧版 JS，页面加载后报错）

4) 后台紧急修复 → 更新 index.html → CDN purge
   用户看到新内容的时间 = 最长 purge 传播延迟

5) 极端情况：如果 purging CDN 时，Cache-Control: max-age=86400
   的 index.html 同时被 purge 掉 → 几百个后端节点同时恢复流量
```

---

## 8. 四个黄金法则（建议记住）

**规则一：ETag 不是免费的。**
当它要求后端计算 hash、访问数据库、做文件 IO 时，高频接口应该关闭它。

**规则二：Cache-Control > ETag。**
正确的缓存生命周期管理比条件请求节省更多带宽和后端资源。优先配置 Cache-Control，ETag 是"锦上添花"的工具。

**规则三：gzip 对小响应体是负优化。**
`gzip_min_length` 应设为至少 512 或 1KB。防止小接口因 gzip 而消耗后端 CPU，反而拖慢响应速度。

**规则四：CDN 与后端的职责必须分离。**
后端 → CDN 不需要 ETag。ETag 只在浏览器到 CDN 层有优化价值。

---

## 9. 常见问题排查清单（快速诊断用）

| 症状 | 可能的 HTTP 头问题 | 验证方法 |
|------|-------------------|---------|
| 后端 CPU 高但网络流量不大 | ETag 计算压缩开销；条件请求风暴 | `ab -c 100` 压测，对比开启/关闭 ETag |
| 前端更新后大量用户拿到旧版 | Cache-Control 过期太长 + CDN purge 异步延迟 | `curl -I <url>` 检查 response headers |
| P99 延时高、吞吐低 | gzip + etag 双重处理；小响应体不值得压缩 | Nginx `access.log` + `upstream_response_time` 统计 |
| 304 状态码占比异常高（>60%）但后端仍跑满 | 大量 HEAD 条件请求打到 Origin | Nginx `status 304` 计数或日志分析 |
| CDN 回源率突然激增 | Purge + ETag 同步不一致导致缓存穿透 | CDN 监控面板回源统计 |

---

## 附录 A：各层 Cache-Control 设置对照表（完整）

### A.1 Nginx 配置模板

```nginx
# ========== 静态资源 ──────────────────────
location ~* \.(js|css)$ {
    expires 7d;                          # Cache-Control: max-age=604800
    add_header Cache-Control "public";   # 可选：覆盖明确设置

    access_log off;                      # Nginx 不记录静态文件日志
}

# ========== 图片 / 字体 ────────────────────
location ~* \.(png|jpg|svg|woff2)$ {
    expires 30d;                         # Cache-Control: max-age=2592000

    access_log off;
}

# ========== HTML 入口页 ────────────────────
location / {
    add_header Cache-Control "no-cache, must-revalidate";

    try_files $uri $uri/ /index.html;
}

# ========== API 接口 ───────────────────────
location /api/ {
    add_header Cache-Control "no-store";  # 不适用任何缓存

    proxy_pass http://backend;
}

# ========== 禁用 Nginx 默认 ETag ────────────
etag off;                                # 全局关闭（推荐）

# ========== 启用 gzip，但对小响应体跳过 ─────
gzip on;
gzip_min_length 1024;                    # 小于 1KB 的不压缩（省 CPU）
gzip_types application/json text/html application/javascript text/css;
```

### A.2 CDN 控制台（云服务商通用）

| 服务 | 设置路径 | Cache-Control 映射 |
|------|---------|-------------------|
| CloudFront (AWS) | Cache Policy → 创建新策略 | `max-age`, `no-cache`, `no-store` → 直接匹配 |
| Aliyun CDN (阿里云) | 缓存设置 → 域名配置 | 有效期（秒）→ 自动映射为 Cache-Control: max-age=N |
| Akamai | Traffic Manager → Caching Rules | `max_age`, `no_cache` 直接配置为 Header Value |

### A.3 动态 max-age（按资源特性分段）

```python
# 根据资源的"新鲜度"动态设置 max-age
def get_cache_control(resource):
    """根据资源类型和热度返回最优 Cache-Control 值"""

    if resource.type in ("js", "css"):
        # JS / CSS（文件名带 hash）→ 长期缓存安全
        return "public, max-age=31536000, immutable"

    elif resource.type == "html":
        # HTML 入口页 → 每次检查（CDN/Etag 条件判断）
        return "no-cache, must-revalidate"

    elif resource.is_hot:
        # 热门 API → 短时效（高频更新）
        return f"max-age={60}, must-revalidate"

    elif resource.is_cold:
        # 冷门 API → 长时效（内容稳定）
        return f"max-age={86400}, must-revalidate"

    else:
        # 普通 API → 中等时效
        return f"max-age={300}, must-revalidate"

# 前端可用的 index.html 引用
<script src="/dist/app.a3c1f9b2.js"></script>
<!-- index.html 本身: Cache-Control: no-cache, must-revalidate → 每次检查 ETag -->
<!-- /dist/app.js (文件名带 hash): Cache-Control: max-age=31536000, immutable → 永久缓存 -->
```
