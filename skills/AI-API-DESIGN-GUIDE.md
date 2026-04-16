# AI 接口设计规范

> **适用工具**：Cursor / ChatGPT / Claude / GitHub Copilot / Windsurf / Cline
>
> **定位**：本文件是 AI 助手执行 API 接口设计任务的系统级指令文件（Skill File）。AI 在设计任何接口之前必须完整阅读本文件，并严格按照其中的规范和约束执行。
>
> 本手册参考 Google API Design Guide、Microsoft REST API Guidelines、阿里巴巴 Java 开发手册（接口篇）、Stripe/GitHub/微信开放平台等业界公认的优秀 API 设计实践编写，适用于 RESTful API 和 GraphQL API 的设计。
>
> **目标读者**：通过 AI 辅助进行接口设计的开发者。AI 阅读后应具备资深后端架构师级别的 API 设计能力。

---

## 目录

**第一部分：设计流程（先想清楚再写代码）**
1. [接口设计流程](#1-接口设计流程)
2. [接口文档规范](#2-接口文档规范)

**第二部分：RESTful API 设计规范（核心章节）**
3. [URL 设计规范](#3-url-设计规范)
4. [HTTP 方法语义](#4-http-方法语义)
5. [请求设计规范](#5-请求设计规范)
6. [响应设计规范](#6-响应设计规范)
7. [状态码使用规范](#7-状态码使用规范)
8. [错误处理规范](#8-错误处理规范)

**第三部分：通用设计模式（每个接口都会用到）**
9. [分页设计](#9-分页设计)
10. [筛选与排序](#10-筛选与排序)
11. [批量操作](#11-批量操作)
12. [文件上传](#12-文件上传)

**第四部分：安全与认证（不做就裸奔）**
13. [认证方案设计](#13-认证方案设计)
14. [接口安全规范](#14-接口安全规范)
15. [敏感数据处理](#15-敏感数据处理)

**第五部分：版本管理与演进（让接口能活十年）**
16. [版本管理策略](#16-版本管理策略)
17. [接口废弃流程](#17-接口废弃流程)
18. [向后兼容原则](#18-向后兼容原则)

**第六部分：性能与可靠性（高并发不翻车）**
19. [性能设计规范](#19-性能设计规范)
20. [幂等性设计](#20-幂等性设计)
21. [限流与降级](#21-限流与降级)

**第七部分：GraphQL 设计规范（可选）**
22. [GraphQL 适用场景](#22-graphql-适用场景)
23. [Schema 设计规范](#23-schema-设计规范)

**附录**
- [A. 常用接口设计速查表](#附录-a-常用接口设计速查表)
- [B. 典型业务接口设计示例](#附录-b-典型业务接口设计示例)
- [C. 接口设计评审检查清单](#附录-c-接口设计评审检查清单)
- [D. AI 接口设计 Prompt 模板](#附录-d-ai-接口设计-prompt-模板)

---

# 第一部分：设计流程

## 1. 接口设计流程

### 1.1 标准流程

AI 接收到接口设计需求后，必须按以下流程执行：

```
业务需求 → 识别资源和操作 → 定义 URL 和方法 → 设计请求/响应结构
         → 定义错误码 → 安全方案 → 文档输出 → 评审检查
```

### 1.2 设计前必须确认的事项

| 确认项 | 为什么重要 |
|--------|-----------|
| **谁调用？** | 前端/小程序/第三方/内部服务，影响认证方案和安全策略 |
| **调用频率？** | 每秒 10 次 vs 10000 次，影响限流和缓存策略 |
| **数据量级？** | 返回 10 条 vs 10 万条，影响分页和响应设计 |
| **实时性要求？** | 毫秒级 vs 秒级，影响缓存和异步方案 |
| **版本兼容？** | 是否有旧版本客户端需要兼容 |

## 2. 接口文档规范

### 2.1 单接口文档模板

AI 输出接口设计时，**必须**使用以下模板：

```markdown
### [接口名称]

**接口说明**：[一句话描述]
**请求方式**：`POST /api/v1/orders`
**认证方式**：Bearer Token
**频率限制**：100 次/分钟

#### 请求参数

**Headers**
| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| Authorization | string | 是 | Bearer {token} |

**Body (JSON)**
| 参数 | 类型 | 必填 | 说明 | 示例 |
|------|------|------|------|------|
| product_id | integer | 是 | 商品ID | 1001 |
| quantity | integer | 是 | 数量，1-999 | 2 |
| address_id | integer | 是 | 收货地址ID | 55 |
| coupon_id | integer | 否 | 优惠券ID | 88 |
| remark | string | 否 | 买家备注，最长200字 | "周末送达" |

#### 响应示例

**成功 (200)**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "order_id": 10001,
    "order_no": "20260416153000001",
    "pay_amount": "99.00",
    "pay_expire_at": "2026-04-16T15:50:00+08:00"
  }
}
```

**失败示例**
```json
{
  "code": 40001,
  "message": "库存不足",
  "data": null
}
```

#### 错误码
| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 40001 | 库存不足 | 提示用户修改数量 |
| 40002 | 优惠券不可用 | 移除优惠券后重试 |
| 40003 | 收货地址不存在 | 引导用户新增地址 |

#### 备注
- 同一用户 5 秒内不能重复提交
- 创建订单后 30 分钟未支付自动取消
```

---

# 第二部分：RESTful API 设计规范

## 3. URL 设计规范

### 3.1 核心规则

| 规则 | 正确示例 | 错误示例 |
|------|---------|---------|
| **URL 代表资源，用名词不用动词** | `/api/v1/users` | `/api/v1/getUsers` |
| **用复数名词** | `/api/v1/orders` | `/api/v1/order` |
| **全小写，用连字符分隔** | `/api/v1/order-items` | `/api/v1/orderItems` |
| **层级表示归属关系** | `/api/v1/users/123/orders` | `/api/v1/getUserOrders` |
| **不超过 3 层嵌套** | `/api/v1/users/123/orders` | `/api/v1/users/123/orders/456/items/789/details` |
| **统一前缀** | `/api/v1/...` | 有的 `/api` 有的 `/service` |

### 3.2 URL 设计模板

```
# 基础 CRUD
GET    /api/v1/users          # 获取用户列表
POST   /api/v1/users          # 创建用户
GET    /api/v1/users/123      # 获取用户详情
PUT    /api/v1/users/123      # 更新用户（全量）
PATCH  /api/v1/users/123      # 更新用户（部分字段）
DELETE /api/v1/users/123      # 删除用户

# 子资源
GET    /api/v1/users/123/orders         # 用户的订单列表
POST   /api/v1/users/123/addresses      # 给用户添加地址

# 操作类（非标准 CRUD 才用动词）
POST   /api/v1/orders/123/cancel        # 取消订单
POST   /api/v1/orders/123/pay           # 支付订单
POST   /api/v1/users/123/verify-phone   # 验证手机号
```

### 3.3 常见资源命名

| 资源 | URL | 说明 |
|------|-----|------|
| 用户 | `/users` | |
| 商品 | `/products` | |
| 订单 | `/orders` | |
| 分类 | `/categories` | |
| 评论 | `/comments` | |
| 优惠券 | `/coupons` | |
| 收货地址 | `/addresses` | |
| 支付 | `/payments` | |
| 通知 | `/notifications` | |
| 配置 | `/configs` 或 `/settings` | |

## 4. HTTP 方法语义

### 4.1 方法对照表

| 方法 | 语义 | 幂等性 | 安全性 | 请求体 | 场景 |
|------|------|--------|--------|--------|------|
| **GET** | 获取资源 | ✅ 是 | ✅ 是 | 无 | 查列表、查详情 |
| **POST** | 创建资源 | ❌ 否 | ❌ 否 | 有 | 新增数据、触发操作 |
| **PUT** | 全量替换 | ✅ 是 | ❌ 否 | 有 | 更新全部字段 |
| **PATCH** | 部分更新 | ✅ 是 | ❌ 否 | 有 | 只更新传入的字段 |
| **DELETE** | 删除资源 | ✅ 是 | ❌ 否 | 无/有 | 删除数据 |

### 4.2 易混淆场景的方法选择

| 场景 | 正确方法 | 原因 |
|------|---------|------|
| 用户登录 | `POST /api/v1/auth/login` | 创建会话，非幂等 |
| 用户登出 | `POST /api/v1/auth/logout` | 销毁会话 |
| 修改密码 | `PUT /api/v1/users/me/password` | 全量替换密码 |
| 修改昵称 | `PATCH /api/v1/users/me` | 部分更新 |
| 取消订单 | `POST /api/v1/orders/123/cancel` | 触发操作 |
| 搜索 | `GET /api/v1/products?keyword=手机` | 查询类 |
| 复杂搜索（参数极多） | `POST /api/v1/products/search` | GET URL 长度有限制 |
| 导出报表 | `POST /api/v1/reports/export` | 触发耗时操作 |
| 批量删除 | `POST /api/v1/products/batch-delete` | 需要请求体传 ID 列表 |

## 5. 请求设计规范

### 5.1 请求参数位置

| 位置 | 用途 | 示例 |
|------|------|------|
| **Path** | 资源标识 | `/users/{id}` |
| **Query** | 筛选、分页、排序 | `?page=1&size=20&status=1` |
| **Body** | 创建/更新的数据 | `{"name": "张三"}` |
| **Header** | 认证、版本、语言 | `Authorization: Bearer xxx` |

### 5.2 请求体规范

```json
// 正确：扁平结构，字段命名 snake_case
{
  "product_id": 1001,
  "quantity": 2,
  "coupon_id": 88,
  "address_id": 55,
  "remark": "周末送达"
}

// 错误：驼峰命名（前后端不一致）
{
  "productId": 1001,
  "addressId": 55
}

// 错误：嵌套过深
{
  "order": {
    "product": {
      "info": {
        "id": 1001
      }
    }
  }
}
```

### 5.3 字段命名规范

| 规则 | 正确 | 错误 |
|------|------|------|
| **统一 snake_case** | `user_name` | `userName` / `UserName` |
| **布尔值用 is_/has_ 前缀** | `is_active` | `active` |
| **时间字段用 _at 后缀** | `created_at` | `create_time` / `ctime` |
| **ID 字段用 _id 后缀** | `user_id` | `userId` / `uid` |
| **金额字段明确单位** | `amount`（元） 或 `amount_cents`（分） | `money` |
| **列表字段用复数** | `order_ids` | `order_id_list` |

> **说明**：snake_case 是 RESTful API 的业界主流（GitHub、Stripe、Twitter 等均使用）。如果团队已有驼峰约定也可保持一致，但必须全项目统一。

## 6. 响应设计规范

### 6.1 统一响应结构

**所有接口**必须返回统一的 JSON 结构：

```json
{
  "code": 0,
  "message": "success",
  "data": { ... }
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| `code` | integer | 业务状态码，0 表示成功，非 0 表示业务错误 |
| `message` | string | 人类可读的提示信息 |
| `data` | object/array/null | 业务数据，失败时为 null |

### 6.2 列表响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "list": [
      { "id": 1, "name": "商品A" },
      { "id": 2, "name": "商品B" }
    ],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 150
    }
  }
}
```

### 6.3 详情响应格式

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "id": 123,
    "order_no": "20260416153000001",
    "status": 1,
    "status_text": "已支付",
    "total_amount": "199.00",
    "created_at": "2026-04-16T15:30:00+08:00"
  }
}
```

### 6.4 响应字段规范

| 规则 | 说明 |
|------|------|
| **金额用字符串** | `"99.00"` 而非 `99.0`，避免浮点精度问题 |
| **时间用 ISO 8601** | `"2026-04-16T15:30:00+08:00"` |
| **枚举值同时返回 code 和 text** | `"status": 1, "status_text": "已支付"` |
| **null 字段可省略或返回 null** | 不要返回 `"name": ""` 代替 null |
| **ID 用整型或字符串**（大数） | JS 中超过 2^53 的整数会丢失精度，用字符串 |
| **空列表返回 []** | 不要返回 null |

## 7. 状态码使用规范

### 7.1 常用状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|---------|
| **200** | OK | 查询成功、更新成功 |
| **201** | Created | 资源创建成功 |
| **204** | No Content | 删除成功（无返回体） |
| **400** | Bad Request | 参数校验失败 |
| **401** | Unauthorized | 未登录或 Token 过期 |
| **403** | Forbidden | 已登录但无权限 |
| **404** | Not Found | 资源不存在 |
| **409** | Conflict | 资源冲突（如重复创建） |
| **422** | Unprocessable Entity | 参数格式正确但业务不允许 |
| **429** | Too Many Requests | 超过频率限制 |
| **500** | Internal Server Error | 服务器内部错误 |

### 7.2 状态码选择决策树

```
请求成功了吗？
├── 是 → 创建了新资源？
│        ├── 是 → 201 Created
│        └── 否 → 有返回内容？
│                  ├── 是 → 200 OK
│                  └── 否 → 204 No Content
└── 否 → 谁的错？
         ├── 客户端 → 认证问题？
         │            ├── 没登录 → 401
         │            ├── 没权限 → 403
         │            └── 不是 → 参数问题？
         │                       ├── 格式错误 → 400
         │                       ├── 业务不允许 → 422
         │                       ├── 资源不存在 → 404
         │                       ├── 重复操作 → 409
         │                       └── 频率过高 → 429
         └── 服务端 → 500
```

## 8. 错误处理规范

### 8.1 错误响应格式

```json
{
  "code": 40001,
  "message": "库存不足，当前剩余 3 件",
  "data": null,
  "details": [
    {
      "field": "quantity",
      "message": "购买数量超过库存"
    }
  ]
}
```

### 8.2 错误码设计规范

**错误码分段规则：**

| 段位 | 范围 | 说明 |
|------|------|------|
| `0` | 0 | 成功 |
| `400xx` | 40001-40099 | 通用业务错误 |
| `401xx` | 40101-40199 | 认证错误 |
| `403xx` | 40301-40399 | 权限错误 |
| `404xx` | 40401-40499 | 资源不存在 |
| `500xx` | 50001-50099 | 服务端错误 |

**常用错误码示例：**

| 错误码 | 说明 | 使用场景 |
|--------|------|---------|
| 0 | 成功 | |
| 40001 | 参数校验失败 | 必填字段为空、格式不对 |
| 40002 | 业务规则不允许 | 库存不足、超出限制 |
| 40003 | 操作重复 | 重复下单、重复提交 |
| 40101 | 未登录 | Token 缺失 |
| 40102 | Token 已过期 | Token 失效 |
| 40103 | Token 无效 | Token 被篡改 |
| 40301 | 无权限 | 角色权限不足 |
| 40401 | 资源不存在 | 数据不存在或已删除 |
| 50001 | 服务器内部错误 | 未知异常 |
| 50002 | 第三方服务异常 | 支付/短信等外部服务故障 |

### 8.3 错误信息编写原则

| 原则 | 正确示例 | 错误示例 |
|------|---------|---------|
| **告诉用户发生了什么** | "库存不足，当前剩余 3 件" | "error" |
| **告诉用户怎么解决** | "手机号格式不正确，请输入11位手机号" | "参数错误" |
| **不暴露技术细节** | "服务暂时不可用，请稍后重试" | "NullPointerException at..." |
| **用中文** | "该订单已取消，无法支付" | "order status invalid" |

---

# 第三部分：通用设计模式

## 9. 分页设计

### 9.1 页码分页（适用大部分场景）

```
GET /api/v1/products?page=1&size=20

响应：
{
  "data": {
    "list": [...],
    "pagination": {
      "page": 1,
      "size": 20,
      "total": 150
    }
  }
}
```

### 9.2 游标分页（适用大数据量/无限滚动）

```
GET /api/v1/messages?cursor=eyJpZCI6MTAwfQ&size=20

响应：
{
  "data": {
    "list": [...],
    "next_cursor": "eyJpZCI6ODB9",
    "has_more": true
  }
}
```

### 9.3 分页方式选择

| 场景 | 推荐方式 | 原因 |
|------|---------|------|
| 后台列表（可跳页） | 页码分页 | 需要显示总数和页码 |
| 移动端无限滚动 | 游标分页 | 性能好，不需要 COUNT |
| 数据量 < 10 万 | 页码分页 | 简单够用 |
| 数据量 > 100 万 | 游标分页 | 页码分页 OFFSET 会慢 |
| 实时数据流（消息列表） | 游标分页 | 数据不断插入，页码会错位 |

## 10. 筛选与排序

### 10.1 筛选参数设计

```
# 等值筛选
GET /api/v1/products?status=1&category_id=5

# 范围筛选
GET /api/v1/orders?created_at_start=2026-04-01&created_at_end=2026-04-30

# 模糊搜索
GET /api/v1/products?keyword=手机

# 多值筛选
GET /api/v1/orders?status=1,2,3
```

### 10.2 排序参数设计

```
# 单字段排序
GET /api/v1/products?sort=price&order=asc

# 多字段排序
GET /api/v1/products?sort=sales_count:desc,price:asc

# 预定义排序
GET /api/v1/products?sort_by=popular    # 热门
GET /api/v1/products?sort_by=newest     # 最新
GET /api/v1/products?sort_by=price_asc  # 价格升序
```

## 11. 批量操作

### 11.1 批量查询

```
POST /api/v1/products/batch-get
{
  "ids": [1, 2, 3, 4, 5]
}
```

### 11.2 批量创建

```
POST /api/v1/products/batch-create
{
  "items": [
    { "name": "商品A", "price": "99.00" },
    { "name": "商品B", "price": "199.00" }
  ]
}

响应（部分成功场景）：
{
  "code": 0,
  "data": {
    "success_count": 1,
    "fail_count": 1,
    "results": [
      { "index": 0, "success": true, "id": 1001 },
      { "index": 1, "success": false, "error": "名称已存在" }
    ]
  }
}
```

### 11.3 批量删除

```
POST /api/v1/products/batch-delete
{
  "ids": [1, 2, 3]
}
```

### 11.4 批量操作规范

| 规则 | 说明 |
|------|------|
| **单次最大数量** | 限制在 100-500 条，防止超大请求 |
| **部分成功处理** | 返回每条的结果，不要全部成功或全部失败 |
| **超时保护** | 超过一定时间改为异步任务 |

## 12. 文件上传

### 12.1 上传方式

| 方式 | 适用场景 | 流程 |
|------|---------|------|
| **直传 OSS** | 大文件/图片/视频 | 后端签名 → 前端直传 OSS → 回调 |
| **后端中转** | 小文件/需要后端处理 | 前端上传后端 → 后端转存 OSS |

### 12.2 直传 OSS 接口设计

```
# 第一步：获取上传凭证
POST /api/v1/upload/token
{
  "file_type": "image",       // image / video / document
  "content_type": "image/jpeg"
}

响应：
{
  "data": {
    "upload_url": "https://oss.example.com/upload",
    "file_key": "images/2026/04/abc123.jpg",
    "token": "upload-token-xxx",
    "expire_at": "2026-04-16T16:00:00+08:00"
  }
}

# 第二步：前端直传 OSS（前端处理，非后端接口）

# 第三步：确认上传完成
POST /api/v1/upload/confirm
{
  "file_key": "images/2026/04/abc123.jpg"
}
```

### 12.3 上传限制规范

| 文件类型 | 最大大小 | 允许格式 |
|---------|---------|---------|
| 头像 | 5MB | jpg, png, webp |
| 商品图 | 10MB | jpg, png, webp |
| 视频 | 500MB | mp4, mov |
| 文档 | 20MB | pdf, doc, docx |

---

# 第四部分：安全与认证

## 13. 认证方案设计

### 13.1 认证方案对比

| 方案 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **JWT** | App / 小程序 / SPA | 无状态，扩展性好 | 无法主动踢下线 |
| **JWT + Redis** | 需要踢下线功能 | 兼顾无状态和可控性 | 需要维护 Redis |
| **Session** | 传统 Web | 简单 | 不适合分布式 |
| **OAuth 2.0** | 开放平台 / 第三方登录 | 标准化 | 复杂 |

### 13.2 JWT 方案设计（推荐）

```
# 登录
POST /api/v1/auth/login
{
  "phone": "13800138000",
  "code": "123456"    // 短信验证码
}

响应：
{
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "dGhpcyBpcyBhIHJlZnJl...",
    "expires_in": 7200,       // access_token 有效期（秒）
    "token_type": "Bearer"
  }
}

# 刷新 Token
POST /api/v1/auth/refresh
{
  "refresh_token": "dGhpcyBpcyBhIHJlZnJl..."
}

# 使用 Token
GET /api/v1/users/me
Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### 13.3 双 Token 机制

| Token | 有效期 | 用途 |
|-------|--------|------|
| `access_token` | 2 小时 | 接口认证 |
| `refresh_token` | 30 天 | 刷新 access_token |

**刷新流程**：access_token 过期 → 用 refresh_token 获取新的 access_token → refresh_token 也过期则重新登录。

## 14. 接口安全规范

### 14.1 安全检查清单

| 检查项 | 说明 |
|--------|------|
| **HTTPS** | 所有接口必须走 HTTPS |
| **参数校验** | 所有输入必须校验类型、长度、范围 |
| **SQL 注入** | 使用参数化查询，禁止拼接 SQL |
| **XSS 防护** | 输出转义，不信任任何用户输入 |
| **CSRF 防护** | API 用 Token 认证天然防 CSRF |
| **越权检查** | 必须验证当前用户有权操作目标资源 |
| **频率限制** | 核心接口必须限流 |
| **日志脱敏** | 日志中不打印密码、Token、身份证等 |

### 14.2 越权防护（最常被遗漏的安全问题）

```
# 水平越权示例：
# 用户 A 修改用户 B 的订单 → 必须检查订单属于当前用户
GET /api/v1/orders/123
→ 后端必须校验：order.user_id == current_user.id

# 垂直越权示例：
# 普通用户调用管理员接口 → 必须检查角色权限
DELETE /api/v1/admin/users/456
→ 后端必须校验：current_user.role == 'admin'
```

## 15. 敏感数据处理

### 15.1 脱敏规则

| 数据类型 | 脱敏规则 | 示例 |
|---------|---------|------|
| 手机号 | 中间 4 位 * | 138\*\*\*\*8000 |
| 身份证 | 中间 10 位 * | 110\*\*\*\*\*\*\*\*\*\*1234 |
| 银行卡 | 只显示后 4 位 | \*\*\*\* \*\*\*\* \*\*\*\* 1234 |
| 邮箱 | @前保留首尾 | z\*\*\*n@gmail.com |
| 姓名 | 姓保留 | 张\*\* |

### 15.2 敏感数据传输规则

| 规则 | 说明 |
|------|------|
| **密码** | 前端加密传输，后端 bcrypt 存储 |
| **支付密码** | 独立加密通道 |
| **Token** | 放 Header 不放 URL，URL 会被日志记录 |
| **文件下载** | 使用临时签名 URL，不暴露真实路径 |

---

# 第五部分：版本管理与演进

## 16. 版本管理策略

### 16.1 版本号方案

**推荐：URL 路径版本号**

```
/api/v1/users
/api/v2/users
```

| 方案 | 示例 | 优点 | 缺点 |
|------|------|------|------|
| **URL 路径** | `/api/v1/users` | 直观清晰 | URL 变长 |
| Header | `Api-Version: 1` | URL 简洁 | 不直观，调试难 |
| Query | `?version=1` | 简单 | 不规范 |

### 16.2 什么时候升级大版本

| 场景 | 是否升版本 |
|------|-----------|
| 新增字段 | ❌ 不需要（向后兼容） |
| 新增接口 | ❌ 不需要 |
| 修改字段名 | ✅ 需要（破坏性变更） |
| 删除字段 | ✅ 需要 |
| 修改字段类型 | ✅ 需要 |
| 修改业务逻辑 | 视情况，不影响响应格式不需要 |

## 17. 接口废弃流程

```
阶段1：标记废弃
  → 响应 Header 添加 Deprecation: true
  → 文档标注废弃时间和替代接口

阶段2：日志监控（持续 1-3 个月）
  → 监控旧接口调用量
  → 通知仍在使用的调用方迁移

阶段3：下线
  → 返回 410 Gone
  → 响应体说明替代接口
```

## 18. 向后兼容原则

| 规则 | 说明 |
|------|------|
| **只增不删** | 可以新增字段，不能删除/改名字段 |
| **新增字段可选** | 新增的请求字段必须有默认值 |
| **枚举值只增** | 状态码可以增加新值，不能修改已有值的含义 |
| **保持响应结构** | `code` / `message` / `data` 结构不变 |

---

# 第六部分：性能与可靠性

## 19. 性能设计规范

### 19.1 响应时间标准

| 接口类型 | P99 目标 | 说明 |
|---------|---------|------|
| 简单查询 | < 100ms | 按 ID 查详情 |
| 列表查询 | < 300ms | 分页列表 |
| 创建/更新 | < 500ms | 写入操作 |
| 复杂查询 | < 1s | 多表关联、聚合统计 |
| 导出/报表 | 异步 | 超过 3 秒改异步任务 |

### 19.2 性能优化手段

| 手段 | 适用场景 | 说明 |
|------|---------|------|
| **字段裁剪** | 列表接口 | 列表只返回必要字段，详情返回全部 |
| **缓存** | 热点数据 | 商品详情、配置数据加 Redis 缓存 |
| **数据库优化** | 慢查询 | 加索引、优化 SQL |
| **异步化** | 耗时操作 | 发短信、生成报表改异步 |
| **CDN** | 静态资源 | 图片、文件走 CDN |
| **压缩** | 大响应体 | 开启 Gzip 压缩 |

## 20. 幂等性设计

### 20.1 为什么需要幂等

网络抖动/用户重复点击 → 请求可能发送多次 → 不做幂等就会重复创建订单/重复扣款。

### 20.2 幂等方案

| 方案 | 适用场景 | 实现方式 |
|------|---------|---------|
| **唯一请求 ID** | 创建订单、支付 | 客户端生成 request_id，服务端去重 |
| **数据库唯一索引** | 防重复创建 | 业务唯一键约束 |
| **Token 机制** | 表单提交 | 先获取 Token，提交时带上，用过即失效 |
| **状态机** | 状态流转 | 只有特定状态才能执行操作 |

### 20.3 唯一请求 ID 方案

```
# 创建订单（幂等）
POST /api/v1/orders
Headers:
  X-Request-Id: 550e8400-e29b-41d4-a716-446655440000
Body:
  { "product_id": 1001, "quantity": 1 }

# 服务端逻辑：
# 1. 检查 X-Request-Id 是否已处理过（查 Redis/DB）
# 2. 已处理 → 直接返回上次的结果
# 3. 未处理 → 执行创建 → 记录 Request-Id 和结果
```

## 21. 限流与降级

### 21.1 限流策略

| 维度 | 说明 | 示例 |
|------|------|------|
| **用户级** | 每用户每接口限流 | 每用户 100 次/分钟 |
| **IP 级** | 每 IP 限流 | 每 IP 1000 次/分钟 |
| **接口级** | 总体限流 | 创建订单接口总计 5000 QPS |

### 21.2 限流响应

```json
HTTP 429 Too Many Requests
Headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 0
  X-RateLimit-Reset: 1713262800

{
  "code": 42901,
  "message": "请求过于频繁，请 60 秒后重试",
  "data": null
}
```

---

# 第七部分：GraphQL 设计规范

## 22. GraphQL 适用场景

### 22.1 REST vs GraphQL 选择

| 场景 | 推荐 | 原因 |
|------|------|------|
| 标准 CRUD 后台 | REST | 简单直观，工具链成熟 |
| 多端需求差异大 | GraphQL | 客户端按需查询 |
| 数据关系复杂 | GraphQL | 一次查询获取关联数据 |
| 对外开放 API | REST | 门槛低，文档友好 |
| 内部微服务通信 | REST / gRPC | GraphQL 不适合服务间调用 |
| 实时数据 | GraphQL Subscription | 原生支持 |

### 22.2 何时使用 GraphQL

- 前端需要的数据结构和后端返回的差异很大
- 一个页面需要调多个 REST 接口才能拼出数据
- 不同端（App/Web/小程序）需要不同的字段组合

## 23. Schema 设计规范

### 23.1 命名规范

```graphql
# 类型名：PascalCase
type Product {
  id: ID!
  name: String!
  price: Float!
  createdAt: DateTime!         # 字段名：camelCase
}

# 查询：动词 + 名词
type Query {
  product(id: ID!): Product
  products(filter: ProductFilter, page: Int, size: Int): ProductConnection!
}

# 变更：动词 + 名词
type Mutation {
  createProduct(input: CreateProductInput!): Product!
  updateProduct(id: ID!, input: UpdateProductInput!): Product!
  deleteProduct(id: ID!): Boolean!
}

# 输入类型加 Input 后缀
input CreateProductInput {
  name: String!
  price: Float!
  categoryId: ID!
}
```

### 23.2 分页（Relay 规范）

```graphql
type ProductConnection {
  edges: [ProductEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type ProductEdge {
  node: Product!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

---

# 附录

## 附录 A. 常用接口设计速查表

### 用户模块

| 接口 | 方法 | URL | 说明 |
|------|------|-----|------|
| 手机号登录 | POST | `/api/v1/auth/login` | 手机号 + 验证码 |
| 刷新 Token | POST | `/api/v1/auth/refresh` | 用 refresh_token |
| 退出登录 | POST | `/api/v1/auth/logout` | |
| 获取当前用户 | GET | `/api/v1/users/me` | |
| 更新个人资料 | PATCH | `/api/v1/users/me` | |
| 获取用户详情 | GET | `/api/v1/users/{id}` | 公开信息 |

### 商品模块

| 接口 | 方法 | URL | 说明 |
|------|------|-----|------|
| 商品列表 | GET | `/api/v1/products` | 支持分页/筛选/排序 |
| 商品详情 | GET | `/api/v1/products/{id}` | |
| 搜索商品 | GET | `/api/v1/products?keyword=xxx` | |
| 分类列表 | GET | `/api/v1/categories` | |
| 创建商品 | POST | `/api/v1/admin/products` | 管理端 |
| 上下架 | PATCH | `/api/v1/admin/products/{id}` | `{"status": 1}` |

### 订单模块

| 接口 | 方法 | URL | 说明 |
|------|------|-----|------|
| 创建订单 | POST | `/api/v1/orders` | 需要幂等 |
| 订单列表 | GET | `/api/v1/orders` | ?status=1 |
| 订单详情 | GET | `/api/v1/orders/{id}` | |
| 取消订单 | POST | `/api/v1/orders/{id}/cancel` | |
| 确认收货 | POST | `/api/v1/orders/{id}/confirm` | |
| 发起支付 | POST | `/api/v1/payments` | |
| 支付回调 | POST | `/api/v1/payments/notify/wechat` | 微信回调 |

## 附录 B. 典型业务接口设计示例

### 手机号验证码登录（完整示例）

```
### 发送验证码

POST /api/v1/auth/sms-code
Content-Type: application/json

请求：
{
  "phone": "13800138000",
  "scene": "login"          // login/register/reset_password
}

成功响应 (200)：
{
  "code": 0,
  "message": "验证码已发送",
  "data": {
    "expire_in": 300        // 有效期 5 分钟
  }
}

错误响应：
- 40001: 手机号格式不正确
- 42901: 发送太频繁，请 60 秒后重试

限制：
- 同一手机号 60 秒内只能发 1 次
- 同一手机号每天最多 10 次
- 同一 IP 每小时最多 30 次


### 验证码登录

POST /api/v1/auth/login
Content-Type: application/json

请求：
{
  "phone": "13800138000",
  "code": "123456"
}

成功响应 (200)：
{
  "code": 0,
  "message": "登录成功",
  "data": {
    "access_token": "eyJhbGci...",
    "refresh_token": "dGhpcyBp...",
    "expires_in": 7200,
    "token_type": "Bearer",
    "is_new_user": false
  }
}

错误响应：
- 40001: 验证码错误
- 40002: 验证码已过期
- 40003: 验证码错误次数过多，请重新获取
- 40004: 该手机号已被封禁
```

## 附录 C. 接口设计评审检查清单

AI 在完成接口设计后，必须逐项检查：

### URL 设计
- [ ] URL 使用名词复数、全小写、连字符分隔？
- [ ] HTTP 方法语义正确？
- [ ] 嵌套不超过 3 层？
- [ ] 统一使用 `/api/v1/` 前缀？

### 请求设计
- [ ] 所有参数都有类型、必填、说明？
- [ ] 字段命名统一（snake_case 或 camelCase，不混用）？
- [ ] 有参数校验规则（长度、范围、格式）？

### 响应设计
- [ ] 使用统一响应结构（code/message/data）？
- [ ] 列表接口有分页？
- [ ] 金额用字符串？时间用 ISO 8601？
- [ ] 枚举值同时返回 code 和 text？

### 错误处理
- [ ] 定义了错误码？
- [ ] 错误信息对用户友好？
- [ ] 没有暴露技术细节？

### 安全
- [ ] 需要认证的接口都有标注？
- [ ] 有越权检查？
- [ ] 敏感数据有脱敏？
- [ ] 有频率限制？

### 性能
- [ ] 列表接口有分页限制？
- [ ] 有缓存策略？
- [ ] 写入接口有幂等设计？

## 附录 D. AI 接口设计 Prompt 模板

### Prompt 1：设计接口

```
请根据以下功能需求，按照接口设计规范设计 RESTful API：
1. 定义 URL 和 HTTP 方法
2. 设计请求参数和响应结构
3. 定义错误码
4. 说明安全和性能考虑

功能需求：
[粘贴需求描述]
```

### Prompt 2：评审接口文档

```
请评审以下接口设计，检查：
1. URL 和方法是否符合 RESTful 规范
2. 请求/响应结构是否合理
3. 错误处理是否完善
4. 安全性是否有遗漏
5. 性能是否有隐患

接口文档：
[粘贴接口文档]
```

### Prompt 3：RESTful 改造

```
以下接口不符合 RESTful 规范，请重新设计：

[粘贴现有接口列表，如：
POST /api/getProductList
POST /api/deleteUser
POST /api/updateOrder
]
```

---

> **本手册持续更新中。** 如果你觉得有用，请给 [AI 黑魔法](https://github.com/lvcn/ai-black-magic) 一个 ⭐ Star！
