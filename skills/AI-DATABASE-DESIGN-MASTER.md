# AI 数据库设计大师

> **适用工具**：Cursor / ChatGPT / Claude / GitHub Copilot / Windsurf / Cline
>
> **定位**：本文件是 AI 助手执行数据库设计任务的系统级指令文件（Skill File）。AI 在设计任何数据库之前必须完整阅读本文件，并严格按照其中的规范、流程和约束执行。
>
> 本手册参考《阿里巴巴 Java 开发手册（数据库篇）》《高性能 MySQL》《数据库设计入门经典》以及字节跳动、阿里、腾讯、美团等一线互联网公司 DBA 团队规范编写，适用于 MySQL / PostgreSQL 为主的关系型数据库设计。
>
> **目标读者**：通过 AI 辅助进行数据库设计的开发者。AI 阅读后应具备资深 DBA + 架构师级别的数据库设计能力。

---

## 目录

**第一部分：设计流程（先想清楚再建表）**
1. [需求到数据模型的转化流程](#1-需求到数据模型的转化流程)
2. [概念模型设计（ER 图）](#2-概念模型设计er-图)
3. [逻辑模型设计](#3-逻辑模型设计)

**第二部分：建表规范（红线，必须遵守）**
4. [命名规范](#4-命名规范)
5. [字段设计规范](#5-字段设计规范)
6. [主键与唯一约束](#6-主键与唯一约束)
7. [必备字段规范](#7-必备字段规范)

**第三部分：索引设计（快慢就看这一章）**
8. [索引设计原则](#8-索引设计原则)
9. [索引类型选择](#9-索引类型选择)
10. [索引失效场景](#10-索引失效场景)

**第四部分：SQL 规范（写出不挖坑的 SQL）**
11. [查询规范](#11-查询规范)
12. [写入规范](#12-写入规范)
13. [事务与锁](#13-事务与锁)

**第五部分：高阶设计（支撑业务增长）**
14. [范式与反范式决策](#14-范式与反范式决策)
15. [分库分表设计](#15-分库分表设计)
16. [读写分离设计](#16-读写分离设计)
17. [缓存与数据库协同](#17-缓存与数据库协同)

**第六部分：典型业务库表设计（拿来就用）**
18. [用户体系设计](#18-用户体系设计)
19. [商品与 SKU 设计](#19-商品与-sku-设计)
20. [订单与支付设计](#20-订单与支付设计)
21. [权限体系设计（RBAC）](#21-权限体系设计rbac)
22. [内容与审核设计](#22-内容与审核设计)

**附录**
- [A. MySQL 字段类型速查表](#附录-a-mysql-字段类型速查表)
- [B. 建表语句模板](#附录-b-建表语句模板)
- [C. 常见性能问题自查清单](#附录-c-常见性能问题自查清单)
- [D. AI 数据库设计 Prompt 模板](#附录-d-ai-数据库设计-prompt-模板)

---

# 第一部分：设计流程

## 1. 需求到数据模型的转化流程

### 1.1 标准设计流程

AI 接收到数据库设计需求后，必须按以下流程执行：

```
业务需求 → 识别核心实体 → 概念模型（ER 图） → 逻辑模型（表结构）
         → 物理模型（字段类型/索引） → 评审检查 → 输出建表 SQL
```

**禁止**直接从需求跳到写建表语句，必须经过建模环节。

### 1.2 需求分析阶段必做的事

| 步骤 | 要做的事 | 输出物 |
|------|---------|--------|
| 识别实体 | 从需求中提取名词 → 转化为实体 | 实体列表 |
| 识别关系 | 确定实体间关系（1:1 / 1:N / M:N） | 关系矩阵 |
| 识别属性 | 每个实体有哪些属性 | 属性列表 |
| 预估数据量 | 未来 1-3 年的数据增长 | 容量规划 |
| 确认查询模式 | 主要读场景和写场景 | 查询模式列表 |

### 1.3 数据量预估模板

```markdown
## 数据量预估

| 实体 | 当前量级 | 日增长 | 1年后量级 | 3年后量级 | 单行大小 |
|------|---------|--------|----------|----------|---------|
| 用户 | 0 | 500/天 | 18 万 | 55 万 | ~200B |
| 商品 | 0 | 50/天 | 1.8 万 | 5.5 万 | ~500B |
| 订单 | 0 | 1000/天 | 36 万 | 110 万 | ~300B |
| 订单项 | 0 | 2500/天 | 90 万 | 275 万 | ~200B |

结论：3 年内单表最大约 275 万，MySQL 单表轻松承载，无需分表。
```

## 2. 概念模型设计（ER 图）

### 2.1 ER 图输出格式

AI 使用文本方式描述 ER 图：

```
[用户] 1 ──── N [订单]
  │                │
  │ 1:N            │ 1:N
  ↓                ↓
[收货地址]       [订单项] N ──── 1 [商品]
                                     │
                                     │ 1:N
                                     ↓
                                   [SKU]

[用户] M ──── N [优惠券]（通过"用户优惠券"关联表）
```

### 2.2 关系类型判断规则

| 关系 | 判断方法 | 处理方式 |
|------|---------|---------|
| **1:1** | A 的一条记录只对应 B 的一条 | 合并为一张表，或用外键关联 |
| **1:N** | A 的一条记录对应 B 的多条 | 在 B 表加 A 的外键字段 |
| **M:N** | A 和 B 互相对应多条 | 创建中间关联表 |

### 2.3 实体关系确认清单

每对实体关系都要回答以下问题：

- [ ] 关系是 1:1、1:N 还是 M:N？
- [ ] 关系是否可选？（用户可以没有订单吗？→ 可以）
- [ ] 删除一方时另一方怎么办？（级联删除？置空？禁止？）
- [ ] 关系本身是否有属性？（"用户-优惠券"关系有"领取时间"属性 → 需要关联表）

## 3. 逻辑模型设计

### 3.1 从 ER 图到表结构的转化规则

| ER 元素 | 转化为 | 规则 |
|---------|--------|------|
| 实体 | 表 | 一个实体一张表 |
| 属性 | 字段 | 一个属性一个字段 |
| 1:1 关系 | 外键或合并 | 访问频率高 → 合并；数据量差异大 → 拆分 |
| 1:N 关系 | 外键 | 在 N 端加 1 端的 ID 字段 |
| M:N 关系 | 关联表 | 新建表，包含两端 ID + 关系属性 |

### 3.2 表结构设计输出格式

AI 输出表结构时，必须使用以下格式：

```markdown
### 表名：t_order（订单表）

**说明**：记录用户提交的订单信息
**预估数据量**：3 年内约 110 万行
**核心查询场景**：
- 按用户查订单列表（高频）
- 按订单号查详情（高频）
- 按状态查订单（中频，后台）
- 按时间范围查订单（低频，报表）

| 字段名 | 类型 | 允许空 | 默认值 | 说明 |
|--------|------|--------|--------|------|
| id | bigint unsigned | NO | 自增 | 主键 |
| order_no | varchar(32) | NO | | 订单编号，唯一 |
| user_id | bigint unsigned | NO | | 用户 ID |
| total_amount | decimal(10,2) | NO | | 订单总金额 |
| pay_amount | decimal(10,2) | NO | | 实付金额 |
| status | tinyint | NO | 0 | 订单状态：0待支付 1已支付 2已发货 3已完成 4已取消 |
| address_snapshot | json | NO | | 收货地址快照 |
| remark | varchar(500) | YES | | 买家备注 |
| pay_time | datetime | YES | | 支付时间 |
| deliver_time | datetime | YES | | 发货时间 |
| complete_time | datetime | YES | | 完成时间 |
| created_at | datetime | NO | CURRENT_TIMESTAMP | 创建时间 |
| updated_at | datetime | NO | CURRENT_TIMESTAMP ON UPDATE | 更新时间 |
| deleted_at | datetime | YES | NULL | 软删除时间 |

**索引设计**：
| 索引名 | 类型 | 字段 | 说明 |
|--------|------|------|------|
| uk_order_no | UNIQUE | order_no | 订单号唯一 |
| idx_user_id_status | NORMAL | (user_id, status) | 用户查订单列表 |
| idx_created_at | NORMAL | created_at | 按时间范围查询 |
```

---

# 第二部分：建表规范

## 4. 命名规范

### 4.1 总体规则

| 规则 | 正确示例 | 错误示例 |
|------|---------|---------|
| **全部小写，下划线分隔** | `user_order` | `UserOrder` / `userorder` |
| **表名加 `t_` 前缀**（推荐） | `t_user` | `user`（可能与关键字冲突） |
| **禁止使用 MySQL 保留字** | `t_order` | `order` / `desc` / `group` |
| **表名用单数** | `t_user` | `t_users` |
| **字段名不加表名前缀** | `name` | `user_name`（在 user 表里多余） |
| **布尔字段用 `is_` 前缀** | `is_deleted` | `deleted` / `is_del` |
| **时间字段用 `_at` 或 `_time` 后缀** | `created_at` | `create_date` / `ctime` |

### 4.2 常用命名约定

| 场景 | 命名规范 | 示例 |
|------|---------|------|
| 主键 | `id` | `id` |
| 外键 | `{关联表}_id` | `user_id`、`order_id` |
| 创建时间 | `created_at` | |
| 更新时间 | `updated_at` | |
| 软删除 | `deleted_at` | |
| 状态字段 | `status` 或具体名称 | `order_status`、`pay_status` |
| 排序字段 | `sort_order` 或 `seq` | |
| 关联表 | `t_{表A}_{表B}` | `t_user_role` |

## 5. 字段设计规范

### 5.1 类型选择原则

**能小就不大**——选满足需求的最小数据类型：

| 场景 | 推荐类型 | 不推荐 | 原因 |
|------|---------|--------|------|
| 主键 | `bigint unsigned` | `int` | int 上限 21 亿，高速增长可能溢出 |
| 金额 | `decimal(10,2)` | `float` / `double` | 浮点数有精度丢失 |
| 状态/类型 | `tinyint` | `int` / `varchar` | 状态值不超过 127 个 |
| 布尔 | `tinyint(1)` | `bit` / `char(1)` | MySQL 中 tinyint 最通用 |
| 手机号 | `varchar(20)` | `bigint` | 手机号有前导零、国际号码带 + |
| 身份证号 | `varchar(18)` | `bigint` | 有字母 X |
| IP 地址 | `int unsigned` / `varchar(45)` | `varchar(15)` | IPv6 需要 45 字符 |
| 大文本 | `text` | `varchar(10000)` | 超过 5000 字符用 text |
| JSON 数据 | `json` | `text` | MySQL 5.7+ 原生 JSON 类型 |
| 时间 | `datetime` | `timestamp` | timestamp 有 2038 年问题 |

### 5.2 字段约束规则

| 规则 | 说明 |
|------|------|
| **尽量 NOT NULL** | 字段默认设为 NOT NULL + 有意义的默认值 |
| **禁止 NULL 参与索引** | NULL 值在索引中行为不一致 |
| **字符串默认值用空字符串** | `DEFAULT ''` 而不是 `DEFAULT NULL` |
| **数值默认值用 0** | `DEFAULT 0` 而不是 `DEFAULT NULL` |
| **禁止存储大文件** | 图片/文件存 OSS，数据库只存 URL |
| **禁止明文存密码** | 必须加密或哈希存储 |

### 5.3 字段长度参考

| 场景 | 推荐长度 | 说明 |
|------|---------|------|
| 用户名/昵称 | `varchar(50)` | 中文约 16 字 |
| 手机号 | `varchar(20)` | 国际号码预留 |
| 邮箱 | `varchar(100)` | RFC 标准最长 254 |
| 密码哈希 | `varchar(128)` | bcrypt 输出 60 字符 |
| URL | `varchar(500)` | 一般足够 |
| 标题 | `varchar(200)` | |
| 简介/描述 | `varchar(1000)` | |
| 详情/正文 | `text` / `mediumtext` | |
| 订单编号 | `varchar(32)` | 如 20260416153000001 |
| UUID | `varchar(36)` | 标准 UUID 格式 |

## 6. 主键与唯一约束

### 6.1 主键设计规则

| 规则 | 说明 |
|------|------|
| **必须有主键** | 每张表必须有主键，无例外 |
| **推荐自增 bigint** | `id bigint unsigned AUTO_INCREMENT` |
| **禁止用业务字段做主键** | 手机号、身份证号不能做主键（会变、有隐私问题） |
| **UUID 做主键要谨慎** | 随机 UUID 导致索引页分裂，性能差。如果要用建议用有序 UUID |
| **分布式 ID** | 大规模系统用 Snowflake / 美团 Leaf / 百度 UidGenerator |

### 6.2 唯一约束设计

```sql
-- 业务唯一键用 UNIQUE INDEX，不用 UNIQUE CONSTRAINT
ALTER TABLE t_user ADD UNIQUE INDEX uk_phone (phone);
ALTER TABLE t_order ADD UNIQUE INDEX uk_order_no (order_no);

-- 联合唯一：用户每天只能签到一次
ALTER TABLE t_sign_in ADD UNIQUE INDEX uk_user_date (user_id, sign_date);
```

## 7. 必备字段规范

### 7.1 每张表必须包含的字段

```sql
-- 每张表的固定结构
CREATE TABLE t_xxx (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT COMMENT '主键',
    -- ... 业务字段 ...
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at  datetime         NULL     DEFAULT NULL COMMENT '软删除时间',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='表说明';
```

### 7.2 字段说明

| 必备字段 | 类型 | 用途 |
|---------|------|------|
| `id` | bigint unsigned AUTO_INCREMENT | 主键，自增 |
| `created_at` | datetime DEFAULT CURRENT_TIMESTAMP | 记录创建时间 |
| `updated_at` | datetime ON UPDATE CURRENT_TIMESTAMP | 记录最后更新时间 |
| `deleted_at` | datetime NULL | 软删除标记，NULL 表示未删除 |

### 7.3 可选通用字段

| 字段 | 适用场景 |
|------|---------|
| `created_by` | 需要记录谁创建的（后台管理系统） |
| `updated_by` | 需要记录谁修改的 |
| `version` | 乐观锁场景 |
| `tenant_id` | SaaS 多租户系统 |
| `sort_order` | 需要自定义排序 |

---

# 第三部分：索引设计

## 8. 索引设计原则

### 8.1 核心原则

| 原则 | 说明 |
|------|------|
| **先有查询，再设索引** | 根据实际 SQL 查询模式设计索引，不要凭空猜 |
| **单表索引不超过 5 个** | 索引太多影响写入性能 |
| **联合索引优先** | 一个联合索引覆盖多个查询 > 多个单列索引 |
| **最左前缀原则** | 联合索引 (A,B,C) 能命中 A / A,B / A,B,C 的查询 |
| **高区分度字段在前** | 性别（2种）< 城市（300+）< 用户ID（百万），用户ID 放前面 |
| **覆盖索引优化** | 查询的字段全在索引中，避免回表 |

### 8.2 索引设计步骤

```
1. 收集所有 SQL 查询（含 WHERE / ORDER BY / GROUP BY / JOIN）
2. 按查询频率排序（高频优先）
3. 为高频查询设计索引
4. 合并可复用的索引（利用最左前缀）
5. 检查是否超过 5 个索引
6. 用 EXPLAIN 验证索引命中情况
```

### 8.3 索引设计输出格式

```markdown
### 索引设计：t_order

**查询场景分析**：
| 场景 | SQL 模式 | 频率 | 当前耗时 |
|------|---------|------|---------|
| 用户查订单 | WHERE user_id=? AND status=? ORDER BY created_at DESC | 极高 | - |
| 订单号查询 | WHERE order_no=? | 极高 | - |
| 后台按日期查 | WHERE created_at BETWEEN ? AND ? | 中等 | - |
| 后台按状态查 | WHERE status=? | 低 | - |

**索引方案**：
| 索引名 | 类型 | 字段 | 覆盖场景 |
|--------|------|------|---------|
| PRIMARY | 主键 | id | - |
| uk_order_no | 唯一 | order_no | 订单号查询 |
| idx_user_status_created | 普通 | (user_id, status, created_at) | 用户查订单 + 按状态筛选 |
| idx_created_at | 普通 | created_at | 后台按日期查 |

**说明**：
- `idx_user_status_created` 利用最左前缀同时覆盖 "按用户查" 和 "按用户+状态查"
- 后台按状态查频率低，不单独建索引，走 `idx_user_status_created` 或全表扫描
- 总计 4 个索引（含主键），在合理范围内
```

## 9. 索引类型选择

### 9.1 常用索引类型

| 类型 | 适用场景 | 语法 |
|------|---------|------|
| **主键索引** | 主键列 | `PRIMARY KEY (id)` |
| **唯一索引** | 业务唯一值（手机号、订单号） | `UNIQUE INDEX uk_xxx (col)` |
| **普通索引** | 高频查询条件 | `INDEX idx_xxx (col)` |
| **联合索引** | 多列组合查询 | `INDEX idx_xxx (col_a, col_b)` |
| **前缀索引** | 长字符串字段 | `INDEX idx_xxx (col(20))` |
| **全文索引** | 全文搜索（不推荐，用 ES） | `FULLTEXT INDEX` |

### 9.2 什么时候不建索引

| 场景 | 原因 |
|------|------|
| 数据量 < 1000 行 | 全表扫描就够快 |
| 频繁更新的字段 | 索引维护成本高 |
| 区分度 < 10% 的字段 | 如性别，索引效果差 |
| 很少用于查询条件的字段 | 浪费空间 |
| text / blob 类型 | 不能直接建，考虑前缀索引或全文索引 |

## 10. 索引失效场景

### 10.1 常见索引失效情况

AI 在做 SQL 审核或设计时，必须检查以下索引失效场景：

| 失效场景 | 反面示例 | 正确写法 |
|---------|---------|---------|
| **对索引列做函数运算** | `WHERE YEAR(created_at) = 2026` | `WHERE created_at >= '2026-01-01' AND created_at < '2027-01-01'` |
| **隐式类型转换** | `WHERE phone = 13800138000`（phone 是 varchar） | `WHERE phone = '13800138000'` |
| **前导模糊查询** | `WHERE name LIKE '%张'` | `WHERE name LIKE '张%'` |
| **OR 连接不同字段** | `WHERE user_id=1 OR phone='138'` | 改用 UNION |
| **NOT IN / NOT EXISTS** | `WHERE status NOT IN (3,4)` | `WHERE status IN (0,1,2)` |
| **索引列允许 NULL + IS NULL** | `WHERE deleted_at IS NULL` | 考虑 `is_deleted tinyint DEFAULT 0` 替代 |
| **联合索引跳列** | 索引 (A,B,C)，查 WHERE A=1 AND C=3 | B 列条件也要带上 |

### 10.2 EXPLAIN 结果解读

| 关键字段 | 好的值 | 差的值 |
|---------|--------|--------|
| `type` | const / eq_ref / ref / range | ALL（全表扫描） / index（全索引扫描） |
| `key` | 有值（命中了索引） | NULL（没命中索引） |
| `rows` | 尽量小 | 接近表总行数就是全表扫描 |
| `Extra` | Using index（覆盖索引） | Using filesort（额外排序）/ Using temporary（临时表） |

---

# 第四部分：SQL 规范

## 11. 查询规范

### 11.1 SELECT 规范

| 规则 | 说明 |
|------|------|
| **禁止 SELECT \*** | 明确列出需要的字段 |
| **大表必须分页** | `LIMIT offset, size`，offset 过大时用游标分页 |
| **禁止在循环中查 SQL** | N+1 问题，改为批量查询 |
| **COUNT 用法** | `COUNT(*)` 统计行数，`COUNT(col)` 统计非 NULL 行 |
| **避免大 IN 列表** | `IN` 的值不超过 1000 个，超过改用临时表或分批 |
| **EXISTS 替代 IN** | 子查询数据量大时 `EXISTS` 优于 `IN` |

### 11.2 分页优化

```sql
-- 普通分页（offset 大时非常慢）
SELECT * FROM t_order ORDER BY id LIMIT 100000, 20;

-- 优化方案一：游标分页（推荐）
SELECT * FROM t_order WHERE id > 100000 ORDER BY id LIMIT 20;

-- 优化方案二：延迟关联
SELECT t.* FROM t_order t
INNER JOIN (SELECT id FROM t_order ORDER BY id LIMIT 100000, 20) tmp
ON t.id = tmp.id;
```

### 11.3 JOIN 规范

| 规则 | 说明 |
|------|------|
| **JOIN 表不超过 3 张** | 超过 3 张考虑拆查询或冗余字段 |
| **小表驱动大表** | 小结果集的表做驱动表（放在 FROM 后面或 LEFT JOIN 左边） |
| **JOIN 条件必须有索引** | ON 子句中的字段必须有索引 |
| **避免笛卡尔积** | 每个 JOIN 必须有 ON 条件 |

## 12. 写入规范

### 12.1 INSERT 规范

```sql
-- 正确：明确指定字段
INSERT INTO t_user (phone, nickname, created_at) VALUES ('13800138000', '张三', NOW());

-- 错误：不指定字段
INSERT INTO t_user VALUES (NULL, '13800138000', '张三', ...);

-- 批量插入（单次不超过 1000 行）
INSERT INTO t_user (phone, nickname) VALUES
('138001', '张三'),
('138002', '李四'),
('138003', '王五');
```

### 12.2 UPDATE 规范

| 规则 | 说明 |
|------|------|
| **必须带 WHERE 条件** | 没有 WHERE 的 UPDATE 就是灾难 |
| **WHERE 条件必须命中索引** | 避免全表锁 |
| **禁止一次更新超过 1 万行** | 分批更新，避免长事务 |
| **避免更新主键** | 主键变更代价极大 |

### 12.3 DELETE 规范

| 规则 | 说明 |
|------|------|
| **业务数据用软删除** | `UPDATE SET deleted_at = NOW()`，不用 `DELETE` |
| **日志类数据可硬删除** | 定期清理过期日志 |
| **硬删除必须带 WHERE** | 没有 WHERE 的 DELETE 就是删库 |
| **大批量删除分批执行** | 每次删 1000 行，循环直到删完 |

## 13. 事务与锁

### 13.1 事务规范

| 规则 | 说明 |
|------|------|
| **事务要尽量短** | 减少锁持有时间 |
| **禁止在事务中调外部接口** | 网络超时会导致长事务 |
| **禁止大事务** | 单事务不超过 1000 行写入 |
| **合理选择隔离级别** | MySQL 默认 REPEATABLE READ，大多数场景够用 |

### 13.2 常见锁问题和解法

| 问题 | 场景 | 解法 |
|------|------|------|
| **死锁** | 两个事务互相等对方的锁 | 统一加锁顺序；重试机制 |
| **锁等待超时** | 大事务持锁时间过长 | 拆分大事务；加索引减少锁范围 |
| **超卖（库存扣减）** | 并发扣库存导致负数 | `UPDATE SET stock = stock - 1 WHERE stock > 0` |
| **重复数据** | 并发插入相同数据 | 唯一索引 + INSERT IGNORE / ON DUPLICATE KEY |

### 13.3 乐观锁 vs 悲观锁

| 方式 | 适用场景 | 实现 |
|------|---------|------|
| **乐观锁** | 冲突少，读多写少 | `version` 字段，`UPDATE WHERE version = ?` |
| **悲观锁** | 冲突多，必须保证一致性 | `SELECT ... FOR UPDATE` |

```sql
-- 乐观锁示例
UPDATE t_product
SET stock = stock - 1, version = version + 1
WHERE id = 100 AND version = 5;
-- 影响行数 = 0 表示被别人改了，需要重试

-- 悲观锁示例（谨慎使用）
BEGIN;
SELECT * FROM t_product WHERE id = 100 FOR UPDATE;
-- 这里其他事务会等待
UPDATE t_product SET stock = stock - 1 WHERE id = 100;
COMMIT;
```

---

# 第五部分：高阶设计

## 14. 范式与反范式决策

### 14.1 什么时候遵循范式

| 场景 | 建议 |
|------|------|
| 数据一致性要求高 | 遵循范式（如金融、订单） |
| 写多读少 | 遵循范式 |
| 数据量不大 | 遵循范式 |

### 14.2 什么时候反范式

| 场景 | 反范式方式 | 示例 |
|------|-----------|------|
| 读多写少，查询频繁需要 JOIN | 冗余字段 | 订单表冗余用户昵称 |
| 需要快速聚合统计 | 预计算字段 | 商品表冗余已售数量 |
| 历史快照 | 数据快照 | 订单表存收货地址快照（不存关联ID） |

### 14.3 反范式的代价

| 代价 | 处理方式 |
|------|---------|
| 数据不一致风险 | 用消息队列异步同步冗余字段 |
| 存储空间增大 | 空间换时间，可接受 |
| 更新变复杂 | 明确记录哪些字段是冗余的、更新策略是什么 |

## 15. 分库分表设计

### 15.1 什么时候需要分库分表

| 指标 | 建议动作 |
|------|---------|
| 单表 < 500 万行 | 不需要分表 |
| 单表 500 万 ~ 2000 万行 | 优化索引和 SQL，暂不分表 |
| 单表 > 2000 万行 | 考虑分表 |
| 单库 TPS > 5000 | 考虑分库 |
| 单库容量 > 500GB | 考虑分库 |

> **重要**：过早分库分表是万恶之源。大多数中小项目永远不需要分库分表。

### 15.2 分表策略

| 策略 | 适用场景 | 示例 |
|------|---------|------|
| **按范围分** | 数据有时间特征 | 订单表按月分：t_order_202601 |
| **按哈希分** | 数据均匀分布 | 用户表按 user_id % 16 分 |
| **按业务分** | 不同业务独立 | 订单库、商品库、用户库分开 |

### 15.3 分库分表带来的问题

| 问题 | 解决方案 |
|------|---------|
| 跨库 JOIN 不了 | 冗余字段或应用层组装 |
| 分布式事务 | 柔性事务（TCC / Saga / 消息事务） |
| 全局唯一 ID | 分布式 ID 生成器（Snowflake） |
| 跨表分页排序 | 归并排序或搜索引擎（ES） |
| 数据迁移困难 | 提前设计好分片键，尽量不改 |

## 16. 读写分离设计

### 16.1 适用场景

- 读写比 > 7:3
- 读请求延迟要求不高（可接受 1-2 秒延迟）
- 报表/搜索类查询压力大

### 16.2 架构模式

```
应用层
  ├── 写请求 → 主库 (Master)
  └── 读请求 → 从库 (Slave) × N
              ↑
         主从复制（异步/半同步）
```

### 16.3 注意事项

| 问题 | 说明 |
|------|------|
| **主从延迟** | 写入后立刻读可能读不到，关键业务强制读主库 |
| **刚写的数据** | "写完立刻读"的场景走主库 |
| **事务中的读** | 事务内所有读写都走主库 |

## 17. 缓存与数据库协同

### 17.1 缓存策略

| 策略 | 读流程 | 写流程 | 适用场景 |
|------|--------|--------|---------|
| **Cache Aside** | 先读缓存，miss 则读 DB 并回填 | 先更新 DB，再删缓存 | 最通用 |
| **Read Through** | 缓存组件自动回源 | 先更新 DB，再删缓存 | 缓存中间件支持 |
| **Write Behind** | 先读缓存 | 先写缓存，异步写 DB | 高写入场景，有丢失风险 |

### 17.2 缓存常见问题

| 问题 | 说明 | 解决方案 |
|------|------|---------|
| **缓存穿透** | 查不存在的数据，每次都打 DB | 布隆过滤器 / 缓存空值 |
| **缓存击穿** | 热点 key 过期瞬间大量请求打 DB | 互斥锁 / 热点 key 不过期 |
| **缓存雪崩** | 大量 key 同时过期 | 过期时间加随机值 |
| **数据不一致** | 缓存和 DB 数据不同步 | 延迟双删 / 订阅 binlog |

---

# 第六部分：典型业务库表设计

## 18. 用户体系设计

### 18.1 用户表

```sql
CREATE TABLE t_user (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT COMMENT '用户ID',
    phone       varchar(20)      NOT NULL DEFAULT '' COMMENT '手机号',
    nickname    varchar(50)      NOT NULL DEFAULT '' COMMENT '昵称',
    avatar_url  varchar(500)     NOT NULL DEFAULT '' COMMENT '头像URL',
    gender      tinyint          NOT NULL DEFAULT 0 COMMENT '性别：0未知 1男 2女',
    birthday    date             NULL     DEFAULT NULL COMMENT '生日',
    status      tinyint          NOT NULL DEFAULT 1 COMMENT '状态：0禁用 1正常',
    last_login_at datetime       NULL     DEFAULT NULL COMMENT '最后登录时间',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间',
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at  datetime         NULL     DEFAULT NULL COMMENT '注销时间',
    PRIMARY KEY (id),
    UNIQUE INDEX uk_phone (phone)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户表';
```

### 18.2 第三方登录表（支持多个平台）

```sql
CREATE TABLE t_user_oauth (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT COMMENT '主键',
    user_id     bigint unsigned  NOT NULL COMMENT '用户ID',
    platform    varchar(20)      NOT NULL COMMENT '平台：wechat/alipay/apple',
    open_id     varchar(128)     NOT NULL COMMENT '平台OpenID',
    union_id    varchar(128)     NOT NULL DEFAULT '' COMMENT '平台UnionID',
    access_token varchar(500)    NOT NULL DEFAULT '' COMMENT '访问令牌',
    expires_at  datetime         NULL     DEFAULT NULL COMMENT '令牌过期时间',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE INDEX uk_platform_openid (platform, open_id),
    INDEX idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='第三方登录表';
```

### 18.3 收货地址表

```sql
CREATE TABLE t_user_address (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    user_id     bigint unsigned  NOT NULL COMMENT '用户ID',
    name        varchar(50)      NOT NULL COMMENT '收货人姓名',
    phone       varchar(20)      NOT NULL COMMENT '收货人手机号',
    province    varchar(20)      NOT NULL COMMENT '省',
    city        varchar(20)      NOT NULL COMMENT '市',
    district    varchar(20)      NOT NULL COMMENT '区',
    address     varchar(200)     NOT NULL COMMENT '详细地址',
    is_default  tinyint(1)       NOT NULL DEFAULT 0 COMMENT '是否默认地址',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at  datetime         NULL     DEFAULT NULL,
    PRIMARY KEY (id),
    INDEX idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='收货地址表';
```

## 19. 商品与 SKU 设计

### 19.1 商品表（SPU）

```sql
CREATE TABLE t_product (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    name        varchar(200)     NOT NULL COMMENT '商品名称',
    subtitle    varchar(500)     NOT NULL DEFAULT '' COMMENT '副标题',
    category_id bigint unsigned  NOT NULL COMMENT '分类ID',
    brand_id    bigint unsigned  NOT NULL DEFAULT 0 COMMENT '品牌ID',
    main_image  varchar(500)     NOT NULL DEFAULT '' COMMENT '主图URL',
    images      json             NULL     COMMENT '图片列表JSON',
    description text             NULL     COMMENT '商品详情（富文本）',
    price_min   decimal(10,2)    NOT NULL DEFAULT 0 COMMENT '最低价（冗余）',
    price_max   decimal(10,2)    NOT NULL DEFAULT 0 COMMENT '最高价（冗余）',
    total_stock int unsigned     NOT NULL DEFAULT 0 COMMENT '总库存（冗余）',
    sales_count int unsigned     NOT NULL DEFAULT 0 COMMENT '已售数量（冗余）',
    status      tinyint          NOT NULL DEFAULT 0 COMMENT '状态：0下架 1上架 2审核中',
    sort_order  int              NOT NULL DEFAULT 0 COMMENT '排序值，越大越靠前',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at  datetime         NULL     DEFAULT NULL,
    PRIMARY KEY (id),
    INDEX idx_category (category_id),
    INDEX idx_status_sort (status, sort_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='商品表';
```

### 19.2 SKU 表

```sql
CREATE TABLE t_product_sku (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    product_id  bigint unsigned  NOT NULL COMMENT '商品ID',
    sku_name    varchar(200)     NOT NULL COMMENT 'SKU名称，如"红色 XL"',
    attrs       json             NOT NULL COMMENT '规格属性JSON，如{"颜色":"红","尺码":"XL"}',
    price       decimal(10,2)    NOT NULL COMMENT '售价',
    original_price decimal(10,2) NOT NULL DEFAULT 0 COMMENT '原价（划线价）',
    stock       int unsigned     NOT NULL DEFAULT 0 COMMENT '库存',
    sku_code    varchar(50)      NOT NULL DEFAULT '' COMMENT 'SKU编码',
    image       varchar(500)     NOT NULL DEFAULT '' COMMENT 'SKU图片',
    status      tinyint          NOT NULL DEFAULT 1 COMMENT '状态：0禁用 1启用',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    INDEX idx_product_id (product_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='商品SKU表';
```

### 19.3 分类表（无限级）

```sql
CREATE TABLE t_category (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    parent_id   bigint unsigned  NOT NULL DEFAULT 0 COMMENT '父分类ID，0为顶级',
    name        varchar(50)      NOT NULL COMMENT '分类名称',
    icon        varchar(500)     NOT NULL DEFAULT '' COMMENT '分类图标',
    sort_order  int              NOT NULL DEFAULT 0 COMMENT '排序',
    level       tinyint          NOT NULL DEFAULT 1 COMMENT '层级：1/2/3',
    status      tinyint          NOT NULL DEFAULT 1 COMMENT '状态：0隐藏 1显示',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    INDEX idx_parent (parent_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='商品分类表';
```

## 20. 订单与支付设计

### 20.1 订单主表

```sql
CREATE TABLE t_order (
    id              bigint unsigned  NOT NULL AUTO_INCREMENT,
    order_no        varchar(32)      NOT NULL COMMENT '订单编号',
    user_id         bigint unsigned  NOT NULL COMMENT '买家ID',
    total_amount    decimal(10,2)    NOT NULL COMMENT '商品总额',
    freight_amount  decimal(10,2)    NOT NULL DEFAULT 0 COMMENT '运费',
    discount_amount decimal(10,2)    NOT NULL DEFAULT 0 COMMENT '优惠金额',
    pay_amount      decimal(10,2)    NOT NULL COMMENT '实付金额',
    status          tinyint          NOT NULL DEFAULT 0 COMMENT '0待支付 1已支付 2已发货 3已完成 4已取消 5退款中 6已退款',
    pay_method      tinyint          NOT NULL DEFAULT 0 COMMENT '支付方式：0未支付 1微信 2支付宝',
    address_snapshot json            NOT NULL COMMENT '收货地址快照',
    remark          varchar(500)     NOT NULL DEFAULT '' COMMENT '买家备注',
    pay_time        datetime         NULL COMMENT '支付时间',
    deliver_time    datetime         NULL COMMENT '发货时间',
    receive_time    datetime         NULL COMMENT '收货时间',
    cancel_time     datetime         NULL COMMENT '取消时间',
    cancel_reason   varchar(200)     NOT NULL DEFAULT '' COMMENT '取消原因',
    created_at      datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at      datetime         NULL     DEFAULT NULL,
    PRIMARY KEY (id),
    UNIQUE INDEX uk_order_no (order_no),
    INDEX idx_user_status (user_id, status),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单表';
```

### 20.2 订单项表

```sql
CREATE TABLE t_order_item (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    order_id    bigint unsigned  NOT NULL COMMENT '订单ID',
    product_id  bigint unsigned  NOT NULL COMMENT '商品ID',
    sku_id      bigint unsigned  NOT NULL COMMENT 'SKU ID',
    product_name varchar(200)    NOT NULL COMMENT '商品名称（快照）',
    sku_name    varchar(200)     NOT NULL DEFAULT '' COMMENT 'SKU名称（快照）',
    image       varchar(500)     NOT NULL DEFAULT '' COMMENT '商品图片（快照）',
    price       decimal(10,2)    NOT NULL COMMENT '单价（快照）',
    quantity    int unsigned     NOT NULL COMMENT '购买数量',
    subtotal    decimal(10,2)    NOT NULL COMMENT '小计金额',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    INDEX idx_order_id (order_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单项表';
```

### 20.3 支付记录表

```sql
CREATE TABLE t_payment (
    id              bigint unsigned  NOT NULL AUTO_INCREMENT,
    payment_no      varchar(32)      NOT NULL COMMENT '支付流水号',
    order_no        varchar(32)      NOT NULL COMMENT '订单编号',
    user_id         bigint unsigned  NOT NULL COMMENT '用户ID',
    amount          decimal(10,2)    NOT NULL COMMENT '支付金额',
    method          tinyint          NOT NULL COMMENT '支付方式：1微信 2支付宝',
    status          tinyint          NOT NULL DEFAULT 0 COMMENT '0待支付 1支付成功 2支付失败 3已退款',
    trade_no        varchar(64)      NOT NULL DEFAULT '' COMMENT '第三方交易号',
    pay_time        datetime         NULL COMMENT '支付成功时间',
    callback_raw    text             NULL COMMENT '回调原始数据',
    created_at      datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE INDEX uk_payment_no (payment_no),
    INDEX idx_order_no (order_no)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='支付记录表';
```

## 21. 权限体系设计（RBAC）

### 21.1 核心表结构

```sql
-- 角色表
CREATE TABLE t_role (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    code        varchar(50)      NOT NULL COMMENT '角色编码，如 admin/editor',
    name        varchar(50)      NOT NULL COMMENT '角色名称',
    description varchar(200)     NOT NULL DEFAULT '' COMMENT '角色描述',
    status      tinyint          NOT NULL DEFAULT 1 COMMENT '0禁用 1启用',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE INDEX uk_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='角色表';

-- 权限表
CREATE TABLE t_permission (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    parent_id   bigint unsigned  NOT NULL DEFAULT 0 COMMENT '父权限ID',
    code        varchar(100)     NOT NULL COMMENT '权限编码，如 user:create',
    name        varchar(50)      NOT NULL COMMENT '权限名称',
    type        tinyint          NOT NULL COMMENT '类型：1菜单 2按钮 3接口',
    path        varchar(200)     NOT NULL DEFAULT '' COMMENT '前端路由/接口路径',
    sort_order  int              NOT NULL DEFAULT 0,
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE INDEX uk_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='权限表';

-- 用户-角色关联表
CREATE TABLE t_user_role (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    user_id     bigint unsigned  NOT NULL,
    role_id     bigint unsigned  NOT NULL,
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE INDEX uk_user_role (user_id, role_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户角色关联表';

-- 角色-权限关联表
CREATE TABLE t_role_permission (
    id            bigint unsigned  NOT NULL AUTO_INCREMENT,
    role_id       bigint unsigned  NOT NULL,
    permission_id bigint unsigned  NOT NULL,
    created_at    datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE INDEX uk_role_perm (role_id, permission_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='角色权限关联表';
```

### 21.2 RBAC 查询模式

```sql
-- 查询用户所有权限
SELECT DISTINCT p.code
FROM t_user_role ur
INNER JOIN t_role_permission rp ON ur.role_id = rp.role_id
INNER JOIN t_permission p ON rp.permission_id = p.id
INNER JOIN t_role r ON ur.role_id = r.id
WHERE ur.user_id = ? AND r.status = 1;
```

## 22. 内容与审核设计

### 22.1 内容表（通用）

```sql
CREATE TABLE t_content (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    user_id     bigint unsigned  NOT NULL COMMENT '作者ID',
    type        tinyint          NOT NULL COMMENT '内容类型：1文章 2帖子 3评论',
    title       varchar(200)     NOT NULL DEFAULT '' COMMENT '标题',
    body        text             NOT NULL COMMENT '正文内容',
    images      json             NULL     COMMENT '图片列表',
    status      tinyint          NOT NULL DEFAULT 0 COMMENT '0草稿 1待审核 2已发布 3已拒绝 4已下架',
    view_count  int unsigned     NOT NULL DEFAULT 0 COMMENT '浏览数（冗余）',
    like_count  int unsigned     NOT NULL DEFAULT 0 COMMENT '点赞数（冗余）',
    reply_count int unsigned     NOT NULL DEFAULT 0 COMMENT '评论数（冗余）',
    audit_remark varchar(200)    NOT NULL DEFAULT '' COMMENT '审核备注',
    published_at datetime        NULL     COMMENT '发布时间',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at  datetime         NULL     DEFAULT NULL,
    PRIMARY KEY (id),
    INDEX idx_user_id (user_id),
    INDEX idx_status_published (status, published_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='内容表';
```

### 22.2 审核记录表

```sql
CREATE TABLE t_audit_log (
    id          bigint unsigned  NOT NULL AUTO_INCREMENT,
    target_type varchar(20)      NOT NULL COMMENT '审核对象类型：content/product/user',
    target_id   bigint unsigned  NOT NULL COMMENT '审核对象ID',
    action      tinyint          NOT NULL COMMENT '操作：1通过 2拒绝 3下架',
    operator_id bigint unsigned  NOT NULL COMMENT '操作人ID',
    remark      varchar(500)     NOT NULL DEFAULT '' COMMENT '审核备注',
    created_at  datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    INDEX idx_target (target_type, target_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='审核记录表';
```

---

# 附录

## 附录 A. MySQL 字段类型速查表

### 整数类型

| 类型 | 字节 | 有符号范围 | 无符号范围 | 适用场景 |
|------|------|-----------|-----------|---------|
| tinyint | 1 | -128~127 | 0~255 | 状态、类型、布尔 |
| smallint | 2 | -32768~32767 | 0~65535 | 较小计数 |
| int | 4 | -21亿~21亿 | 0~42亿 | 常规整数 |
| bigint | 8 | 极大 | 极大 | 主键、ID |

### 字符串类型

| 类型 | 最大长度 | 适用场景 |
|------|---------|---------|
| varchar(N) | 65535 字节 | 变长字符串，N ≤ 5000 |
| text | 65535 字节 | 长文本 |
| mediumtext | 16MB | 富文本/HTML |
| longtext | 4GB | 极长文本（很少用） |
| json | - | 结构化 JSON 数据 |

### 时间类型

| 类型 | 范围 | 精度 | 适用场景 |
|------|------|------|---------|
| datetime | 1000-01-01 ~ 9999-12-31 | 秒/微秒 | 大多数时间字段（推荐） |
| timestamp | 1970-01-01 ~ 2038-01-19 | 秒/微秒 | 有 2038 问题，不推荐 |
| date | 1000-01-01 ~ 9999-12-31 | 天 | 只需要日期 |
| time | -838:59:59 ~ 838:59:59 | 秒 | 只需要时间 |

### 精确数值

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| decimal(M,D) | M 位总长，D 位小数 | 金额（必须用这个） |

## 附录 B. 建表语句模板

```sql
-- =============================================
-- 标准建表模板
-- =============================================
CREATE TABLE t_table_name (
    -- 主键
    id              bigint unsigned  NOT NULL AUTO_INCREMENT COMMENT '主键',

    -- 业务字段
    name            varchar(100)     NOT NULL DEFAULT '' COMMENT '名称',
    status          tinyint          NOT NULL DEFAULT 0 COMMENT '状态：0xxx 1xxx',
    amount          decimal(10,2)    NOT NULL DEFAULT 0 COMMENT '金额',
    description     text             NULL     COMMENT '描述',
    extra_data      json             NULL     COMMENT 'JSON扩展数据',

    -- 通用字段
    created_at      datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at      datetime         NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    deleted_at      datetime         NULL     DEFAULT NULL COMMENT '软删除时间',

    -- 主键和索引
    PRIMARY KEY (id),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='表说明';
```

## 附录 C. 常见性能问题自查清单

AI 在完成数据库设计后，必须逐项检查：

### 建表检查
- [ ] 每张表都有主键？
- [ ] 用了 `utf8mb4` 字符集？（不是 `utf8`，后者不支持 emoji）
- [ ] 金额字段用 `decimal` 不是 `float/double`？
- [ ] 有 `created_at` 和 `updated_at`？
- [ ] 字段尽量 `NOT NULL`？
- [ ] 没有使用 MySQL 保留字做表名/字段名？
- [ ] 单表字段数不超过 30 个？
- [ ] 没有存大文件（图片/文件应存 OSS）？

### 索引检查
- [ ] 每个高频查询都有对应索引？
- [ ] 单表索引不超过 5 个？
- [ ] 联合索引的字段顺序正确（高区分度在前）？
- [ ] 没有重复索引？（如已有 (A,B) 就不需要单独的 (A)）
- [ ] 外键关联字段都有索引？

### SQL 检查
- [ ] 没有 `SELECT *`？
- [ ] 大表查询都有 `LIMIT`？
- [ ] 没有在循环中执行 SQL（N+1 问题）？
- [ ] JOIN 不超过 3 张表？
- [ ] WHERE 条件都能命中索引？
- [ ] 没有对索引列做函数运算？

### 数据安全检查
- [ ] 密码用哈希存储？
- [ ] 手机号/身份证等敏感信息有脱敏方案？
- [ ] 业务数据用软删除？
- [ ] 有定期备份策略？

## 附录 D. AI 数据库设计 Prompt 模板

### Prompt 1：从需求设计数据库

```
请根据以下产品需求，按照数据库设计流程完成数据库设计：
1. 先识别核心实体和关系，输出 ER 图
2. 设计每张表的结构（字段、类型、索引）
3. 给出建表 SQL
4. 预估数据量并评估是否需要分表

产品需求：
[粘贴需求或 PRD]
```

### Prompt 2：审核现有表结构

```
请审核以下建表语句，检查：
1. 命名是否规范
2. 字段类型是否合理
3. 索引设计是否合理
4. 有没有性能隐患
5. 有没有遗漏字段

建表语句：
[粘贴 SQL]
```

### Prompt 3：SQL 优化

```
请分析以下 SQL 的执行效率，给出优化建议。
当前表结构和数据量：[说明]

SQL：
[粘贴 SQL]
```

### Prompt 4：分库分表方案

```
当前业务数据量如下：
[粘贴数据量信息]

请评估是否需要分库分表，如果需要请给出：
1. 分片键选择
2. 分片策略
3. 数据迁移方案
4. 需要解决的问题（跨库查询、分布式事务等）
```

---

> **本手册持续更新中。** 如果你觉得有用，请给 [AI 黑魔法](https://github.com/lvcn/ai-black-magic) 一个 ⭐ Star！
