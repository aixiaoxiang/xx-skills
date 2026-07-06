---
name: xx-setup
description: 微信小程序基础设施与合规备案技能。当用户要初始化小程序项目、配置云开发环境、部署云函数、创建数据库集合、处理小程序常见错误码、配置页面分享、或需要做ICP备案和隐私合规时使用。覆盖从项目骨架搭建到合规上线的完整基础设施流程。AI 按本规范生成项目骨架与配置，用户按检查清单验收。
---

# 基础设施与合规备案

> 先把地基打好，再盖房子。基础设施不对，后面全是返工。

## 这个技能做什么

帮你在项目启动阶段一次性搞定三件事：
1. **项目骨架** — 标准目录结构、配置文件、云开发环境
2. **云能力** — 云函数部署、数据库集合、数据传递策略
3. **合规备案** — ICP 备案、隐私保护指引、UGC 声明

跳过任何一步，后期都会踩坑。最常见的教训：没做备案就提交审核，被打回；云函数没调超时，AI 接口 3 秒超时报错。

**读者说明：** 本 skill 主要给 AI 执行规范（生成目录、配置、云函数模板），次要给人判断标准（检查清单验收）。用户不需要自己敲配置，只需要会描述项目类型、能看懂报错截图反馈给 AI。

贯穿案例：**小象取色**（摄像头取色 → AI 中式命名 → 配色海报 → 历史记录）。

---

## 一、项目初始化

### 1.1 标准目录结构

**AI 参考实现（小象取色项目骨架）：**

```
miniprogram-project/
├── miniprogram/              # 小程序前端代码
│   ├── pages/                # 页面
│   │   ├── picker/           # 取色页（核心）
│   │   │   ├── picker.wxml
│   │   │   ├── picker.wxss
│   │   │   ├── picker.js
│   │   │   └── picker.json
│   │   ├── poster/           # 海报生成页
│   │   └── history/          # 颜色历史页
│   ├── components/           # 自定义组件
│   ├── utils/                # 工具函数
│   │   └── util.js
│   ├── config/               # 配置
│   │   ├── cloud.js          # 云环境配置
│   │   └── theme.js          # 主题 Token（见 xx-brand）
│   ├── app.js                # 应用入口
│   ├── app.json              # 应用配置
│   ├── app.wxss              # 全局样式
│   └── sitemap.json
├── cloudfunctions/           # 云函数
│   └── aiNameColor/          # AI 取色命名
│       ├── index.js
│       ├── package.json
│       └── config.json
├── project.config.json       # 项目配置
└── README.md
```

**AI 执行约束：** 目录结构必须按上述分层，pages 按功能拆分，config 集中管理配置，禁止把配置写死在页面 js 里。

### 1.2 project.config.json

**AI 参考实现：**

```json
{
  "miniprogramRoot": "miniprogram/",
  "cloudfunctionRoot": "cloudfunctions/",
  "setting": {
    "urlCheck": true,
    "es6": true,
    "enhance": true,
    "postcss": true,
    "preloadBackgroundData": false,
    "minified": true,
    "newFeature": false,
    "coverView": true,
    "nodeModules": false,
    "autoAudits": false,
    "showShadowRootInWxmlPanel": true,
    "scopeDataCheck": false,
    "uglifyFileName": false,
    "checkInvalidKey": true,
    "checkSiteMap": true,
    "uploadWithSourceMap": true,
    "compileHotReLoad": false,
    "useMultiFrameRuntime": true,
    "useApiHook": true,
    "useApiHostProcess": true,
    "babelSetting": {
      "ignore": [],
      "disablePlugins": [],
      "outputPath": ""
    },
    "enableEngineNative": false,
    "useIsolateContext": true,
    "userConfirmedBundleSwitch": false,
    "packNpmManually": false,
    "packNpmRelationList": [],
    "minifyWXSS": true,
    "disableUseStrict": false,
    "minifyWXML": true,
    "showES6CompileOption": false,
    "useCompilerPlugins": false
  },
  "appid": "替换为你的AppID",
  "projectname": "xiaoxiang-color",
  "libVersion": "3.5.0",
  "condition": {},
  "editorSetting": {
    "tabIndent": "insertSpaces",
    "tabSize": 2
  }
}
```

**AI 执行约束：** `appid` 必须替换为用户真实 AppID，不能留占位符上线。

### 1.3 app.json 基础配置

**AI 参考实现（小象取色）：**

```json
{
  "pages": [
    "pages/picker/picker",
    "pages/poster/poster",
    "pages/history/history"
  ],
  "window": {
    "backgroundTextStyle": "light",
    "navigationBarBackgroundColor": "#8B0000",
    "navigationBarTitleText": "小象取色",
    "navigationBarTextStyle": "white",
    "backgroundColor": "#F2F2F7"
  },
  "style": "v2",
  "sitemapLocation": "sitemap.json",
  "lazyCodeLoading": "requiredComponents",
  "permission": {
    "scope.camera": {
      "desc": "用于摄像头实时取色"
    },
    "scope.writePhotosAlbum": {
      "desc": "用于保存配色海报到相册"
    }
  }
}
```

> `lazyCodeLoading` 设为 `requiredComponents` 可按需注入组件代码，减少启动耗时。

**AI 执行约束：** `permission` 里声明的 scope 必须与实际使用的 API 一一对应，多声明会被审核打回；每条 desc 必须写清用途。

---

## 二、云开发环境

### 2.1 wx.cloud.init 配置

**AI 参考实现（app.js）：**

```js
App({
  globalData: {
    userInfo: null,
    cloudEnv: 'your-env-id'
  },

  onLaunch() {
    if (!wx.cloud) {
      console.error('请使用 2.2.3 或以上的基础库以使用云能力')
      return
    }
    wx.cloud.init({
      env: this.globalData.cloudEnv,
      traceUser: true
    })
  }
})
```

### 2.2 环境ID配置

**关键原则：环境ID只在一处定义，其他地方引用，不硬编码。**

**AI 参考实现（config/cloud.js）：**

```js
const CLOUD_ENV = 'your-env-id'

module.exports = {
  CLOUD_ENV,
  COLLECTIONS: {
    COLORS: 'colors',
    POSTERS: 'posters',
    FEEDBACK: 'feedback'
  }
}
```

```js
const { CLOUD_ENV } = require('../config/cloud')
const db = wx.cloud.database({ env: CLOUD_ENV })
```

**判断标准（给人）：** 全项目搜 `your-env-id` 应该只出现在 `config/cloud.js` 一处。出现多处 = 不合格。

### 2.3 多环境管理（可选）

**AI 参考实现：**

```js
const ENV_MAP = {
  develop: 'dev-env-id',
  trial: 'trial-env-id',
  release: 'prod-env-id'
}

const envVersion = __wxConfig.envVersion
const CLOUD_ENV = ENV_MAP[envVersion] || ENV_MAP.release
```

---

## 三、云函数部署

### 3.1 云函数标准结构

```
cloudfunctions/
└── aiNameColor/
    ├── index.js          # 入口
    ├── package.json      # 依赖声明（必需）
    └── config.json       # 云函数配置（可选）
```

### 3.2 package.json（必需）

**每次新增依赖或修改 package.json 后，必须重新上传并部署（云端安装依赖）。**

```json
{
  "name": "aiNameColor",
  "version": "1.0.0",
  "description": "AI 中式颜色命名",
  "main": "index.js",
  "dependencies": {
    "wx-server-sdk": "~2.6.3"
  }
}
```

> **常见坑**：本地 `npm install` 了但没上传 `node_modules`，或者改了 `package.json` 没点"上传并部署：云端安装依赖"。两者都会导致 `require` 报错 `MODULE_NOT_FOUND`。

### 3.3 云函数入口模板

**AI 参考实现：**

```js
const cloud = require('wx-server-sdk')
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV })

exports.main = async (event, context) => {
  const { action, data } = event

  try {
    switch (action) {
      case 'getName':
        return await handleGetName(data)
      case 'getScheme':
        return await handleGetScheme(data)
      default:
        return { code: 400, message: '未知操作' }
    }
  } catch (err) {
    console.error('云函数错误:', err)
    return { code: 500, message: err.message }
  }
}

async function handleGetName(data) {
  const db = cloud.database()
  const result = await db.collection('colors')
    .where({ _openid: '{openid}' })
    .orderBy('createdAt', 'desc')
    .limit(20)
    .get()
  return { code: 0, data: result.data }
}
```

### 3.4 上传部署步骤

1. **右键云函数文件夹** → "上传并部署：云端安装依赖"
2. 等待上传完成（控制台会显示日志）
3. 在云开发控制台 → 云函数 → 测试

### 3.5 超时调整（重要）

AI 相关云函数默认 3 秒超时，调用大模型必超时。

**调整方法：**
- 云开发控制台 → 云函数 → 选择函数 → 配置 → 超时时间 → 改为 **60 秒**
- 或在 `config.json` 中配置：

```json
{
  "permissions": {
    "openapi": []
  },
  "timeout": 60
}
```

> **必须调整超时的场景**：调用大模型API、图片处理、批量数据库操作、文件上传处理。建议统一设为 60 秒。

**AI 执行约束：** 任何调用 AI 模型的云函数，超时必须显式设为 60 秒，不能留默认 3 秒。

---

## 四、数据库

### 4.1 集合创建

在云开发控制台 → 数据库 → 创建集合（小象取色示例）：

| 集合名 | 用途 | 权限设置 |
|--------|------|---------|
| `colors` | 用户取色历史 | 仅创建者可读写 |
| `posters` | 生成的海报记录 | 仅创建者可读写 |
| `feedback` | 用户反馈 | 仅创建者可读写 |

**权限说明：**
- `仅创建者可读写` — 默认推荐，用户只能操作自己的数据
- `所有用户可读，仅创建者可写` — 排行榜等公开数据
- `仅管理端可读写` — 敏感配置数据

### 4.2 数据库操作模板

**AI 参考实现：**

```js
const addResult = await db.collection('colors').add({
  data: {
    hex: '#8B0000',
    name: '故宫红',
    createdAt: db.serverDate(),
    status: 1
  }
})

const queryResult = await db.collection('colors')
  .where({ _openid: '{openid}' })
  .orderBy('createdAt', 'desc')
  .limit(20)
  .get()

const updateResult = await db.collection('colors')
  .doc(recordId)
  .update({ data: { status: 0 } })

const deleteResult = await db.collection('colors')
  .doc(recordId)
  .remove()
```

**AI 执行约束：** 时间字段必须用 `db.serverDate()`，不能用前端 `new Date()`（用户时区/改机时间不可信）。

### 4.3 数据传递优化

**核心原则：能在前端传参的，不要让云函数再去数据库查一遍。**

```
❌ 慢：前端 → 云函数 → 查数据库取用户信息 → 调用API → 返回
✅ 快：前端已有用户信息 → 直接传给云函数 → 调用API → 返回
```

**AI 参考实现（反例 vs 正例）：**

```js
exports.main = async (event, context) => {
  const user = await db.collection('users')
    .where({ _openid: event.openid }).get()
  const result = await callAI(event.input, user.data[0].preference)
  return result
}

exports.main = async (event, context) => {
  const result = await callAI(event.input, event.preference)
  return result
}
```

**URL 传参优于数据库读取的场景：**
- 列表页 → 详情页：直接传 id 和必要字段，详情页用 id 查详情
- 用户信息：登录后缓存到 globalData，传参使用
- 临时数据：不需要持久化的中间状态，用页面栈或 URL 传递

```js
wx.navigateTo({
  url: `/pages/poster/poster?id=${item._id}&hex=${encodeURIComponent(item.hex)}`
})

onLoad(options) {
  const { id, hex } = options
  this.setData({ hex })
  this.loadDetail(id)
}
```

**判断标准（给人）：** 列表点进详情，详情页加载要 < 500ms。如果慢，检查是不是云函数多查了一次数据库。

---

## 五、常见错误码速查表

| 错误码 | 含义 | 原因 | 解决方案 |
|--------|------|------|---------|
| `-604100` | 云函数调用失败 | 云函数未部署/超时/代码报错 | 检查云函数是否上传部署；超时则调至60秒；查看云函数日志 |
| `-502005` | 数据库权限不足 | 集合权限设置不允许当前操作 | 控制台修改集合权限为"仅创建者可读写" |
| `-504003` | 数据库记录不存在 | 查询的 doc id 不存在 | 检查 id 是否正确；先判断记录是否存在 |
| `-10009` | 系统错误 | 基础库版本过低或系统异常 | 升级基础库版本；重启微信开发者工具 |
| `ERR_FILE_NOT_FOUND` | 文件未找到 | 云存储文件路径错误或文件已删除 | 检查 cloudPath；确认文件已上传 |

**错误处理统一封装（AI 参考实现）：**

```js
function handleError(err, context = '') {
  console.error(`[${context}] 错误:`, err)

  const errorMap = {
    '-604100': '服务繁忙，请稍后重试',
    '-502005': '操作权限不足',
    '-504003': '数据不存在',
    '-10009': '系统异常，请更新微信',
    'ERR_FILE_NOT_FOUND': '文件不存在'
  }

  const message = errorMap[err.errCode] || err.errMsg || '未知错误，请稍后重试'
  wx.showToast({ title: message, icon: 'none' })
  return { success: false, message }
}

module.exports = { handleError }
```

**AI 执行约束：** 返回给用户的错误提示不能含技术细节（堆栈、key 片段、SQL），只能给友好文案；技术细节只进日志。

---

## 六、页面分享规范

**所有页面必须定义 `onShareAppMessage` 和 `onShareTimeline`，否则用户无法分享。**

**AI 参考实现：**

```js
Page({
  onShareAppMessage() {
    return {
      title: '小象取色 - 一键取中式色名',
      path: '/pages/picker/picker',
      imageUrl: '/assets/share-cover.png'
    }
  },

  onShareTimeline() {
    return {
      title: '小象取色 - 一键取中式色名',
      query: 'from=timeline',
      imageUrl: '/assets/share-cover.png'
    }
  }
})
```

> 不定义 `onShareAppMessage` 的页面，右上角菜单的"转发"按钮会灰色不可点。
> 不定义 `onShareTimeline` 的页面，无法分享到朋友圈。

**AI 执行约束：** 分享标题不超过 20 字；分享封面图必须是 5:4 比例。

---

## 七、文件编辑安全规范

**追加方法前必须检查文件是否已闭合（`}` 或 `]`）。**

这是最常见的破坏性操作：在已有文件末尾追加代码时，如果原文件最后一个字符不是闭合符号，追加会导致语法错误。

### 安全追加检查清单

```
追加前检查：
1. 读取目标文件完整内容
2. 确认最后一个字符是 } 或 ]（对象/数组闭合）
3. 如果最后一个字符是 } → 追加新方法需要先加逗号
4. 如果文件为空 → 从头写，不需要逗号
5. 追加后验证语法正确性
```

**AI 执行约束：** 改任何 .js / .json / .wxml 文件前，先 Read 完整内容，确认结构闭合，再做修改。永远不要假设文件末尾的状态。

---

## 八、ICP 备案

### 8.1 主体类型选择

| 类型 | 适用场景 | 所需材料 | 审核时间 |
|------|---------|---------|---------|
| **个人主体** | 个人开发者、自媒体 | 身份证、人脸识别 | 7-20个工作日 |
| **企业主体** | 公司、个体工商户 | 营业执照、法人身份证、公章 | 7-20个工作日 |

**选择建议：**
- 如果小程序涉及支付、经营性内容 → 必须企业主体
- 如果只是工具类、无收费 → 个人主体即可
- 企业主体审核更严格但功能权限更多

**判断标准（给人）：** 小象取色要做付费配色/海报模板 → 涉及经营性内容 → 必须企业主体。

### 8.2 隐私保护指引填写

**关键原则：不勾选"自定义"，只声明微信API实际获取的信息。**

在 `mp.weixin.qq.com` → 设置 → 服务内容声明 → 用户隐私保护指引：

**填写步骤：**
1. 不要勾选"自定义信息"
2. 根据实际使用的 API 勾选对应的信息类型：

| 使用的微信API | 对应信息项 |
|-------------|-----------|
| `wx.getUserProfile` | 微信昵称、头像 |
| `wx.getLocation` | 位置信息 |
| `<camera>` 取色 | 相机 |
| `wx.saveImageToPhotosAlbum` | 相册（写入） |
| `wx.getPhoneNumber` | 手机号 |
| 云开发数据库存储 | 不算用户信息 |

3. 填写信息用途说明（小象取色示例）：

```
- 相机：用于实时取色
- 相册：用于保存生成的配色海报
```

4. 第三方信息共享声明（如果用了第三方 SDK/API）：

```
- 本程序使用了 混元大模型 进行 颜色命名，
  会将您的 取色色值 传输至 腾讯云 用于 生成中式色名。
```

> **常见坑**：勾选了未实际使用的信息项，审核会被打回。只勾选真实用到的。

**AI 执行约束：** 隐私指引里声明的信息项必须与代码里实际调用的 API 完全一致，多声明或少声明都会被审核打回。

### 8.3 UGC 声明

**用户输入文本/图片不算 UGC，但用户发布的内容被其他用户看到才算。**

| 场景 | 是否UGC | 需要声明 |
|------|---------|---------|
| 用户取色结果仅自己看 | ❌ 不是 | 不需要 |
| 用户保存海报到相册 | ❌ 不是 | 不需要 |
| 用户发布配色被其他用户看到 | ✅ 是 | 需要 |
| 用户评论/评分其他人可见 | ✅ 是 | 需要 |

**如果产品包含 UGC，需要在备案中声明：**

```
UGC内容安全管理：
1. 用户发布内容前需同意内容规范
2. 系统对UGC内容进行关键词过滤
3. 提供举报入口，收到举报后24小时内处理
4. 违规内容予以删除，严重违规者封禁账号
```

**判断标准（给人）：** 小象取色 MVP 不做社区 → 不需要 UGC 声明；后期加社区 → 必须补声明。

### 8.4 备案流程

```
1. 准备材料
   ├─ 个人：身份证正反面照片、人脸识别
   └─ 企业：营业执照、法人身份证、网站负责人信息

2. 提交备案
   └─ mp.weixin.qq.com → 设置 → 基本设置 → ICP备案

3. 等待审核
   ├─ 小程序平台初审（1-3个工作日）
   ├─ 通信管理局审核（7-20个工作日）
   └─ 审核结果通过短信和站内信通知

4. 备案通过后
   └─ 在小程序"关于"页面展示备案号
```

### 8.5 备案号展示

备案通过后，**必须在小程序中展示备案号**：

**AI 参考实现：**

```html
<view class="about-footer">
  <view class="icp-info">
    <text>备案号：</text>
    <text class="icp-number" bindtap="copyIcp">{{icpNumber}}</text>
  </view>
</view>
```

```js
Page({
  data: {
    icpNumber: '京ICP备XXXXXXXX号-X'
  },

  copyIcp() {
    wx.setClipboardData({
      data: this.data.icpNumber,
      success: () => {
        wx.showToast({ title: '已复制备案号', icon: 'success' })
      }
    })
  }
})
```

**AI 执行约束：** 备案号必须真实，不能用占位符上线；展示位置必须在用户可见的"关于"页。

---

## 九、开发检查清单

在提交审核前，逐项检查：

### 基础设施

- [ ] `project.config.json` 中 `appid` 已替换为真实 AppID
- [ ] 云开发环境ID已配置且在 `config/cloud.js` 中统一管理
- [ ] `wx.cloud.init` 在 `app.js` 中正确调用
- [ ] 所有云函数已上传部署（含 `package.json` 依赖）
- [ ] AI相关云函数超时已调整为 60 秒
- [ ] 数据库集合已创建，权限设置正确

### 代码规范

- [ ] 所有页面定义了 `onShareAppMessage` 和 `onShareTimeline`
- [ ] 分享标题不超过20字，分享封面图5:4比例
- [ ] 错误处理使用统一封装（`errorHandler`）
- [ ] 数据传递优先用URL参数，减少不必要的数据库查询
- [ ] 修改文件前已确认文件闭合状态

### 合规备案

- [ ] ICP备案已通过，备案号已展示
- [ ] 隐私保护指引已填写（只勾选实际使用的API对应信息）
- [ ] 未勾选"自定义信息"
- [ ] 第三方信息共享已声明（如有）
- [ ] UGC声明已填写（如有社区/评论功能）
- [ ] UGC内容审核机制已实现（如有）

### 上线前

- [ ] 基础库版本 ≥ 2.2.3
- [ ] `lazyCodeLoading` 已开启
- [ ] `sitemap.json` 已配置
- [ ] 体验版已完整测试通过

**AI 执行约束：** 任一项未通过不能提交审核。备案类项目未完成不能上线。

---

## 使用方式

当你需要搭建小程序基础设施时，告诉我你的项目类型，我来生成项目骨架和配置。

**对话示例：**
- "我要做小象取色这个小程序，帮我搭项目结构" → 我生成标准目录 + app.json + 云函数模板
- "云函数调用报 -604100 错误" → 我帮你排查是没部署还是超时
- "小象取色要做 ICP 备案，需要准备什么" → 我帮你梳理材料和流程
- "用户反馈分享按钮点不了" → 检查是否定义了 onShareAppMessage

---

## 与其他技能的关系

- **前置：** xx-prd 定义功能规格（项目骨架的页面来源）
- **并行：** xx-brand 主题 Token（config/theme.js）、xx-blocks 组件库（components/）
- **下游：** xx-ai 云函数里调混元、xx-backend key 管理与缓存
- **安全交汇：** xx-safety 的隐私授权、生成式 AI 备案（基础设施层落地）

## 方法论来源

- **微信小程序官方文档** — 云开发、基础能力、备案规范
- **微信开放平台备案指南** — ICP备案操作规范
- **个人信息保护法** — 隐私保护指引填写依据
- **云开发最佳实践** — 数据传递与性能优化策略
