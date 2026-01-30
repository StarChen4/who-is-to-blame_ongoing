---
name: who-is-to-blame
description: Guide for finding out who is to blame in a DoTA2 game using trustable date analysis and algorithm. This skill should be used when users want to find out who is to blame in a Dota2 game giving the game ID.
---

# who-is-to-blame

DOTA2比赛抓畜分析方法论 - 基于科学方法论找出那个畜生

## 简介

一个专业的DOTA2比赛分析方法论,能够基于多维度数据和科学方法论,客观地识别出比赛中表现最差的玩家(俗称"畜生")。

**核心原则**:
- 只分析输掉比赛的一方
- 基于客观数据,而非主观判断
- 考虑位置差异、克制关系、经济背景等调节因素
- 使用SSS到F级失误分类系统
- 智能控制畜生数量(通常1-2个)

## 触发条件

### 斜杠命令
- `/抓畜 <比赛ID>`
- `/找畜生 <比赛ID>`
- `/背锅侠 <比赛ID>`

### 自然语言(支持变体)
- "找出比赛{ID}中的畜生"
- "抓畜 {ID}"
- "分析比赛{ID}"
- "这局比赛谁背锅?ID是{ID}"
- "这局谁最坑?{ID}"
- "比赛{ID}谁最菜"

## 工作流程

### 1. 比赛ID验证与胜负判定
提取比赛ID，打开OpenDota判断哪方输了。

### 2. 数据获取
从STRATZ获取玩家-英雄映射和选人阶段数据，从OpenDota获取比赛数据，从Dota2.Tools获取克制关系。
→ 详见 [data-mapping.md](./references/data-mapping.md)

### 3. 位置识别
基于分路和经济数据确定1-5号位。
→ 详见 [data-mapping.md](./references/data-mapping.md#位置映射规则)

### 4. 失误识别与畜度计算
识别常规失误（SSS-F级）、选人责任（仅核心位）、视野责任（仅辅助位），计算最终畜度。
→ 详见 [analysis-logic.md](./references/analysis-logic.md)

### 5. 畜生判定
根据畜度排名确定1-2个畜生，考虑位置平衡。
→ 详见 [analysis-logic.md](./references/analysis-logic.md#畜生判定)

### 6. 生成报告并保存
生成Markdown报告，包含罪状、数据对比、惩罚建议，生成 `results/{match_id}.md`。
→ 详见 [output-template.md](./references/output-template.md)

## 详细说明

完整的实现细节请参考以下模块文档:

- **数据获取**: [./references/data-mapping.md](./references/data-mapping.md) - 详细的数据字段映射表
- **分析逻辑**: [./references/analysis-logic.md](./references/analysis-logic.md) - 失误识别规则和畜度计算
- **错误处理**: [./references/error-handling.md](./references/error-handling.md) - 异常情况处理
- **输出格式**: [./references/output-template.md](./references/output-template.md) - 报告生成模板

## 快速开始

### 使用示例

```bash
/抓畜 8669004890
```

或者:

```
帮我分析一下比赛8669004890,看看谁最坑
```

### 预期输出

工具会:
1. 显示进度提示(获取数据、分析等)
2. 生成详细的分析报告
3. 指出畜生并列举罪状
4. 给出幽默的惩罚建议
5. 保存结果到文件

## 技术依赖

- **Chrome DevTools MCP**: 用于网页数据爬取
- **OpenDota**: 比赛数据来源（基础数据、表现、分路、团战、视野）
- **STRATZ**: 选人阶段数据来源 (`https://stratz.com/matches/{match_id}`)
- **Dota2.Tools**: 英雄克制数据来源 (`https://dota2.tools/tools/hero-counter-viewer`)
- **方法论文档**: `/d/projects/who-is-to-blame/docs/methodology.md`

## 注意事项

- 需要稳定的网络连接
- 首次分析可能需要触发OpenDota的"分析"功能,耗时1-2分钟
- Dota2.Tools克制数据通过点击英雄按钮获取
- 仅分析输掉比赛的一方
- 分析结果基于客观数据,但可能无法完全反映游戏中的所有因素

## 版本信息

- **版本**: 1.0.0
- **创建时间**: 2026-01-30
- **方法论版本**: 基于 `/d/projects/who-is-to-blame/docs/methodology.md`
