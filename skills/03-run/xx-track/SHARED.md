# SHARED · 数据埋点（平台无关）

> 本文件描述跨平台通用的数据埋点原则。平台具体实现在 wechat/SKILL.md 和 web/SKILL.md。
> 主要读者是 AI（执行埋点规范），次要读者是人（判断埋点是否覆盖核心旅程）。

## 通用原则

### 1. 事件命名规范
- 动词_名词 结构（如 view_home、click_button）
- 全小写下划线
- 业务前缀分组（如 order_create、order_pay）

**AI 执行约束：** 事件名一旦发布不可修改，发布前必须确认命名；命名只用英文小写+下划线，不用拼音、不用驼峰。

**判断标准：** 看事件名能否一眼看出"在哪个页面做了什么"，看不懂就是不合格。

### 2. 埋点分层
- 页面级：页面浏览、停留
- 交互级：点击、滑动
- 业务级：下单、支付、注册

**AI 执行约束：** 每个核心业务动作都要有对应的业务级埋点，不能只埋页面级。

### 3. 漏斗分析
- 定义关键转化路径
- 每步转化率监控
- 流失节点定位

**判断标准：** 漏斗步数 ≤ 5 步；每步流失率有对应行动建议，而不是只报数字。

### 4. 数据驱动决策
- 指标分级（北极星 → 二级 → 三级）
- A/B 测试机制
- 数据看板每日查看

### 5. 含 AI 能力的产品的埋点增量
> 含 AI 能力的产品除了业务埋点，还要监控"AI 输出质量"——业务指标有滞后性，AI 变差时用户会默默流失，等业务指标降时已经晚了。

- 记录 AI 调用结果（成功/失败/重试）
- 采样 AI 输出做人工评分
- 监控用户负反馈率（点"不满意"比例）

**AI 执行约束：** AI 调用必须埋 `ai_call_success` / `ai_call_fail` / `ai_retry` 三类事件，缺一不可；失败事件必须带 error_code。

## 示例：小象取色的核心埋点

| 层级 | 事件 | 说明 |
|------|------|------|
| 页面级 | home_view / picker_view / history_view | 首页/取色页/历史页浏览 |
| 交互级 | picker_start_click / picker_save_click | 开始取色/保存颜色 |
| 业务级 | color_pick_success / ai_name_success / poster_generate_success | 取色成功/AI命名成功/海报生成成功 |
| AI 级 | ai_call_success / ai_call_fail / ai_retry | AI 调用结果与重试 |

核心漏斗：`picker_view → picker_start_click → color_pick_success → ai_name_success → picker_save_click`

## 平台对应

| 平台 | 实现文件 | 核心技术 |
|------|---------|---------|
| 微信小程序 | wechat/SKILL.md | WeAnalysis、wx.reportEvent、元事件配置 |
| 出海网站 | web/SKILL.md（待补充）| GA4、PostHog、Mixpanel、Segment |
