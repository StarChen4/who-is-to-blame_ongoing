# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个 DOTA2 比赛分析工具（"抓畜工具"），基于科学方法论从 OpenDota 和 ProTracker 获取数据，客观识别输掉比赛中表现最差的玩家。

**核心功能**：分析比赛数据 → 计算畜度 → 找出"畜生" → 生成 Markdown 报告

## 项目结构

```
├── skills/who-is-to-blame/     # Skill 主目录
│   ├── skill.md                # Skill 入口文件（触发条件、工作流程）
│   └── references/             # 详细参考文档
│       ├── data-mapping.md     # 数据字段映射（如何从 OpenDota 获取数据）
│       ├── analysis-logic.md   # 分析逻辑（失误识别、畜度计算）
│       ├── error-handling.md   # 错误处理指南
│       └── output-template.md  # 报告输出模板
├── docs/methodology.md         # 科学方法论 v3.0（核心算法文档）
├── data/history.json           # 历史分析记录
└── results/                    # 分析报告输出目录（{match_id}.md）
```

## 核心架构

### 数据流

1. **输入**: 比赛 ID（8-10位数字）
2. **数据获取**:
   - OpenDota（概览/表现/分路/团战/曲线图 共5个页面）
   - ProTracker（英雄克制数据）
3. **分析引擎**: 失误识别 → 畜度计算 → 系数调整
4. **输出**: Markdown 报告 + history.json 更新

### 畜度计算公式

```
最终畜度 = 基础畜度 × 克制系数 × 位置系数 × 经济背景系数
```

- **基础畜度**: Σ(失误等级权重)，SSS级=100分，SS级=80分...
- **克制系数**: 0.7-1.3（被克制减免，克制对手加重）
- **位置系数**: 1/2号位×1.2，4/5号位×0.8-0.9
- **经济背景系数**: 0.7-1.2（全队劣势时减免）

### 失误分级系统

- **SSS级(+100)**: 核心位对线大崩、经济转化极差
- **SS级(+80)**: 对线严重劣势、团战贡献极低
- **S级(+60)**: 资源浪费、团战严重失误
- **A-F级(+10-40)**: 较轻失误

## 关键技术依赖

- **Chrome DevTools MCP**: 网页数据爬取（必须使用 MCP 工具访问 OpenDota/ProTracker）
- **数据源**:
  - OpenDota: `https://www.opendota.com/matches/{match_id}`
  - ProTracker: `https://dota2protracker.com/hero/{hero_name}`（克制数据唯一来源）

## 触发方式

```bash
# 斜杠命令
/抓畜 8669004890
/找畜生 8669004890
/背锅侠 8669004890

# 自然语言
"帮我分析比赛8669004890,看看谁最坑"
```

## 重要约定

1. **只分析输掉比赛的一方**
2. **克制数据严格使用 ProTracker**，不使用 OpenDota 的 matchups
3. **畜生数量控制**: 默认1个，最多2个
4. **位置平衡机制**: 避免"辅助总是背锅"
5. **报告保存路径**: `results/{match_id}.md`

## 开发状态

- ✅ 文档框架完成
- ⏳ 数据获取模块待实现
- ⏳ 分析引擎待实现
- ⏳ 报告生成待实现
