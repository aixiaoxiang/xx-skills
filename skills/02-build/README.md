# 第二层·做出来

> 需要基本开发能力。把第一层想清楚的方案做出来。

## 本层包含

> 难度 1-3：1 入门、2 进阶、3 较难。平台"通用"指所有平台通用，"微信"指偏微信小程序实现。

| 顺序 | skill | 解决什么 | 难度 | 预计耗时 | 前置 skill | 后续 skill | 平台 |
|------|-------|---------|------|---------|-----------|-----------|------|
| 6 | xx-prd | MVP 该做什么功能 | 2 | 30 分钟 | xx-research | xx-data | 通用 |
| 7 | xx-data | 用数据迭代 AI | 2 | 30 分钟 | xx-prd | xx-safety | 通用 |
| 8 | xx-safety | 内容安全与风险防控 | 2 | 20 分钟 | xx-data | xx-backend | 通用 |
| 9 | xx-backend | 后端与数据存储怎么搭 | 2 | 30 分钟 | xx-safety | xx-setup | 微信 |
| 10 | xx-setup | 项目初始化与合规 | 2 | 30 分钟 | xx-backend | xx-brand | 微信 |
| 11 | xx-brand | 设计 Token 与品牌视觉 | 2 | 30 分钟 | xx-setup | xx-blocks | 微信 |
| 12 | xx-blocks | 可复用组件库 | 2 | 30 分钟 | xx-brand | xx-ai | 微信 |
| 13 | xx-ai | AI 能力接入 + Agentic 入门 | 3 | 45 分钟 | xx-blocks | xx-track | 微信 |

## 平台特化说明

xx-backend / xx-setup / xx-brand / xx-blocks / xx-ai 这 5 个 skill 偏微信小程序实现，归平台特化：

- xx-backend：单文件 SKILL.md，偏微信云开发实现
- xx-setup / xx-brand / xx-blocks / xx-ai：采用三层结构
  - SHARED.md：跨平台通用原则
  - wechat/SKILL.md：微信小程序实现（已完整）
  - web/：出海网站实现（待补充）

## 学完这一层后

| 你的状态 | 下一步 |
|---------|--------|
| 产品能跑了 → 上线 | 进 ../03-run/ 层，做数据埋点 |
| 还缺 AI 能力 → 重点看 xx-ai | 按 xx-ai 的模块一步步接入 |
| AI 质量不稳定 → 看 xx-data | 建评估集，用数据迭代 |
