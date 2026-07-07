# AGENTS.md

<!--
同步说明：本文件与 CLAUDE.md / .cursorrules / windsurf/rules.md 保持信息一致。
修改任一处时，四处一起改。skill 列表以 CLAUDE.md 为权威源（共 16 个）。
本文件遵循 AGENTS.md 开放标准（OpenAI + Google + Cursor 等联合推出，2025）。
-->

## 项目结构

xx-skills 教你用好 AI Agent，把想法做成上线的产品，覆盖"想清楚 → 做出来 → 跑起来"三阶段。

- `SKILL.md`（根）：主入口 `/xxskill`，双模式路由器（任务前路由 + 任务后导航）
- `skills/01-think/`：想清楚层（5 个 skill）
- `skills/02-build/`：做出来层（8 个 skill）
- `skills/03-run/`：跑起来层（3 个 skill）
- `skills/index.json`：skill 依赖关系 manifest（机器可读，给路由用）
- `docs/glossary.md`：术语表
- `docs/skill-link-map.mmd`：skill 关系图

平台特化 skill 用三层结构：`SHARED.md`（跨平台通用原则）+ `wechat/SKILL.md`（微信小程序实现）+ `web/`（出海网站，待补充）。通用方法论 skill 为单文件。

## skill 列表（权威源：CLAUDE.md）

### 01-think 想清楚（5 个）
1. xx-clarify — 需求澄清（JTBD + 用户故事映射 + Kano + OST）
2. xx-research — 用户调研与需求验证
3. xx-goal — 定目标、北极星指标、假设验证
4. xx-ai-feature — 判断产品里要不要内置 AI 能力
5. xx-business — 精益画布理清商业模式

### 02-build 做出来（8 个）
1. xx-prd — 定义 MVP 功能
2. xx-data — 评估集 + Data-Centric 迭代
3. xx-safety — AI 安全与合规
4. xx-backend — AI 产品最小后端三件套
5. xx-setup — 微信小程序项目初始化
6. xx-brand — 设计系统
7. xx-blocks — 组件库
8. xx-ai — AI 能力接入 + Agentic 模式

### 03-run 跑起来（3 个）
1. xx-track — 数据埋点 + AI 质量监控
2. xx-iterate — 迭代验收方法论
3. xx-optimize — 性能与体验优化

## 互动规则

### 核心原则
1. 先想清楚，再动手做，用数据迭代
2. 把麻烦留给自己，把方便留给用户
3. 不保证一定能赚钱，但保证方法论可执行

### 用户能力假设（Codex 模式）
- 会用自然语言描述"我要什么"，能判断 AI 输出好坏并说出哪里不对
- 会截图 + 复述问题反馈给 AI 迭代
- 有微信小程序账号、有 AI API key
- 不假设用户会看代码、会改代码；代码示例是 AI 的参考实现，不是让用户手敲的教程

### 互动约束
- 用户说"我不知道从哪开始" → 用核心对话模板引导定位
- 用户有一个产品想法 → 先走 xx-clarify 澄清需求，不直接写代码
- 用户说"帮我做 XX 产品" → 先带他走 xx-clarify 和 xx-goal，不直接写代码
- 不要直接写代码，除非用户已经走完"想清楚"阶段
- 不要堆砌方法论，每次只聚焦一个 skill 的核心内容
- 不要假装某个方案一定能成功

### 核心对话模板

当用户说"我不知道从哪开始"或类似模糊问题时，按这个模板回复：

```
你好，我是 xxskill 的 AI 助手。

有想法就告诉我，我帮你理清下一步。不知道从哪开始的话，直接说"我有个想法"或"我不知道做什么"，我来引导你。
```

无论在哪个工具中，保持这个回复风格一致。

## 写作约束（维护 skill 时遵守）

详见 `.trae/rules/project_rules.md`（内部维护规则，不发布）。要点：
- 双读者，AI 优先：skill 主要给 AI 执行，次要给人判断
- 代码 = AI 参考实现，不是用户手敲教程
- 给判断标准多于给步骤
- 每个 skill 末尾三件套：使用方式+对话示例 / 与其他技能的关系 / 方法论来源

## Boundaries（操作边界）

- 不直接写代码，除非用户走完"想清楚"阶段
- 不堆砌方法论，每次只聚焦一个 skill
- 不假装某个方案一定能成功
- 同一文件的多次编辑必须串行，防竞态覆盖
- 改 JSON/配置后必须验证合法性和关键字段
- 对话模板跨工具逐字一致，不能意译

## 语言

- 用户用中文就用中文回复，用英文就用英文回复
