---
name: xx-ai
description: AI能力接入与Agentic模式技能。当用户需要在小程序中调用大模型API（混元/豆包）、实现AI流式输出、处理AI错误降级、用摄像头取色、用Canvas生成配色海报、做颜色算法处理，或需要Agentic工具调用与结构化输出时使用。覆盖从模型调用到Agentic入门的完整AI开发能力。AI 按本规范生成 AI 调用与降级代码，用户按质量标准验收。
---

# AI 能力接入与 Agentic 模式

> AI 不是魔法，是工程。降级、缓存、流式、成本，每一样都要写代码兜底。

## 这个技能做什么

帮你在小程序里接入 AI 能力，分四个模块：
1. **模块一：AI 大模型调用** — 混元/豆包接入、流式输出、错误降级
2. **模块二：Camera + Canvas 海报生成** — 摄像头取色、Canvas 绘制海报
3. **模块三：颜色处理算法** — RGB/HSL 转换、相似度计算
4. **模块四：Agentic 模式入门** — 工具调用、结构化输出、防死循环

**读者说明：** 本 skill 主要给 AI 执行规范（模型调用、降级、Canvas 绘制代码），次要给人判断标准（AI 质量是否达标、降级是否健壮）。用户不需要自己写云函数，只需要会描述"我要取色后让 AI 起中式名字"、能截图说"这个命名不对，应该叫故宫红"。

贯穿案例：**小象取色**，全模块围绕"取色→AI 命名→配色海报→历史记录"展开。

---

## 模块一：AI 大模型调用

### 1.1 前置准备

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1 | 腾讯云控制台开通混元大模型 | 免费额度内可用，境内已备案 |
| 2 | 获取 SecretId / SecretKey | 存入云函数环境变量，不写代码 |
| 3 | 火山引擎开通豆包大模型 | 作为备选模型 |
| 4 | 云函数安装 tencentcloud-sdk | `npm i tencentcloud-sdk-nodejs-hunyuan` |

> key 管理与缓存见 xx-backend；此处只展示调用逻辑。

### 1.2 混元调用模板

**AI 参考实现（云函数 aiNameColor/index.js）：**

```js
const cloud = require('wx-server-sdk')
const tencentcloud = require('tencentcloud-sdk-nodejs-hunyuan')

cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV })

const client = new tencentcloud.hunyuan.v20230901.Client({
  credential: {
    secretId: process.env.TENCENT_SECRET_ID,
    secretKey: process.env.TENCENT_SECRET_KEY
  },
  region: 'ap-beijing',
  profile: {
    httpProfile: { endpoint: 'hunyuan.tencentcloudapi.com' }
  }
})

exports.main = async (event, context) => {
  const { hex } = event
  try {
    const result = await callHunyuan(hex)
    return { code: 0, data: result }
  } catch (err) {
    console.error('混元调用失败:', err)
    const fallback = await callDoubao(hex)
    if (fallback) return { code: 0, data: fallback }
    return { code: 0, data: { name: genericName(hex), source: 'fallback' } }
  }
}

async function callHunyuan(hex) {
  const params = {
    Model: 'hunyuan-pro',
    Messages: [
      { Role: 'system', Content: '你是中式色彩专家。根据色值返回一个中国传统色名，只返回色名，不要解释。' },
      { Role: 'user', Content: `色值：${hex}` }
    ]
  }
  const resp = await client.ChatCompletions(params)
  return { name: resp.Choices[0].Message.Content.trim(), source: 'hunyuan' }
}

async function callDoubao(hex) {
  return null
}

function genericName(hex) {
  const { r, g, b } = hexToRgb(hex)
  if (r > 200 && g < 100 && b < 100) return '红'
  if (r < 100 && g > 150 && b < 150) return '青'
  if (r > 200 && g > 200 && b < 100) return '黄'
  return '色'
}

function hexToRgb(hex) {
  hex = hex.replace('#', '')
  return {
    r: parseInt(hex.substring(0, 2), 16),
    g: parseInt(hex.substring(2, 4), 16),
    b: parseInt(hex.substring(4, 6), 16)
  }
}
```

**AI 执行约束：** key 必须从 `process.env` 读，禁止硬编码；主模型失败必须降级到备模型，备模型失败必须返回兜底结果（不能让用户看报错）。

### 1.3 流式输出（SSE）

**AI 参考实现（云函数流式调用 + 前端接收）：**

```js
async function callHunyuanStream(hex, onChunk) {
  const params = {
    Model: 'hunyuan-pro',
    Stream: true,
    Messages: [
      { Role: 'system', Content: '你是中式色彩专家。根据色值返回一个中国传统色名和简短释义。' },
      { Role: 'user', Content: `色值：${hex}` }
    ]
  }

  const stream = await client.ChatCompletions(params)
  let fullText = ''
  for await (const chunk of stream) {
    const delta = chunk.Choices[0].Delta.Content || ''
    fullText += delta
    onChunk(delta)
  }
  return fullText
}
```

**前端伪流式（云函数不支持 SSE 透传时）：**

```js
async function callNameColor(hex) {
  const res = await wx.cloud.callFunction({
    name: 'aiNameColor',
    data: { hex, stream: true }
  })
  return res.result.data
}

Page({
  async onPicked(hex) {
    const streamComp = this.selectComponent('#aiStream')
    streamComp.setData({ thinking: true })

    const result = await callNameColor(hex)
    streamComp.setData({ thinking: false })
    streamComp.startStream(result.name)
  }
})
```

**判断标准（给人）：** 命名结果有没有"打字机"效果？中断后已生成内容是否保留？

**AI 执行约束：** 流式组件 detached 时必须清定时器（见 xx-blocks ai-stream）；中断不能整段清空。

### 1.4 错误降级三级策略

| 级别 | 触发条件 | 降级动作 |
|------|---------|---------|
| L1 | 混元超时/限流 | 切豆包 |
| L2 | 豆包也失败 | 返回通用色名（红/青/黄） |
| L3 | 用户已无网络 | 用本地缓存的上次结果 |

**AI 参考实现（降级链）：**

```js
async function getNameWithFallback(hex) {
  try {
    return await callHunyuan(hex)
  } catch (e1) {
    console.warn('混元失败，降级豆包:', e1.message)
    try {
      return await callDoubao(hex)
    } catch (e2) {
      console.warn('豆包失败，降级兜底:', e2.message)
      return { name: genericName(hex), source: 'fallback', confidence: 'low' }
    }
  }
}
```

**AI 执行约束：** 兜底结果必须标 `confidence: 'low'`，前端据此显示 risk-item 提示（见 xx-blocks）。

### 1.5 模型对比

| 维度 | 混元 hunyuan-pro | 豆包 doubao-pro | 本地兜底 |
|------|------------------|-----------------|---------|
| 成本 | 有免费额度 | 按量付费 | 0 |
| 速度 | 首字 1-3s | 首字 1-2s | <10ms |
| 质量 | 高 | 高 | 低（仅大类） |
| 备案 | 境内已备案 | 境内已备案 | 无需 |
| 适用 | 首选 | 降级 | 兜底 |

---

## 模块二：Camera + Canvas 海报生成

### 2.1 摄像头取色

**AI 参考实现（取色页）：**

```html
<camera
  device-position="back"
  flash="off"
  frame-size="medium"
  binderror="onCameraError"
  class="camera"
></camera>

<view class="center-crosshair">
  <view class="crosshair-circle"></view>
</view>

<button class="btn-primary" bindtap="onPick">取色</button>
```

```js
Page({
  data: { hex: '' },

  onPick() {
    const ctx = wx.createCameraContext()
    ctx.takePhoto({
      quality: 'high',
      success: (res) => {
        this.extractColor(res.tempImagePath)
      },
      fail: (err) => {
        wx.showToast({ title: '取色失败', icon: 'none' })
      }
    })
  },

  extractColor(imagePath) {
    const ctx = wx.createCanvasContext('pickerCanvas')
    wx.getImageInfo({
      src: imagePath,
      success: (info) => {
        const canvas = wx.createOffscreenCanvas({
          type: '2d',
          width: info.width,
          height: info.height
        })
        const c = canvas.getContext('2d')
        const img = canvas.createImage()
        img.onload = () => {
          c.drawImage(img, 0, 0)
          const cx = Math.floor(info.width / 2)
          const cy = Math.floor(info.height / 2)
          const pixel = c.getImageData(cx, cy, 1, 1).data
          const hex = this.rgbToHex(pixel[0], pixel[1], pixel[2])
          this.setData({ hex })
          this.getName(hex)
        }
        img.src = imagePath
      }
    })
  },

  rgbToHex(r, g, b) {
    return '#' + [r, g, b].map(x => x.toString(16).padStart(2, '0')).join('').toUpperCase()
  },

  async getName(hex) {
    const res = await wx.cloud.callFunction({
      name: 'aiNameColor',
      data: { hex }
    })
    if (res.result.code === 0) {
      this.setData({ name: res.result.data.name })
    }
  }
})
```

### 2.2 Canvas 配色海报

**AI 参考实现（用 xx-brand 的 canvasTheme.js 短别名）：**

```js
const C = require('../../config/canvasTheme')

function drawPoster(ctx, data, canvasWidth, canvasHeight) {
  ctx.fillStyle = C.card
  ctx.fillRect(0, 0, canvasWidth, canvasHeight)

  const blockHeight = canvasHeight * 0.5
  ctx.fillStyle = data.hex
  ctx.fillRect(0, 0, canvasWidth, blockHeight)

  ctx.fillStyle = C.t
  ctx.font = 'bold 48px sans-serif'
  ctx.fillText(data.name, 40, blockHeight + 80)

  ctx.fillStyle = C.ts
  ctx.font = '28px sans-serif'
  ctx.fillText(data.hex, 40, blockHeight + 130)

  ctx.fillStyle = C.p
  ctx.fillRect(0, canvasHeight - 100, canvasWidth, 100)
  ctx.fillStyle = C.ti
  ctx.font = '24px sans-serif'
  ctx.fillText('小象取色', canvasWidth / 2 - 50, canvasHeight - 40)

  ctx.draw()
}
```

### 2.3 保存到相册

**AI 参考实现：**

```js
async function savePoster(canvasId) {
  try {
    const { authSetting } = await wx.getSetting()
    if (!authSetting['scope.writePhotosAlbum']) {
      await wx.authorize({ scope: 'scope.writePhotosAlbum' })
    }

    const res = await new Promise((resolve, reject) => {
      wx.canvasToTempFilePath({
        canvasId: canvasId,
        success: resolve,
        fail: reject
      })
    })

    await wx.saveImageToPhotosAlbum({ filePath: res.tempFilePath })
    wx.showToast({ title: '已保存到相册', icon: 'success' })
  } catch (err) {
    if (err.errMsg.includes('auth deny')) {
      wx.showModal({
        title: '需要相册权限',
        content: '请在设置中开启相册权限以保存海报',
        confirmText: '去设置',
        success: (res) => {
          if (res.confirm) wx.openSetting()
        }
      })
    } else {
      wx.showToast({ title: '保存失败', icon: 'none' })
    }
  }
}
```

**AI 执行约束：** 保存相册前必须检查授权，授权被拒要引导去设置页，不能静默失败。

---

## 模块三：颜色处理算法

### 3.1 RGB ↔ HSL 转换

**AI 参考实现（utils/color.js）：**

```js
function rgbToHsl(r, g, b) {
  r /= 255; g /= 255; b /= 255
  const max = Math.max(r, g, b)
  const min = Math.min(r, g, b)
  let h, s, l = (max + min) / 2

  if (max === min) {
    h = s = 0
  } else {
    const d = max - min
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min)
    switch (max) {
      case r: h = (g - b) / d + (g < b ? 6 : 0); break
      case g: h = (b - r) / d + 2; break
      case b: h = (r - g) / d + 4; break
    }
    h /= 6
  }
  return { h: Math.round(h * 360), s: Math.round(s * 100), l: Math.round(l * 100) }
}

function hslToRgb(h, s, l) {
  h /= 360; s /= 100; l /= 100
  let r, g, b
  if (s === 0) {
    r = g = b = l
  } else {
    const hue2rgb = (p, q, t) => {
      if (t < 0) t += 1
      if (t > 1) t -= 1
      if (t < 1/6) return p + (q - p) * 6 * t
      if (t < 1/2) return q
      if (t < 2/3) return p + (q - p) * (2/3 - t) * 6
      return p
    }
    const q = l < 0.5 ? l * (1 + s) : l + s - l * s
    const p = 2 * l - q
    r = hue2rgb(p, q, h + 1/3)
    g = hue2rgb(p, q, h)
    b = hue2rgb(p, q, h - 1/3)
  }
  return {
    r: Math.round(r * 255),
    g: Math.round(g * 255),
    b: Math.round(b * 255)
  }
}
```

### 3.2 颜色相似度（CIE76）

**AI 参考实现（用于配色去重）：**

```js
function colorDistance(hex1, hex2) {
  const { r: r1, g: g1, b: b1 } = hexToRgb(hex1)
  const { r: r2, g: g2, b: b2 } = hexToRgb(hex2)
  const dr = r1 - r2
  const dg = g1 - g2
  const db = b1 - b2
  return Math.sqrt(dr * dr + dg * dg + db * db)
}

function isSimilarColor(hex1, hex2, threshold = 30) {
  return colorDistance(hex1, hex2) < threshold
}
```

### 3.3 配色方案生成

**AI 参考实现（互补色/类似色）：**

```js
function getComplementary(hex) {
  const { r, g, b } = hexToRgb(hex)
  return rgbToHex(255 - r, 255 - g, 255 - b)
}

function getAnalogous(hex) {
  const { h, s, l } = rgbToHsl(...Object.values(hexToRgb(hex)))
  return [
    hslToHex((h + 30) % 360, s, l),
    hex,
    hslToHex((h + 330) % 360, s, l)
  ]
}

function hslToHex(h, s, l) {
  const { r, g, b } = hslToRgb(h, s, l)
  return rgbToHex(r, g, b)
}
```

**判断标准（给人）：** 生成的配色方案，互补色是否真的是视觉互补？类似色是否相邻不刺眼？看着别扭 = 算法或阈值有问题。

**AI 执行约束：** 配色方案生成必须返回 hex 字符串数组，不要返回对象，方便 Canvas 直接用。

---

## 模块四：Agentic 模式入门

### 4.1 什么是 Agentic

传统 AI 调用：用户问 → AI 答（一问一答）。

Agentic：用户问 → AI 决定调哪个工具 → 拿结果 → 继续推理 → 直到完成。

**小象取色 Agentic 场景：**
- 用户："帮我取个故宫红的配色，再生成海报"
- AI 自主决策：① 调取色工具拿 #9B2335 → ② 调命名工具得"故宫红" → ③ 调配色工具得方案 → ④ 调海报工具生成 → ⑤ 返回结果

### 4.2 工具调用结构

**AI 参考实现（工具定义）：**

```js
const TOOLS = [
  {
    name: 'getColor',
    description: '从摄像头获取当前画面中心点的色值',
    parameters: {}
  },
  {
    name: 'nameColor',
    description: '给色值起中式色名',
    parameters: { hex: { type: 'string', description: '16进制色值' } }
  },
  {
    name: 'getScheme',
    description: '根据主色生成配色方案',
    parameters: { hex: { type: 'string' }, type: { type: 'string', enum: ['complementary', 'analogous'] } }
  },
  {
    name: 'drawPoster',
    description: '生成配色海报图片',
    parameters: { hex: { type: 'string' }, name: { type: 'string' }, scheme: { type: 'array' } }
  }
]
```

### 4.3 结构化输出

**AI 参考实现（用 JSON schema 约束）：**

```js
async function agenticPlan(userInput) {
  const params = {
    Model: 'hunyuan-pro',
    Messages: [
      {
        Role: 'system',
        Content: `你是小象取色的助手。可用工具：${JSON.stringify(TOOLS)}。
用户说需求，你返回一个 JSON 数组，表示要依次调用的工具及参数。
格式：[{"tool":"nameColor","args":{"hex":"#9B2335"}},...]
只返回 JSON，不要解释。`
      },
      { Role: 'user', Content: userInput }
    ]
  }

  const resp = await client.ChatCompletions(params)
  const plan = JSON.parse(resp.Choices[0].Message.Content)
  return plan
}
```

### 4.4 执行计划 + 防死循环

**AI 参考实现：**

```js
async function executePlan(plan) {
  const results = []
  const MAX_STEPS = 8

  for (let i = 0; i < Math.min(plan.length, MAX_STEPS); i++) {
    const step = plan[i]
    try {
      const result = await executeTool(step.tool, step.args)
      results.push({ tool: step.tool, result, success: true })
    } catch (err) {
      results.push({ tool: step.tool, error: err.message, success: false })
      break
    }
  }

  return results
}

async function executeTool(name, args) {
  switch (name) {
    case 'getColor':
      return await getCurrentColor()
    case 'nameColor':
      return await callNameColor(args.hex)
    case 'getScheme':
      return getAnalogous(args.hex)
    case 'drawPoster':
      return await generatePoster(args.hex, args.name, args.scheme)
    default:
      throw new Error(`未知工具: ${name}`)
  }
}
```

**AI 执行约束：**
- 必须设 `MAX_STEPS` 上限（建议 8），防止 Agentic 死循环烧钱
- 工具执行失败必须 break，不能继续跑
- 每个 step 结果都要记录，便于审计

**判断标准（给人）：** Agentic 跑一次花了多少 token？有没有超过 MAX_STEPS 还在转？超了 = 有死循环风险。

---

## 模块五：质量与成本（汇总）

| 维度 | 标准 | 来源 skill |
|------|------|-----------|
| 命名准确率 | ≥ 85%（按评估集） | xx-data |
| 幻觉容忍度 | 不可编造不存在的典故 | xx-prd |
| 响应时间 | 首字 ≤ 2s，全文 ≤ 8s | xx-prd |
| 失败降级 | 三级降级，不裸露报错 | 本 skill |
| 单次成本 | < ¥0.01 | xx-prd |
| 月预算 | ¥500，超限限流 | xx-prd |
| 生成式 AI 备案 | 已申报 | xx-safety |

**AI 执行约束：** 上线前这 7 项必须全部达标，缺一项不能发版。

---

## 使用方式

当你需要在小程序接入 AI 能力时，告诉我具体场景，我帮你生成调用与降级代码。

**对话示例：**
- "取色后让 AI 起个中式名字" → 我生成混元调用 + 三级降级云函数
- "命名结果要打字机效果" → 我生成流式输出 + 前端 ai-stream 对接
- "AI 挂了怎么办" → 我帮你补三级降级链（混元→豆包→通用色名）
- "要生成配色海报" → 我生成 Camera 取色 + Canvas 绘制 + 保存相册
- "让 AI 自己决定调哪个工具" → 我生成 Agentic 工具调用 + 防死循环

---

## 与其他技能的关系

- **前置：** xx-prd 定义 AI 质量标准与降级要求（本 skill 的目标）
- **前置：** xx-setup 云函数骨架与超时配置（60 秒）
- **并行：** xx-backend key 管理与缓存（本 skill 的 key 来源）
- **并行：** xx-data 评估集驱动 prompt 迭代（本 skill 的质量来源）
- **并行：** xx-blocks 的 ai-stream 组件展示本 skill 的流式输出
- **并行：** xx-brand 的 canvasTheme.js 用于本 skill 的海报绘制
- **安全交汇：** xx-safety 的生成式 AI 备案与内容安全（本 skill 上线前提）

## 方法论来源

- **混元大模型官方文档** — 腾讯云，ChatCompletions 接口
- **Prompt Engineering** — OpenAI 官方指南
- **Agentic Workflow** — Andrew Ng，工具调用与规划
- **Data-Centric AI** — Andrew Ng，评估集驱动迭代
- **色彩空间理论** — RGB/HSL/CIEXYZ，色彩科学基础
