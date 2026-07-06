---
name: xx-backend
description: AI 产品最小后端技能。当需要把 AI API 接入小程序但不想搞复杂后端时使用。只讲 AI 产品最小三件套：API 代理（隐藏 key）、key 管理、简单缓存降本。不碰微服务、不碰数据库设计。
---

# AI 产品最小后端

> 前端直接调 AI API = 把 key 写在公网上。先用一个云函数挡住。

## 这个技能做什么

帮你用微信云开发搭一个 AI 产品够用的最小后端，覆盖三件套：**API 代理 + key 管理 + 简单缓存**。再加一个限流防滥用。

**明确边界：** 本 skill 不教数据库设计、不教微服务、不教 Docker、不教消息队列。只够让 AI 产品跑起来且 key 不泄露。需要更复杂后端时见第 7 节"什么时候该升级"。

---

## 1. 为什么不能直接在前端调 AI API

| 问题 | 后果 |
|------|------|
| key 写在前端代码 | 反编译可拿走，等于公开，被人盗刷你买单 |
| 跨域 | AI 服务商一般不允许小程序域名直接调 |
| 无法鉴权计费 | 任何人都能用你的 key 调用，你没法定额 |
| 无法限流 | 一个用户狂调把你的额度烧光 |
| 无法缓存 | 相同输入重复调，浪费钱 |

**铁律：key 写死在小程序前端 = 公开。** 任何前端代码都可以被反编译。

正确架构：

```
小程序前端 → 云函数（API 代理）→ AI API
              ↑
         key 在这里
```

---

## 2. 最小三件套

| 组件 | 解决什么 | 复杂度 |
|------|---------|--------|
| API 代理 | 隐藏 key + 统一错误处理 | 低 |
| key 管理 | 多 key 轮询降本 + 不写死 | 低 |
| 简单缓存 | 相同输入复用结果，降本提速 | 低 |
| 限流（附加） | 防 openid 滥用 | 低 |

四个加起来一个云函数就能装下。

---

## 3. API 代理：云函数做转发

**AI 参考实现（cloudfunctions/aiProxy/index.js）：**

```javascript
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });

const AI_API_URL = 'https://ai.example.com/v1/chat';
const TIMEOUT_MS = 15000;

exports.main = async (event, context) => {
  const openid = cloud.getWXContext().OPENID;

  if (!event.userInput) {
    return { code: 400, msg: '缺少输入' };
  }

  const controller = new AbortController();
  const timer = setTimeout(() => controller.abort(), TIMEOUT_MS);

  try {
    const res = await fetch(AI_API_URL, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.AI_API_KEY}`
      },
      body: JSON.stringify({
        model: 'hunyuan-pro',
        messages: [
          { role: 'system', content: event.systemPrompt || '你是助手' },
          { role: 'user', content: event.userInput }
        ]
      }),
      signal: controller.signal
    });

    clearTimeout(timer);

    if (!res.ok) {
      console.error('AI API 错误', res.status, await res.text());
      return { code: 500, msg: 'AI 服务暂时不可用' };
    }

    const data = await res.json();
    return { code: 0, data: data.choices[0].message.content };
  } catch (e) {
    clearTimeout(timer);
    if (e.name === 'AbortError') {
      return { code: 504, msg: 'AI 响应超时' };
    }
    console.error('云函数异常', e);
    return { code: 500, msg: '服务异常' };
  }
};
```

**AI 执行约束：**
- key 从 `process.env.AI_API_KEY` 读，不写死
- 必须设超时（AbortController），AI 接口可能挂很久
- 错误日志记 status + body，不记 key
- 返回给前端的错误信息不能含技术细节

**前端调用：**

```javascript
wx.cloud.callFunction({
  name: 'aiProxy',
  data: { userInput: '故宫红', systemPrompt: '你是中式颜色命名助手...' },
  success: res => console.log(res.result)
});
```

---

## 4. key 管理

### 4.1 key 放环境变量

云函数后台 → 函数配置 → 环境变量，加 `AI_API_KEY`。代码里只读 `process.env.AI_API_KEY`。

**判断标准：** 代码里搜不到 key 明文 = 合格。

### 4.2 多 key 轮询降本

单个 key 有 QPS 限制 / 额度限制。多 key 轮询能突破单 key 上限 + 分散成本。

**AI 参考实现：**

```javascript
const KEY_POOL = [
  process.env.AI_KEY_1,
  process.env.AI_KEY_2,
  process.env.AI_KEY_3
].filter(Boolean);

let keyIndex = 0;

function getNextKey() {
  if (KEY_POOL.length === 0) {
    throw new Error('无可用 key');
  }
  const key = KEY_POOL[keyIndex % KEY_POOL.length];
  keyIndex++;
  return key;
}

exports.main = async (event) => {
  const key = getNextKey();
  return callAIWithKey(key, event);
};
```

**AI 执行约束：** 单 key 失败要自动切下一个 key 重试，重试次数 ≤ key 总数。

---

## 5. 简单缓存

相同输入复用结果，降本 + 提速。AI 命名这类任务特别适合缓存（同一种颜色命名结果稳定）。

**缓存 key 设计：** 对 `systemPrompt + userInput` 做哈希，作为缓存 key。

**AI 参考实现（云函数 + 简单 KV，用云开发数据库当缓存）：**

```javascript
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });
const db = cloud.database();

const CACHE_TTL_MS = 24 * 60 * 60 * 1000;

function hashKey(text) {
  let h = 0;
  for (let i = 0; i < text.length; i++) {
    h = ((h << 5) - h + text.charCodeAt(i)) | 0;
  }
  return 'cache_' + Math.abs(h).toString(36);
}

async function getCache(cacheKey) {
  const res = await db.collection('ai_cache').where({
    _id: cacheKey,
    expireAt: db.command.gt(new Date())
  }).get();
  return res.data.length > 0 ? res.data[0].result : null;
}

async function setCache(cacheKey, result) {
  await db.collection('ai_cache').doc(cacheKey).set({
    data: {
      result,
      expireAt: new Date(Date.now() + CACHE_TTL_MS)
    }
  });
}

exports.main = async (event) => {
  const cacheKey = hashKey((event.systemPrompt || '') + '|' + event.userInput);

  const cached = await getCache(cacheKey);
  if (cached) {
    return { code: 0, data: cached, fromCache: true };
  }

  const aiResult = await callAI(event);
  try {
    await setCache(cacheKey, aiResult);
  } catch (e) {
    console.error('缓存写入失败', e);
  }
  return { code: 0, data: aiResult, fromCache: false };
};
```

**AI 执行约束：**
- 缓存 key 必须包含 systemPrompt，否则换 prompt 会读到旧结果
- 缓存必须有 TTL，不能永久缓存
- 缓存写入失败不能阻塞主流程（try/catch 吞掉）

---

## 6. 限流

按 openid 简单限流，防一个人把额度烧光。

**AI 参考实现（基于云开发数据库计数）：**

```javascript
const cloud = require('wx-server-sdk');
cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });
const db = cloud.database();

const RATE_LIMIT = 20;
const RATE_WINDOW_MS = 60 * 60 * 1000;

async function checkRateLimit(openid) {
  const now = Date.now();
  const since = new Date(now - RATE_WINDOW_MS);

  const res = await db.collection('rate_limit').where({
    openid,
    createdAt: db.command.gt(since)
  }).count();

  if (res.total >= RATE_LIMIT) {
    return { pass: false, msg: '调用过于频繁，请稍后再试' };
  }

  await db.collection('rate_limit').add({
    data: { openid, createdAt: new Date(now) }
  });
  return { pass: true };
}

exports.main = async (event, context) => {
  const openid = cloud.getWXContext().OPENID;
  const limit = await checkRateLimit(openid);
  if (!limit.pass) {
    return { code: 429, msg: limit.msg };
  }
  return callAI(event);
};
```

**判断标准：** 限流维度按业务选——普通用户按 openid / 小时，付费用户放宽或按套餐。

---

## 7. 什么时候该升级后端

本 skill 的方案能撑到一定规模。出现以下任一信号，该找后端工程师或换方案：

| 信号 | 阈值 | 升级方向 |
|------|------|---------|
| QPS 过高 | > 50 | 独立后端服务（Node/Go） |
| 需要持久化业务数据 | 用户 / 订单 / 内容 | 引入正式数据库设计 |
| 需要异步任务 | 生成长视频 / 批量处理 | 消息队列 + Worker |
| 需要多端共享 | 小程序 + App + Web | 独立 API 服务 |
| 云函数冷启动严重 | 首次调用 > 3s | 常驻后端服务 |

**AI 执行约束：** 没到阈值不要提前优化，先把三件套跑稳。

---

## 示例：小象取色 后端架构

```
小程序（取色 → 显示颜色）
    ↓ wx.cloud.callFunction
云函数 aiNameColor
    ├─ 限流：openid 每小时 20 次
    ├─ 缓存：hashKey(systemPrompt + 颜色hex) → 命名结果，TTL 24h
    ├─ key 轮询：3 个混元 key 轮换
    └─ 调混元 API → 返回中式色名
```

典型调用：用户取到 #8B0000 → 云函数查缓存（首次没有）→ 调混元 → 返回"故宫红"→ 写缓存。下一个用户取到同样的色，直接命中缓存，0 成本。

---

## 使用方式

把你的 AI 调用场景告诉我，我帮你生成云函数代码、配置 key、加缓存和限流。

**对话示例：**
- "我要在小程序里调混元，key 怎么放" → 我给你云函数代理代码
- "AI 调用太贵了怎么省钱" → 我帮你加缓存 + 多 key 轮询
- "有人狂调我的接口怎么办" → 我给你 openid 限流代码
- "QPS 上来了云函数扛不住" → 我帮你判断是不是该升级后端

---

## 与其他技能的关系

- **前置：** xx-setup 完成云开发初始化
- **并行：** xx-ai 前端调用 AI 的写法，本 skill 是其后端落地
- **安全交汇：** xx-safety 管 key 安全（key 只放云函数环境变量是两个 skill 的共同铁律）
- **下游：** xx-track 监控缓存命中率 / 限流触发率，反向调参

## 方法论来源

- **BFF（Backend for Frontend）模式** — Sam Newman
- **微信云开发官方文档** — 云函数 / 数据库 / 内容安全
- **Rate Limiting 模式** — 令牌桶 / 滑动窗口简化版
