---
name: xx-track
description: 微信小程序数据埋点与 WeAnalysis 后台配置全流程：事件命名规范、五阶段埋点事件表、转化漏斗分析、数据驱动决策8步框架、wx.reportEvent 代码示例、批量创建JSON模板、元事件配置陷阱、AI 质量监控。主要给 AI 执行埋点规范，给人判断埋点是否覆盖核心旅程。
---

# xx-track 数据埋点与分析技能包

> 上线运营阶段 · 数据驱动决策：埋点策略 + WeAnalysis 配置 + AI 质量监控。
> 先埋对核心旅程，再看 AI 输出质量。

本技能包主要给 AI 执行埋点规范，给人判断埋点是否覆盖核心旅程。代码示例是 AI 的参考实现，不是让用户手敲的教程。用户只需用自然语言描述"我想看哪条路径的转化"，AI 按本规范产出事件表和参考实现。

贯穿案例：**小象取色**（摄像头实时取色 → AI 中式颜色命名 → AI 配色方案 → Canvas 生成配色海报 → 历史记录）。

---

## 目录

- [一、埋点策略](#一埋点策略)
- [二、WeAnalysis 后台配置](#二weanalysis-后台配置)
- [三、最佳实践](#三最佳实践)
- [四、AI 质量监控](#四ai-质量监控)

---

## 一、埋点策略

### 1.1 事件命名规范

**统一格式：`<page>_<action>[_<detail>]`**

| 组成部分 | 说明 | 示例 |
|----------|------|------|
| `<page>` | 页面标识（小写） | `home`、`picker`、`history` |
| `<action>` | 动作标识（小写） | `view`、`click`、`submit` |
| `<detail>` | 详情标识（可选） | `success`、`fail`、`cancel` |

**小象取色命名示例：**

```
home_view                    # 首页浏览
home_button_click            # 首页按钮点击
picker_view                  # 取色页浏览
picker_start_click           # 开始取色点击
color_pick_success           # 取色成功
ai_name_success              # AI 命名成功
ai_name_fail                 # AI 命名失败
poster_generate_success      # 海报生成成功
picker_save_click            # 保存颜色点击
```

**AI 执行约束：**
- 全小写，单词用下划线 `_` 分隔
- 不超过 32 个字符
- 不使用拼音，统一用英文
- 发布后不可修改事件名（改了 = 历史数据断裂）

**判断标准：** 看事件名能否一眼看出"在哪个页面做了什么"。看不懂就是不合格。

### 1.2 推荐埋点事件表

按用户旅程五个阶段设计埋点。以小象取色为例：

#### 阶段一：入口（用户从哪里来）

| 事件名 | 触发时机 | 属性 |
|--------|----------|------|
| `app_launch` | 小程序冷启动 | `scene`（场景值）、`source`（来源） |
| `page_view` | 任意页面展示 | `page_name`、`referrer`（来源页） |
| `home_view` | 首页展示 | `entry_type`（扫码/搜索/分享） |

#### 阶段二：操作（用户做了什么）

| 事件名 | 触发时机 | 属性 |
|--------|----------|------|
| `home_button_click` | 首页主按钮点击 | `button_name`、`position` |
| `picker_start_click` | 开始取色点击 | `entry_type`（相机/相册） |
| `ai_name_click` | 触发 AI 命名 | `color_hex`、`source` |

#### 阶段三：成功（转化完成）

| 事件名 | 触发时机 | 属性 |
|--------|----------|------|
| `color_pick_success` | 取色成功 | `color_hex`、`pick_method` |
| `ai_name_success` | AI 命名成功 | `color_hex`、`name`、`duration_ms` |
| `poster_generate_success` | 海报生成成功 | `template_id`、`duration_ms` |
| `picker_save_success` | 颜色保存成功 | `color_hex`、`name` |

#### 阶段四：错误（出错了）

| 事件名 | 触发时机 | 属性 |
|--------|----------|------|
| `color_pick_fail` | 取色失败 | `error_code`、`error_msg` |
| `ai_name_fail` | AI 命名失败 | `error_code`、`error_msg`、`duration_ms` |
| `api_error` | 接口请求失败 | `api_name`、`status_code`、`error_msg` |

#### 阶段五：退出（用户离开）

| 事件名 | 触发时机 | 属性 |
|--------|----------|------|
| `page_exit` | 页面离开 | `page_name`、`stay_duration`（停留时长） |
| `app_exit` | 小程序切后台 | `session_duration`（会话时长） |

**AI 执行约束：** 成功事件必须带 `duration_ms`（耗时），用于性能监控；失败事件必须带 `error_code`，不带等于白埋。

### 1.3 转化漏斗分析

**小象取色核心漏斗：`picker_view → picker_start_click → color_pick_success → ai_name_success → picker_save_click`**

```
                picker_view
               /         \
        picker_start_click      (流失：60%)
           /       \
     color_pick_success    (流失：10%)
        /        \
   ai_name_success     (流失：30%)
      /         \
picker_save_click    (流失：50%)
```

#### 漏斗计算示例（AI 参考实现）

```javascript
// utils/funnel-analysis.js

/**
 * 计算转化漏斗数据
 * @param {object} rawData - 各阶段事件触发数
 * @returns {object} 漏斗分析结果
 */
function calcFunnel(rawData) {
  const stages = [
    { name: '取色页浏览', event: 'picker_view', count: rawData.picker_view },
    { name: '开始取色', event: 'picker_start_click', count: rawData.picker_start_click },
    { name: '取色成功', event: 'color_pick_success', count: rawData.color_pick_success },
    { name: 'AI命名成功', event: 'ai_name_success', count: rawData.ai_name_success },
    { name: '保存颜色', event: 'picker_save_click', count: rawData.picker_save_click },
  ];

  let prevCount = stages[0].count;
  stages.forEach((stage, index) => {
    if (index === 0) {
      stage.conversionRate = '100%';
    } else {
      // 用数字运算，最后再格式化为字符串，避免隐式转换
      const rateNum = stage.count / prevCount * 100;
      stage.conversionRate = `${rateNum.toFixed(1)}%`;
      stage.dropOffRate = `${(100 - rateNum).toFixed(1)}%`;
    }
    prevCount = stage.count;
  });

  // 整体转化率
  const overallRate = (stages[4].count / stages[0].count * 100).toFixed(1);

  return { stages, overallConversion: `${overallRate}%` };
}

// 示例数据
const sampleData = {
  picker_view: 10000,
  picker_start_click: 4000,
  color_pick_success: 3600,
  ai_name_success: 2520,
  picker_save_click: 1260,
};

console.log(calcFunnel(sampleData));
// 输出：
// 取色页浏览: 10000 → 100%
// 开始取色: 4000  → 40.0%（流失 60.0%）
// 取色成功: 3600  → 90.0%（流失 10.0%）
// AI命名成功: 2520  → 70.0%（流失 30.0%）
// 保存颜色: 1260  → 50.0%（流失 50.0%）
// 整体转化率: 12.6%
```

#### 漏斗关键问题

| 指标 | 关注点 | 行动 |
|------|--------|------|
| 页面→开始取色流失 60% | 首屏吸引力不足？取色按钮位置不对？ | 优化首屏设计、调整 CTA 位置 |
| 开始→取色成功流失 10% | 相机授权拒绝？取色逻辑报错？ | 检查授权引导、取色稳定性 |
| 取色→AI命名流失 30% | 命名按钮不明显？AI 等待太久？ | 强化命名入口、加骨架屏 |
| AI命名→保存流失 50% | 命名结果不满意？保存入口远？ | 优化命名质量、保存入口前置 |

**判断标准：** 每个流失节点都要能回答"为什么流失"和"怎么优化"，只报数字不合格。

### 1.4 代码埋点封装（AI 参考实现）

```javascript
// utils/track.js

/**
 * 统一埋点上报函数
 * @param {string} eventId - 事件ID（命名规范：<page>_<action>[_<detail>]）
 * @param {object} data - 事件属性（key-value，value 必须是字符串）
 */
function track(eventId, data = {}) {
  // 所有 value 转为字符串（WeAnalysis 要求）
  const stringData = {};
  for (const key in data) {
    stringData[key] = String(data[key]);
  }

  wx.reportEvent(eventId, stringData);
}

/**
 * 页面浏览埋点（自动埋点）
 * 在 Page onLoad 中调用
 */
function trackPageView(pageName) {
  const referrer = getCurrentPages().length > 1
    ? getCurrentPages()[getCurrentPages().length - 2].route
    : 'direct';

  track('page_view', {
    page_name: pageName,
    referrer: referrer,
  });
}

/**
 * 点击埋点
 */
function trackClick(pageName, buttonName, position = '') {
  track(`${pageName}_button_click`, {
    button_name: buttonName,
    position: position,
  });
}

/**
 * 成功埋点
 */
function trackSuccess(pageName, action, extra = {}) {
  track(`${pageName}_${action}_success`, extra);
}

/**
 * 失败埋点
 */
function trackFail(pageName, action, errorCode, errorMsg) {
  track(`${pageName}_${action}_fail`, {
    error_code: errorCode,
    error_msg: errorMsg,
  });
}

module.exports = { track, trackPageView, trackClick, trackSuccess, trackFail };
```

**小象取色取色页使用示例（AI 参考实现）：**

```javascript
// pages/picker/picker.js
const { trackPageView, trackClick, trackSuccess, trackFail } = require('../../utils/track');

Page({
  onLoad() {
    trackPageView('picker');
  },

  onTapStartPick() {
    trackClick('picker', 'start_pick', 'top');

    // 业务逻辑...
    startPickColor()
      .then((res) => {
        trackSuccess('picker', 'color_pick', {
          color_hex: res.colorHex,
          pick_method: res.method,
        });
      })
      .catch((err) => {
        trackFail('picker', 'color_pick', err.code, err.message);
      });
  },

  onTapAiName() {
    trackClick('picker', 'ai_name', 'bottom');
    callAiName(this.data.currentColor)
      .then((res) => {
        trackSuccess('picker', 'ai_name', {
          color_hex: this.data.currentColor,
          name: res.name,
          duration_ms: res.duration,
        });
      })
      .catch((err) => {
        trackFail('picker', 'ai_name', err.code, err.message);
      });
  },
});
```

**AI 执行约束：** 成功回调里必须埋成功事件，catch 里必须埋失败事件，二者成对出现，不能只埋成功不埋失败。

### 1.5 微信数据分析后台使用

**路径：微信公众平台 → 数据分析 → 自定义分析 → 事件分析**

在后台可以查看以下关键指标：

| 指标 | 说明 | 关注点 |
|------|------|--------|
| 触发次数 | 事件被触发的总次数 | 看趋势，不看绝对值 |
| 触发人数 | 触发该事件的独立用户数 | **核心指标**，看这个 |
| 人均次数 | 触发次数 / 触发人数 | 判断用户是否重复操作 |
| 趋势图 | 按天/周/月展示变化 | 看异常波动 |

### 1.6 数据驱动决策 8 步框架

```
①定义目标 → ②确定指标 → ③设计埋点 → ④开发上线
                                        ↓
⑧迭代优化 ← ⑦制定方案 ← ⑥分析数据 ← ⑤收集数据
```

**Step 1：定义目标**
- 这次要看什么？（例：取色→保存转化率低，想知道哪一步流失最多）

**Step 2：确定指标**
- 用什么指标衡量？（例：5 步漏斗每步的转化率）

**Step 3：设计埋点**
- 需要哪些事件？（例：5 个阶段的埋点事件 + 属性）

**Step 4：开发上线**
- 代码埋点 + 后台配置事件 → 发布

**Step 5：收集数据**
- 等 24 小时，数据出现在后台

**Step 6：分析数据**
- 看漏斗哪步流失最多？看趋势图哪天异常？

**Step 7：制定方案**
- 流失最多的环节怎么优化？

**Step 8：迭代优化**
- 上线优化 → 回到 Step 5 继续观察

---

## 二、WeAnalysis 后台配置

### 2.1 后台路径

**公众平台 → We分析 → 数据管理 → 上报管理 → 元事件**

> We分析是微信官方的数据分析平台，比旧版"自定义分析"更强大，支持事件属性、漏斗分析、留存分析等。

### 2.2 代码示例（AI 参考实现）

在小程序前端使用 `wx.reportEvent` 上报事件：

```javascript
// 基础用法：无属性事件
wx.reportEvent('home_view');

// 带属性事件：value 必须是字符串
wx.reportEvent('picker_start_click', {
  entry_type: 'camera',
  position: 'top',
});

// 数字也必须转成字符串
wx.reportEvent('ai_name_success', {
  color_hex: '#A23B3B',
  name: '故宫红',
  duration_ms: String(1200),      // 数字转字符串
});

// 布尔值转字符串
wx.reportEvent('color_pick_success', {
  is_first_pick: 'true',      // 布尔转字符串
});
```

**封装后的安全上报函数（AI 参考实现，推荐使用）：**

```javascript
// utils/report-event.js

/**
 * 安全上报事件（自动处理 value 类型转换）
 * @param {string} eventId - 事件ID
 * @param {object} params - 事件参数
 */
function reportEvent(eventId, params = {}) {
  // 所有 value 转为字符串
  const safeParams = {};
  for (const key in params) {
    if (params[key] !== null && params[key] !== undefined) {
      safeParams[key] = String(params[key]);
    }
  }

  try {
    wx.reportEvent(eventId, safeParams);
  } catch (e) {
    console.error(`[上报失败] ${eventId}:`, e);
  }
}

module.exports = { reportEvent };
```

**AI 执行约束：** 上报前必须做 `String()` 转换，不能直接传数字/布尔；上报失败要 try-catch，不能让埋点报错崩了业务。

### 2.3 批量创建 JSON 模板

We分析后台支持通过 JSON 批量导入事件定义，免去手动逐个创建：

```json
[
  {
    "event_id": "home_view",
    "event_name": "首页浏览",
    "event_comment": "用户进入首页时触发",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "picker_start_click",
    "event_name": "开始取色点击",
    "event_comment": "用户点击开始取色按钮",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "color_pick_success",
    "event_name": "取色成功",
    "event_comment": "用户成功取到一个颜色",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "ai_name_success",
    "event_name": "AI命名成功",
    "event_comment": "AI成功为颜色生成中式命名",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "ai_name_fail",
    "event_name": "AI命名失败",
    "event_comment": "AI命名接口失败",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "picker_save_success",
    "event_name": "保存颜色成功",
    "event_comment": "用户成功保存颜色到历史",
    "report_src": 0,
    "event_key_list": []
  }
]
```

**字段说明：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `event_id` | string | 事件ID，**大小写敏感**，发布后不可修改 |
| `event_name` | string | 事件显示名称（中英文均可） |
| `event_comment` | string | 事件描述备注 |
| `report_src` | number | 上报来源：`0` = 客户端上报（小程序前端） |
| `event_key_list` | array | 事件属性列表，**初始创建时设为空数组 `[]`** |

### 2.4 关键陷阱：属性必须先注册

**这是最常见的坑：** 带属性的事件，如果属性未在"属性"tab 中先注册，创建事件时会失败。

```json
// ❌ 错误做法：直接在 event_key_list 中写属性
{
  "event_id": "ai_name_success",
  "event_name": "AI命名成功",
  "report_src": 0,
  "event_key_list": [
    {
      "key": "color_hex",
      "name": "颜色值",
      "type": "string"
    }
  ]
  // 如果 color_hex 属性未在"属性"tab注册，创建失败！
}
```

```json
// ✅ 正确做法一：先创建无属性事件，成功后再添加属性
// 第一步：创建事件（空属性）
{
  "event_id": "ai_name_success",
  "event_name": "AI命名成功",
  "report_src": 0,
  "event_key_list": []
}
// 第二步：在"属性"tab注册 color_hex 属性
// 第三步：回到事件编辑，添加 color_hex 属性
```

```json
// ✅ 正确做法二：初始创建时 event_key_list 设为空数组（推荐）
{
  "event_id": "ai_name_success",
  "event_name": "AI命名成功",
  "report_src": 0,
  "event_key_list": []
}
// 创建成功后，再单独在"属性"tab中注册所有需要的属性
// 然后在事件编辑中关联这些属性
```

**AI 执行约束：** 生成批量导入 JSON 时，所有 `event_key_list` 一律设为 `[]`，保证 100% 成功率，属性后续单独注册。

### 2.5 推荐做法

**初始创建时 `event_key_list` 设为空数组 `[]`，保证 100% 成功率。**

```json
[
  {
    "event_id": "app_launch",
    "event_name": "应用启动",
    "event_comment": "小程序冷启动",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "page_view",
    "event_name": "页面浏览",
    "event_comment": "任意页面展示",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "page_exit",
    "event_name": "页面退出",
    "event_comment": "页面离开",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "ai_name_click",
    "event_name": "AI命名点击",
    "event_comment": "用户触发AI命名",
    "report_src": 0,
    "event_key_list": []
  },
  {
    "event_id": "api_error",
    "event_name": "接口错误",
    "event_comment": "API请求失败",
    "report_src": 0,
    "event_key_list": []
  }
]
```

> **流程：** 批量导入空属性事件 → 全部成功 → 在"属性"tab注册属性 → 回到事件编辑关联属性 → 代码中使用 `wx.reportEvent(eventId, { key: 'value' })`。

### 2.6 验证流程

```
1. 后台创建事件（JSON批量导入）
       ↓
2. 前端代码埋点（wx.reportEvent）
       ↓
3. 发布小程序（体验版/正式版）
       ↓
4. 等待 24 小时 ⏳（重要！数据不是实时的）
       ↓
5. 后台查看：We分析 → 数据管理 → 元事件
       ↓
6. 确认事件有触发数据 ✅
       ↓
7. 配置漏斗/留存分析
```

### 2.7 关键规则汇总

| 规则 | 说明 | 违反后果 |
|------|------|----------|
| `report_src: 0` | 客户端上报（小程序前端） | 数据无法上报 |
| ID 大小写敏感 | `Home_View` 和 `home_view` 是不同事件 | 数据分散到两个事件 |
| 发布后不改 ID | 事件ID发布后不可修改 | 历史数据断裂 |
| value 必须是字符串 | `String(99.9)` 而非 `99.9` | 上报失败或数据异常 |
| 属性先注册 | 在"属性"tab注册后才能用于事件 | 事件创建失败 |
| 初始空属性 | `event_key_list: []` 保证创建成功 | — |
| 等 24 小时 | 数据非实时，T+1 出现 | 以为埋点失败，反复修改 |

**判断标准：** 发布当天不要反复检查"为什么没数据"，T+1 是正常的；如果第二天还没数据，再排查上报代码和事件配置。

---

## 三、最佳实践

### 3.1 发布后等 24 小时数据才出现

We分析的数据是 **T+1** 的，今天发布的小程序，明天才能在后台看到数据。

```
今天 14:00 发布 → 明天 14:00 左右数据出现
```

**不要**在发布当天反复检查"为什么没数据"。

### 3.2 看触发人数，不看总触发次数

| 指标 | 为什么看 | 为什么不看 |
|------|----------|-----------|
| 触发**人数** | 反映有多少真实用户触发了事件 | ✅ 核心指标 |
| 触发**次数** | 一个用户可能触发多次，数据被稀释 | ❌ 容易误导 |

**示例：**
- 1000 次触发 / 100 人 = 每人触发 10 次 → 可能是 bug 导致重复触发
- 1000 次触发 / 800 人 = 每人触发 1.25 次 → 正常使用行为

### 3.3 不要埋所有事件，聚焦核心用户旅程

**埋点不是越多越好。** 太多事件会导致：
- 后台数据噪音大，难以分析
- 维护成本高
- 事件数量有上限

**AI 执行约束：** 只埋核心路径的 5-10 个事件，非核心交互不埋。

```
小象取色核心旅程：取色页 → 开始取色 → 取色成功 → AI命名 → 保存
埋点：    picker_view → picker_start_click → color_pick_success → ai_name_success → picker_save_click
```

### 3.4 事件名发布后不改

事件ID发布后修改，会导致：
- 历史数据和新数据断裂
- 漏斗分析无法连续

**AI 执行约束：** 如果必须修改，新建一个新事件，不要改旧的。

### 3.5 每周 review 而非每天

数据波动是正常的，天天看容易过度反应。

**推荐节奏：**

| 频率 | 看什么 | 动作 |
|------|--------|------|
| 每天 | 异常告警（错误率飙升） | 紧急修复 |
| 每周 | 核心漏斗转化率趋势 | 制定优化方案 |
| 每月 | 全量指标 review | 产品迭代决策 |

---

## 完整配置流程清单

```
□ 1. 设计埋点事件表（按五阶段）
□ 2. 确定事件命名（<page>_<action>[_<detail>]）
□ 3. 编写批量创建 JSON（event_key_list 全部设为 []）
□ 4. 后台导入 JSON：We分析 → 数据管理 → 上报管理 → 元事件
□ 5. 在"属性"tab注册事件属性
□ 6. 回到事件编辑，关联属性
□ 7. 前端代码埋点（封装 track 函数）
□ 8. 确认所有 value 为字符串
□ 9. 发布小程序
□ 10. 等待 24 小时
□ 11. 后台验证数据出现
□ 12. 配置漏斗分析
□ 13. 设置每周 review 节奏
```

**判断标准：** 13 项全部打勾才算埋点上线完成。任一项未完成不能进入数据分析阶段。

---

## 四、AI 质量监控

> 业务埋点告诉你"用户点了什么"，AI 监控告诉你"AI 输出质量在变好还是变坏"。
> 小象取色的 AI 命名质量下降时，用户不会点"不满意"，而是默默不保存——等保存率降时已经晚了。

### 4.1 为什么 AI 产品需要单独的质量监控

业务指标（点击率、转化率）反映用户行为，但有滞后性：
- AI 命名质量下降 → 用户不满意 → 默默不保存 → 保存率才下降
- 等业务指标降时，已经流失一批用户了

AI 质量监控的作用：在业务指标下降前，提前发现 AI 质量问题。

### 4.2 三个关键风险

| 风险 | 说明 | 监控方式 |
|------|------|---------|
| 数据漂移 | 用户输入分布变了（取色从红色系变到蓝色系） | 采样输入做聚类，看分布变化 |
| 概念漂移 | 用户期望变了（从要"故宫红"变到要"莫兰迪色"） | 定期人工评估同一批样本 |
| 静默失败 | AI 还在返回但质量已差（命名越来越"不像中式名"） | 重试率 + 人工抽样 |

### 4.3 最小 AI 监控看板

| 指标 | 怎么采 | 阈值 |
|------|--------|------|
| 人工评分 | 每天采样 N 条 AI 命名打分 | 均分 < 阈值告警 |
| 用户重试率 | 重试次数 / 总次数 | > 20% 告警 |
| 平均 token 消耗 | 记录每次 AI 调用 | 突增 50% 告警 |
| 负反馈率 | 用户点"重新命名"比例 | > 10% 告警 |
| 保存率 | 保存颜色 / AI命名成功 | 下降 10% 告警 |

### 4.4 采样策略

全量评估太贵，采样即可：
- 每天采样 20-50 条 AI 命名输出
- 覆盖不同颜色和时段
- 优先采"重新命名"和"未保存"的 case

### 4.5 告警机制

```
触发告警 → 不是立刻回滚，而是：
1. 确认是不是数据问题（用户取色颜色分布变了）
2. 确认是不是 prompt 问题（最近改过命名 prompt）
3. 确认是不是模型问题（混元/豆包服务方更新）
→ 对症下药
```

### 4.6 与 xx-data 的关系

- xx-data 讲离线评估（上线前用评估集跑分）
- 本节讲在线监控（上线后看真实表现）
- 两者结合：离线指标好 + 在线监控稳 = 可持续

---

## 示例：小象取色 埋点全景

| 关注点 | 事件 | 用途 |
|--------|------|------|
| 用户从哪来 | app_launch / home_view | 看场景值分布，优化拉新渠道 |
| 取色转化 | picker_view → picker_start_click → color_pick_success | 看取色漏斗，定位流失 |
| AI 命名质量 | ai_name_success / ai_name_fail / 重新命名率 | 监控 AI 输出质量 |
| 商业转化 | poster_generate_success / 付费模板点击 | 看付费转化 |
| 性能 | ai_name_success.duration_ms / poster_generate_success.duration_ms | 监控 AI 和海报生成耗时 |

---

## 使用方式

把你的埋点需求或数据问题告诉我，我帮你设计方案、产出事件表和参考实现代码。你只需用自然语言描述"我想看什么"，我按本规范产出。

**对话示例：**
- "小象取色上线了，我想知道从取色到保存哪一步流失最多" → 我帮你规划取色漏斗的 5 个埋点事件 + 属性
- "我看后台 AI 命名的成功率掉了，怎么监控质量" → 我帮你搭 AI 命名质量监控看板（重试率 + 采样评分）
- "转化率低，怎么定位是哪一步的问题" → 我帮你分析漏斗流失节点，给优化方向
- "帮我生成能在 We分析 后台批量导入的事件 JSON" → 我按 event_key_list 全空规范产出 JSON

> **技能包信息**：xx-track | AI 产品 Builder 技能包 | 数据埋点与分析模块

## 与其他技能的关系

- **前置：** xx-data 建立离线评估集，本 skill 监控上线后在线表现
- **并行：** xx-iterate 用本 skill 的数据验证迭代效果
- **下游：** xx-optimize 性能问题靠本 skill 的 `duration_ms` 字段定位
- **回溯：** xx-safety 监控"内容过滤触发率"，反向校准敏感词库

## 方法论来源

- **We分析官方文档** — 微信开放平台，元事件/属性/漏斗配置
- **AARRR 海盗指标** — Dave McClure，漏斗分阶段设计参考
- **Data-Centric AI** — 在线监控 + 离线评估结合的数据飞轮
- **Google Analytics 事件模型** — 事件+属性结构的跨平台通用性参考
