# xxskill — 小象AI产品Builder技能包

> 用 AI Agent 把想法变成上线的产品。

---

## 快速元信息

| 项目 | 说明 |
|------|------|
| 最新版本 | v3.0.4（2026-07-07） |
| 包含技能 | 16 个 |
| 覆盖阶段 | 想清楚 → 做出来 → 跑起来 |
| 当前聚焦 | 微信小程序（出海网站场景规划中） |
| 适合人群 | 想用 AI 做产品、但不靠手写代码的人——有想法没方法的初学者、有技术没产品的独立开发者、有流量不知道怎么变现的人 |
| 预计总学习 | 6-10 小时（纯 skill 学习约 7 小时，含练习更长） |
| 使用方式 | Codex 模式：自然语言提需求，AI 执行，你看结果提优化，不要求会写代码 |
| 核心原则 | 先想清楚，再动手做，用数据迭代 |

---

## 这个技能包解决什么问题

如果你遇到下面这些情况，这个技能包就是为你写的：

- 看了很多 AI 编程教程，但最后还是做不出一个能用的产品
- 有想法，但不确定值不值得做、要不要用 AI
- 已经做了 MVP，但不知道用户会不会用、下一步该优化什么
- 有流量，但不知道怎么把流量变成可持续的生意
- 想做 AI 产品，但担心 AI 输出不稳定、成本不可控

**我们不教你写代码，我们教你用好 AI Agent，把想法做成上线的产品。**

---

## 这个技能包不做什么

为了让你不浪费时间，先说明边界：

- 不教大模型训练、微调、数学原理
- 不保证学了一定能赚钱（我们要诚实）
- 不做海外网站场景的具体实现（目前只覆盖微信小程序，出海网站场景正在规划中）
- 不适合已经有成熟产品方法论、不需要手把手引导的人

---

## Codex 模式：怎么用这个技能包

本技能包面向"用自然语言表达需求、看输出结果提优化"的用户。你不需要会写代码，只需要：

- 会用自然语言描述"我要什么"
- 能判断 AI 输出的好坏，并说出哪里不对
- 会截图复述问题，把问题反馈给 AI
- 有微信小程序账号和 AI API key

代码示例是 AI 的参考实现，不是让你手敲的教程。你的角色是"提需求 + 验收"，AI 的角色是"执行 + 改进"。

| 环节 | 你做什么 | AI 做什么 |
|------|---------|----------|
| 提需求 | 用自然语言说清"我要什么、给谁用、长什么样" | 理解需求，给出实现方案 |
| 看输出 | 运行结果，判断好不好用，截图复述问题 | 按你的反馈修改，再给一版 |
| 提优化 | 指出哪里不对、想要什么效果 | 调整实现，直到你满意 |

> 贯穿全包的案例是"小象取色"——一个用 AI 帮你从照片里提取配色的小程序。所有 skill 都用它做示范，方便你对照学习。

---

## 你真正学会的是：用好 AI Agent

表面看，这个技能包带你做一个小程序产品。实质是：**通过做一个产品，掌握"用好 AI Agent"这项能力。**

- 产品是练习场，也是成果证明——做出来，就证明你学会了和 AI Agent 协作
- 这项能力可迁移：下一个产品、换个赛道，用同样的协作方法就能做
- 所以本技能包聚焦微信小程序，但不止于微信小程序

---

## 我们的方法：3 层学习路径

这套技能包把做产品分成 3 个阶段。不要跳过第一层直接写代码——这是大多数人失败的原因。

### 第一层·想清楚（零技术门槛，适合所有人）

**你有一个想法，但不确定值不值得做、要不要用 AI。**

| 顺序 | 技能 | 解决什么 | 预计时间 |
|------|------|---------|---------|
| 1 | 01-think/xx-clarify | 需求澄清，把模糊想法变结构化 | 30 分钟 |
| 2 | 01-think/xx-research | 怎么做轻量调研，验证假设 | 20 分钟 |
| 3 | 01-think/xx-goal | 为谁解决什么问题、怎么判断成功 | 20 分钟 |
| 4 | 01-think/xx-ai-feature | 判断产品要不要 AI 能力 | 15 分钟 |
| 5 | 01-think/xx-business | 商业模式跑不跑得通 | 20 分钟 |

### 第二层·做出来（需要基本开发能力）

**你想清楚了，开始动手开发。**

| 顺序 | 技能 | 解决什么 | 平台 |
|------|------|---------|------|
| 6 | 02-build/xx-prd | MVP 该做什么功能 | 通用 |
| 7 | 02-build/xx-data | 用数据迭代 AI，不只靠 prompt | 通用 |
| 8 | 02-build/xx-safety | 内容安全与风险防控 | 通用 |
| 9 | 02-build/xx-backend | 后端与数据存储怎么搭 | 通用 |
| 10 | 02-build/xx-setup | 项目初始化与合规 | 微信小程序 |
| 11 | 02-build/xx-brand | 设计 Token 与品牌视觉 | 微信小程序 |
| 12 | 02-build/xx-blocks | 可复用组件库 | 微信小程序 |
| 13 | 02-build/xx-ai | AI 能力接入 + Agentic 模式入门 | 微信小程序 |

### 第三层·跑起来（产品上线后）

**产品上线了，需要数据驱动迭代。**

| 顺序 | 技能 | 解决什么 | 平台 |
|------|------|---------|------|
| 14 | 03-run/xx-track | 数据埋点 + AI 质量监控 | 微信小程序 |
| 15 | 03-run/xx-iterate | 迭代方法论与验收 | 通用 |
| 16 | 03-run/xx-optimize | 性能与体验优化 | 微信小程序 |

---

## 平台支持说明

当前聚焦**微信小程序**场景，因为微信小程序是普通人用 AI 做产品最容易形成闭环的地方：门槛低、有自然流量、用户反馈快。

架构上已预留**出海网站**扩展空间。16 个 skill 分两类：

**通用方法论**（单文件 SKILL.md，所有平台通用）：

- 01-think 全部 5 个：xx-clarify、xx-research、xx-goal、xx-ai-feature、xx-business
- 02-build 中的：xx-prd、xx-data、xx-safety、xx-backend
- 03-run 中的：xx-iterate

**平台特化**（SHARED.md + wechat/SKILL.md + web/ 三层结构）：

- 02-build 中的：xx-setup、xx-brand、xx-blocks、xx-ai
- 03-run 中的：xx-track、xx-optimize

平台特化目录结构：

- SHARED.md：跨平台通用原则
- wechat/SKILL.md：微信小程序实现（已完整）
- web/：出海网站实现（待补充）

如果你有出海网站开发经验想贡献，欢迎补充 web/ 目录。

---

## 每个 skill 的导航元信息

每个 skill 除了"解决什么"，还带一组导航元信息，帮你判断"现在该不该学、学完去哪"。在各层 README 里有一张完整导航表，字段如下：

| 字段 | 含义 |
|------|------|
| 难度（1-3） | 1 入门、2 进阶、3 较难 |
| 预计耗时 | 纯学习时间，不含动手做产品 |
| 前置 skill | 学这个之前最好先学哪个 |
| 后续 skill | 学完这个自然衔接哪个 |
| 平台 | 通用 / 微信 |

详见：

- [skills/01-think/README.md](skills/01-think/README.md)
- [skills/02-build/README.md](skills/02-build/README.md)
- [skills/03-run/README.md](skills/03-run/README.md)

---

## 不确定从哪开始？

按你的状态选：

| 你的状态 | 从哪开始 |
|---------|---------|
| 完全新手，没做过 AI 产品 | 从  01-think/xx-clarify 开始，把想法澄清 |
| 有想法，但不确定要不要用 AI | 先看  01-think/xx-ai-feature |
| 决定做了，但没想清楚目标 | 从  01-think/xx-goal 开始 |
| 已经在开发，遇到具体问题 | 从  02-build/ 层选对应的 skill |
| 已经上线，想优化转化/质量 | 从  03-run/xx-track 开始 |

---

## 入口 skill：xxskill

SKILL.md 是整个技能包的入口。它是一个双模式路由器：任务前帮你定位从哪开始，任务后帮你导航到下一个 skill。

在 Trae 中，输入 /xxskill 即可唤起入口 skill，它会问你：

> 你在哪一层？告诉我你现在的状态，我帮你定位。

然后它会根据你的回答，推荐你进入：

- **想清楚** → 01-think/ 层
- **做出来** → 02-build/ 层
- **跑起来** → 03-run/ 层

你也可以直接打开 SKILL.md，把内容复制到 AI 对话中使用。

---

## 在不同 AI 编程工具中使用

这个 skill 包是纯 Markdown 格式，不依赖任何特定平台。你可以在各种 AI 编程工具中使用。

### Claude Code

Claude Code 会读取项目根目录下的 CLAUDE.md 作为全局上下文。

1. 将本仓库根目录的 CLAUDE.md 复制到你项目的根目录（或直接用本仓库作为项目）
2. 启动 Claude Code，它会自动识别 CLAUDE.md
3. 输入类似"帮我从 xx-clarify 开始"的指令，Claude 会按 3 层路径引导你

如果你不想用 CLAUDE.md，也可以直接打开任意 skill 的 SKILL.md，复制内容粘贴到 Claude Code 对话中。

### OpenAI Codex / Codex CLI

Codex 可以通过项目级指令文件或对话 prompt 使用。

1. 将 AGENTS.md 的内容复制到你项目的根目录 AGENTS.md（AGENTS.md 是 OpenAI 2025 年推出的开放标准，被 Codex 等工具识别）
2. 启动 Codex 时，它会自动加载这些指令
3. 你也可以直接复制某个 skill 的 SKILL.md 内容到 Codex 对话中

### Trae IDE

Trae 原生支持 skill。

1. 将整个 xx-skills 文件夹复制到你项目的 .trae/skills/ 目录下
2. 打开 Trae 的 AI 对话，输入 /xxskill 唤起入口 skill，或输入对应 skill 名（如 /xx-clarify）
3. AI 会自动读取 SKILL.md 中的 name 和 description，并按方法论引导你

### Cursor

Cursor 支持 .cursorrules 文件作为项目级规则。

1. 将本仓库根目录的 .cursorrules 复制到你项目的根目录
2. Cursor 会自动读取并应用这些规则
3. 在 Composer 或 Chat 中，直接引用 skill 名或复制 SKILL.md 内容即可

### 其他 AI 编程工具

通用方法：

1. 找到你当前阶段对应的 skill 目录
2. 打开 SKILL.md
3. 复制全部内容
4. 粘贴到你的 AI 对话中
5. 告诉 AI"请按这个 skill 的方法论帮我"

支持此方式的工具包括但不限于：Windsurf、Cline / Roo Code 等。

---

## 三种使用场景

除了不同工具，你也可以按不同场景使用：

### 场景一：让 AI 全程带你做

把整个 skill 包作为 AI 的上下文，让它按 3 层路径引导你。适合完全新手。

### 场景二：针对具体问题查 skill

遇到具体问题（如"要不要用 AI""怎么做数据埋点"）时，只打开对应的 skill，复制到 AI 对话中。

### 场景三：当检查清单用

每个 SKILL.md 都可以独立阅读，按里面的清单逐项自检。适合已经有经验的人查漏补缺。

---

## 推荐下一步

如果你刚拿到这个技能包，建议按这个顺序：

1. **10 分钟**：读完这篇 README，判断自己是否适合用
2. **15 分钟**：打开  01-think/xx-clarify，把你的想法澄清
3. **30 分钟**：走一遍  01-think/xx-ai-feature 和  01-think/xx-goal，确认自己的方向
4. **3 小时**：用 xx-clarify 把你的想法澄清成结构化问题定义，跑通第一个端到端 AI 功能
5. **1 天**：用  02-build/ 的技能把你的第一个 MVP 做出来
6. **上线后**：用  03-run/xx-track 埋点，用  03-run/xx-iterate 迭代

---

## 这个技能包的来源

这个技能包不是凭空写的。它来自：

- 10+ 个已上线微信小程序项目的实战经验
- Andrew Ng 的 Data-Centric AI 思想
- Lean Startup、JTBD、MoSCoW、North Star 等产品方法论
- 多轮审核和优化，重点解决初学者最常踩的坑

我们把它整理成一套可以直接照着做的技能，目标是**让你少走弯路**。

---

## 加入付费答疑群

技能包是免费的，但做产品的过程里你肯定会遇到具体问题。

**加微信号：HelpAgent**，备注"xxskill"，拉你进群。

---

## 已知问题与待验证

我们诚实地列出还没完全验证的部分：

- **web/ 出海网站实现待补充**：目前只覆盖微信小程序，出海网站场景需要更多真实项目验证
- **xx-data 的评估方法论**：在复杂 AI 场景下可能需要调整评估集规模和指标
- **初学者真实学习曲线**：16 个 skill 的总学习时长是估算，需要根据用户反馈调整
- **Codex 模式的真实可用性**：全部 skill 转为 codex 模式后，"不写代码、只提需求"的工作流是否真的顺畅，需要用户反馈验证
- **Agentic 章节深度**：模块四只讲入门，复杂 Agent 工作流需要后续扩展
- **平台命名是否便于记忆**：xx-clarify / xx-ai-feature 等新命名需要用户验证

如果你在使用过程中发现问题，欢迎反馈。

---

## 附录

### 文件结构

```text
xx-skills/
├── README.md              # 本文件
├── SKILL.md               # Trae IDE 入口 skill
├── CLAUDE.md              # Claude Code 项目级指令
├── VERSION                # 版本号
├── LICENSE
├── .cursorrules           # Cursor 项目级规则
├── AGENTS.md              # OpenAI Codex 开放标准指令
├── .claude-plugin/
│   └── marketplace.json
├── windsurf/
│   └── rules.md           # Windsurf 项目级规则
├── docs/
│   ├── glossary.md        # 术语表
│   └── skill-link-map.mmd # skill 关系图
├── skills/
│   ├── index.json         # skill 依赖 manifest
│   ├── 01-think/          # 想清楚
│   │   ├── README.md
│   │   ├── xx-clarify/
│   │   ├── xx-research/
│   │   ├── xx-goal/
│   │   ├── xx-ai-feature/
│   │   └── xx-business/
│   ├── 02-build/          # 做出来
│   │   ├── README.md
│   │   ├── xx-prd/
│   │   ├── xx-data/
│   │   ├── xx-safety/
│   │   ├── xx-backend/
│   │   ├── xx-setup/
│   │   ├── xx-brand/
│   │   ├── xx-blocks/
│   │   └── xx-ai/
│   └── 03-run/            # 跑起来
│       ├── README.md
│       ├── xx-track/
│       ├── xx-iterate/
│       └── xx-optimize/
```

### 版本历史

| 版本 | 时间 | 主要变化 |
|------|------|---------|
| v3.0.4 | 2026-07-07 | 初学者视角全面审核：修正 02-build/README 依赖关系错误（线性链→三条线 DAG）、AI 产品表述改条件式（6 个 skill）、xx-safety/xx-backend 补读者说明、补产品不含 AI 路径、glossary 补 Mom Test/修 FID→INP/修标题 |
| v3.0.3 | 2026-07-07 | 修复版本号同步 bug（README 最新版本行）；定位表述统一（「做 AI 产品」→「做产品/用 AI 做产品」）；SKILL.md description 补「上线」 |
| v3.0.2 | 2026-07-07 | 价值定位同步为「用好 AI Agent 把想法做成上线的产品」；README 新增「你真正学会的是」段；修复定位文案残留 |
| v3.0.1 | 2026-07-07 | xx-clarify 默认轻量出卡、入口对话模板精简为 3 句、xx-ai-feature 附录化，修复 xx-clarify 定位与 xx-ai-feature 前置依赖 bug |
| v3.0 | 2026-07-06 | 入口改名 xxskill 并吸收 xx-start 导览（双模式路由）；新增 xx-clarify（需求澄清，JTBD+用户故事映射+Kano+OST）；xx-ai-or-not 改名 xx-ai-feature 并改语义（产品里要不要 AI 能力）；新增 AGENTS.md（Codex 开放标准）；删除 .codex/instructions.md |
| v2.1 | 2026-07-06 | 新增 xx-research / xx-safety / xx-backend，全部 skill 转 codex 模式，统一小象取色案例，新增术语表和依赖 manifest，IDE 规则文件一致性补齐 |
| v2.0 | 2026-07-05 | 重组为 3 层结构，新增 xx-start / xx-ai-or-not / xx-data，补齐 AI 评估、监控、Agentic 入门 |
| v1.0 | 2026-07-04 | 初始 10 个 skill，扁平结构 |

### 反馈方式

如果你在使用过程中遇到问题，或有改进建议：

- 在 GitHub 提 issue
- 加微信号 **HelpAgent**，备注"xxskill"

### License 说明

本项目采用**自定义授权协议**（详见 [LICENSE](./LICENSE)），一句话概括：

- ✅ **个人学习、开发自己的产品、非付费教学、社区分享** — 随便用，保留署名即可
- ❌ **付费课程、企业内训、批量分发、嵌入商业产品销售** — 需取得商业授权

商业授权联系微信：**HelpAgent**
