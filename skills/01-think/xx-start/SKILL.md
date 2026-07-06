---
name: xx-start
description: AI 产品入门导览技能。当用户是 AI 产品开发新手、不知道从哪里开始、或想快速了解 AI 产品和传统软件的区别时使用。给出 AI 产品的认知判断标准与执行约束，含一个 3 小时可跑通的 AI 起名工具参考实现，以及迁移到小象取色的说明。是整个技能包的入口。
---

# 从零开始：AI 产品入门

> 在你看任何方法论之前，先花 10 分钟建立认知，再决定下一步。

## 这个技能做什么

帮你在 10 分钟内搞清楚 3 件事：
1. AI 产品到底是什么，和普通软件有什么区别
2. 用户需要会什么、不需要会什么
3. 第一个 AI 产品该从哪开始

本技能面向"用自然语言提需求、看结果提优化"的用户：你不需要手敲代码，下面的代码示例是 AI 的参考实现。你要做的是判断 AI 输出好坏、用截图和复述把不对的地方反馈回去。

**AI 执行约束：** 讲解时优先给"认知判断标准"和"执行约束"，不要把代码示例拆成手把手教程。用户能跑通端到端流程即可，不要求用户读懂每一行。

---

## 1. 什么是 AI 产品

AI 产品 = 用 AI 模型解决用户问题的产品。

和传统软件的 3 个根本差异：

| 维度 | 传统软件 | AI 产品 |
|------|---------|---------|
| 输出确定性 | 相同输入永远相同输出 | 相同输入可能有不同输出 |
| 成本结构 | 主要是服务器成本 | 每次调用都要付费（token 成本）|
| 失败模式 | 崩溃或报错 | 可能"看起来对但其实错了"（幻觉）|

这 3 个差异决定了做 AI 产品的思路不同：要管成本、要降级容错、要评估质量。

**判断标准：** 如果一个产品同时满足"输出不确定 + 按调用付费 + 可能静默出错"，它就是 AI 产品，必须按 AI 产品的思路做（成本、容错、质量评估三件套）。

---

## 2. 一个最小 AI 产品长什么样

端到端结构：

```
用户输入 → 你的代码 → 调 AI 接口 → AI 返回结果 → 你的代码解析 → 用户看到
```

代码层只需要做 3 件事：
- 调 API（云函数里 fetch 一下）
- 解析 JSON（拿到 AI 返回的内容）
- 处理失败（AI 超时或报错时不崩）

**AI 执行约束：** 生成代码时必须同时实现这 3 件事。缺"处理失败"的代码不算完成——AI 接口会超时、会限流、会返回空，必须有降级或提示，不允许白屏。

---

## 3. 用户需要会的最少知识

**需要会的（人）：**
- 能用自然语言描述需求
- 能判断 AI 输出好坏并说出哪里不对
- 会截图复述问题反馈给 AI
- 有微信小程序账号和 AI API key

**不需要会的（别被吓到）：**
- 机器学习原理
- 模型训练
- 数学
- GPU / 算力
- 手写每一行代码

你不需要懂模型怎么训练的，就像你不需要懂数据库怎么实现的，照样能用。

**判断标准：** 用户能回答"这次生成的结果对不对、哪里不对、该怎么改"就够了。不需要用户解释代码为什么这么写。

---

## 4. 第一个练习：3 小时做一个 AI 起名工具

场景：用户输入行业关键词，AI 返回 5 个品牌名建议。

为什么选这个练习：
- 最简单：一个输入框 + 一个按钮 + 一个结果展示
- 能跑通：验证端到端流程
- 有成就感：3 小时拿到一个能用的东西

> 以下代码是 AI 的参考实现。用户只需用自然语言描述"我要一个起名工具"，AI 按此实现；用户看结果、提优化，不必逐行对照手敲。

### 4.1 云函数参考实现

```javascript
// cloudfunctions/nameGen/index.js
const cloud = require('wx-server-sdk')
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV })

exports.main = async (event, context) => {
  const { keyword } = event
  if (!keyword) {
    return { ok: false, msg: '关键词不能为空' }
  }

  try {
    const res = await cloud.callModel({
      name: 'hunyuan-pro',
      data: {
        prompt: `你是一个品牌起名专家。用户输入行业关键词"${keyword}"，请返回 5 个有创意、易记忆、有寓意的品牌名建议。每个名字用一行输出，格式：名字 - 寓意说明。`
      }
    })

    return { ok: true, names: res.choices[0].message.content }
  } catch (err) {
    return {
      ok: false,
      msg: 'AI 服务暂时不可用，请稍后再试',
      fallback: ['建议1', '建议2', '建议3']
    }
  }
}
```

### 4.2 前端页面参考实现

```html
<!--pages/index/index.wxml-->
<view class="container">
  <view class="title">AI 起名工具</view>
  <input class="input" placeholder="输入行业关键词，如：咖啡"
         value="{{keyword}}" bindinput="onInput" />
  <button class="btn" bindtap="onGenerate" loading="{{loading}}">
    生成品牌名
  </button>
  <view class="result" wx:if="{{result}}">
    <view wx:for="{{names}}" wx:key="index" class="item">
      {{item}}
    </view>
  </view>
</view>
```

```javascript
// pages/index/index.js
Page({
  data: {
    keyword: '',
    loading: false,
    result: '',
    names: []
  },

  onInput(e) {
    this.setData({ keyword: e.detail.value })
  },

  async onGenerate() {
    if (!this.data.keyword) {
      wx.showToast({ title: '请输入关键词', icon: 'none' })
      return
    }

    this.setData({ loading: true })
    try {
      const res = await wx.cloud.callFunction({
        name: 'nameGen',
        data: { keyword: this.data.keyword }
      })

      if (res.result.ok) {
        this.setData({
          names: res.result.names.split('\n').filter(n => n.trim()),
          result: res.result.names
        })
      } else {
        wx.showToast({ title: res.result.msg, icon: 'none' })
      }
    } catch (err) {
      wx.showToast({ title: '网络错误', icon: 'none' })
    }
    this.setData({ loading: false })
  }
})
```

### 4.3 验收清单

- [ ] 能在真机上跑通
- [ ] 输入有输出（不是 loading 转圈）
- [ ] AI 报错时有提示，不白屏

**AI 执行约束：** 三项任一不满足，必须回去修，不能算完成。尤其第三项——AI 报错白屏是最常见的初学者坑。

---

## 5. 从起名工具迁移到小象取色

起名工具验证的是"输入文本 → AI 生成文本"的最小闭环。小象取色在此基础上多了"输入色值 → AI 理解 → 生成结构化输出"：

| 能力 | 起名工具 | 小象取色 |
|------|---------|---------|
| 输入 | 文本关键词 | 摄像头取色的 Hex |
| AI 任务 | 生成 5 个名字 | 命名（故宫红/天青色）+ 配色方案 |
| 输出 | 文本列表 | 结构化 JSON（色名 + 出处 + 配色）|
| 失败降级 | 返回占位名字 | 返回 Hex + 通用色名（深红色）|

**迁移要点：**
- 把"输入框"换成"摄像头取色"，取色拿到 Hex 后当关键词传给 AI
- 把 prompt 从"返回 5 个名字"改成"返回 JSON：{name, origin, palette:[]}"
- 解析 JSON 部分逻辑一致，只是字段更多
- 降级方案：AI 挂了，至少返回 Hex + "深红色"这种通用名，不能白屏

**AI 执行约束：** 迁移时不要重写整个项目，应基于起名工具的结构增量改造：输入源、prompt、解析、降级四处替换即可。用户说"把起名工具改成取色命名"时，按此四处对应改，不要推倒重来。

---

## 6. 学完之后去哪

根据状态选下一步：

| 状态 | 下一步 |
|---------|--------|
| 还没决定做不做 | 用 `xx-ai-or-not` 判断要不要用 AI |
| 决定做了但没想清楚 | 用 `xx-goal` 定目标 → `xx-business` 理商业模式 |
| 想清楚了准备开发 | 进 `02-build/` 层，从 `xx-prd` 开始 |
| 已经上线了 | 直接进 `03-run/` 层 |

---

## 使用方式

把状态告诉我，我帮你定位下一步该用哪个技能，或直接帮你跑通第一个练习。

**对话示例：**
- "我是纯新手，想做个能用摄像头取色命名的微信小程序" → 我先带你跑通起名工具，再讲怎么迁移到取色命名
- "我跑通了起名工具，怎么改成取色命名的" → 我按迁移要点帮你改输入源、prompt、解析、降级
- "我不知道这个练习跑通后算不算成功" → 我用验收清单 3 项帮你检查

---

## 与其他技能的关系

- **入口：** 本技能是整个技能包的起点，跑通第一个练习后进入后续技能
- **后续：** 决定做不做 → `xx-ai-or-not`；想清楚目标 → `xx-goal` → `xx-business`；准备开发 → `02-build/` 层
- **并行：** 跑练习时如想做用户验证，可并行用 `xx-research`
- **回溯：** 上线后用 `xx-track` 监控 AI 质量，回看这里讲的"成本、容错、质量评估"是否落地

## 方法论来源

- **最小可行闭环** — Lean Startup（Eric Ries）的 Build-Measure-Learn 最小版本
- **AI 产品三差异** — 实践总结：确定性、成本结构、失败模式
