<div align="center">

# 🧙‍♂️ AI 黑魔法

### 一个技能文件，让 AI 拥有十年架构师的能力

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/lvcn/ai-black-magic?style=social)](https://github.com/lvcn/ai-black-magic/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/lvcn/ai-black-magic?style=social)](https://github.com/lvcn/ai-black-magic/network/members)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/lvcn/ai-black-magic/pulls)

**别再把 AI 当自动补全了——给它一份技能文件，它就是你的资深架构师。**

[English](README_EN.md) | [快速开始](#-三步起飞) · [技能清单](#-技能清单) · [路线图](#-路线图) · [参与贡献](#-参与贡献)

---

*⭐ 如果这个项目帮到了你，请给一个 Star，让更多人发现它！*

</div>

## 🤔 这是什么？

大多数人用 AI 写代码，只是把它当成**高级自动补全**。

而这个仓库里的每一个 **技能文件（Skill File）**，都是一份精心编写的**系统级指令**——把它丢给 AI（Cursor / ChatGPT / Claude / Copilot），AI 瞬间变成某个领域的资深专家。

> **一句话总结：这不是教程，是给 AI 吃的"武功秘籍"。你不需要学会，AI 学会就行。**

## 💡 为什么用 AI 黑魔法？

```
传统方式：程序员学习 → 程序员执行 → 加班到头秃
黑魔法：  程序员指挥 → AI 按专家标准执行 → 准时下班
```

| 维度 | 普通 AI 使用 | AI 黑魔法 |
|------|------------|-----------|
| **提问方式** | "帮我写个登录功能" | 加载技能文件 → AI 自动按阿里规范设计全套方案 |
| **代码质量** | 能跑就行 | 对标大厂 CR 标准，安全/性能/规范一次到位 |
| **知识依赖** | 你得懂，AI 才能帮你 | 你不懂也行，技能文件里全写了 |
| **适用场景** | 写单个函数 | 从需求分析到部署上线全流程 |

### 兼容工具

<table>
<tr>
<td align="center"><b>Cursor</b></td>
<td align="center"><b>GitHub Copilot</b></td>
<td align="center"><b>ChatGPT</b></td>
<td align="center"><b>Claude</b></td>
<td align="center"><b>Windsurf</b></td>
<td align="center"><b>Cline</b></td>
</tr>
<tr>
<td align="center">✅</td>
<td align="center">✅</td>
<td align="center">✅</td>
<td align="center">✅</td>
<td align="center">✅</td>
<td align="center">✅</td>
</tr>
</table>

## 🚀 三步起飞

**第一步**：Clone 仓库

```bash
git clone https://github.com/lvcn/ai-black-magic.git
```

**第二步**：选择你需要的技能文件（在 `skills/` 目录下）

**第三步**：把技能文件丢给你的 AI 助手

| 工具 | 使用方式 |
|------|---------|
| **Cursor** | 放到项目 `.cursor/rules/` 目录，AI 自动加载 |
| **ChatGPT / Claude** | 对话开头粘贴技能文件内容，或上传文件 |
| **GitHub Copilot** | 在 Copilot Chat 中引用技能文件 |
| **Windsurf / Cline** | 添加到项目规则或直接粘贴到对话中 |

就这么简单。AI 会按照技能文件里的专家标准来执行任务。

## 📦 技能清单

### ✅ 已发布

| 技能文件 | 一句话说明 | 适用场景 |
|---------|-----------|---------|
| [AI 全栈开发实战手册](skills/AI-FULLSTACK-DEVELOPMENT-GUIDE.md) | 让 AI 拥有资深全栈工程师的架构决策能力 | 从 0 到 1 开发完整项目，含中国市场专项（微信/支付宝/备案） |
| [AI 代码审核实战手册](skills/AI-CODE-AUDIT-GUIDE.md) | 让 AI 拥有大厂技术专家的 Code Review 能力 | 代码审核、安全审计、上线前质量把关 |

### 🔨 开发中（即将发布）

| 技能文件 | 一句话说明 | 预计发布 |
|---------|-----------|---------|
| AI 需求分析与 PRD 生成 | 让 AI 像产品总监一样拆解需求、输出 PRD | 第 2 周 |
| AI 数据库设计大师 | 让 AI 按阿里规范设计高性能数据库 | 第 2 周 |
| AI 接口设计规范 | RESTful / GraphQL 一步到位 | 第 3 周 |

> 完整路线图见 [路线图章节](#-路线图)

## 🗺 路线图

按照**真实开发流程**组织，覆盖程序员工作的方方面面。

### 第一阶段：开发核心（基础能力） ✅ 进行中

> 覆盖日常开发中最高频、最痛的场景

| # | 技能文件 | 说明 | 状态 |
|---|---------|------|------|
| 1 | AI 全栈开发实战手册 | 从需求到部署的全流程开发规范 | ✅ 已发布 |
| 2 | AI 代码审核实战手册 | 大厂级 Code Review 标准 | ✅ 已发布 |
| 3 | AI 需求分析与 PRD 生成 | 像产品总监一样拆解需求 | 🔨 开发中 |
| 4 | AI 数据库设计大师 | 高性能数据库设计与优化 | 🔨 开发中 |
| 5 | AI 接口设计规范 | RESTful / GraphQL API 设计 | 📋 排期中 |

### 第二阶段：工程进阶（提升上限）

> 让个人开发者具备团队级工程能力

| # | 技能文件 | 说明 | 状态 |
|---|---------|------|------|
| 6 | AI 自动化测试全攻略 | 单测/集成测试/E2E 一键生成 | 📋 排期中 |
| 7 | AI DevOps 实战 | CI/CD、Docker、K8s 部署自动化 | 📋 排期中 |
| 8 | AI 性能优化指南 | 前端性能 + 后端调优 + SQL 优化 | 📋 排期中 |
| 9 | AI 重构大师 | 屎山代码优雅翻新，不改功能改结构 | 📋 排期中 |
| 10 | AI 技术方案评审 | 像 P8 一样做技术方案评审 | 📋 排期中 |

### 第三阶段：垂直场景（深度覆盖）

> 针对具体技术栈和业务场景的深度技能

| # | 技能文件 | 说明 | 状态 |
|---|---------|------|------|
| 11 | AI 微服务架构师 | 微服务拆分、服务治理、分布式事务 | 📋 排期中 |
| 12 | AI 安全攻防手册 | OWASP Top 10 + 渗透测试 + 合规 | 📋 排期中 |
| 13 | AI 小程序开发专家 | 微信/支付宝小程序全流程 | 📋 排期中 |
| 14 | AI 移动端开发指南 | Flutter / React Native / 原生 | 📋 排期中 |
| 15 | AI 数据分析与可视化 | 数据清洗、分析、Dashboard 搭建 | 📋 排期中 |

### 第四阶段：职场加速（软实力）

> 不只是写代码，还要会汇报、会协作、会晋升

| # | 技能文件 | 说明 | 状态 |
|---|---------|------|------|
| 16 | AI 技术文档写作 | 设计文档/技术博客/RFC 一键生成 | 📋 排期中 |
| 17 | AI 项目管理助手 | 排期、风险管理、站会汇报 | 📋 排期中 |
| 18 | AI 面试通关秘籍 | 简历优化 + 模拟面试 + 八股文精讲 | 📋 排期中 |
| 19 | AI 晋升述职助手 | PPT 大纲、业绩总结、答辩准备 | 📋 排期中 |
| 20 | AI 技术选型决策 | 技术选型的系统化方法论 | 📋 排期中 |

### 第五阶段：商业变现（副业/创业）

> 用 AI 赚钱，而不只是用 AI 打工

| # | 技能文件 | 说明 | 状态 |
|---|---------|------|------|
| 21 | AI 独立开发者指南 | 从 idea 到上线到收费的全流程 | 📋 排期中 |
| 22 | AI SaaS 产品设计 | SaaS 产品的设计、定价与增长 | 📋 排期中 |
| 23 | AI 技术自媒体运营 | 用 AI 批量生产技术内容 | 📋 排期中 |
| 24 | AI 接单赚钱指南 | 外包/freelance 的高效交付方法 | 📋 排期中 |
| 25 | AI 开源项目运营 | 从 0 到 10K Star 的运营策略 | 📋 排期中 |

## 📖 使用示例

### 场景一：你要开发一个新项目

```
1. 把 AI-FULLSTACK-DEVELOPMENT-GUIDE.md 丢给 Cursor
2. 告诉 AI："我要开发一个在线教育平台"
3. AI 会自动按照手册标准帮你：
   ├── 做需求分析和技术选型
   ├── 设计数据库和 API
   ├── 按大厂规范写代码
   └── 处理微信登录/支付宝支付等中国特色需求
```

### 场景二：上线前的代码审核

```
1. 把 AI-CODE-AUDIT-GUIDE.md 丢给 AI
2. 告诉 AI："帮我审核这个模块的代码"
3. AI 会像资深架构师一样：
   ├── P0 红线：安全漏洞、资损风险
   ├── P1 重大问题：架构、事务、并发
   ├── P2 改进建议：命名、性能、可维护性
   └── 输出一份专业的审核报告
```

## 🎯 为什么选择 AI 黑魔法？

### 对标大厂，适合中国

所有技能文件的标准参考：
- 📘 《阿里巴巴 Java 开发手册》
- 📗 《Google Code Review Guidelines》
- 📙 《OWASP Top 10》
- 📕 字节/阿里/腾讯/美团一线工程实践

### 即插即用，不用学习

你不需要花两周看完一本书才能用。**复制 → 粘贴 → 开干**，三秒钟完成技能加载。

### 真实落地，不画大饼

不画饼、不讲概念。每个技能文件都包含：
- ✅ 具体的检查清单
- ✅ 可复用的模板
- ✅ 真实场景的例子
- ✅ 边界条件的处理

### 持续更新

每周更新新的技能文件，紧跟 AI 发展和程序员实际需求。

## 🤝 参与贡献

欢迎贡献你的技能文件！详见 [CONTRIBUTING.md](CONTRIBUTING.md)。

1. Fork 本仓库
2. 在 `skills/` 目录下创建你的技能文件
3. 遵循现有技能文件的格式规范
4. 提交 Pull Request

## ⭐ Star History

如果觉得有用，请点个 Star！每一个 Star 都能帮助更多开发者发现这些技能文件。

[![Star History Chart](https://api.star-history.com/svg?repos=lvcn/ai-black-magic&type=Date)](https://star-history.com/#lvcn/ai-black-magic&Date)

## 📄 License

[Apache License 2.0](LICENSE) — 个人和商业使用均免费。

---

<div align="center">

**用魔法打败魔法。**

**AI 不会取代程序员，但会用 AI 的程序员会取代不会用的。**

如果你觉得有用，请点个 ⭐ Star，这是对我们最大的支持！

</div>
