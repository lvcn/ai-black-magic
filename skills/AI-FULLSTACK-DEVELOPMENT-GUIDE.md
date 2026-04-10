# AI 全栈开发实战手册

> **定位**：本文件是 AI 编码助手的系统级指令文件。AI 在开发任何项目前必须完整阅读本文件，并严格遵循其中的规范、流程和约束。
>
> 本手册参考《阿里巴巴 Java 开发手册》《Google 前端规范》及国内一线互联网公司工程实践，针对中国市场的商业化 Web/移动应用场景编写。
>
> **目标读者**：通过 AI 辅助开发的非专业开发者。AI 阅读后应具备资深全栈工程师的决策能力。

---

## 目录

**第一部分：项目启动（想清楚再动手）**
1. [需求分析与方案设计](#1-需求分析与方案设计)
2. [技术选型决策树](#2-技术选型决策树)
3. [项目初始化标准流程](#3-项目初始化标准流程)

**第二部分：架构设计（地基决定上限）**
4. [前端架构设计](#4-前端架构设计)
5. [后端架构设计](#5-后端架构设计)
6. [数据库设计](#6-数据库设计)
7. [API 设计](#7-api-设计)

**第三部分：编码规范（代码是给人读的）**
8. [前端编码规范](#8-前端编码规范)
9. [后端编码规范](#9-后端编码规范)
10. [CSS/样式规范](#10-css样式规范)

**第四部分：中国市场专项（做中国人用的产品）**
11. [UI/UX 中国化设计](#11-uiux-中国化设计)
12. [支付集成](#12-支付集成)
13. [微信生态开发](#13-微信生态开发)
14. [合规与备案](#14-合规与备案)

**第五部分：质量保障（上线前的最后防线）**
15. [安全规范](#15-安全规范)
16. [性能优化](#16-性能优化)
17. [测试规范](#17-测试规范)

**第六部分：交付上线（最后一公里）**
18. [部署与运维](#18-部署与运维)
19. [监控与告警](#19-监控与告警)
20. [Git 工作流与版本管理](#20-git-工作流与版本管理)
21. [项目交付清单](#21-项目交付清单)

**附录**
- [A. 常用技术栈版本速查表](#附录-a-常用技术栈版本速查表)
- [B. 中国主流云服务商对比](#附录-b-中国主流云服务商对比)
- [C. 项目自测 Checklist（逐项打勾）](#附录-c-项目自测-checklist)
- [D. AI 开发工作流模板](#附录-d-ai-开发工作流模板)

---

# 第一部分：项目启动

## 1. 需求分析与方案设计

### 1.1 黄金法则：先设计再编码

**任何项目必须按以下顺序执行，不可跳步。违反此流程是项目失败的首要原因。**

```
阶段一：需求理解 ──→ 阶段二：方案设计 ──→ 阶段三：技术选型
     │                    │                    │
     ▼                    ▼                    ▼
阶段四：原型确认 ──→ 阶段五：开发实施 ──→ 阶段六：测试上线
```

### 1.2 需求分析模板

在动手写任何代码之前，AI 必须先输出以下分析文档：

```markdown
## 需求分析报告

### 1. 项目概述
- 项目名称：
- 一句话描述：
- 目标用户：[C 端消费者 / B 端企业用户 / 内部系统]
- 预期用户量级：[百级 / 千级 / 万级 / 十万级以上]
- 核心业务流程：[用一段话描述主干链路]

### 2. 功能清单（按优先级排序）
| 优先级 | 功能模块 | 功能描述 | 涉及角色 | 复杂度 |
|--------|---------|---------|---------|--------|
| P0（必须）| | | | |
| P1（重要）| | | | |
| P2（nice to have）| | | | |

### 3. 非功能需求
- 并发量要求：
- 响应时间要求：
- 数据存储量预估：
- 是否需要实时通信：
- 是否需要支付：
- 是否涉及敏感数据（身份证、手机号等）：

### 4. 约束条件
- 预算范围：
- 上线时间：
- 团队技术栈偏好：
- 部署环境限制：
```

### 1.3 五问法——避免遗漏

在方案设计前，对每个功能模块问自己五个问题：

1. **谁用？** 用户角色是什么？有几种角色？权限怎么划分？
2. **怎么用？** 用户的完整操作路径是什么？从进入到离开经历几步？
3. **异常呢？** 如果网络断了、数据错了、用户乱填了，怎么处理？
4. **量多大？** 数据量、并发量分别是多少？需要分页吗？需要缓存吗？
5. **能赚钱吗？** 这个功能对商业目标的贡献是什么？有没有比这更重要的？

---

## 2. 技术选型决策树

### 2.1 选型原则

| 原则 | 说明 | 反面教材 |
|------|------|---------|
| **够用就好** | 不追求最新最热，选择生态最成熟的方案 | 用 Rust 写博客系统 |
| **生态优先** | 中文文档多、社区活跃、有大厂背书 | 用冷门框架找不到轮子 |
| **团队能力匹配** | AI 辅助开发优先选 AI 训练语料最多的技术栈 | 用小众语言指望 AI 帮你写 |
| **运维友好** | 优先选择国内云服务商有托管方案的 | 自建 K8s 集群维护不起 |
| **成本可控** | MVP 阶段用免费/低成本方案验证 | 一上来就买企业版数据库 |

### 2.2 前端技术选型

```
项目类型是什么？
│
├─ 管理后台（B端）
│   └─→ Vue 3 + Vite + Element Plus + Pinia + TypeScript
│       备选：React + Ant Design Pro（团队偏好 React 时）
│
├─ 面向消费者的 Web 应用（C端）
│   └─→ Vue 3 + Vite + Vant 4（移动优先）
│       或 React + Next.js（需要 SEO 时）
│
├─ 微信小程序
│   └─→ 原生小程序 + Vant Weapp
│       或 uni-app（需要多端时）
│
├─ 支付宝小程序
│   └─→ 支付宝原生 + antd-mini
│
├─ H5 页面（嵌入 App/公众号）
│   └─→ Vue 3 + Vite + Vant 4
│
├─ 桌面应用
│   └─→ Electron + Vue 3
│
└─ 官网/落地页
    └─→ Nuxt 3（需要 SEO）或 Vue 3 + Vite（纯 SPA）
```

### 2.3 后端技术选型

```
团队规模和项目复杂度？
│
├─ 个人/小团队 + 中小项目
│   └─→ Node.js (Express/Koa/NestJS) + MySQL + Redis
│       优点：前后端同语言，AI 生成效率高，部署简单
│
├─ 中型项目（3-10人团队）
│   ├─ 偏好 Java
│   │   └─→ Spring Boot 2.7/3.x + MyBatis-Plus + MySQL + Redis
│   └─ 偏好 Go
│       └─→ Gin/Hertz + GORM + MySQL + Redis
│
├─ 大型项目（微服务）
│   └─→ Spring Cloud Alibaba + Nacos + Gateway + Sentinel
│       或 Go 微服务 + gRPC + Consul/Nacos
│
└─ Serverless / 快速验证
    └─→ 云函数（腾讯云 SCF / 阿里云 FC）+ 云数据库
```

### 2.4 数据库选型

| 场景 | 推荐方案 | 说明 |
|------|---------|------|
| 常规业务数据 | **MySQL 8.0** | 国内生态最好，云服务商全支持 |
| 缓存/会话/排行榜 | **Redis 6/7** | 几乎所有项目都需要 |
| 搜索/日志分析 | **Elasticsearch 7/8** | 全文搜索场景 |
| 文件存储 | **OSS / COS** | 用云存储，不要存本地 |
| 文档型数据 | **MongoDB** | 非结构化数据多时使用 |
| 消息队列 | **RabbitMQ / RocketMQ** | 异步解耦、削峰填谷 |

---

## 3. 项目初始化标准流程

### 3.1 前端项目初始化（以 Vue 3 + Vite 为例）

```bash
# 1. 创建项目
npm create vite@latest my-project -- --template vue-ts

# 2. 安装核心依赖
cd my-project
npm install vue-router@4 pinia axios
npm install -D sass unplugin-auto-import unplugin-vue-components

# B 端后台额外安装
npm install element-plus @element-plus/icons-vue

# C 端移动优先额外安装
npm install vant @vant/use
npm install -D @vant/auto-import-resolver

# 3. 安装代码质量工具
npm install -D eslint prettier eslint-plugin-vue @typescript-eslint/eslint-plugin
```

### 3.2 标准前端目录结构

```
src/
├── api/                    # 接口请求模块（按业务域拆分）
│   ├── modules/            # 各模块的 API 定义
│   │   ├── user.ts         # 用户相关接口
│   │   └── order.ts        # 订单相关接口
│   └── request.ts          # axios 封装（拦截器、错误处理）
├── assets/                 # 静态资源
│   ├── images/
│   └── styles/
│       ├── variables.scss  # 全局变量（颜色、字号、间距）
│       ├── mixins.scss     # 通用 mixin
│       └── global.scss     # 全局样式重置
├── components/             # 公共组件
│   ├── common/             # 通用基础组件（按钮、弹窗等）
│   └── business/           # 通用业务组件（用户选择器等）
├── composables/            # 组合式函数（hooks）
│   ├── useAuth.ts          # 认证相关
│   └── useLoading.ts       # 加载状态
├── layouts/                # 布局组件
│   ├── DefaultLayout.vue   # 默认布局
│   └── BlankLayout.vue     # 空白布局
├── router/                 # 路由
│   ├── index.ts
│   ├── routes.ts           # 路由配置
│   └── guard.ts            # 路由守卫
├── stores/                 # 状态管理（Pinia）
│   ├── modules/
│   │   ├── user.ts
│   │   └── app.ts
│   └── index.ts
├── types/                  # TypeScript 类型定义
│   ├── api.d.ts            # 接口类型
│   └── global.d.ts         # 全局类型
├── utils/                  # 工具函数
│   ├── auth.ts             # Token 管理
│   ├── storage.ts          # 本地存储封装
│   ├── validate.ts         # 表单校验规则
│   └── format.ts           # 格式化工具（日期、金额等）
├── views/                  # 页面组件（按业务模块分目录）
│   ├── login/
│   │   └── index.vue
│   ├── dashboard/
│   │   └── index.vue
│   └── user/
│       ├── list.vue
│       └── detail.vue
├── App.vue
├── main.ts
└── env.d.ts
```

### 3.3 后端项目初始化

#### Node.js (NestJS) 方案

```bash
# 1. 创建项目
npx @nestjs/cli new my-backend

# 2. 安装核心依赖
npm install @nestjs/typeorm typeorm mysql2
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install class-validator class-transformer
npm install ioredis @nestjs/cache-manager cache-manager
npm install helmet cors
```

#### Java (Spring Boot) 方案

```xml
<!-- 核心 starter -->
<dependencies>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-web</artifactId></dependency>
    <dependency><groupId>com.baomidou</groupId><artifactId>mybatis-plus-boot-starter</artifactId></dependency>
    <dependency><groupId>mysql</groupId><artifactId>mysql-connector-java</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-data-redis</artifactId></dependency>
    <dependency><groupId>org.springframework.boot</groupId><artifactId>spring-boot-starter-validation</artifactId></dependency>
    <dependency><groupId>org.projectlombok</groupId><artifactId>lombok</artifactId></dependency>
    <dependency><groupId>cn.hutool</groupId><artifactId>hutool-all</artifactId></dependency>
    <dependency><groupId>com.github.xiaoymin</groupId><artifactId>knife4j-openapi3-spring-boot-starter</artifactId></dependency>
</dependencies>
```

### 3.4 标准后端目录结构（单体应用）

```
src/main/java/com/example/project/
├── config/                # 配置类
│   ├── WebConfig.java     # Web 配置（CORS、拦截器等）
│   ├── RedisConfig.java   # Redis 配置
│   ├── SecurityConfig.java# 安全配置
│   └── SwaggerConfig.java # API 文档配置
├── common/                # 公共模块
│   ├── result/            # 统一返回值
│   │   ├── Result.java    # 统一响应封装
│   │   └── ResultCode.java# 响应码枚举
│   ├── exception/         # 异常处理
│   │   ├── BusinessException.java    # 业务异常
│   │   └── GlobalExceptionHandler.java# 全局异常处理器
│   ├── constant/          # 常量
│   └── util/              # 工具类
├── modules/               # 业务模块（按功能域划分）
│   ├── user/              # 用户模块
│   │   ├── controller/
│   │   ├── service/
│   │   │   ├── UserService.java
│   │   │   └── impl/UserServiceImpl.java
│   │   ├── mapper/
│   │   ├── entity/
│   │   ├── dto/
│   │   └── vo/
│   └── order/             # 订单模块
│       └── ...
└── ProjectApplication.java
```

### 3.5 初始化必做清单

无论什么项目，初始化时必须完成以下配置：

- [ ] **环境变量管理**：`.env.development`、`.env.production` 分离
- [ ] **统一响应格式**：所有接口返回值统一 `{ code, message, data }` 结构
- [ ] **全局异常处理**：前端 axios 拦截器 + 后端全局异常处理器
- [ ] **日志配置**：后端日志分级输出，前端错误上报
- [ ] **CORS 配置**：后端正确配置跨域允许规则
- [ ] **代码格式化**：ESLint + Prettier（前端）、代码风格配置（后端）
- [ ] **Git 配置**：`.gitignore` 排除 `node_modules/`、`.env`、`dist/`、`target/` 等

---

# 第二部分：架构设计

## 4. 前端架构设计

### 4.1 路由设计

#### 路由配置规范

```typescript
// router/routes.ts
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/login/index.vue'),
    meta: { title: '登录', requireAuth: false }
  },
  {
    path: '/',
    component: () => import('@/layouts/DefaultLayout.vue'),
    redirect: '/dashboard',
    children: [
      {
        path: 'dashboard',
        name: 'Dashboard',
        component: () => import('@/views/dashboard/index.vue'),
        meta: { title: '首页', icon: 'home', requireAuth: true }
      }
    ]
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: () => import('@/views/error/404.vue'),
    meta: { title: '404' }
  }
]
```

#### 路由守卫标准实现

```typescript
// router/guard.ts
import router from './index'
import { useUserStore } from '@/stores/modules/user'
import { getToken } from '@/utils/auth'

const whiteList = ['/login', '/register', '/forget-password']

router.beforeEach(async (to, from, next) => {
  document.title = `${to.meta.title || ''} - ${import.meta.env.VITE_APP_TITLE}`

  const token = getToken()

  if (token) {
    if (to.path === '/login') {
      next({ path: '/' })
    } else {
      const userStore = useUserStore()
      if (userStore.roles.length === 0) {
        try {
          await userStore.getUserInfo()
          next({ ...to, replace: true })
        } catch {
          userStore.logout()
          next(`/login?redirect=${to.path}`)
        }
      } else {
        next()
      }
    }
  } else {
    if (whiteList.includes(to.path)) {
      next()
    } else {
      next(`/login?redirect=${to.path}`)
    }
  }
})
```

### 4.2 HTTP 请求封装

```typescript
// api/request.ts
import axios, { type AxiosRequestConfig, type AxiosResponse } from 'axios'
import { getToken, removeToken } from '@/utils/auth'
import { showToast } from '@/utils/message'
import router from '@/router'

const service = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 15000,
  headers: { 'Content-Type': 'application/json' }
})

service.interceptors.request.use(
  (config) => {
    const token = getToken()
    if (token) {
      config.headers.Authorization = `Bearer ${token}`
    }
    return config
  },
  (error) => Promise.reject(error)
)

service.interceptors.response.use(
  (response: AxiosResponse) => {
    const { code, message, data } = response.data

    if (code === 200) {
      return data
    }

    // 登录过期
    if (code === 401) {
      removeToken()
      router.push('/login')
      return Promise.reject(new Error('登录已过期，请重新登录'))
    }

    showToast(message || '请求失败')
    return Promise.reject(new Error(message))
  },
  (error) => {
    if (error.response) {
      const { status } = error.response
      const messageMap: Record<number, string> = {
        400: '请求参数错误',
        401: '未授权，请重新登录',
        403: '拒绝访问',
        404: '请求的资源不存在',
        500: '服务器内部错误',
        502: '网关错误',
        503: '服务不可用'
      }
      showToast(messageMap[status] || `请求失败(${status})`)
    } else if (error.code === 'ECONNABORTED') {
      showToast('请求超时，请稍后重试')
    } else {
      showToast('网络连接异常，请检查网络')
    }
    return Promise.reject(error)
  }
)

export default service
```

### 4.3 状态管理设计

```typescript
// stores/modules/user.ts
import { defineStore } from 'pinia'
import { loginApi, getUserInfoApi, logoutApi } from '@/api/modules/user'
import { getToken, setToken, removeToken } from '@/utils/auth'

interface UserState {
  token: string
  userInfo: UserInfo | null
  roles: string[]
  permissions: string[]
}

export const useUserStore = defineStore('user', {
  state: (): UserState => ({
    token: getToken() || '',
    userInfo: null,
    roles: [],
    permissions: []
  }),

  actions: {
    async login(loginForm: LoginForm) {
      const { token } = await loginApi(loginForm)
      this.token = token
      setToken(token)
    },

    async getUserInfo() {
      const data = await getUserInfoApi()
      this.userInfo = data
      this.roles = data.roles
      this.permissions = data.permissions
    },

    async logout() {
      try {
        await logoutApi()
      } finally {
        this.token = ''
        this.userInfo = null
        this.roles = []
        this.permissions = []
        removeToken()
      }
    }
  }
})
```

### 4.4 前端分层原则

```
┌─────────────────────────────────────────────┐
│  View 层（.vue 页面）                         │
│  职责：页面布局、用户交互、调用 Store/API       │
│  规则：不写业务逻辑，只做 "胶水" 和 "展示"      │
├─────────────────────────────────────────────┤
│  Composable 层（hooks/组合式函数）             │
│  职责：可复用的业务逻辑                        │
│  规则：一个 hook 只做一件事                    │
├─────────────────────────────────────────────┤
│  Store 层（Pinia）                           │
│  职责：全局状态管理                            │
│  规则：只存需要跨组件/页面共享的状态             │
├─────────────────────────────────────────────┤
│  API 层                                      │
│  职责：HTTP 请求定义                           │
│  规则：一个模块一个文件，函数名与接口语义对应      │
├─────────────────────────────────────────────┤
│  Utils 层                                    │
│  职责：纯工具函数                              │
│  规则：无副作用、可单元测试                     │
└─────────────────────────────────────────────┘
```

---

## 5. 后端架构设计

### 5.1 分层架构（强制）

无论使用什么语言和框架，后端必须严格分层：

```
┌──────────────────────────────────────────────┐
│  Controller 层（接口入口）                      │
│  职责：参数校验、调用 Service、返回结果            │
│  禁止：不写业务逻辑，不直接操作数据库              │
├──────────────────────────────────────────────┤
│  Service 层（业务逻辑）                         │
│  职责：核心业务逻辑、事务管理、调用 DAO            │
│  规则：接口 + 实现类分离                         │
├──────────────────────────────────────────────┤
│  DAO / Mapper / Repository 层（数据访问）        │
│  职责：数据库 CRUD                              │
│  规则：只做数据操作，不含业务判断                  │
├──────────────────────────────────────────────┤
│  Entity / Model 层（数据模型）                   │
│  职责：映射数据库表结构                           │
├──────────────────────────────────────────────┤
│  DTO / VO 层（数据传输）                         │
│  职责：接口入参(DTO)、出参(VO)的定义              │
│  规则：禁止直接将 Entity 暴露给前端               │
└──────────────────────────────────────────────┘
```

### 5.2 统一响应格式

所有接口必须返回统一的 JSON 结构：

```json
{
  "code": 200,
  "message": "success",
  "data": {},
  "timestamp": 1712736000000
}
```

**状态码约定**：

| code | 含义 | 说明 |
|------|------|------|
| 200 | 成功 | |
| 400 | 参数错误 | 前端传参有误 |
| 401 | 未认证 | Token 过期或缺失 |
| 403 | 无权限 | 角色/权限不足 |
| 404 | 资源不存在 | |
| 500 | 服务器内部错误 | 代码 bug 或依赖服务异常 |
| 10xx | 业务错误 | 各业务模块自定义错误码 |

**Java 实现**：

```java
@Data
public class Result<T> {
    private int code;
    private String message;
    private T data;
    private long timestamp;

    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        result.setTimestamp(System.currentTimeMillis());
        return result;
    }

    public static <T> Result<T> error(int code, String message) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        result.setTimestamp(System.currentTimeMillis());
        return result;
    }
}
```

**Node.js 实现**：

```typescript
// 统一响应封装
export class Result<T = any> {
  code: number
  message: string
  data: T | null
  timestamp: number

  static success<T>(data: T, message = 'success'): Result<T> {
    return { code: 200, message, data, timestamp: Date.now() }
  }

  static error(code: number, message: string): Result<null> {
    return { code, message, data: null, timestamp: Date.now() }
  }
}
```

### 5.3 全局异常处理（强制）

**Java（Spring Boot）**：

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public Result<?> handleBusinessException(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return Result.error(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result<?> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.joining(", "));
        return Result.error(400, message);
    }

    @ExceptionHandler(Exception.class)
    public Result<?> handleException(Exception e) {
        log.error("系统异常", e);
        return Result.error(500, "系统繁忙，请稍后重试");
    }
}
```

### 5.4 认证与授权

#### JWT 认证流程

```
客户端                          服务端
  │                              │
  │──── POST /login ────────────→│ 验证账号密码
  │                              │ 生成 accessToken + refreshToken
  │←── { accessToken, refresh } ─│
  │                              │
  │──── GET /api/xxx ───────────→│ 验证 accessToken
  │     Authorization: Bearer xx │
  │←── { data } ────────────────│
  │                              │
  │  accessToken 过期             │
  │──── POST /refresh ──────────→│ 验证 refreshToken
  │     { refreshToken }         │ 生成新 accessToken
  │←── { accessToken } ─────────│
```

**Token 存储规范**：

| 场景 | 存储位置 | 原因 |
|------|---------|------|
| Web 应用 | `localStorage` + `httpOnly cookie`（推荐） | httpOnly 防 XSS |
| 小程序 | `wx.setStorageSync` | 小程序无 cookie |
| H5 / 公众号 | `localStorage` | 跨页面保持登录 |

#### RBAC 权限模型

```
用户(User) ──N:N──→ 角色(Role) ──N:N──→ 权限(Permission)
                                            │
                                       ┌────┴────┐
                                    菜单权限    按钮权限
                                   (route)    (action)
```

---

## 6. 数据库设计

### 6.1 命名规范

| 项目 | 规范 | 正确示例 | 错误示例 |
|------|------|---------|---------|
| 数据库名 | 小写下划线，项目前缀 | `myapp_order` | `MyAppOrder` |
| 表名 | 小写下划线，模块前缀 | `t_order`、`sys_user` | `Order`、`SysUser` |
| 字段名 | 小写下划线 | `create_time` | `createTime`、`CreateTime` |
| 索引名-普通 | `idx_{表名缩写}_{字段}` | `idx_order_user_id` | `index1` |
| 索引名-唯一 | `uk_{表名缩写}_{字段}` | `uk_user_phone` | `unique_phone` |
| 主键 | `id` | `id` | `orderId`、`ID` |

### 6.2 必备字段（所有表强制包含）

```sql
CREATE TABLE t_example (
    id          BIGINT       NOT NULL AUTO_INCREMENT COMMENT '主键ID',
    -- 业务字段 ...
    create_time DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    update_time DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    is_deleted  TINYINT(1)   NOT NULL DEFAULT 0 COMMENT '逻辑删除：0否 1是',
    PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='示例表';
```

**说明**：
- **`id`**：统一使用 `BIGINT` 自增主键
- **`create_time`**：记录创建时间，不可修改
- **`update_time`**：每次更新自动刷新
- **`is_deleted`**：逻辑删除标记。**永远不要物理删除业务数据**
- **`utf8mb4`**：必须使用 `utf8mb4` 而非 `utf8`（支持 emoji 和完整中文）

### 6.3 字段类型规范

| 业务场景 | 推荐类型 | 说明 |
|---------|---------|------|
| 主键 | `BIGINT AUTO_INCREMENT` | 不用 UUID，查询效率差 |
| 金额 | `INT UNSIGNED` 或 `BIGINT` | **单位：分**。绝对禁止用浮点数存金额 |
| 手机号 | `VARCHAR(20)` | 考虑国际号码 |
| 邮箱 | `VARCHAR(128)` | |
| 身份证号 | `VARCHAR(18)` | 存加密后的值 |
| 状态/类型 | `TINYINT UNSIGNED` | 0/1/2... 对应枚举值 |
| 是/否 | `TINYINT(1) UNSIGNED` | 0=否, 1=是 |
| 百分比 | `DECIMAL(5,2)` | 如 99.99% |
| 经纬度 | `DECIMAL(10,6)` | |
| JSON 数据 | `JSON` 或 `VARCHAR(2048)` | MySQL 5.7+ 支持 JSON 类型 |
| 长文本 | `TEXT` | 文章内容、备注等 |
| 日期时间 | `DATETIME` | 不用 TIMESTAMP（2038年问题） |
| IP 地址 | `VARCHAR(45)` | 兼容 IPv6 |

### 6.4 索引设计原则

1. **必建索引**：
   - 外键字段（如 `user_id`、`order_id`）
   - 经常出现在 `WHERE` 条件中的字段
   - `ORDER BY` 排序字段
   - `is_deleted`（如果经常用于查询条件）

2. **联合索引遵循最左前缀原则**：
   ```sql
   -- 如果查询经常是 WHERE platform_id = ? AND status = ? ORDER BY create_time
   ALTER TABLE t_order ADD INDEX idx_platform_status_time (platform_id, status, create_time);
   ```

3. **禁止事项**：
   - 单表索引不超过 5 个
   - 禁止在大文本字段（TEXT/BLOB）上建索引
   - 禁止冗余索引（已有 `idx_a_b` 就不需要单独的 `idx_a`）

### 6.5 SQL 编写规范

```sql
-- ✅ 正确：明确指定字段
SELECT id, user_name, phone, status FROM t_user WHERE is_deleted = 0;

-- ❌ 错误：禁止 SELECT *
SELECT * FROM t_user;

-- ✅ 正确：大量数据必须分页
SELECT id, title FROM t_order WHERE user_id = 100 ORDER BY create_time DESC LIMIT 20 OFFSET 0;

-- ❌ 错误：不加 LIMIT 的查询可能返回百万行
SELECT id, title FROM t_order WHERE user_id = 100 ORDER BY create_time DESC;

-- ✅ 正确：批量操作用 IN，但控制数量
DELETE FROM t_order WHERE id IN (1, 2, 3, 4, 5);

-- ❌ 错误：IN 中不要超过 1000 个值
DELETE FROM t_order WHERE id IN (1, 2, ..., 10000);

-- ✅ 正确：更新必须带 WHERE 条件
UPDATE t_order SET status = 2 WHERE id = 100 AND status = 1;

-- ❌ 危险：没有 WHERE 条件的更新
UPDATE t_order SET status = 2;
```

---

## 7. API 设计

### 7.1 RESTful 规范

| 操作 | HTTP 方法 | 路径示例 | 说明 |
|------|----------|---------|------|
| 查询列表 | `GET` | `/api/users` | 查询参数放 Query String |
| 查询详情 | `GET` | `/api/users/:id` | |
| 创建 | `POST` | `/api/users` | 请求体放 JSON |
| 更新（全量）| `PUT` | `/api/users/:id` | |
| 更新（部分）| `PATCH` | `/api/users/:id` | |
| 删除 | `DELETE` | `/api/users/:id` | 业务系统通常是逻辑删除 |

### 7.2 接口路径规范

```
/api/{版本}/{模块}/{资源}

示例：
GET    /api/v1/users              # 用户列表
GET    /api/v1/users/123          # 用户详情
POST   /api/v1/users              # 创建用户
PUT    /api/v1/users/123          # 更新用户
DELETE /api/v1/users/123          # 删除用户
POST   /api/v1/users/123/disable  # 禁用用户（非标准 CRUD 操作用 POST + 动词）
GET    /api/v1/orders?status=1&page=1&pageSize=20  # 带筛选和分页的列表
```

### 7.3 分页接口规范

**请求参数**：

```json
{
  "page": 1,
  "pageSize": 20,
  "keyword": "搜索关键词",
  "status": 1,
  "startTime": "2025-01-01 00:00:00",
  "endTime": "2025-12-31 23:59:59",
  "orderBy": "create_time",
  "orderDir": "desc"
}
```

**响应格式**：

```json
{
  "code": 200,
  "message": "success",
  "data": {
    "list": [{ ... }],
    "total": 1234,
    "page": 1,
    "pageSize": 20,
    "totalPages": 62
  }
}
```

### 7.4 文件上传接口规范

```
POST /api/v1/upload
Content-Type: multipart/form-data

响应：
{
  "code": 200,
  "data": {
    "url": "https://oss.example.com/uploads/2025/01/abc.jpg",
    "name": "abc.jpg",
    "size": 102400,
    "type": "image/jpeg"
  }
}
```

**规则**：
- 文件存储必须用云 OSS（阿里云 OSS / 腾讯云 COS），禁止存服务器本地
- 上传前后端都要校验文件类型和大小
- 图片建议限制 5MB，普通文件限制 20MB
- 返回的 URL 必须是 HTTPS

### 7.5 接口文档

- 后端必须集成 Swagger/Knife4j，自动生成接口文档
- 每个接口必须有中文描述
- 每个参数必须标注是否必填、类型、示例值
- 枚举字段必须在描述中列出所有可选值

---

# 第三部分：编码规范

## 8. 前端编码规范

### 8.1 Vue 组件规范

```vue
<template>
  <!-- 模板中不写复杂逻辑，用 computed 或 method 代替 -->
  <div class="user-list">
    <div v-for="item in filteredList" :key="item.id" class="user-item">
      <span>{{ item.name }}</span>
      <span>{{ formatMoney(item.balance) }}</span>
    </div>
    <el-empty v-if="filteredList.length === 0" description="暂无数据" />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { getUserList } from '@/api/modules/user'
import type { UserItem } from '@/types/api'
import { formatMoney } from '@/utils/format'

// Props 定义
interface Props {
  status?: number
}
const props = withDefaults(defineProps<Props>(), {
  status: 0
})

// Emits 定义
const emit = defineEmits<{
  select: [user: UserItem]
}>()

// 响应式数据
const list = ref<UserItem[]>([])
const loading = ref(false)

// 计算属性
const filteredList = computed(() =>
  list.value.filter(item => item.status === props.status)
)

// 方法
async function fetchList() {
  loading.value = true
  try {
    list.value = await getUserList({ status: props.status })
  } finally {
    loading.value = false
  }
}

// 生命周期
onMounted(() => {
  fetchList()
})

// 暴露给父组件的方法
defineExpose({ fetchList })
</script>

<style lang="scss" scoped>
.user-list {
  padding: 16px;
}
.user-item {
  display: flex;
  justify-content: space-between;
  padding: 12px 0;
  border-bottom: 1px solid #eee;
}
</style>
```

### 8.2 命名规范

| 类型 | 规范 | 正确示例 | 错误示例 |
|------|------|---------|---------|
| 组件文件名 | PascalCase | `UserList.vue` | `userList.vue`、`user-list.vue` |
| 组件引用 | PascalCase | `<UserList />` | `<user-list />` |
| 变量/函数 | camelCase | `userName`、`getUserList` | `user_name`、`GetUserList` |
| 常量 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` | `maxPageSize` |
| CSS 类名 | kebab-case | `.user-list-item` | `.userListItem` |
| 路由 path | kebab-case | `/user-center` | `/userCenter` |
| API 函数 | `动词 + 名词` | `getUserList`、`createOrder` | `list`、`add` |
| 事件名 | `on + 动词` | `onSubmit`、`onDelete` | `submit`、`handleClick1` |

### 8.3 TypeScript 使用规范

```typescript
// ✅ 正确：为 API 返回值定义类型
interface UserInfo {
  id: number
  name: string
  phone: string
  avatar: string
  status: UserStatus
  createTime: string
}

enum UserStatus {
  DISABLED = 0,
  ENABLED = 1
}

// ✅ 正确：为组件 Props 定义类型
interface Props {
  userId: number
  showAvatar?: boolean
}

// ❌ 错误：使用 any
const data: any = await fetchData()

// ✅ 正确：不确定类型时用 unknown，然后做类型收窄
const data: unknown = await fetchData()
if (typeof data === 'string') {
  // 在这里 data 是 string 类型
}
```

### 8.4 金额处理规范（极其重要）

```typescript
// utils/format.ts

/**
 * 分转元：后端存分，前端展示元
 * 1234 → "12.34"
 */
export function fen2yuan(fen: number): string {
  return (fen / 100).toFixed(2)
}

/**
 * 元转分：前端输入元，传给后端分
 * "12.34" → 1234
 */
export function yuan2fen(yuan: string | number): number {
  return Math.round(Number(yuan) * 100)
}

/**
 * 格式化金额展示（带千分位）
 * 1234567 → "12,345.67"
 */
export function formatMoney(fen: number): string {
  const yuan = (fen / 100).toFixed(2)
  return yuan.replace(/\B(?=(\d{3})+\.)/g, ',')
}

// ❌ 绝对禁止：直接用浮点数计算金额
// 0.1 + 0.2 = 0.30000000000000004
const wrong = 0.1 + 0.2

// ✅ 正确：用分（整数）计算
const correct = 10 + 20  // 30分 = 0.30元
```

### 8.5 错误处理规范

```typescript
// ✅ 正确：接口调用统一 try/catch
async function submitOrder() {
  if (submitting.value) return  // 防重复提交
  submitting.value = true
  try {
    await createOrder(form.value)
    showToast('提交成功')
    router.push('/order/success')
  } catch (error) {
    // axios 拦截器已处理通用错误，这里处理特殊逻辑
    console.error('创建订单失败:', error)
  } finally {
    submitting.value = false
  }
}

// ✅ 正确：表单提交前校验
async function handleSubmit() {
  const valid = await formRef.value?.validate().catch(() => false)
  if (!valid) return
  await submitOrder()
}
```

---

## 9. 后端编码规范

### 9.1 Controller 规范

```java
@RestController
@RequestMapping("/api/v1/users")
@Api(tags = "用户管理")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    @ApiOperation("用户列表（分页）")
    public Result<PageResult<UserVO>> list(@Valid UserQueryDTO query) {
        return Result.success(userService.pageList(query));
    }

    @GetMapping("/{id}")
    @ApiOperation("用户详情")
    public Result<UserVO> detail(@PathVariable Long id) {
        return Result.success(userService.getDetail(id));
    }

    @PostMapping
    @ApiOperation("创建用户")
    public Result<Long> create(@RequestBody @Valid UserCreateDTO dto) {
        return Result.success(userService.create(dto));
    }

    @PutMapping("/{id}")
    @ApiOperation("更新用户")
    public Result<Boolean> update(@PathVariable Long id, @RequestBody @Valid UserUpdateDTO dto) {
        dto.setId(id);
        return Result.success(userService.update(dto));
    }

    @DeleteMapping("/{id}")
    @ApiOperation("删除用户")
    public Result<Boolean> delete(@PathVariable Long id) {
        return Result.success(userService.delete(id));
    }
}
```

**Controller 铁律**：
- Controller 方法体不超过 **10 行**
- 不写任何业务逻辑，只做：参数接收 → 调用 Service → 返回结果
- 参数校验用 `@Valid` + `@NotBlank` / `@NotNull` 等注解
- 返回值永远是 `Result<T>`

### 9.2 Service 规范

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserMapper userMapper;
    private final UserConvert userConvert;

    @Override
    @Transactional(rollbackFor = Exception.class)
    public Long create(UserCreateDTO dto) {
        // 1. 校验：手机号是否已注册
        Long existCount = userMapper.selectCount(
            new LambdaQueryWrapper<User>().eq(User::getPhone, dto.getPhone())
        );
        if (existCount > 0) {
            throw new BusinessException(1001, "该手机号已注册");
        }

        // 2. 转换：DTO → Entity
        User user = userConvert.toEntity(dto);
        user.setStatus(UserStatus.ENABLED);

        // 3. 持久化
        userMapper.insert(user);

        return user.getId();
    }
}
```

**Service 铁律**：
- 业务异常用自定义 `BusinessException`，携带错误码和消息
- 写操作必须加 `@Transactional(rollbackFor = Exception.class)`
- 禁止在 Service 中直接操作 `HttpServletRequest`/`HttpServletResponse`
- 方法入参用 DTO，返回值用 VO，不要直接暴露 Entity

### 9.3 日志规范

```java
// ✅ 正确：使用 Slf4j + 占位符
log.info("创建用户成功, userId={}, phone={}", user.getId(), user.getPhone());
log.warn("用户登录失败, phone={}, 原因={}", phone, "密码错误");
log.error("支付回调处理异常, orderId={}", orderId, exception);

// ❌ 错误：字符串拼接
log.info("创建用户成功, userId=" + user.getId());

// ❌ 错误：打印敏感信息
log.info("用户登录, phone={}, password={}", phone, password);

// ✅ 正确：敏感信息脱敏后打印
log.info("用户登录, phone={}", DesensitizeUtil.phone(phone));
```

**日志级别使用原则**：

| 级别 | 使用场景 |
|------|---------|
| `ERROR` | 系统异常、影响功能的错误（需要立即关注） |
| `WARN` | 业务异常、可预期的错误（如参数校验失败、余额不足） |
| `INFO` | 关键业务流程节点（如下单成功、支付回调、用户登录） |
| `DEBUG` | 调试信息（生产环境关闭） |

---

## 10. CSS/样式规范

### 10.1 全局设计 Token

每个项目必须在初始化时定义全局设计变量：

```scss
// assets/styles/variables.scss

// 品牌色
$color-primary: #1677ff;
$color-success: #52c41a;
$color-warning: #faad14;
$color-error: #ff4d4f;

// 中性色
$color-text-primary: #1f1f1f;
$color-text-secondary: #666666;
$color-text-placeholder: #bfbfbf;
$color-border: #e8e8e8;
$color-bg-page: #f5f5f5;
$color-bg-card: #ffffff;

// 字号（中国市场偏大）
$font-size-xs: 11px;
$font-size-sm: 13px;
$font-size-md: 15px;
$font-size-lg: 17px;
$font-size-xl: 20px;
$font-size-xxl: 24px;

// 间距系统（4的倍数）
$spacing-xs: 4px;
$spacing-sm: 8px;
$spacing-md: 12px;
$spacing-lg: 16px;
$spacing-xl: 24px;
$spacing-xxl: 32px;

// 圆角
$radius-sm: 4px;
$radius-md: 8px;
$radius-lg: 12px;
$radius-round: 999px;

// 阴影
$shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.06);
$shadow-md: 0 2px 8px rgba(0, 0, 0, 0.08);
$shadow-lg: 0 4px 16px rgba(0, 0, 0, 0.12);

// 字体族（中国标准）
$font-family: -apple-system, BlinkMacSystemFont, 'PingFang SC', 'Hiragino Sans GB',
  'Microsoft YaHei', 'Helvetica Neue', Arial, sans-serif;

// 安全区域
$safe-area-bottom: env(safe-area-inset-bottom, 0px);
```

### 10.2 移动端适配方案

```typescript
// vite.config.ts 中配置 postcss-pxtorem（自动将 px 转为 rem）
import postcssPxToRem from 'postcss-pxtorem'

export default defineConfig({
  css: {
    postcss: {
      plugins: [
        postcssPxToRem({
          rootValue: 37.5,    // 设计稿宽度 375px，根字号 37.5
          propList: ['*'],
          selectorBlackList: ['.norem']  // 不需要转换的类名前缀
        })
      ]
    }
  }
})
```

```typescript
// utils/rem.ts — 根字号动态计算
function setRem() {
  const width = document.documentElement.clientWidth
  const rem = (width / 375) * 16
  document.documentElement.style.fontSize = Math.min(rem, 20) + 'px'
}
setRem()
window.addEventListener('resize', setRem)
```

### 10.3 CSS 编写规则

```scss
// ✅ 正确：使用 scoped 避免样式污染
<style lang="scss" scoped>
.order-card {
  padding: $spacing-lg;
  background: $color-bg-card;
  border-radius: $radius-md;
  box-shadow: $shadow-sm;

  &__title {
    font-size: $font-size-lg;
    font-weight: 600;
    color: $color-text-primary;
  }

  &__price {
    color: $color-error;
    font-size: $font-size-xl;
    font-weight: bold;
  }
}
</style>

// ❌ 错误：硬编码颜色值
.title { color: #333; font-size: 16px; }

// ❌ 错误：使用 !important（除非覆盖第三方组件库）
.btn { color: red !important; }

// ❌ 错误：过深的嵌套（最多 3 层）
.page .content .list .item .title .icon { ... }
```

---

# 第四部分：中国市场专项

## 11. UI/UX 中国化设计

### 11.1 设计原则

| 原则 | 说明 | 对比国外 |
|------|------|---------|
| **信息密度高** | 中国用户习惯在一屏内看到更多信息 | 欧美偏好大留白 |
| **引导性强** | 每一步操作都有明确的视觉引导和文案 | 欧美偏好自主探索 |
| **信任感构建** | 展示资质、案例、实时数据增强信任 | 品牌力即信任 |
| **社交属性** | 分享、邀请、社群是标配功能 | 社交相对独立 |
| **促销氛围** | 红色、金色、倒计时、限时抢购是常态 | 促销更克制 |

### 11.2 颜色规范（中国审美）

```scss
// 中国市场常用品牌色方案

// 方案 A：科技蓝（适合 B 端产品、SaaS、金融）
$brand-primary: #1677ff;
$brand-dark: #0958d9;
$brand-light: #e6f4ff;

// 方案 B：中国红（适合电商、营销、C 端）
$brand-primary: #ff4d4f;
$brand-dark: #cf1322;
$brand-light: #fff1f0;

// 方案 C：活力橙（适合本地生活、外卖、社交）
$brand-primary: #ff6a00;
$brand-dark: #d94800;
$brand-light: #fff7e6;

// 方案 D：自然绿（适合健康、教育、社区）
$brand-primary: #52c41a;
$brand-dark: #389e0d;
$brand-light: #f6ffed;

// 金色（会员、VIP、高级功能）
$color-gold: #faad14;
$color-gold-dark: #d48806;
```

### 11.3 常用页面布局模式

#### 管理后台（B 端）标准布局

```
┌──────────────────────────────────────────────────┐
│ 顶部导航栏：Logo + 全局搜索 + 消息 + 用户头像     │
├────────┬─────────────────────────────────────────┤
│        │ 面包屑导航                               │
│        ├─────────────────────────────────────────┤
│  侧边  │                                         │
│  菜单  │  内容区                                   │
│  栏    │  ┌─────────────────────────────────┐     │
│        │  │ 搜索筛选区                        │     │
│  可    │  ├─────────────────────────────────┤     │
│  折    │  │ 操作按钮区（新增、批量操作）       │     │
│  叠    │  ├─────────────────────────────────┤     │
│        │  │ 表格/列表数据展示区               │     │
│        │  ├─────────────────────────────────┤     │
│        │  │ 分页器                           │     │
│        │  └─────────────────────────────────┘     │
│        │                                         │
├────────┴─────────────────────────────────────────┤
│ 底部：版权信息                                     │
└──────────────────────────────────────────────────┘
```

#### 移动端（C 端）标准布局

```
┌───────────────────────────┐
│ 状态栏（手机系统）          │
├───────────────────────────┤
│ 顶部导航栏                 │
│ 标题 / 返回 / 操作按钮     │
├───────────────────────────┤
│                           │
│     内容区（可滚动）        │
│                           │
│  ┌─────────────────────┐  │
│  │ 轮播图/Banner        │  │
│  ├─────────────────────┤  │
│  │ 功能入口宫格          │  │
│  ├─────────────────────┤  │
│  │ 列表/卡片内容         │  │
│  └─────────────────────┘  │
│                           │
├───────────────────────────┤
│ 底部 TabBar               │
│ 首页 | 分类 | 我的         │
├───────────────────────────┤
│ 安全区域 (iPhone 底部)     │
└───────────────────────────┘
```

### 11.4 表单设计规范

```
✅ 中国用户习惯的表单设计：

1. 手机号输入框 — 自动格式化：138 0000 0000
2. 验证码 — 倒计时按钮："获取验证码" → "59秒后重试"
3. 密码 — 显示/隐藏切换按钮
4. 身份证 — 自动识别出生日期和性别
5. 地址 — 省/市/区三级联动选择器
6. 金额 — 右对齐，自动补小数点
7. 日期 — 用日期选择器，不要让用户手动输入
8. 必填 — 用红色星号 * 标记
9. 提交按钮 — 禁用状态 + Loading 状态 + 防重复点击
10. 错误提示 — 实时校验，错误信息显示在对应输入框下方
```

### 11.5 空状态与异常页设计

```
每个列表/页面必须覆盖以下状态：

1. 加载中 — 骨架屏（Skeleton）而非转圈 Loading
2. 空数据 — 插画 + 文案 + 操作引导（如 "去发布"）
3. 加载失败 — 插画 + 错误描述 + 重试按钮
4. 网络异常 — 插画 + "网络连接异常，请检查网络" + 重试按钮
5. 无权限 — 插画 + "暂无权限访问" + 联系管理员
6. 404 — 插画 + "页面不存在" + 返回首页
```

---

## 12. 支付集成

### 12.1 支付方式优先级（中国市场）

| 优先级 | 支付方式 | 使用场景 | 市场份额 |
|--------|---------|---------|---------|
| 1 | 微信支付 | 小程序/公众号/H5/APP | ~40% |
| 2 | 支付宝 | H5/APP/PC | ~35% |
| 3 | 云闪付/银联 | APP/PC | ~10% |
| 4 | Apple Pay | iOS APP | 少量 |

### 12.2 支付流程标准实现

```
用户端                    商户后端                     支付平台
  │                         │                           │
  │── 1. 提交订单 ─────────→│                           │
  │                         │── 2. 创建内部订单 ────────→│
  │                         │── 3. 调用统一下单 API ────→│
  │                         │←── 4. 返回预支付信息 ─────│
  │←── 5. 返回支付参数 ─────│                           │
  │                         │                           │
  │── 6. 唤起支付控件 ─────────────────────────────────→│
  │←── 7. 支付结果（客户端）←──────────────────────────│
  │                         │                           │
  │                         │←── 8. 异步回调通知 ──────│
  │                         │── 9. 验签 + 更新订单状态 ─│
  │                         │── 10. 返回 SUCCESS ──────→│
  │                         │                           │
  │── 11. 查询订单状态 ────→│                           │
  │←── 12. 返回支付结果 ────│                           │
```

### 12.3 支付开发铁律

1. **金额用分**：所有接口传输和数据库存储，金额单位为**分**（整数）
2. **以回调为准**：支付结果以服务端收到的异步回调为准，客户端返回的结果仅用于展示
3. **幂等处理**：同一笔回调可能多次推送，必须做幂等校验
4. **验签必做**：收到回调后必须验证签名，防止伪造
5. **订单超时**：未支付订单设置超时自动取消（通常 15-30 分钟）
6. **对账机制**：每日与支付平台对账，排查差异
7. **退款异步**：退款结果也是异步通知，不要同步等待

### 12.4 订单号生成规范

```
格式：{业务类型}{年月日}{时分秒}{毫秒}{6位随机数}
示例：OD20250410143025001234567

规则：
- 全局唯一
- 可排序（按时间）
- 包含业务语义前缀
- 长度 24-32 位
- 不要暴露信息量（如连续递增的数字，会暴露订单量）
```

---

## 13. 微信生态开发

### 13.1 微信小程序开发要点

```
项目结构：
├── pages/              # 页面
│   ├── index/          # 首页
│   ├── user/           # 用户中心
│   └── order/          # 订单
├── components/         # 组件
├── utils/              # 工具
├── api/                # 接口
├── miniprogram_npm/    # npm 构建产物
├── app.js              # 入口
├── app.json            # 全局配置
├── app.wxss            # 全局样式
├── project.config.json # 项目配置
└── sitemap.json        # 搜索配置
```

**关键注意事项**：

1. **包大小限制**：主包 ≤ 2MB，总计 ≤ 20MB。必须做分包加载。
2. **分包策略**：
   ```json
   {
     "subPackages": [
       { "root": "pages/order", "pages": ["list", "detail"] },
       { "root": "pages/user", "pages": ["profile", "setting"] }
     ],
     "preloadRule": {
       "pages/index/index": {
         "network": "all",
         "packages": ["pages/order"]
       }
     }
   }
   ```
3. **登录流程**：`wx.login()` 获取 code → 发给后端 → 后端用 code 换 openid/session_key
4. **授权流程**：手机号需要用 `<button open-type="getPhoneNumber">`
5. **支付流程**：后端统一下单后返回支付参数，前端调 `wx.requestPayment()`
6. **审核要点**：
   - 首页不能有弹窗引导关注
   - 不能跳转外部 H5（除非配了业务域名）
   - 虚拟支付走 iOS 内购会被拒（但最新政策有变化）
   - 涉及 UGC 内容必须有审核机制

### 13.2 公众号 H5 开发要点

```
1. 授权登录：OAuth2.0 网页授权
   - 静默授权（scope=snsapi_base）：只能拿 openid
   - 主动授权（scope=snsapi_userinfo）：能拿昵称头像

2. JSSDK 配置：
   - 每个使用 JS-SDK 的页面都需要注入配置
   - SPA 应用注意：iOS 用入口 URL，Android 用当前 URL
   - 必须配置安全域名（JS接口安全域名）

3. 分享配置：
   - 自定义分享标题、描述、图片、链接
   - 分享链接必须是当前公众号的安全域名

4. 支付：
   - 公众号支付必须在微信浏览器内
   - H5 支付可以在外部浏览器（但体验差）
```

---

## 14. 合规与备案

### 14.1 网站上线必备清单

| 项目 | 必要性 | 说明 | 办理周期 |
|------|--------|------|---------|
| **ICP 备案** | 强制 | 所有中国境内服务器的网站必须备案 | 7-20 个工作日 |
| **公安备案** | 强制 | ICP 备案完成后 30 天内到公安局备案 | 3-7 个工作日 |
| **SSL 证书** | 强制 | 所有线上环境必须 HTTPS | 免费证书即日生效 |
| **隐私政策** | 强制 | 页面底部或注册时展示 | - |
| **用户协议** | 强制 | 注册时勾选同意 | - |
| **ICP 经营许可证** | 视情况 | 经营性网站需要（如电商） | 20-40 个工作日 |
| **等保测评** | 视情况 | 处理大量用户数据的系统 | 1-3 个月 |

### 14.2 隐私合规要点

```
1. 注册/登录时：
   - 必须展示《隐私政策》和《用户协议》链接
   - 用户必须主动勾选同意（不能默认勾选）
   - 首次登录弹出隐私政策弹窗

2. 个人信息收集：
   - 最小必要原则：只收集业务必需的信息
   - 敏感信息（身份证、银行卡）加密存储
   - 展示时脱敏处理（138****0000）

3. 第三方 SDK：
   - 在隐私政策中列明所有第三方 SDK 及其用途
   - 微信 SDK、支付宝 SDK、极光推送、友盟统计等

4. 数据出境：
   - 中国用户数据必须存储在中国境内服务器
   - 如有跨境需求，需通过安全评估

5. 注销权利：
   - 必须提供账号注销功能
   - 注销后个人数据必须删除或匿名化
```

### 14.3 页面底部必放信息

```html
<footer>
  <p>Copyright © 2025 {公司名称} 版权所有</p>
  <p>
    <a href="https://beian.miit.gov.cn/" target="_blank">粤ICP备XXXXXXXX号-1</a>
  </p>
  <p>
    <img src="police-badge.png" />
    <a href="http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=XXXXXXXX" target="_blank">
      粤公网安备 XXXXXXXXXXXXXX号
    </a>
  </p>
</footer>
```

---

# 第五部分：质量保障

## 15. 安全规范

### 15.1 安全开发清单（每个项目必检）

| 威胁 | 防御措施 | 实现方式 |
|------|---------|---------|
| **SQL 注入** | 参数化查询 | 使用 ORM（MyBatis-Plus/TypeORM），禁止拼接 SQL |
| **XSS 攻击** | 输出转义 | 前端框架默认转义（Vue/React），后端过滤 HTML 标签 |
| **CSRF** | Token 校验 | 使用 JWT + SameSite Cookie，或 CSRF Token |
| **越权访问** | 权限校验 | 每个接口校验当前用户是否有权操作目标资源 |
| **暴力破解** | 频率限制 | 登录失败 5 次锁定 15 分钟，短信验证码 60 秒间隔 |
| **敏感数据泄露** | 加密+脱敏 | 密码 bcrypt 加密，手机号脱敏展示 |
| **文件上传漏洞** | 类型+大小校验 | 白名单校验文件后缀和 MIME，限制大小 |
| **接口滥用** | 限流 | 全局限流 + 单用户限流 |
| **中间人攻击** | HTTPS | 全站 HTTPS，HSTS 头 |

### 15.2 密码存储规范

```java
// ✅ 正确：bcrypt 加密（Java）
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
String hashedPassword = encoder.encode(rawPassword);           // 加密
boolean matches = encoder.matches(rawPassword, hashedPassword); // 校验
```

```typescript
// ✅ 正确：bcrypt 加密（Node.js）
import bcrypt from 'bcryptjs'

const hashed = await bcrypt.hash(password, 10)         // 加密
const isValid = await bcrypt.compare(password, hashed)  // 校验
```

**密码铁律**：
- 永远不要明文存储密码
- 永远不要使用 MD5/SHA 系列（彩虹表可破解）
- 永远不要在日志中打印密码
- 传输过程使用 HTTPS 加密

### 15.3 敏感数据脱敏

```typescript
// utils/desensitize.ts

export function maskPhone(phone: string): string {
  return phone.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2')
  // 138****0000
}

export function maskIdCard(idCard: string): string {
  return idCard.replace(/(\d{4})\d{10}(\d{4})/, '$1**********$2')
  // 4401**********1234
}

export function maskBankCard(card: string): string {
  return card.replace(/(\d{4})\d+(\d{4})/, '$1 **** **** $2')
  // 6222 **** **** 1234
}

export function maskEmail(email: string): string {
  return email.replace(/(.{2}).+(@.+)/, '$1***$2')
  // zh***@example.com
}

export function maskName(name: string): string {
  if (name.length <= 1) return '*'
  if (name.length === 2) return name[0] + '*'
  return name[0] + '*'.repeat(name.length - 2) + name[name.length - 1]
  // 张*三
}
```

### 15.4 接口安全防护

```typescript
// 防重复提交（前端）
function useDebounceSubmit(fn: Function, delay = 500) {
  const loading = ref(false)

  async function execute(...args: any[]) {
    if (loading.value) return
    loading.value = true
    try {
      await fn(...args)
    } finally {
      setTimeout(() => { loading.value = false }, delay)
    }
  }

  return { loading, execute }
}

// 接口签名（防篡改）
function generateSign(params: Record<string, any>, secret: string): string {
  const sortedKeys = Object.keys(params).sort()
  const queryString = sortedKeys
    .map(key => `${key}=${params[key]}`)
    .join('&')
  return md5(queryString + secret)
}
```

---

## 16. 性能优化

### 16.1 前端性能优化清单

| 优化项 | 做法 | 效果 |
|--------|------|------|
| **路由懒加载** | `() => import('./Page.vue')` | 首屏加载减少 40-60% |
| **组件按需引入** | `unplugin-vue-components` 自动按需导入 | 打包体积减少 30%+ |
| **图片优化** | WebP 格式 + 懒加载 + 压缩 + CDN | 加载提速 50%+ |
| **代码分割** | Vite/Webpack 自动 chunk split | 并行加载，提升速度 |
| **Gzip/Brotli** | Nginx 开启压缩 | 传输体积减少 60-80% |
| **CDN 静态资源** | 将 JS/CSS/图片放 CDN | 加载提速 |
| **虚拟滚动** | 长列表（>100条）使用 `vue-virtual-scroller` | 渲染性能提升 10x |
| **防抖/节流** | 搜索用防抖，滚动用节流 | 减少无效请求 |
| **缓存策略** | 合理设置 `Cache-Control`，使用 Service Worker | 二次访问秒开 |
| **预加载** | `<link rel="prefetch">` 预加载下一页资源 | 页面切换更流畅 |

### 16.2 后端性能优化清单

| 优化项 | 做法 | 场景 |
|--------|------|------|
| **数据库索引** | 根据慢查询日志添加合适索引 | 查询响应 >200ms |
| **Redis 缓存** | 热点数据缓存，设置合理 TTL | 高频读取、低频变更的数据 |
| **分页查询** | 禁止无条件全表查询，必须 LIMIT | 所有列表接口 |
| **SQL 优化** | EXPLAIN 分析、避免 SELECT *、避免子查询 | 慢查询 |
| **连接池** | 配置合理的数据库连接池大小 | 高并发 |
| **异步处理** | 耗时操作放消息队列（短信、邮件、推送） | 响应时间要求 <500ms |
| **批量操作** | 批量插入/更新代替循环单条操作 | 数据导入/同步 |
| **接口合并** | BFF 层合并多个接口调用 | 首屏需要多个接口数据 |

### 16.3 性能指标参考

| 指标 | 优秀 | 合格 | 不合格 |
|------|------|------|--------|
| 首屏加载时间（FCP） | < 1.5s | < 3s | > 3s |
| 最大内容渲染（LCP） | < 2.5s | < 4s | > 4s |
| 接口响应时间（P95） | < 200ms | < 500ms | > 1s |
| 页面切换时间 | < 300ms | < 500ms | > 1s |
| JS 打包体积（gzip后） | < 200KB | < 500KB | > 1MB |

---

## 17. 测试规范

### 17.1 测试类型与覆盖要求

| 测试类型 | 覆盖范围 | 工具推荐 | 最低要求 |
|---------|---------|---------|---------|
| 单元测试 | 工具函数、核心业务逻辑 | Vitest / Jest / JUnit | 工具函数 100%，核心逻辑 80% |
| 接口测试 | 所有 API 接口 | Postman / Apifox | 所有接口 100% |
| E2E 测试 | 核心业务流程 | Cypress / Playwright | 核心链路 100% |
| 兼容性测试 | 主流浏览器/设备 | 真机测试 | 见 17.2 |

### 17.2 兼容性测试矩阵（中国市场）

**移动端**（必测）：

| 设备 | 浏览器/环境 |
|------|------------|
| iPhone（iOS 14+） | Safari、微信内置浏览器 |
| 安卓（Android 10+） | Chrome、微信内置浏览器 |
| 微信小程序 | 微信开发者工具 + 真机 |

**PC 端**（必测）：

| 浏览器 | 版本 |
|--------|------|
| Chrome | 最新 2 个大版本 |
| Edge | 最新版 |
| Safari | 最新版（如有 Mac 用户） |
| Firefox | 最新版 |
| 360 浏览器 | 最新版（中国市场占比不低） |

### 17.3 测试要点清单

```
功能测试：
- [ ] 所有表单的正常提交和异常校验
- [ ] 列表的空数据、单条数据、多页数据展示
- [ ] 搜索、筛选、排序功能
- [ ] 分页的首页、末页、中间页切换
- [ ] 登录/登出/Token 过期处理
- [ ] 权限控制（不同角色看到不同菜单/按钮）
- [ ] 支付流程（正常支付、取消支付、支付超时）

体验测试：
- [ ] 加载中状态有 Loading 或骨架屏
- [ ] 操作成功有成功提示
- [ ] 操作失败有友好的错误提示
- [ ] 网络断开有离线提示
- [ ] 表单有防重复提交保护
- [ ] 长列表滚动流畅
- [ ] 图片有 loading 占位和加载失败兜底

安全测试：
- [ ] SQL 注入测试（在输入框输入 ' OR 1=1 --）
- [ ] XSS 测试（在输入框输入 <script>alert(1)</script>）
- [ ] 越权测试（用户 A 的 Token 访问用户 B 的数据）
- [ ] 并发测试（同一操作快速点击多次）
```

---

# 第六部分：交付上线

## 18. 部署与运维

### 18.1 推荐部署架构

#### 小型项目（日 PV < 1 万）

```
┌──────────────┐
│   云服务器     │  1 台 2核4G
│              │
│  Nginx       │  反向代理 + 静态资源
│  Node.js/Java│  后端服务
│  MySQL       │  数据库（或用云数据库 RDS）
│  Redis       │  缓存
└──────────────┘
```

月成本预估：200-500 元

#### 中型项目（日 PV 1-10 万）

```
                    ┌─────────────┐
                    │  CDN + OSS   │  静态资源
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Nginx/SLB   │  负载均衡
                    └──────┬──────┘
                    ┌──────┴──────┐
               ┌────▼────┐  ┌────▼────┐
               │  App 1   │  │  App 2   │  应用服务器（2 台）
               └────┬────┘  └────┬────┘
                    └──────┬──────┘
               ┌───────────┼───────────┐
          ┌────▼────┐ ┌────▼────┐ ┌────▼────┐
          │ MySQL   │ │ Redis   │ │ OSS     │
          │ (主从)  │ │ (哨兵)  │ │         │
          └─────────┘ └─────────┘ └─────────┘
```

月成本预估：1000-3000 元

### 18.2 Nginx 配置模板

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    ssl_certificate /etc/nginx/ssl/example.com.pem;
    ssl_certificate_key /etc/nginx/ssl/example.com.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    # 前端静态资源
    location / {
        root /var/www/frontend/dist;
        index index.html;
        try_files $uri $uri/ /index.html;

        # 缓存策略
        location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff2?)$ {
            expires 30d;
            add_header Cache-Control "public, immutable";
        }
    }

    # 后端 API 代理
    location /api/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 超时设置
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
    }

    # Gzip 压缩
    gzip on;
    gzip_min_length 1024;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    gzip_vary on;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
}
```

### 18.3 Docker 部署模板

```dockerfile
# 前端 Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

```dockerfile
# 后端 Dockerfile（Node.js）
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=mysql
      - REDIS_HOST=redis
    depends_on:
      - mysql
      - redis

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"

volumes:
  mysql_data:
  redis_data:
```

### 18.4 环境管理

| 环境 | 用途 | 域名示例 | 数据 |
|------|------|---------|------|
| 本地开发 | 日常开发调试 | `localhost:5173` | 本地/测试库 |
| 测试环境 | 功能验证 | `test.example.com` | 测试数据 |
| 预发布环境 | 上线前最终验证 | `staging.example.com` | 生产数据副本 |
| 生产环境 | 正式运行 | `www.example.com` | 真实数据 |

**环境变量文件**：

```bash
# .env.development
VITE_API_BASE_URL=http://localhost:3000/api
VITE_APP_TITLE=项目名-开发

# .env.production
VITE_API_BASE_URL=https://api.example.com/api
VITE_APP_TITLE=项目名
```

---

## 19. 监控与告警

### 19.1 必须监控的指标

| 类型 | 指标 | 告警阈值 | 工具 |
|------|------|---------|------|
| 服务器 | CPU 使用率 | > 80% 持续 5 分钟 | 云监控 |
| 服务器 | 内存使用率 | > 85% | 云监控 |
| 服务器 | 磁盘使用率 | > 80% | 云监控 |
| 应用 | 接口响应时间 P95 | > 1 秒 | APM |
| 应用 | 接口错误率 | > 1% | APM |
| 应用 | JVM 堆内存 | > 80% | Prometheus |
| 数据库 | 慢查询数量 | > 10/分钟 | 云数据库监控 |
| 数据库 | 连接数 | > 80% 最大连接数 | 云数据库监控 |
| 业务 | 下单失败率 | > 5% | 自定义 |
| 业务 | 支付回调超时 | > 5 分钟 | 自定义 |
| 前端 | JS 错误数 | 突增 50% | Sentry |
| 前端 | 白屏率 | > 1% | 前端监控 |

### 19.2 告警通知渠道

| 优先级 | 渠道 | 场景 |
|--------|------|------|
| P0 紧急 | 电话 + 短信 + 钉钉/企微 | 服务宕机、数据库异常 |
| P1 重要 | 钉钉/企微机器人 | 错误率升高、响应变慢 |
| P2 一般 | 邮件 | 磁盘空间不足、证书即将过期 |

### 19.3 日志管理规范

```
日志文件命名：{服务名}-{日期}.log
日志保留策略：
  - 生产环境：保留 30 天
  - 测试环境：保留 7 天
  - 日志按天滚动，单文件不超过 200MB

日志内容格式：
[时间] [级别] [traceId] [类名] - 消息内容
2025-04-10 14:30:25.123 [INFO] [abc123def456] [UserService] - 用户登录成功, userId=1001
```

---

## 20. Git 工作流与版本管理

### 20.1 分支策略（Git Flow 简化版）

```
main (master)
  │
  ├── develop         # 开发分支（日常开发合入）
  │     │
  │     ├── feature/user-login      # 功能分支
  │     ├── feature/order-module    # 功能分支
  │     └── feature/payment         # 功能分支
  │
  ├── release/v1.2.0  # 预发布分支（从 develop 切出）
  │
  └── hotfix/fix-login # 紧急修复分支（从 main 切出）
```

### 20.2 分支命名规范

| 类型 | 格式 | 示例 |
|------|------|------|
| 功能分支 | `feature/{功能名}` | `feature/user-login` |
| 修复分支 | `fix/{问题描述}` | `fix/login-token-expired` |
| 紧急修复 | `hotfix/{问题描述}` | `hotfix/payment-callback-fail` |
| 发布分支 | `release/v{版本号}` | `release/v1.2.0` |

### 20.3 Commit Message 规范

```
格式：<type>(<scope>): <subject>

type 类型：
  feat     - 新功能
  fix      - Bug 修复
  docs     - 文档变更
  style    - 代码格式（不影响逻辑）
  refactor - 重构（既不是 feat 也不是 fix）
  perf     - 性能优化
  test     - 测试
  chore    - 构建/工具变更
  sql      - 数据库变更

scope：影响范围（可选）

示例：
  feat(user): 新增手机号验证码登录功能
  fix(order): 修复订单金额计算精度问题
  docs: 更新 API 接口文档
  sql(order): 新增 t_order_refund 退款表
  perf(list): 订单列表查询添加索引优化
```

### 20.4 版本号规范（语义化版本）

```
格式：v{主版本}.{次版本}.{修订号}

主版本：不兼容的 API 变更
次版本：向后兼容的功能新增
修订号：向后兼容的 Bug 修复

示例：
  v1.0.0 → v1.0.1 （修了个 Bug）
  v1.0.1 → v1.1.0 （新增了功能）
  v1.1.0 → v2.0.0 （大重构，接口不兼容）
```

---

## 21. 项目交付清单

项目上线前，逐项确认：

### 功能完整性
- [ ] 所有 P0 功能已完成并通过测试
- [ ] 所有 P1 功能已完成并通过测试
- [ ] 登录/注册/找回密码流程正常
- [ ] 核心业务流程跑通（下单/支付/退款等）
- [ ] 权限控制正确（不同角色看到不同内容）

### 用户体验
- [ ] 所有页面有 Loading 状态
- [ ] 所有列表有空数据状态
- [ ] 所有操作有成功/失败反馈
- [ ] 表单有输入校验和错误提示
- [ ] 移动端适配正常（iOS + Android）
- [ ] 页面加载速度达标（FCP < 3s）

### 安全合规
- [ ] 所有接口使用 HTTPS
- [ ] 密码加密存储（bcrypt）
- [ ] 敏感数据脱敏展示
- [ ] 接口有防重复提交保护
- [ ] 接口有权限校验
- [ ] 隐私政策和用户协议已就位
- [ ] ICP 备案号已展示在页面底部

### 部署运维
- [ ] 生产环境配置已就绪
- [ ] 数据库已初始化（表结构 + 初始数据）
- [ ] SSL 证书已配置
- [ ] Nginx 配置已优化（Gzip、缓存）
- [ ] 监控告警已配置
- [ ] 数据库定时备份已配置
- [ ] 日志收集已配置

### 文档
- [ ] 接口文档（Swagger/Knife4j）可访问
- [ ] 部署文档（如何部署、如何回滚）
- [ ] 数据库文档（表结构说明）
- [ ] 管理员使用手册（如何使用后台）

---

# 附录

## 附录 A. 常用技术栈版本速查表

> 截至 2025/2026 年推荐版本，选型时优先查看官方最新 LTS 版本。

| 技术 | 推荐版本 | 说明 |
|------|---------|------|
| Node.js | 20 LTS / 22 LTS | 偶数版为长期支持版 |
| Vue | 3.5+ | Vue 2 已停止维护 |
| React | 18/19 | |
| Vite | 6.x | 构建工具首选 |
| TypeScript | 5.x | |
| Element Plus | 2.x | Vue 3 管理后台 UI |
| Ant Design Vue | 4.x | Vue 3 备选 UI |
| Vant | 4.x | Vue 3 移动端 UI |
| TDesign | 1.x | 腾讯出品，美观大方 |
| Java | 8 / 17 / 21 | 8 仍是国内主流 |
| Spring Boot | 2.7 / 3.x | 3.x 需要 Java 17+ |
| MyBatis-Plus | 3.5.x | |
| MySQL | 8.0 | |
| Redis | 6/7 | |
| Nginx | 1.24+ | |
| Docker | 24+ | |

## 附录 B. 中国主流云服务商对比

| 对比项 | 阿里云 | 腾讯云 | 华为云 |
|--------|--------|--------|--------|
| 市场份额 | 第一 | 第二 | 第三 |
| 服务器 | ECS | CVM | ECS |
| 云数据库 | RDS | CDB | RDS |
| 对象存储 | OSS | COS | OBS |
| CDN | CDN | CDN | CDN |
| 域名注册 | 支持 | 支持 | 支持 |
| ICP 备案 | 支持 | 支持 | 支持 |
| SSL 证书 | 免费 DV | 免费 DV | 免费 DV |
| 小程序云 | - | 云开发 | - |
| 入门优惠 | 有 | 有 | 有 |
| 推荐场景 | 综合首选 | 微信/游戏 | 政企 |

**新手建议**：首选阿里云或腾讯云，文档最全、社区最大。如果项目涉及微信小程序，腾讯云有原生集成优势。

## 附录 C. 项目自测 Checklist

将此清单复制到项目中，每次上线前逐项检查：

```markdown
## 上线前 Checklist

### 代码质量
- [ ] 无 ESLint/TSC 错误
- [ ] 无 console.log 遗留（使用 vite-plugin-remove-console）
- [ ] 无硬编码的测试数据/账号/密码
- [ ] 无未处理的 TODO/FIXME

### 接口
- [ ] 所有接口已联调通过
- [ ] 接口错误码处理完整
- [ ] 接口超时有提示
- [ ] 文件上传大小限制合理

### 数据库
- [ ] 新增表包含必备字段（id, create_time, update_time, is_deleted）
- [ ] 查询接口有分页
- [ ] 常用查询字段有索引
- [ ] 无全表扫描的慢查询
- [ ] 敏感字段已加密存储

### 安全
- [ ] 所有接口有鉴权
- [ ] 无 SQL 注入风险
- [ ] 无 XSS 风险
- [ ] 文件上传有类型和大小校验
- [ ] 密码用 bcrypt 加密
- [ ] 生产环境关闭 Swagger 访问（或加密码保护）

### 体验
- [ ] 页面标题（document.title）正确
- [ ] 页面 favicon 已设置
- [ ] Loading 状态完整
- [ ] 空数据状态完整
- [ ] 错误状态完整
- [ ] 移动端触摸体验正常
- [ ] 页面无横向溢出滚动

### 兼容性
- [ ] Chrome 最新版正常
- [ ] 微信内置浏览器正常
- [ ] iOS Safari 正常
- [ ] Android 正常
- [ ] 小屏幕（320px 宽）正常
- [ ] 大屏幕（1920px 宽）正常

### 部署
- [ ] 环境变量配置正确
- [ ] 构建产物体积合理（检查是否有大文件未压缩）
- [ ] HTTPS 配置正常
- [ ] 域名解析正确
- [ ] 数据库已备份
- [ ] 可以回滚到上一版本
```

## 附录 D. AI 开发工作流模板

将以下流程告诉你的 AI 编码助手，作为每次开发的标准流程：

```markdown
## AI 开发工作流

### 接到需求时
1. 理解需求，输出需求分析（见第 1 章）
2. 确认技术选型（见第 2 章）
3. 设计数据库表结构（见第 6 章）
4. 设计 API 接口（见第 7 章）
5. 等我确认方案后再写代码

### 写代码时
1. 先搭项目骨架（目录结构 + 基础配置）
2. 后端：Entity → DAO → Service → Controller 顺序
3. 前端：路由 → 页面 → 组件 → API 调用顺序
4. 每写完一个模块，暂停让我确认

### 写完代码后
1. 对照自测 Checklist 逐项检查（见附录 C）
2. 确认所有 TODO 已处理
3. 确认无硬编码的测试数据
4. 确认环境变量正确分离

### 禁止事项
- 不要跳过方案设计直接写代码
- 不要在代码中硬编码密码、密钥、Token
- 不要使用 SELECT *
- 不要忘记接口的错误处理
- 不要忘记表单的输入校验
- 不要忘记列表的分页和空状态
- 不要忘记 Loading 状态
- 不要用浮点数处理金额
- 不要把 Entity 直接返回给前端
- 不要在 Controller 写业务逻辑
```

---

> **本手册持续更新，遇到新的通用规范会不断补充。**
>
> 最后更新：2026-04-10
