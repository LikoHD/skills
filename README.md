# Liko's Skills

这是一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) Skills 集合，包含自定义的专业技能模块，用于增强 Claude Code 在特定领域的分析能力。

## 什么是 Claude Code Skills?

Claude Code Skills 是 Claude Code CLI 工具的扩展能力，允许用户创建可复用的专业知识模块。每个 Skill 包含特定领域的专业知识、工作流程和最佳实践，让 Claude Code 能够更好地处理特定类型的任务。

## 包含的 Skills

### Analytics Tracker (网站埋点分析器)

一个专门用于分析网站埋点实现的 Skill，帮助理解网站如何收集用户行为数据。

**支持的埋点系统：**

| 平台类型 | 支持的系统 |
|---------|-----------|
| 国际平台 | Google Analytics (GA4/UA), Google Tag Manager, Facebook Pixel, Mixpanel, Amplitude, Segment |
| 国内平台 | 百度统计, 神策数据 (Sensors Data), 字节跳动 Finder/Ranger, 友盟, 腾讯分析 |

**核心功能：**

- 监控和识别网站的网络请求
- 自动识别主流埋点系统
- 解析和解码埋点数据（Base64, URL Encoding, Protobuf）
- 评估埋点质量（完整性、准确性、隐私合规）
- 生成结构化分析报告

**目录结构：**

```
analytics-tracker/
├── SKILL.md                    # 主技能文件
├── references/                 # 参考文档
│   ├── google-analytics.md     # GA 埋点规范
│   ├── sensors-data.md         # 神策数据规范
│   ├── finder-tracker.md       # 字节 Finder 规范
│   └── baidu-analytics.md      # 百度统计规范
└── templates/
    └── report-template.md      # 分析报告模板
```

## 如何使用

### 安装到 Claude Code

将本仓库的内容复制到你的 Claude Code skills 目录：

```bash
# 克隆仓库
git clone https://github.com/<your-username>/likos-skills.git

# 复制到 Claude Code skills 目录
cp -r likos-skills/* ~/.claude/skills/
```

或者直接在 Claude Code 项目目录下创建软链接：

```bash
ln -s /path/to/likos-skills ~/.claude/skills
```

### 使用 Skill

在 Claude Code 中，当你需要分析网站埋点时，可以直接调用：

```
/skill analytics-tracker
```

然后提供目标网站 URL 或 HAR 文件，Skill 会引导你完成整个分析流程。

## 贡献指南

欢迎提交 Issue 和 Pull Request 来改进这些 Skills！

### 添加新的 Skill

1. 在根目录创建新的 Skill 文件夹
2. 添加 `SKILL.md` 主文件，包含 YAML 前置数据
3. 添加必要的参考文档和模板
4. 更新本 README

### Skill 文件格式

每个 Skill 需要包含一个 `SKILL.md` 文件，格式如下：

```markdown
---
name: skill-name
description: Skill description for Claude Code to understand when to use it.
---

# Skill Title

Skill content...
```

## 许可证

MIT License

## 作者

Liko

---

> 本仓库的 Skills 设计用于 Claude Code，Anthropic 官方的 CLI 工具。
