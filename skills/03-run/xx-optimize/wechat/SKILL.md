---
name: xx-optimize
description: 性能优化技能。当产品在低端机卡顿、接口等待白屏、历史列表滚动掉帧、或需要优化帧循环计算、运动交互时使用。覆盖典型性能场景（含 AI 调用的流式渲染、长等待、长列表、冷启动、节流防抖）和零体验降级原则，按P0/P1/P2分级优化，输出可直接应用的参考实现。
---

# 性能优化与运动交互

> 零体验降级。先治"等接口"和"长列表"，再治计算，最后才考虑降级体验。
> 主要给 AI 执行优化策略，给人判断优化是否到位、是否过度降级。性能优化原则对所有小程序通用，含 AI 调用的场景单列。

贯穿案例：**小象取色**（摄像头实时取色 → AI 中式颜色命名 → AI 配色方案 → Canvas 生成配色海报 → 历史记录）。

## 这个技能做什么

帮你在低端机（Android 8 以下）和 AI 调用场景下保持流畅体验。核心原则：**先榨干计算优化和 AI 等待优化，再考虑降级。**

大多数性能优化的误区：
- 一上来就降分辨率、删效果（牺牲体验）
- 不查根因就加节流（治标不治本）
- 只在中端机测，低端机直接崩
- AI 接口等 3 秒白屏，却去优化取色算法的 5ms

正确顺序：
1. **AI 性能场景**（接口等待 / 长列表 / 冷启动）—— AI 产品最常见瓶颈
2. **P0 纯计算优化** —— 帧循环内的高频计算
3. **P1 采样频率** —— 减少非关键刷新
4. **P2 视觉降级** —— 经产品确认才降级

---

## 核心原则：零体验降级

| 级别 | 措施 | 是否需确认 | 示例 |
|------|------|-----------|------|
| **不降级** | 纯计算优化 / 接口等待优化 | 无需确认 | 查表、缓存、流式渲染、骨架屏、请求并行 |
| **轻微调整** | 采样频率调整 | 无需确认 | 降低非关键模块刷新率、加速度计降频 |
| **视觉降级** | 画面表现变差 | 需产品确认 | 移除 blur、降 Canvas 分辨率、减粒子 |
| **交互变更** | 改变用户操作方式 | 必须确认 | 改变核心逻辑、减少动画、改交互方式 |

**决策流程：**

```
性能有问题？
├── 是接口等待 / 长列表 / 冷启动吗？
│    ├── 是 → 做性能场景优化（不降级）
│    └── 否 → 能用纯计算优化解决吗？
│              ├── 能 → 做"不降级"优化
│              └── 不能 → 能用"轻微调整"解决吗？
│                        ├── 能 → 做"轻微调整"
│                        └── 不能 → 需要降级体验
│                                  ├── 产品确认 → 做"视觉降级"
│                                  └── 不确认 → 报告问题，等确认
```

**判断标准：** 性能问题，先确认是不是"在等接口"或"在渲染长列表"，再考虑计算优化。不要一上来就降分辨率。

---

## 一、含 AI 能力的产品的典型性能场景

> 含 AI 调用的产品性能瓶颈通常在接口等待和长列表渲染，不在纯计算。本章覆盖 5 个最常见场景，每个给"判断标准 + AI 参考实现"。产品不含 AI 的话，长列表/冷启动/节流防抖场景仍然适用。

### 1.1 流式响应的渲染性能

**问题：** AI 命名/配色方案返回一整段文本，前端等全部返回再一次性 `setData` 大文本，用户盯着空白等 3 秒。

**方案：** 用 SSE / 分片接收，边收边渲染，避免一次性 `setData` 大文本。

**判断标准：** AI 文本响应超过 200 字时必须有流式渲染；首字出现时间 < 1s。

**AI 参考实现（小象取色 AI 命名流式渲染）：**

```javascript
// pages/picker/picker.js
Page({
  data: {
    nameText: '',     // 渐进显示的命名文本
    streaming: false,
  },

  onAiName() {
    this.setData({ streaming: true, nameText: '' });

    const task = wx.request({
      url: 'https://xxx/ai-name-stream',
      enableChunked: true,   // 开启分片接收
      method: 'POST',
      data: { colorHex: this.data.currentColor },
      responseType: 'arraybuffer',
    });

    // 分片接收：每收到一段就追加渲染
    task.onChunkReceived((res) => {
      const chunk = this.decodeChunk(res.data);
      // 追加而非覆盖，避免一次性 setData 大文本
      this.setData({ nameText: this.data.nameText + chunk });
    });

    task.onComplete(() => {
      this.setData({ streaming: false });
    });

    task.onError(() => {
      this.setData({ streaming: false });
    });
  },

  decodeChunk(arraybuffer) {
    // arraybuffer 转文本
    const u8 = new Uint8Array(arraybuffer);
    // 处理多字节字符边界，避免截断中文
    const text = decodeURIComponent(escape(String.fromCharCode.apply(null, u8)));
    return text;
  },
});
```

**AI 执行约束：** 流式渲染时每次 `setData` 只追加增量文本，不要每次都 `setData` 完整文本；中文要处理多字节边界，不能截断。

---

### 1.2 长 token 响应的等待体验

**问题：** AI 配色方案返回慢（2-5s），用户盯着 loading 转圈不知道还要等多久，超过 8s 直接放弃。

**方案：** 骨架屏占位 + 渐进式显示 + 超时兜底。

**判断标准：** AI 响应 > 1s 必须有骨架屏；> 8s 必须有超时兜底提示，不能无限转圈。

**AI 参考实现（小象取色 AI 命名等待态）：**

```javascript
// pages/picker/picker.js
Page({
  data: {
    loadingState: 'idle', // idle | skeleton | streaming | timeout | error
  },

  onAiName() {
    this.setData({ loadingState: 'skeleton' });

    // 超时兜底：8s 后切换到 timeout 态，给用户"重试"入口
    const timeoutId = setTimeout(() => {
      if (this.data.loadingState === 'skeleton') {
        this.setData({ loadingState: 'timeout' });
      }
    }, 8000);

    callAiName(this.data.currentColor)
      .then((res) => {
        clearTimeout(timeoutId);
        if (res && res.name) {
          this.setData({ loadingState: 'streaming', name: res.name });
        } else {
          // 空结果兜底
          this.setData({ loadingState: 'error', name: '命名失败，请重试' });
        }
      })
      .catch((err) => {
        clearTimeout(timeoutId);
        this.setData({ loadingState: 'error' });
      });
  },

  onRetry() {
    this.onAiName();
  },
});
```

```xml
<!-- picker.wxml：按 loadingState 切换显示 -->
<view wx:if="{{loadingState === 'skeleton'}}" class="skeleton">命名生成中…</view>
<view wx:elif="{{loadingState === 'timeout'}}" class="timeout">
  响应太慢了，<text bindtap="onRetry">点此重试</text>
</view>
<view wx:elif="{{loadingState === 'error'}}" class="error">
  命名失败，<text bindtap="onRetry">重试</text>
</view>
<view wx:else>{{name}}</view>
```

**AI 执行约束：** 任何 AI 调用都要有超时兜底和 error/timeout 态，不能让用户盯着 loading 无限等；空结果要有兜底文案。

---

### 1.3 历史对话 / 历史记录长列表

**问题：** 小象取色历史颜色记录积累到几百条，一次性 `setData` 全部，列表渲染卡顿、内存暴涨。

**方案：** 分页加载 + 触底加载更多；超长列表用虚拟列表（只渲染可见区域）。

**判断标准：** 列表 > 50 条必须分页；> 500 条考虑虚拟列表；不能一次性 `setData` 全量。

**AI 参考实现（小象取色历史记录分页加载）：**

```javascript
// pages/history/history.js
Page({
  data: {
    historyList: [],
    page: 1,
    pageSize: 20,
    hasMore: true,
    loading: false,
  },

  onLoad() {
    this.loadHistory();
  },

  // 触底加载更多
  onReachBottom() {
    if (this.data.hasMore && !this.data.loading) {
      this.loadHistory();
    }
  },

  loadHistory() {
    if (this.data.loading) return;
    this.setData({ loading: true });

    fetchHistory(this.data.page, this.data.pageSize).then((res) => {
      // 追加而非覆盖，每次只加一页
      this.setData({
        historyList: this.data.historyList.concat(res.list),
        page: this.data.page + 1,
        hasMore: res.list.length >= this.data.pageSize,
        loading: false,
      });
    }).catch(() => {
      this.setData({ loading: false });
    });
  },
});
```

**AI 参考实现（虚拟列表，超长历史记录）：**

```javascript
// pages/history/history.js — recycle-view 虚拟列表
const recycleView = require('../../miniprogram-recycle-view');

Page({
  data: {
    list: [],          // 全量数据（只存不渲染）
    visibleList: [],   // 只渲染可见区域
    startIndex: 0,
    itemHeight: 120,   // 每项固定高度
  },

  onPageScroll(e) {
    const scrollTop = e.scrollTop;
    const startIndex = Math.floor(scrollTop / this.data.itemHeight);
    const endIndex = startIndex + 20; // 可视区 + 缓冲

    if (startIndex !== this.data.startIndex) {
      this.setData({
        startIndex,
        visibleList: this.data.list.slice(startIndex, endIndex),
      });
    }
  },
});
```

**AI 执行约束：** 长列表严禁一次性 `setData` 全量；分页必须带 `loading` 防重复请求；`hasMore` 为 false 时不再请求。

---

### 1.4 AI 接口冷启动白屏

**问题：** 进入取色页，AI 配置/用户偏好/上次结果都要请求，串行请求导致首屏白屏 2-3s。

**方案：** 首屏占位（缓存上次结果）+ 请求并行 + 关键路径优先。

**判断标准：** 首屏不能白屏 > 1s；有缓存先显示缓存；多个独立请求必须并行。

**AI 参考实现（小象取色取色页冷启动）：**

```javascript
// pages/picker/picker.js
Page({
  data: {
    currentColor: '',
    name: '',
    config: null,
    pref: null,
  },

  onLoad() {
    // 1. 立即显示缓存的上次取色结果（避免白屏）
    const cached = wx.getStorageSync('lastColorResult');
    if (cached) {
      this.setData({ currentColor: cached.colorHex, name: cached.name });
    }

    // 2. 并行请求：取色配置 + 用户偏好（不串行等待）
    Promise.all([
      fetchPickConfig(),
      fetchUserPref(),
    ]).then(([config, pref]) => {
      this.setData({ config, pref });
    }).catch((err) => {
      console.error('冷启动请求失败', err);
      // 降级：用默认配置
      this.setData({ config: DEFAULT_CONFIG });
    });
  },

  onPickSuccess(colorHex, name) {
    // 缓存最新结果，下次冷启动直接显示
    wx.setStorageSync('lastColorResult', { colorHex, name });
  },
});
```

**AI 执行约束：** 首屏多个独立请求必须 `Promise.all` 并行，不能串行 await；有缓存优先显示缓存，再后台更新（stale-while-revalidate）。

---

### 1.5 AI 请求的节流与防抖

**问题：** 用户连续点击"AI 命名"按钮，触发多次 AI 请求，浪费 token、结果互相覆盖（竞态）。

**方案：** 触发新请求时取消上一个未完成请求 + 节流防抖。

**判断标准：** 用户连续触发同一 AI 调用时，只能有一个在途请求；结果只展示最新请求的返回。

**AI 参考实现（小象取色 AI 命名防抖 + 取消）：**

```javascript
// utils/ai-name.js
let currentTask = null;
let lastColorHex = null;

function nameColor(colorHex, callback) {
  // 1. 防抖：同一颜色 500ms 内重复点击只发一次
  if (colorHex === lastColorHex && currentTask) {
    return; // 已有相同请求在途，跳过
  }
  lastColorHex = colorHex;

  // 2. 取消上一个未完成的请求（防竞态）
  if (currentTask) {
    currentTask.abort();
  }

  currentTask = wx.request({
    url: 'https://xxx/ai-name',
    method: 'POST',
    data: { colorHex },
    success: (res) => {
      // 3. 只处理最新请求的结果
      if (colorHex === lastColorHex && res.data && res.data.name) {
        callback(res.data.name);
      }
    },
    complete: () => {
      currentTask = null;
    },
  });
}

module.exports = { nameColor };
```

**AI 参考实现（按钮禁用 + 节流）：**

```javascript
// pages/picker/picker.js
Page({
  data: {
    naming: false, // 命名中，禁用按钮
  },

  onTapAiName() {
    if (this.data.naming) return; // 节流：命名中不再触发

    this.setData({ naming: true });
    nameColor(this.data.currentColor, (name) => {
      this.setData({ name, naming: false });
    });
  },
});
```

```xml
<!-- picker.wxml：命名中禁用按钮，显示 loading -->
<button bindtap="onTapAiName" disabled="{{naming}}">
  {{naming ? '命名中…' : 'AI 命名'}}
</button>
```

**AI 执行约束：** AI 请求必须可取消（abort）；连续触发时取消旧请求；按钮在请求中要 disabled，防止重复触发。

---

## 二、P0 纯计算优化（无需确认）

> 以下针对帧循环内的高频计算。AI 产品的计算瓶颈较少，但运动交互 / Canvas 渲染场景仍需关注。

### 2.1 循环内数学函数预计算

**问题：** `Math.exp`、`Math.sqrt`、`Math.pow` 在帧循环（如 onAccelerometerChange、requestAnimationFrame）内被高频调用，每次都要做浮点运算。

**方案：** 预计算为 `Float32Array` 查表，用索引代替实时计算。

**AI 参考实现：**

```javascript
// ❌ 危险：每帧调用 Math.exp，低端机每帧多耗 2-5ms
onAccelerometerChange(res) {
  const intensity = Math.exp(res.x * 0.1);
  if (intensity > threshold) {
    this.triggerShake();
  }
}

// ✅ 优化：预计算查表，O(1) 查找
// 初始化时构建查找表（只需构建一次）
const EXP_TABLE_SIZE = 256;
const expTable = new Float32Array(EXP_TABLE_SIZE);
const EXP_INPUT_MIN = -5;
const EXP_INPUT_RANGE = 10; // -5 到 5

function initExpTable() {
  for (let i = 0; i < EXP_TABLE_SIZE; i++) {
    const input = EXP_INPUT_MIN + (i / EXP_TABLE_SIZE) * EXP_INPUT_RANGE;
    expTable[i] = Math.exp(input);
  }
}

// 查表函数：输入值映射到索引
function fastExp(x) {
  // 限制范围
  if (x <= EXP_INPUT_MIN) return expTable[0];
  if (x >= EXP_INPUT_MIN + EXP_INPUT_RANGE) return expTable[EXP_TABLE_SIZE - 1];
  // 计算索引
  const index = Math.floor(
    ((x - EXP_INPUT_MIN) / EXP_INPUT_RANGE) * EXP_TABLE_SIZE
  );
  return expTable[index];
}

// 使用：查表代替实时计算
onAccelerometerChange(res) {
  const intensity = fastExp(res.x * 0.1);
  if (intensity > threshold) {
    this.triggerShake();
  }
}

// 初始化时调用
initExpTable();
```

**同理处理 sqrt 和 pow（AI 参考实现）：**

```javascript
// 预计算 sqrt 查表（0-1000 范围）
const SQRT_TABLE_SIZE = 1001;
const sqrtTable = new Float32Array(SQRT_TABLE_SIZE);

function initSqrtTable() {
  for (let i = 0; i < SQRT_TABLE_SIZE; i++) {
    sqrtTable[i] = Math.sqrt(i);
  }
}

function fastSqrt(x) {
  if (x < 0) return 0;
  if (x >= SQRT_TABLE_SIZE) return Math.sqrt(x); // 超范围回退
  return sqrtTable[Math.floor(x)];
}
```

**判断标准：** 帧循环内数学运算耗时降低 80-90%。

---

### 2.2 数据预缓存

**问题：** 循环内重复解析数据（如颜色 RGB 值），每次都重新转换。

**方案：** 启动时一次性缓存，循环内直接读取。

**AI 参考实现（小象取色颜色解析缓存）：**

```javascript
// ❌ 危险：每次取色都重新解析颜色
function applyColorEffect(colorHex, intensity) {
  const r = parseInt(colorHex.slice(1, 3), 16);
  const g = parseInt(colorHex.slice(3, 5), 16);
  const b = parseInt(colorHex.slice(5, 7), 16);
  return [
    Math.min(255, r * intensity),
    Math.min(255, g * intensity),
    Math.min(255, b * intensity)
  ];
}

// 每帧调用
onColorPick(colors) {
  colors.forEach(color => {
    const rgb = applyColorEffect(color, 1.2); // 每次都重新解析
    this.drawColor(rgb);
  });
}

// ✅ 优化：启动时预缓存 RGB 数组
let colorCache = {}; // { '#FF0000': { r: 255, g: 0, b: 0 } }

function preCacheColors(colorHexArray) {
  colorCache = {};
  colorHexArray.forEach(hex => {
    colorCache[hex] = {
      r: parseInt(hex.slice(1, 3), 16),
      g: parseInt(hex.slice(3, 5), 16),
      b: parseInt(hex.slice(5, 7), 16)
    };
  });
}

// 帧循环内直接读缓存
onColorPick(colors) {
  colors.forEach(color => {
    const cached = colorCache[color];
    if (cached) {
      const rgb = [
        Math.min(255, cached.r * 1.2),
        Math.min(255, cached.g * 1.2),
        Math.min(255, cached.b * 1.2)
      ];
      this.drawColor(rgb);
    }
  });
}

// 初始化时调用
preCacheColors(['#FF0000', '#00FF00', '#0000FF', /* ... 所有颜色 */]);
```

**判断标准：** 消除循环内的 parseInt 调用，每帧减少 30-50% 的计算量。

---

### 2.3 减少内存分配

**问题：** 每帧 `new Uint8Array` / `new Array` 触发 GC，导致间歇性卡顿。

**方案：** 复用缓冲区，避免每帧分配。

**AI 参考实现：**

```javascript
// ❌ 危险：每帧创建新数组，触发频繁 GC
onAccelerometerChange(res) {
  const data = new Float32Array(3); // 每帧 new
  data[0] = res.x;
  data[1] = res.y;
  data[2] = res.z;
  this.processMotion(data);
}

// ✅ 优化：复用预分配的缓冲区
const motionBuffer = new Float32Array(3); // 只分配一次

onAccelerometerChange(res) {
  motionBuffer[0] = res.x;
  motionBuffer[1] = res.y;
  motionBuffer[2] = res.z;
  this.processMotion(motionBuffer);
}
```

**AI 参考实现（Canvas 像素处理复用缓冲区）：**

```javascript
// ❌ 危险：Canvas 像素处理每帧 new
function processFrame(imageData) {
  const pixels = new Uint8Array(imageData.data.length); // 每帧 new
  for (let i = 0; i < imageData.data.length; i++) {
    pixels[i] = imageData.data[i] * 0.8;
  }
  return pixels;
}

// ✅ 优化：复用缓冲区
let pixelBuffer = null;

function processFrame(imageData) {
  if (!pixelBuffer || pixelBuffer.length !== imageData.data.length) {
    pixelBuffer = new Uint8Array(imageData.data.length);
  }
  for (let i = 0; i < imageData.data.length; i++) {
    pixelBuffer[i] = imageData.data[i] * 0.8;
  }
  return pixelBuffer;
}
```

**判断标准：** 消除帧内 GC，帧率稳定性提升 40-60%。

---

## 三、P1 采样频率调整（无需确认）

### 3.1 帧率节流

**问题：** 高频回调中做重计算，每秒执行 60 次。

**方案：** 限制处理间隔，用时间戳判断。

**AI 参考实现：**

```javascript
// ❌ 危险：每次加速度变化都处理，20fps 也要处理 20 次/秒
onAccelerometerChange(res) {
  this.detectShake(res); // 每次都执行重计算
}

// ✅ 优化：限制处理间隔为 250ms（4fps）
let lastProcessTime = 0;
const PROCESS_INTERVAL = 250; // ms

onAccelerometerChange(res) {
  const now = Date.now();
  if (now - lastProcessTime < PROCESS_INTERVAL) {
    return; // 跳过本次
  }
  lastProcessTime = now;
  this.detectShake(res);
}
```

**AI 参考实现（Canvas 绘制节流）：**

```javascript
// ✅ Canvas 绘制节流
let lastDrawTime = 0;
const DRAW_INTERVAL = 100; // 10fps，海报预览够用

function onColorUpdate(color) {
  const now = Date.now();
  if (now - lastDrawTime < DRAW_INTERVAL) {
    return;
  }
  lastDrawTime = now;
  this.drawCanvas(color);
}
```

---

### 3.2 setData 阈值

**问题：** 颜色微小变化也触发 setData，导致渲染抖动。

**方案：** 同色且 RGB 差异小于阈值时跳过更新。

**AI 参考实现（小象取色取色 setData 阈值）：**

```javascript
// ❌ 危险：每次取色都 setData，即使颜色几乎没变
onColorPick(color) {
  this.setData({ currentColor: color }); // 频繁触发渲染
}

// ✅ 优化：RGB 差异 < 12 时跳过
const RGB_THRESHOLD = 12;
let lastColor = null;

function colorDistance(c1, c2) {
  if (!c1 || !c2) return Infinity;
  const dr = c1.r - c2.r;
  const dg = c1.g - c2.g;
  const db = c1.b - c2.b;
  return Math.sqrt(dr * dr + dg * dg + db * db);
}

onColorPick(color) {
  // 解析当前颜色
  const current = {
    r: parseInt(color.slice(1, 3), 16),
    g: parseInt(color.slice(3, 5), 16),
    b: parseInt(color.slice(5, 7), 16)
  };

  // 颜色差异小于阈值，跳过更新
  if (colorDistance(current, lastColor) < RGB_THRESHOLD) {
    return;
  }

  lastColor = current;
  this.setData({ currentColor: color }); // 只有显著变化才更新
}
```

**判断标准：** setData 调用次数减少 60-80%。

> 加速度计采样频率选择见 [五、运动交互专题](#五运动交互专题)。

---

## 四、P2 视觉降级（需产品确认）

> 以下措施会降低视觉效果，必须经产品确认后再实施。

### 4.1 backdrop-filter: blur → 半透明纯色

**AI 参考实现：**

```css
/* ❌ 原效果：毛玻璃，低端机渲染极慢 */
.glass-panel {
  backdrop-filter: blur(20px);
  background: rgba(255, 255, 255, 0.1);
}

/* ✅ 降级方案：半透明纯色，性能提升 5-10x */
.glass-panel {
  background: rgba(255, 255, 255, 0.85);
  /* 无 backdrop-filter，渲染快 */
}
```

**AI 参考实现（低端机自动降级）：**

```javascript
// 低端机自动降级
function isLowEndDevice() {
  const systemInfo = wx.getSystemInfoSync();

  // 优先用官方性能评分（覆盖 iOS + Android，benchmarkLevel < 10 为低端机）
  // 注：部分老版本基础库可能不存在此字段，需做兜底
  if (systemInfo.benchmarkLevel !== undefined) {
    return systemInfo.benchmarkLevel < 10;
  }

  // 兜底：Android 版本检测（iOS 无此字段，不会进入此分支）
  const androidVersion = systemInfo.system.match(/Android\s([\d.]+)/);
  if (androidVersion && parseFloat(androidVersion[1]) < 9) {
    return true;
  }

  return false;
}

// 根据设备选择样式
Page({
  data: {
    glassStyle: ''
  },
  onLoad() {
    if (isLowEndDevice()) {
      this.setData({ glassStyle: 'glass-fallback' });
    } else {
      this.setData({ glassStyle: 'glass-blur' });
    }
  }
});
```

**判断标准：** 低端机检测到帧率 < 30fps 时自动降级。

---

### 4.2 Canvas 分辨率降低

**AI 参考实现：**

```javascript
// ❌ 原分辨率：900×1200，低端机绘制慢
const canvas = {
  width: 900,
  height: 1200
};

// ✅ 降级方案：675×900（75%），视觉差异小，性能提升明显
const canvas = isLowEndDevice()
  ? { width: 675, height: 900 }
  : { width: 900, height: 1200 };
```

**判断标准：** Canvas 绘制像素减少 44%，绘制耗时降低 40-50%。

---

### 4.3 CSS 动画数量合并

**AI 参考实现：**

```css
/* ❌ 多个元素各自动画，触发多次合成 */
.element-1 { animation: fadeIn 0.3s; }
.element-2 { animation: slideUp 0.3s; }
.element-3 { animation: scaleIn 0.3s; }

/* ✅ 合并为容器动画，只触发一次合成 */
.container {
  animation: combinedEntrance 0.3s;
}
@keyframes combinedEntrance {
  from { opacity: 0; transform: translateY(10px) scale(0.95); }
  to { opacity: 1; transform: translateY(0) scale(1); }
}
```

---

### 4.4 粒子/噪点数量减少

**AI 参考实现：**

```javascript
// ❌ 原方案：600 个粒子，低端机卡顿
const PARTICLE_COUNT = 600;

// ✅ 分级降级
const PARTICLE_COUNT = isLowEndDevice() ? 100 : 200;
// 高端机也降到 200，视觉差异不大，性能提升明显

// 海报噪点优化
const NOISE_COUNT = isLowEndDevice() ? 100 : 200;
// 从 600 降到 200，纹理效果仍自然，耗时降 60%
```

**效果对比：**

| 粒子数 | 耗时(低端机) | 视觉效果 | 可接受？ |
|--------|-------------|---------|---------|
| 600 | 120ms/帧 | 丰富细腻 | ❌ 卡顿 |
| 200 | 48ms/帧 | 自然 | ✅ 可接受 |
| 100 | 24ms/帧 | 可见简化 | ⚠️ 需确认 |

---

## 五、运动交互专题

> 本章节是运动交互（加速度计 / 摇一摇 / 相机状态）的专项优化。非运动交互产品可跳过本节。
> 主要给 AI 执行运动交互的参考实现，给人判断采样频率是否合理。

### 5.1 加速度计采样频率选择

| 频率 | 值 | 适用场景 | 耗电 |
|------|-----|---------|------|
| game | 60fps | 体感游戏、高精度运动追踪 | 高 |
| ui | 30fps | UI 交互、手势识别 | 中 |
| normal | 20fps | 摇一摇、运动检测 | 低 |

**AI 参考实现：**

```javascript
// 场景1：摇一摇 → normal (20fps) 足够
wx.startAccelerometer({ frequency: 'normal' });

// 场景2：手势识别 → ui (30fps)
wx.startAccelerometer({ frequency: 'ui' });

// 场景3：体感游戏 → game (60fps)
wx.startAccelerometer({ frequency: 'game' });

// 选择原则：
// 1. 先用最低频率
// 2. 测试是否满足交互需求
// 3. 不满足再升一档
```

**判断标准：**
- 用户需要实时感知加速度变化吗？（体感操作 → game）
- 只是检测某个动作吗？（摇一摇 → normal）
- 不确定？先用 normal，不够再升 ui

---

### 5.2 摇一摇触发：加速度变化阈值检测

**AI 参考实现：**

```javascript
Page({
  data: {
    isShaking: false
  },

  // 摇一摇检测参数
  SHAKE_THRESHOLD: 15,      // 加速度变化阈值
  SHAKE_INTERVAL: 1000,     // 两次摇动最小间隔(ms)
  lastShakeTime: 0,
  lastAccel: { x: 0, y: 0, z: 0 },

  onLoad() {
    // 使用 normal 频率，摇一摇不需要高精度
    wx.startAccelerometer({ frequency: 'normal' });
    wx.onAccelerometerChange(this.onAccelerometerChange.bind(this));
  },

  onUnload() {
    wx.stopAccelerometer();
    wx.offAccelerometerChange();
  },

  onAccelerometerChange(res) {
    const now = Date.now();

    // 节流：两次检测间隔至少 1s
    if (now - this.lastShakeTime < this.SHAKE_INTERVAL) {
      return;
    }

    // 计算加速度变化量
    const deltaX = Math.abs(res.x - this.lastAccel.x);
    const deltaY = Math.abs(res.y - this.lastAccel.y);
    const deltaZ = Math.abs(res.z - this.lastAccel.z);

    // 更新上次加速度
    this.lastAccel = { x: res.x, y: res.y, z: res.z };

    // 变化量超过阈值 → 触发摇一摇
    const totalDelta = deltaX + deltaY + deltaZ;
    if (totalDelta > this.SHAKE_THRESHOLD) {
      this.lastShakeTime = now;
      this.triggerShake();
    }
  },

  triggerShake() {
    if (this.data.isShaking) return; // 防重复触发
    this.setData({ isShaking: true });

    // 摇一摇后的业务逻辑（如：随机切换配色方案）
    this.handleShakeAction();

    // 重置状态
    setTimeout(() => {
      this.setData({ isShaking: false });
    }, 1500);
  },

  handleShakeAction() {
    console.log('摇一摇触发');
  }
});
```

---

### 5.3 运动检测状态管理：cameraReady 与 showCamera 同步

**问题：** `cameraReady` 和 `showCamera` 不同步，导致相机状态混乱（如后台返回时相机无法重启）。

**AI 参考实现：**

```javascript
Page({
  data: {
    cameraReady: false,
    showCamera: false
  },

  // 开启相机
  startCamera() {
    // ✅ 同步设置，避免状态不一致
    this.setData({
      cameraReady: true,
      showCamera: true
    });
  },

  // 关闭相机
  stopCamera() {
    this.setData({
      cameraReady: false,
      showCamera: false
    });
  },

  // 热启动状态恢复
  onShow() {
    // ✅ 如果 cameraReady 已为 true，直接重启相机
    if (this.data.cameraReady) {
      this.setData({ showCamera: true });
      // 重新启动加速度计监听
      wx.startAccelerometer({ frequency: 'normal' });
    }
  },

  // 后台时暂停
  onHide() {
    // ✅ 保持 cameraReady 状态，只关闭显示
    this.setData({ showCamera: false });
    wx.stopAccelerometer();
  }
});
```

**关键规则：**
- `cameraReady`：表示相机模块已初始化（逻辑状态）
- `showCamera`：表示相机 UI 是否显示（视觉状态）
- 两者必须同步设置，不能只改一个
- `onHide` 只关 `showCamera`，保留 `cameraReady`
- `onShow` 检查 `cameraReady`，若为 true 直接恢复 `showCamera`

---

### 5.4 热启动状态恢复

**场景：** 用户切到后台再回来，相机和加速度计需要恢复。

**AI 参考实现：**

```javascript
Page({
  data: {
    cameraReady: false,
    showCamera: false,
    accelerometerActive: false
  },

  onShow() {
    // 热启动恢复：检查之前的状态
    if (this.data.cameraReady) {
      // 相机已初始化，直接恢复显示
      this.setData({ showCamera: true });
    }

    if (this.data.accelerometerActive) {
      // 重新启动加速度计
      wx.startAccelerometer({ frequency: 'normal' });
      wx.onAccelerometerChange(this.onAccelerometerChange.bind(this));
    }
  },

  onHide() {
    // 暂停但不重置状态
    this.setData({ showCamera: false });
    wx.stopAccelerometer();
    wx.offAccelerometerChange();
  },

  onUnload() {
    // 完全销毁时才重置
    this.setData({
      cameraReady: false,
      showCamera: false,
      accelerometerActive: false
    });
    wx.stopAccelerometer();
    wx.offAccelerometerChange();
  }
});
```

---

### 5.5 Canvas 海报噪点优化

**问题：** 海报生成时噪点过多（600个），低端机生成耗时 >2s。

**方案：** 噪点数量从 600 降到 200，纹理效果仍自然，耗时降 60%。

**AI 参考实现：**

```javascript
// ❌ 原方案：600 个噪点
function generateNoise(ctx, width, height) {
  for (let i = 0; i < 600; i++) {
    const x = Math.random() * width;
    const y = Math.random() * height;
    const alpha = Math.random() * 0.15;
    ctx.fillStyle = `rgba(0, 0, 0, ${alpha})`;
    ctx.fillRect(x, y, 1, 1);
  }
}

// ✅ 优化：200 个噪点 + 预计算随机值
const NOISE_COUNT = 200;
let noiseCache = null;

function generateNoise(ctx, width, height) {
  // 预计算随机位置（只需一次）
  if (!noiseCache || noiseCache.width !== width) {
    noiseCache = {
      width: width,
      points: new Float32Array(NOISE_COUNT * 3) // x, y, alpha
    };
    for (let i = 0; i < NOISE_COUNT; i++) {
      noiseCache.points[i * 3] = Math.random() * width;
      noiseCache.points[i * 3 + 1] = Math.random() * height;
      noiseCache.points[i * 3 + 2] = Math.random() * 0.15;
    }
  }

  // 使用预计算的值绘制
  for (let i = 0; i < NOISE_COUNT; i++) {
    const x = noiseCache.points[i * 3];
    const y = noiseCache.points[i * 3 + 1];
    const alpha = noiseCache.points[i * 3 + 2];
    ctx.fillStyle = `rgba(0, 0, 0, ${alpha})`;
    ctx.fillRect(x, y, 1, 1);
  }
}
```

**效果对比：**

| 方案 | 噪点数 | 耗时(低端机) | 视觉效果 |
|------|--------|-------------|---------|
| 原方案 | 600 | 120ms | 丰富细腻 |
| 优化后 | 200 | 48ms（降60%） | 自然，差异不明显 |
| 极限 | 100 | 24ms | 可见简化 |

---

## 低端机性能检查清单

> 上线前对照此清单逐项检查，全部通过才能发版。

```
### 低端机性能检查清单（小象取色）

#### AI 性能场景
- [ ] AI 文本响应 > 200 字有流式渲染（分片 setData）
- [ ] AI 响应 > 1s 有骨架屏，> 8s 有超时兜底
- [ ] 历史记录列表 > 50 条有分页，> 500 条考虑虚拟列表
- [ ] 首屏无白屏 > 1s（缓存上次结果 + 请求并行）
- [ ] AI 请求可取消，连续触发不产生竞态
- [ ] AI 调用按钮在请求中 disabled

#### P0 纯计算优化
- [ ] Math.exp / sqrt / pow 不在帧循环内（已预计算查表）
- [ ] 颜色/数据查找已预缓存（启动时一次性缓存）
- [ ] 无每帧 new Array / new Uint8Array（已复用缓冲区）

#### P1 采样频率
- [ ] 帧处理有节流（时间戳判断间隔）
- [ ] setData 有变化阈值（RGB 差异 < 阈值时跳过）

#### P2 视觉降级（如已确认）
- [ ] backdrop-filter: blur 已降级为半透明纯色（低端机）
- [ ] Canvas 分辨率已降低（低端机 675×900）
- [ ] CSS 动画已合并
- [ ] 粒子/噪点数量已减少（200 或以下）

#### 运动交互
- [ ] 加速度计频率为 normal 或 ui（非 game，除非体感游戏）
- [ ] cameraReady 与 showCamera 同步设置
- [ ] onShow 时检查 cameraReady 并恢复
- [ ] onHide 时暂停加速度计和相机显示
- [ ] 摇一摇有防重复触发（间隔 ≥ 1s）

#### 真机测试
- [ ] Android 8 以下机型测试通过
- [ ] 首屏加载 < 3s
- [ ] 核心操作帧率 ≥ 30fps
- [ ] 连续操作 5 分钟内存不泄漏
- [ ] 断网/弱网场景不崩溃
```

**判断标准：** 真机测试 5 项为硬门槛（所有产品），含 AI 能力的产品另加 AI 性能场景 6 项。任一硬门槛未通过不发版。

---

## 使用方式

把你的性能问题告诉我，我帮你分级优化、给参考实现代码。你只需用自然语言描述"哪里卡"，我按本规范定位瓶颈并产出方案。

**对话示例：**
- "用户点 AI 命名后白屏等 3 秒，太慢了" → 我帮你做流式渲染 + 骨架屏 + 超时兜底（1.1 + 1.2）
- "历史记录多了之后滚动卡顿" → 我帮你做分页加载或虚拟列表（1.3）
- "进入取色页要白屏一会" → 我帮你做冷启动优化：缓存上次结果 + 请求并行（1.4）
- "用户连点 AI 命名，结果乱跳" → 我帮你做请求取消 + 防抖（1.5）
- "低端机取色卡顿" → 我帮你按 P0/P1/P2 分级定位计算瓶颈
- "海报生成要 2 秒" → 我帮你优化 Canvas 噪点数量和分辨率

---

## 与其他技能的关系

| 技能 | 关系 |
|------|------|
| `xx-blocks` | 组件实现时遵循性能优化规范 |
| `xx-ai` | AI 功能的流式响应、骨架屏、请求取消直接套用本 skill 第一章 |
| `xx-track` | 用 `duration_ms` 字段定位 AI 接口和海报生成的性能瓶颈 |
| `xx-iterate` | 性能问题按 P0/P1/P2 分级，纳入迭代流程 |

---

## 方法论来源

- **零体验降级原则** — 先优化不降级，产品确认后才降级
- **RAIL 性能模型** — Google，Response/Animation/Idle/Load 分场景优化
- **Stale-while-revalidate** — HTTP 缓存策略在首屏冷启动的应用
- **虚拟列表** — 仅渲染可见区域的长列表优化通用方案
