# AI 代码审核实战手册

> **定位**：本文件是 AI 编码助手执行代码审核任务的系统级指令文件。AI 在审核任何模块/项目前必须完整阅读本文件，并严格按照其中的流程、检查项和输出格式执行。
>
> 本手册参考《阿里巴巴 Java 开发手册》《Google Code Review Guidelines》《OWASP Top 10》及国内一线互联网公司（字节、阿里、腾讯、美团）CR 实践编写，适用于任何技术栈的后端/前端/全栈工程。
>
> **目标读者**：通过 AI 辅助进行代码审核的技术负责人或开发者。AI 阅读后应具备资深架构师级别的审核能力。

---

## 目录

**第一部分：审核流程（标准化作业）**
1. [审核执行流程](#1-审核执行流程)
2. [审核范围确认](#2-审核范围确认)
3. [输出文档规范](#3-输出文档规范)

**第二部分：绝对问题检查（红线，必须修复）**
4. [安全漏洞](#4-安全漏洞)
5. [资损与数据一致性](#5-资损与数据一致性)
6. [逻辑缺陷与空实现](#6-逻辑缺陷与空实现)
7. [并发与线程安全](#7-并发与线程安全)

**第三部分：重大质量问题（强烈建议修复）**
8. [架构与设计](#8-架构与设计)
9. [事务与异常处理](#9-事务与异常处理)
10. [API 设计规范](#10-api-设计规范)
11. [代码规范与可维护性](#11-代码规范与可维护性)

**第四部分：建议改进（可选优化）**
12. [性能优化](#12-性能优化)
13. [可观测性](#13-可观测性)
14. [可测试性](#14-可测试性)

**第五部分：技术栈专项检查**
15. [Java / Spring Boot 专项](#15-java--spring-boot-专项)
16. [前端 Vue / React 专项](#16-前端-vue--react-专项)
17. [数据库与 SQL 专项](#17-数据库与-sql-专项)

**附录**
- [A. 问题严重程度定义](#附录-a-问题严重程度定义)
- [B. 审核报告模板](#附录-b-审核报告模板)
- [C. 常见反模式速查表](#附录-c-常见反模式速查表)
- [D. AI 审核 Prompt 模板](#附录-d-ai-审核-prompt-模板)

---

# 第一部分：审核流程

## 1. 审核执行流程

### 1.1 黄金法则：先全局后细节

**审核必须按以下顺序执行，不可跳步。先理解整体再深入细节，否则容易只见树木不见森林。**

```
阶段一：范围确认 ──→ 阶段二：结构扫描 ──→ 阶段三：核心逻辑深审
     │                    │                    │
     ▼                    ▼                    ▼
阶段四：交叉检查 ──→ 阶段五：问题分级 ──→ 阶段六：输出报告
```

### 1.2 各阶段详细动作

| 阶段 | 动作 | 产出 |
|------|------|------|
| **范围确认** | 确认审核目标（模块/PR/全量），了解业务背景 | 模块边界、文件清单 |
| **结构扫描** | 目录结构、文件数量、代码行数、依赖关系 | 模块拓扑图 |
| **核心逻辑深审** | 逐文件阅读核心 Service/Controller，重点关注写操作 | 问题列表草稿 |
| **交叉检查** | 检查模块间调用关系、外部依赖、配置一致性 | 补充问题 |
| **问题分级** | 按 P0/P1/P2 分级，标注文件名和行号 | 分级问题清单 |
| **输出报告** | 按模板输出 Markdown 格式审核报告 | 正式报告文件 |

### 1.3 审核时间分配建议

| 模块规模 | 建议时间 | 重点分配 |
|---------|---------|---------|
| 小模块（< 1000 行） | 30 分钟 | 70% 逻辑审查，30% 规范检查 |
| 中模块（1000-5000 行） | 1-2 小时 | 50% 核心逻辑，30% 安全/事务，20% 规范 |
| 大模块（> 5000 行） | 半天-1天 | 40% 架构设计，30% 核心逻辑，20% 安全，10% 规范 |

---

## 2. 审核范围确认

### 2.1 范围确认清单

在开始审核前，必须明确以下信息：

```markdown
## 审核范围确认

### 1. 审核目标
- 模块/项目名称：
- 审核类型：[全量审核 / 增量审核(PR) / 专项审核(安全/性能)]
- 模块核心业务：[一句话描述]

### 2. 模块边界
- 包含文件数量：
- 代码总行数：
- 核心 Service 文件：[列出最重要的 3-5 个文件]
- 外部依赖模块：[列出模块外被调用/调用的模块]

### 3. 业务上下文
- 是否涉及资金操作：[是/否]
- 是否涉及用户敏感数据：[是/否]
- 是否有并发场景：[是/否]
- 是否对外暴露 API：[是/否]
- 当前线上运行状态：[新模块未上线 / 已上线运行中]
```

### 2.2 审核优先级决策

```
模块是否涉及资金？
│
├─ 是 → 按【资金安全审核】标准，P0 检查项全部覆盖
│
└─ 否 → 模块是否对外暴露 API？
         │
         ├─ 是 → 按【接口安全审核】标准，重点检查权限和输入校验
         │
         └─ 否 → 按【常规代码质量审核】标准
```

---

## 3. 输出文档规范

### 3.1 报告命名规则

```
{模块名}-代码审核.md
```

存放位置：项目根目录下 `ai-doc/` 文件夹。

### 3.2 报告结构（强制）

所有审核报告必须包含以下结构：

```markdown
# {模块名} 模块代码审核报告

> **审核范围**: {文件路径}
> **核心文件**: {最重要的文件及行数}
> **审核日期**: {YYYY-MM-DD}

---

## 一、绝对问题（必须修复）
### P0-01 | {问题标题}
**文件**: {文件路径} L{行号}
**严重程度**: {致命/严重/高风险}
{问题描述 + 代码片段 + 修复建议}

## 二、重大质量问题（强烈建议修复）
### P1-01 | {问题标题}
...

## 三、建议改进（可选优化）
### P2-01 | {问题标题}
...

## 四、问题统计
| 优先级 | 数量 | 说明 |
|-------|------|------|

## 五、优先修复建议
1. **立即修复**: ...
2. **本迭代修复**: ...
3. **下迭代规划**: ...
```

### 3.3 问题描述铁律

每个问题必须包含：

1. **精确定位**：文件路径 + 行号范围
2. **问题本质**：一句话说清楚问题是什么
3. **代码证据**：贴出有问题的代码片段
4. **影响范围**：说明这个问题会导致什么后果
5. **修复建议**：给出具体的修复方向或代码示例

**禁止事项**：
- 禁止只说"代码不规范"但不说哪里不规范
- 禁止只说"建议优化"但不说怎么优化
- 禁止罗列"可能有问题"的猜测性结论
- 禁止把同一类问题拆成多条凑数量

---

# 第二部分：绝对问题检查（P0 红线）

> P0 问题 = 线上事故级别。发现即必须修复，不修完不能上线。

## 4. 安全漏洞

### 4.1 越权访问（最常见，最致命）

**检查要点**：每个写操作接口，是否验证了「当前用户有权操作目标资源」。

```java
// ❌ 致命：只校验了 orderId 存在，没校验订单是否属于当前用户
public ApiResult cancelOrder(Long userId, Long orderId) {
    FirmShopOrder order = this.getById(orderId);
    if (order == null) throw bizException("订单不存在");
    // 缺少: if (!order.getUserId().equals(userId)) throw bizException("无权操作");
    order.setStatus(OrderStatus.CANCELLED);
    this.updateById(order);
    return ApiResult.success();
}

// ✅ 正确：双重校验
public ApiResult cancelOrder(Long userId, Long orderId) {
    FirmShopOrder order = this.getById(orderId);
    if (order == null) throw bizException("订单不存在");
    if (!order.getUserId().equals(userId)) throw bizException("无权操作此订单");
    order.setStatus(OrderStatus.CANCELLED);
    this.updateById(order);
    return ApiResult.success();
}
```

**逐条检查清单**：

| 检查项 | 说明 |
|--------|------|
| C 端接口是否校验 userId 与资源归属 | 用户 A 不能操作用户 B 的数据 |
| B 端接口是否校验 shopId/tenantId 归属 | 店铺 A 不能操作店铺 B 的数据 |
| 删除/修改操作是否同时校验 id + 归属 | 不能仅凭 id 就允许操作 |
| 列表查询是否强制带归属过滤 | 不能查到其他租户的数据 |

### 4.2 SQL 注入

```java
// ❌ 致命：字符串拼接 SQL
String sql = "SELECT * FROM user WHERE name = '" + name + "'";

// ❌ 危险：MyBatis 中使用 ${}
@Select("SELECT * FROM user WHERE name = '${name}'")

// ✅ 正确：参数化查询
@Select("SELECT * FROM user WHERE name = #{name}")
```

### 4.3 XSS 攻击

```
检查所有用户输入存储后展示的场景：
- 评论内容
- 用户昵称
- 文章/作品标题和正文
- 商品描述
- 搜索关键词回显

修复：入库前过滤 HTML 标签，或出库时转义
```

### 4.4 敏感数据泄露

```
检查接口返回值中是否包含：
- 密码（即使是加密后的 hash）
- 完整手机号（应脱敏为 138****0000）
- 完整身份证号
- 数据库内部 ID 的规律性（是否可被遍历）
- 其他用户的个人信息

检查日志中是否打印了：
- 密码明文
- Token / Secret Key
- 完整手机号/身份证号
```

---

## 5. 资损与数据一致性

### 5.1 金额计算

```java
// ❌ 致命：浮点数计算金额
double price = 19.9;
double total = price * 3;  // 59.699999999999996

// ✅ 正确：整数（分）计算
int priceFen = 1990;
int total = priceFen * 3;  // 5970 分 = 59.70 元

// ✅ 正确：BigDecimal 计算
BigDecimal price = new BigDecimal("19.90");
BigDecimal total = price.multiply(new BigDecimal("3"));
```

**金额审核检查清单**：

| 检查项 | 说明 |
|--------|------|
| 金额字段是否为整数（分） | 数据库、接口传输、内部计算全链路 |
| 是否使用了 float/double 做金额运算 | 致命：浮点数精度问题 |
| 退款金额是否超过原订单金额 | 防止超退 |
| 多次退款总和是否校验 | 防止重复退款 |
| 优惠/抵扣后金额是否做了 `Math.max(0, ...)` | 防止负数金额 |
| 支付回调是否做了幂等校验 | 防止重复到账 |

### 5.2 支付流程审核

```
支付必检九条：
1. 以服务端回调为准，不信客户端返回结果
2. 回调必须验签
3. 回调必须幂等（同一通知多次推送不能重复处理）
4. 订单金额与支付金额必须校验一致
5. 未支付订单必须有超时自动关闭
6. 退款后必须更新订单状态 + 记录退款流水
7. 退款走异步通知，不能同步等待
8. 资金操作必须有操作日志（谁、什么时间、做了什么）
9. 财务记录不能 try-catch 吞异常
```

### 5.3 库存/余额扣减

```java
// ❌ 危险：先读后写，并发下超卖/超扣
int stock = goods.getStock();
if (stock > 0) {
    goods.setStock(stock - 1);
    goodsMapper.updateById(goods);
}

// ✅ 正确：数据库乐观锁 / CAS 更新
int affected = goodsMapper.decreaseStock(goodsId, 1);
// UPDATE goods SET stock = stock - 1 WHERE id = ? AND stock >= 1
if (affected == 0) {
    throw bizException("库存不足");
}
```

---

## 6. 逻辑缺陷与空实现

### 6.1 TODO / 空方法体

```
搜索关键词：TODO、FIXME、HACK、XXX、空方法体
重点关注：
- 方法内只有 TODO 注释但返回了"成功"
- 方法声明了但 body 为空
- 被注释掉的关键逻辑（如退款、退库存）

判定标准：
- 如果方法已被 Controller 暴露为接口 → P0
- 如果方法只在内部使用且有替代方案 → P1
```

### 6.2 条件逻辑反转

```java
// ❌ 常见 Bug：条件写反导致逻辑永远走不进去或永远走进去
if (order.getAwardId() != null && order.getAwardId() != 0) {
    // 如果订单刚创建时 awardId 为 null，此条件永远为 false
    applyAward(...);
}

// 审核技巧：
// 1. 看条件中每个子表达式在实际场景下的值
// 2. 用真实数据代入跑一遍逻辑
// 3. 检查 && 和 || 是否写反
// 4. 检查 == 和 != 是否写反
// 5. 检查 > 和 >= 的边界
```

### 6.3 不可达代码

```java
// ❌ Bug：操作日志永远不会执行
if (payType == 1) {
    // ... 处理逻辑 ...
    return order.getId();   // 直接 return
}
if (payType == 2) {
    // ... 处理逻辑 ...
    return order.getId();   // 直接 return
}

// 下面的代码永远不会执行
addOperationLog(order);  // 不可达
throw bizException("不支持的支付类型");

// 审核技巧：检查所有 if/switch 分支是否都有提前 return
```

---

## 7. 并发与线程安全

### 7.1 分布式锁检查

```java
// ❌ 危险：Spring AOP 代理的内部调用不生效
@Service
public class OrderService {
    @WebLock(keys = "#roomId")          // 这个锁通过 AOP 实现
    public boolean checkCanUse(Long roomId) { ... }

    public void submitOrder(...) {
        checkCanUse(roomId);             // 同类内部调用，AOP 不生效，锁无效！
    }
}

// ✅ 正确：通过注入自身代理或使用 ApplicationContext 获取代理对象
@Autowired
private ApplicationContext ctx;

public void submitOrder(...) {
    ctx.getBean(OrderService.class).checkCanUse(roomId);
}
```

**并发审核检查清单**：

| 检查项 | 说明 |
|--------|------|
| 订单操作是否加了分布式锁 | 防止同一订单被并发操作 |
| 锁的粒度是否合理 | 不要锁整个表，应该锁具体的订单 ID |
| 锁超时时间是否合理 | 太短会导致业务没执行完锁就释放 |
| finally 中是否释放锁 | 防止死锁 |
| 计数器更新（点赞数、库存）是否原子操作 | 先读后写有并发问题 |

### 7.2 先读后写（Read-Modify-Write）

```java
// ❌ 典型并发 Bug：点赞数在高并发下丢失
work.setLikeCount(work.getLikeCount() + 1);
workService.updateById(work);

// ✅ 正确：SQL 原子自增
workMapper.incrementLikeCount(workId);
// UPDATE community_work SET like_count = like_count + 1 WHERE id = #{id}
```

---

# 第三部分：重大质量问题（P1）

> P1 问题 = 不影响现有功能但影响长期维护，或存在潜在风险。

## 8. 架构与设计

### 8.1 上帝类 / 上帝方法

**判定标准**：

| 指标 | 健康 | 警告 | 必须重构 |
|------|------|------|---------|
| 单文件行数 | < 500 行 | 500-1000 行 | > 1000 行 |
| 单方法行数 | < 50 行 | 50-100 行 | > 100 行 |
| 类依赖注入数 | < 7 个 | 7-12 个 | > 12 个 |
| 类职责数 | 1 个 | 2-3 个 | > 3 个 |

**拆分建议**（以订单 Service 为例）：

```
OrderServiceImpl (2000+ 行) → 拆分为：
├── OrderSubmitService        # 提交订单
├── OrderPayService           # 支付相关
├── OrderOperationService     # 结账/续时/换房/取消/退款
├── OrderQueryService         # 查询/统计/报表
└── OrderScheduleService      # 定时任务
```

### 8.2 Entity 充当 VO

```java
// ❌ 反模式：Entity 上挂满非数据库字段
@TableName("t_order")
public class Order extends BaseModel {
    private Long id;           // 数据库字段
    private Integer money;     // 数据库字段

    @TableField(exist = false)
    private String userName;   // 非数据库字段 ×
    @TableField(exist = false)
    private Object userInfo;   // 非数据库字段 × 且用了 Object 类型
    @TableField(exist = false)
    private String shopName;   // 非数据库字段 ×
    // ... 还有 20 个非数据库字段
}

// ✅ 正确：Entity 只映射数据库字段，查询返回用 VO
@Data
public class OrderDetailVO {
    private Long id;
    private Integer money;
    private String userName;
    private UserInfoVO userInfo;  // 强类型
    private String shopName;
}
```

**危害**：
- API 返回完整数据库字段，存在信息泄漏风险
- 不同接口返回同一 Entity 但填充字段不同，前端无法区分
- Object 类型字段无法序列化控制，缺乏类型安全

### 8.3 重复代码

```
重复代码判定标准：
- 完全相同的代码块出现 2 次以上 → 必须抽取
- 结构相同仅参数不同的代码块出现 3 次以上 → 必须抽取
- switch/case 映射逻辑出现 2 次以上 → 抽取为公共方法或 Map

审核技巧：搜索特征关键词，看是否在多处出现
例如：搜索 "FirmShopFinancialDataBizSubTypeEnum" 看财务类型映射是否重复
```

---

## 9. 事务与异常处理

### 9.1 缺少事务注解

**必须加 @Transactional 的场景**：

| 场景 | 示例 |
|------|------|
| 涉及两个以上表的写操作 | 创建订单 + 扣库存 |
| 先写后依赖写入结果再写 | 创建用户 + 初始化钱包 |
| 有条件回滚需求 | 支付成功后更新订单，财务记录失败需回滚 |
| 扣减余额 + 记录流水 | 会员扣费 + 扣费记录 |

```java
// ❌ 危险：多表写入没有事务，扣了余额但记录流水失败
public void memberPay(Long userId, Long orderId) {
    memberService.deductBalance(userId, amount);   // 1. 扣余额
    orderMapper.updateStatus(orderId, PAID);        // 2. 更新订单
    financialService.addRecord(order);              // 3. 如果这步失败，余额已扣但没记录
}

// ✅ 正确：加事务保证原子性
@Transactional(rollbackFor = Exception.class)
public void memberPay(Long userId, Long orderId) {
    memberService.deductBalance(userId, amount);
    orderMapper.updateStatus(orderId, PAID);
    financialService.addRecord(order);
}
```

### 9.2 异常处理不统一

```
审核要点：一个模块内只应使用一种业务异常抛出方式。

常见混乱场景：
- throw new RuntimeException("xxx")        // 方式 A
- throw new ApiBizException("xxx")         // 方式 B
- throw bizException("xxx")               // 方式 C（静态工厂）
- return ApiResult.fail("xxx")            // 方式 D（静默失败）

危害：
- RuntimeException 可能泄漏堆栈信息到前端
- 混用导致全局异常处理器无法统一拦截
- return fail 与 throw 混用导致调用方无法统一 catch

修复：统一使用一种方式，推荐 BusinessException + 全局异常处理器
```

### 9.3 异常被吞

```java
// ❌ 严重：财务记录失败但异常被吞，导致资金数据丢失
try {
    financialService.addRecord(order);
} catch (Exception e) {
    log.error("添加财务记录失败", e);
    // 然后什么都不做，流程继续
}

// 判定标准：
// - 核心数据（资金、订单状态）的写入失败被 catch 住 → P0
// - 非核心数据（日志、统计）的写入失败被 catch 住 → P2（但建议加告警）
```

---

## 10. API 设计规范

### 10.1 HTTP 方法使用

```
有副作用的操作必须用 POST/PUT/DELETE，禁止使用 GET：

❌ GET /order/{id}/cancel     → 取消订单（有副作用）
❌ GET /order/{id}/end        → 结束订单（有副作用）
❌ GET /room/{id}/change      → 换房间（有副作用）

✅ POST /order/{id}/cancel
✅ PUT  /order/{id}/status
✅ POST /order/{id}/change-room

原因：
1. GET 请求可能被浏览器预取（prefetch）
2. GET 请求可能被 CDN/Proxy 缓存
3. GET 请求会出现在浏览器历史记录和 Nginx access log 中
4. 爬虫可能触发 GET 请求
```

### 10.2 Controller 返回值

```java
// ❌ 混乱：同一模块有的用泛型有的不用
public ApiResult cancelOrder(...)       // 无泛型 → 前端不知道 data 类型
public ApiResult<Long> submitOrder(...) // 有泛型 ✓

// ✅ 正确：统一使用泛型
public ApiResult<Void> cancelOrder(...)
public ApiResult<Long> submitOrder(...)
public ApiResult<PageVo<OrderVO>> orderPage(...)
```

### 10.3 接口参数校验

```java
// ❌ 危险：Controller 参数没有 @RequestParam 注解
@PostMapping("/time/{orderId}")
public ApiResult addTime(Integer addMin) {  // POST 请求中裸参数可能接收不到
    ...
}

// ✅ 正确
@PostMapping("/time/{orderId}")
public ApiResult addTime(@RequestParam Integer addMin) { ... }

// 或者用 RequestBody
@PostMapping("/time/{orderId}")
public ApiResult addTime(@RequestBody @Valid AddTimeReqVo vo) { ... }
```

---

## 11. 代码规范与可维护性

### 11.1 魔法数字 / 魔法字符串

```java
// ❌ 反模式：到处写数字，没人知道 4 是什么意思
if (order.getPayType() != null && order.getPayType() == 4) { ... }
switch (payType) {
    case 1: bizSubType = FUSE_KIT_OTA_MT; break;
    case 3: bizSubType = FUSE_KIT_CASH; break;
}

// ✅ 正确：使用枚举常量
if (PaymentType.MEMBER_PAYMENT.getType().equals(order.getPayType())) { ... }
```

**审核方法**：搜索代码中的纯数字（排除 0 和 1），检查是否应该用常量/枚举替代。

### 11.2 枚举 ordinal 的危险使用

```java
// ❌ 高风险：依赖枚举声明顺序
if (status.ordinal() > FuseKitOrderStatus.UNPAID.ordinal()) { ... }
FuseKitOrderStatus.values()[statusIndex]  // 越界风险

// ✅ 正确：通过 code/name 比较
if (!status.equals(FuseKitOrderStatus.UNPAID)) { ... }
FuseKitOrderStatus.getByCode(statusCode)  // 安全查找
```

### 11.3 命名规范检查

| 检查项 | 标准 | 反面教材 |
|--------|------|---------|
| 方法名应表达业务含义 | `calculateOrderPrice()` | `calc()` |
| 布尔变量/方法用 is/has/can 前缀 | `isExpired()` | `expired()` |
| 集合变量用复数 | `List<Order> orders` | `List<Order> orderList` |
| 常量全大写下划线分隔 | `MAX_RETRY_COUNT` | `maxRetryCount` |
| VO/DTO 后缀明确 | `OrderDetailVO` | `OrderInfo`（含义模糊） |
| 字段名不能与业务含义矛盾 | `minDuration`（最小时长） | `hours`（实际存的是秒） |

---

# 第四部分：建议改进（P2）

## 12. 性能优化

### 12.1 N+1 查询

```java
// ❌ 经典 N+1：循环中查数据库
for (User user : userList) {
    List<Order> orders = orderMapper.selectByUserId(user.getId());  // N 次查询
}

// ✅ 正确：批量查询 + Map 组装
List<Long> userIds = userList.stream().map(User::getId).collect(Collectors.toList());
Map<Long, List<Order>> orderMap = orderMapper.selectByUserIds(userIds)
    .stream().collect(Collectors.groupingBy(Order::getUserId));
```

### 12.2 全量查询无分页

```java
// ❌ 危险：不加 LIMIT 的查询可能返回百万条
List<Order> orders = this.list(queryWrapper);  // 全量加载到内存

// 审核标准：
// 所有返回 List 的查询方法，检查是否有 LIMIT 或分页
// 如果是定时任务批量处理，检查是否分批处理
```

### 12.3 缓存检查

```
需要缓存的典型场景：
- 配置数据（店铺设置、系统参数）：读多写少
- 热点数据（商品详情、首页推荐）：高频访问
- 计算结果（价格计算、权限树）：计算成本高

需要检查的缓存问题：
- 缓存穿透：查不存在的数据导致每次都打到数据库
- 缓存击穿：热点 key 过期瞬间大量请求打到数据库
- 缓存雪崩：大量 key 同时过期
- 缓存与数据库一致性：更新数据后是否清除缓存
```

---

## 13. 可观测性

### 13.1 日志审核

| 检查项 | 说明 |
|--------|------|
| 关键业务节点是否有日志 | 下单、支付、退款、状态变更 |
| 异常是否打印了堆栈 | `log.error("msg", e)` 而非 `log.error("msg")` |
| 日志级别是否正确 | 业务异常用 WARN，系统异常用 ERROR |
| 是否打印了敏感信息 | 密码、Token、完整手机号 |
| 日志信息是否有业务上下文 | 包含 orderId、userId 等关键标识 |

### 13.2 操作日志审核

```
涉及资金和核心业务的操作，必须有操作日志：
- 谁（operatorId）
- 什么时间（opTime）
- 对什么对象（orderId / userId）
- 做了什么操作（opType）
- 操作前后的关键数据变化
- 客户端 IP

审核要点：
- 操作日志是否在所有路径都被记录（注意提前 return 的分支）
- 异步写日志是否有失败补偿机制
```

---

## 14. 可测试性

```
可测试性审核要点：
1. Service 是否依赖了 HttpServletRequest / HttpServletResponse
   → 难以单元测试，应通过参数传入需要的值
2. 是否使用了过多的 static 方法
   → 无法 mock，难以测试
3. 方法是否过长（> 50 行）
   → 难以写出有效的单元测试
4. 是否有硬编码的外部依赖（URL、文件路径）
   → 应通过配置注入
```

---

# 第五部分：技术栈专项检查

## 15. Java / Spring Boot 专项

### 15.1 Spring 注入检查

```java
// ❌ 不推荐：字段注入（难以测试，隐式依赖）
@Autowired
private OrderService orderService;

// ✅ 推荐：构造器注入
@RequiredArgsConstructor
public class OrderController {
    private final OrderService orderService;
}
```

### 15.2 MyBatis-Plus 检查

| 检查项 | 说明 |
|--------|------|
| `LambdaQueryWrapper` 是否缺少必要过滤条件 | 如 `is_deleted = 0`、`firm_shop_id = ?` |
| `getOne()` 是否加了 `last("LIMIT 1")` | 防止多条数据报错 |
| 批量操作是否使用了 `saveBatch` | 而非循环单条 `save` |
| 是否有 `SELECT *` 的全字段查询 | 列表查询应只查需要的字段 |
| 枚举类型是否正确配置了 TypeHandler | 防止存储/读取类型不匹配 |

### 15.3 常见 Spring Boot 陷阱

| 陷阱 | 说明 |
|------|------|
| `@Transactional` 在 private 方法上不生效 | AOP 代理限制 |
| `@Transactional` 在同类内部调用不生效 | 同上 |
| `@Async` 在同类内部调用不生效 | 同上 |
| `@Cacheable` 在同类内部调用不生效 | 同上 |
| `BeanUtils.copyProperties` 会忽略 null 以外的类型不匹配 | 静默丢失数据 |

---

## 16. 前端 Vue / React 专项

### 16.1 安全检查

| 检查项 | 说明 |
|--------|------|
| `v-html` / `dangerouslySetInnerHTML` 是否使用了未过滤的用户输入 | XSS 风险 |
| Token 是否存储在 localStorage 且无 HttpOnly | 建议 HttpOnly Cookie |
| 路由守卫是否完整 | 未登录不能访问需认证页面 |
| 前端是否存储了敏感信息（密码、密钥） | 绝对禁止 |

### 16.2 性能检查

| 检查项 | 说明 |
|--------|------|
| 路由是否懒加载 | `() => import('./Page.vue')` |
| 大列表是否用了虚拟滚动 | > 100 条数据时 |
| 图片是否懒加载 | 列表中的图片 |
| 组件是否有不必要的重渲染 | React: useMemo/useCallback |
| 是否有内存泄漏 | 定时器/事件监听未在组件销毁时清除 |

### 16.3 体验检查

| 检查项 | 说明 |
|--------|------|
| 提交按钮是否有防重复点击 | loading + disabled |
| 列表是否有空状态 | 空数据、加载中、加载失败 |
| 金额展示是否分转元 | 后端存分，前端显示元 |
| 时间展示是否格式化 | 不要展示时间戳或 ISO 字符串 |

---

## 17. 数据库与 SQL 专项

### 17.1 表设计检查

| 检查项 | 说明 |
|--------|------|
| 是否有 `create_time`、`update_time` | 所有表必备 |
| 是否使用 `utf8mb4` 字符集 | 支持 emoji 和完整中文 |
| 金额字段是否为 INT/BIGINT（分） | 禁止 FLOAT/DOUBLE |
| 外键字段是否有索引 | `user_id`、`order_id` 等 |
| 是否使用逻辑删除 | 业务数据禁止物理删除 |

### 17.2 SQL 检查

| 检查项 | 说明 |
|--------|------|
| 是否有 `SELECT *` | 应该只查需要的字段 |
| 是否有不带 WHERE 的 UPDATE/DELETE | 致命：全表更新/删除 |
| IN 子句元素是否超过 1000 | 应分批处理 |
| 是否有慢查询风险（无索引的 WHERE 条件） | 检查 EXPLAIN |
| 时间范围查询是否走了索引 | 联合索引最左前缀原则 |

---

# 附录

## 附录 A. 问题严重程度定义

| 等级 | 代号 | 含义 | 定义 | 要求 |
|------|------|------|------|------|
| **P0** | 绝对问题 | 线上事故级 | 安全漏洞、资损、数据丢失、核心功能不可用 | 立即修复，不修完不上线 |
| **P1** | 重大问题 | 严重隐患 | 架构缺陷、事务缺失、性能瓶颈、规范严重违反 | 当前迭代修复 |
| **P2** | 改进建议 | 优化空间 | 代码可读性、可维护性、性能优化、最佳实践 | 下迭代或 Tech Debt 排期 |

## 附录 B. 审核报告模板

```markdown
# {模块名} 模块代码审核报告

> **审核范围**: `{根包路径}` 全部 {N} 个文件
> **核心文件**: `{核心Service文件名}` ({N} 行)
> **审核日期**: {YYYY-MM-DD}

---

## 一、绝对问题（必须修复）

### P0-01 | {一句话标题}

**文件**: `{文件路径}` L{起始行}-L{结束行}
**严重程度**: **{致命/严重/高风险}**

{问题描述：2-3 句话说清楚问题本质和影响}

// 有问题的代码片段（5-10行，标注问题行）

**修复**: {具体修复建议}

---

## 二、重大质量问题（强烈建议修复）

### P1-01 | {一句话标题}
...

---

## 三、建议改进（可选优化）

### P2-01 | {一句话标题}
...

---

## 四、问题统计

| 优先级 | 数量 | 说明 |
|-------|------|------|
| P0 绝对问题 | {N} | {概括说明} |
| P1 重大问题 | {N} | {概括说明} |
| P2 改进建议 | {N} | {概括说明} |
| **合计** | **{N}** | |

---

## 五、优先修复建议

1. **立即修复**: {列出最紧急的 P0}
2. **本迭代修复**: {列出剩余 P0 + 重要 P1}
3. **下迭代规划**: {P1 中的架构类 + P2}
```

## 附录 C. 常见反模式速查表

| 反模式 | 检测方法 | 等级 |
|--------|---------|------|
| 退款/退库存方法为空实现但返回成功 | 搜索 TODO + return success | P0 |
| 写操作接口缺少归属校验 | 每个 POST/PUT/DELETE 检查 | P0 |
| 支付回调未验签 | 搜索 callback/notify 方法 | P0 |
| 支付回调未做幂等 | 同上 | P0 |
| 浮点数做金额运算 | 搜索 double/float + money/price | P0 |
| 单文件超 1000 行 | 统计行数 | P1 |
| 多表写入无事务 | 搜索 save/update 连续调用 | P1 |
| catch 吞掉关键异常 | 搜索 catch + 检查是否只 log | P1 |
| GET 方法做写操作 | 检查 @GetMapping 内是否有 save/update | P1 |
| Entity 超过 5 个 @TableField(exist=false) | 统计 | P1 |
| 循环中查数据库 | 搜索 for + selectById/getById | P2 |
| 列表查询无分页 | 搜索 this.list() 无 LIMIT | P2 |
| 硬编码数字代替枚举 | 搜索 `== 1` `== 2` 等 | P2 |

## 附录 D. AI 审核 Prompt 模板

将以下 Prompt 发送给 AI 编码助手即可触发标准化代码审核：

```markdown
## 代码审核指令

请对 {模块名} 模块进行全量代码审核。

### 审核范围
- 模块路径：{路径}
- 核心业务：{一句话描述}
- 是否涉及资金：{是/否}

### 审核要求
1. 先读取所有文件，理解模块整体结构
2. 按 P0 → P1 → P2 的优先级检查
3. 每个问题标注具体文件和行号
4. 输出格式按审核报告模板
5. 报告保存到 ai-doc/{模块名}-代码审核.md

### 重点关注
- 安全漏洞（越权、注入、敏感数据泄露）
- 资金安全（金额计算、支付流程、退款逻辑）
- 并发安全（锁、原子操作、竞态条件）
- 事务完整性（多表写入是否有事务保护）
- 逻辑缺陷（条件反转、空实现、不可达代码）
```

---

> **本手册持续更新，遇到新的通用审核规则会不断补充。**
>
> 最后更新：2026-04-16
