# Contributing to AI Black Magic / 贡献指南

Thank you for your interest in contributing! 感谢你的参与！

This project thrives on community contributions. Whether you're fixing a typo or submitting a brand new Skill File, your help is welcome.

## How to Contribute a Skill File / 如何贡献技能文件

### 1. Understand the Format / 了解格式

Every Skill File follows this structure:

```markdown
# AI [Domain] [Role] Guide

> **定位**：说明这个文件让 AI 具备什么能力
> **目标读者**：说明适用于什么场景的什么人
> **参考标准**：列出引用的行业规范

---

## 目录
(按逻辑分章)

## 正文
(具体的规范、检查清单、模板、示例)

## 附录
(速查表、模板、参考资料)
```

### 2. Writing Principles / 编写原则

| Principle | Description |
|-----------|-------------|
| **Clear positioning** | State exactly what capability the AI gains after reading this file |
| **Professional standards** | Reference recognized industry practices (Alibaba, Google, OWASP, etc.) |
| **Actionable** | Include concrete checklists, templates, and decision trees — not vague advice |
| **China-ready** | Address real scenarios for Chinese developers (WeChat, Alipay, ICP, etc.) |
| **Bilingual friendly** | Chinese content is primary; key terms can include English equivalents |

### 3. Submission Process / 提交流程

1. **Fork** this repository
2. **Create** your Skill File in the `skills/` directory
3. **Name** it following the pattern: `AI-[DOMAIN]-[TYPE].md` (e.g., `AI-TESTING-AUTOMATION.md`)
4. **Test** your Skill File with at least one AI tool (Cursor, ChatGPT, or Claude) to verify it works
5. **Submit** a Pull Request with:
   - A clear title describing the Skill File
   - A brief description of what capability the AI gains
   - Which AI tools you tested with

### 4. Quality Checklist / 质量检查

Before submitting, make sure your Skill File:

- [ ] Has a clear positioning statement at the top
- [ ] References at least one industry standard or best practice
- [ ] Contains concrete checklists (not just principles)
- [ ] Includes at least one reusable template
- [ ] Has been tested with at least one AI tool
- [ ] Follows the existing file naming convention
- [ ] Is written primarily in Chinese with key English terms

## Other Ways to Contribute / 其他贡献方式

- **Report issues**: Found a problem in an existing Skill File? Open an Issue.
- **Suggest topics**: Have an idea for a new Skill File? Open an Issue with the `[Suggestion]` tag.
- **Improve docs**: Typo fixes, better examples, clearer instructions — all welcome.
- **Spread the word**: Star the repo, share it on social media, write about it.

## Code of Conduct / 行为准则

- Be respectful and constructive
- Focus on quality over quantity
- When in doubt, open an Issue to discuss before submitting a large PR

---

Thank you for helping make AI work smarter for everyone! 🧙‍♂️
