---
name: xx-brand
description: 设计系统与品牌视觉一致性技能。当用户需要配置设计Token、全局样式、按钮/卡片/Hero Banner等组件样式、建立品牌资产三件套（BRAND_GUIDELINES.md + brand.js + assets/）、或需要统一多个页面的品牌视觉风格时使用。覆盖从Token配置到Canvas渲染场景的完整设计体系。AI 按本规范生成设计 Token 与组件样式，用户按一致性自查清单验收。
---

# 设计系统与品牌一致性

> 设计不是装饰，是体验的骨架。Token 不统一，再多页面也只是各自漂亮的碎片。

## 这个技能做什么

帮你在开发初期建立一套**可复用的设计系统**，解决两个问题：
1. **Apple 风格 UI 结构** — Token、组件样式、页面布局，开箱即用
2. **品牌一致性** — 品牌资产三件套，确保多页面、多场景视觉统一

不建立设计系统的后果：每个页面颜色不一样、间距靠感觉、按钮各写各的、Canvas 渲染时品牌色对不上。

**读者说明：** 本 skill 主要给 AI 执行规范（生成 Token、组件样式、品牌资产），次要给人判断标准（一致性自查清单）。用户不需要自己写 CSS，只需要会描述风格倾向（如"我要国风感"）、能截图说"这个按钮颜色和首页不一样"。

贯穿案例：**小象取色**，品牌色用中式色（故宫红为主色），方法论结构沿用 Apple HIG。

---

## 一、Design Token 配置

### 1.1 THEME 对象

**AI 参考实现（小象取色主题，config/theme.js）：**

```js
const THEME = {
  brand: {
    primary: '#9B2335',        // 故宫红，主按钮/链接/强调
    primaryLight: '#C44569',
    primaryDark: '#6B1018',
    secondary: '#4A90A4',      // 天青，辅色
    accent: '#C9A961'          // 鎏金，点缀
  },

  bg: {
    page: '#F7F3EE',           // 米白底（国风感）
    card: '#FFFFFF',
    input: '#F2EDE5'
  },

  text: {
    primary: '#2B1810',        // 墨色
    secondary: '#8B7355',      // 赭石
    light: '#BFAE99'           // 浅赭
  },

  border: '#D9C9B6',
  divider: '#E8DDC9',

  semantic: {
    danger: '#C0392B',
    warning: '#D68910',
    success: '#5D6D3D',        // 松柏绿
    info: '#4A90A4'
  },

  radius: '24rpx',
  radiusBtn: '28rpx',
  radiusInput: '16rpx',
  radiusFull: '999rpx',

  shadow: '0 8rpx 32rpx rgba(60, 30, 10, 0.08)',
  shadowBtn: '0 4rpx 16rpx rgba(155, 35, 53, 0.3)',
  shadowCard: '0 4rpx 24rpx rgba(60, 30, 10, 0.06)',

  space: {
    xs: '8rpx',
    sm: '16rpx',
    md: '24rpx',
    lg: '32rpx',
    xl: '48rpx',
    xxl: '64rpx'
  },

  fontSize: {
    page: '52rpx',
    card: '36rpx',
    section: '32rpx',
    body: '28rpx',
    desc: '26rpx',
    caption: '24rpx',
    tag: '22rpx'
  }
}

module.exports = THEME
```

**AI 执行约束：** 品牌色必须有 primary/primaryLight/primaryDark 三档（用于按压/禁用态）；semantic 必须有 danger/warning/success/info 四态。

### 1.2 字体大小规范表

| 用途 | Token | 大小 | 字重 | 使用场景 |
|------|-------|------|------|---------|
| 页面标题 | `fontSize.page` | 52rpx | 700 | 页面顶部大标题 |
| 卡片标题 | `fontSize.card` | 36rpx | 600 | 卡片/弹窗标题 |
| 区域标题 | `fontSize.section` | 32rpx | 600 | 内容区块标题 |
| 正文 | `fontSize.body` | 28rpx | 400 | 主要文本内容 |
| 描述 | `fontSize.desc` | 26rpx | 400 | 次要说明文字 |
| 辅助 | `fontSize.caption` | 24rpx | 400 | 时间、标签说明 |
| 标签 | `fontSize.tag` | 22rpx | 500 | 状态标签、徽标 |

**判断标准（给人）：** 页面上出现的字号种类不超过这 7 种。出现第 8 种 = 层级没理清。

---

## 二、全局页面样式

### 2.1 app.wxss

**AI 参考实现：**

```css
page {
  background: #F7F3EE;
  color: #2B1810;
  font-family: -apple-system, 'PingFang SC', 'Songti SC', serif;
  font-size: 28rpx;
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}

view, text {
  box-sizing: border-box;
}

.page-container {
  padding: 32rpx;
  min-height: 100vh;
}

.page-title {
  font-size: 52rpx;
  font-weight: 700;
  color: #2B1810;
  margin-bottom: 32rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #2B1810;
  margin: 32rpx 0 16rpx;
}

.card-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #2B1810;
}

.text-primary { color: #9B2335; }
.text-danger  { color: #C0392B; }
.text-success { color: #5D6D3D; }
.text-warning { color: #D68910; }
.text-secondary { color: #8B7355; }

.text-page    { font-size: 52rpx; font-weight: 700; }
.text-card    { font-size: 36rpx; font-weight: 600; }
.text-section { font-size: 32rpx; font-weight: 600; }
.text-body    { font-size: 28rpx; }
.text-desc    { font-size: 26rpx; color: #8B7355; }
.text-caption { font-size: 24rpx; color: #8B7355; }
.text-tag     { font-size: 22rpx; }

.mb-xs { margin-bottom: 8rpx; }
.mb-sm { margin-bottom: 16rpx; }
.mb-md { margin-bottom: 24rpx; }
.mb-lg { margin-bottom: 32rpx; }
.mb-xl { margin-bottom: 48rpx; }

.safe-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
```

**AI 执行约束：** app.wxss 里定义的 class 必须与 theme.js 的 Token 一一对应，不能出现两套色值。

---

## 三、按钮组件

### 3.1 主按钮（渐变 + 阴影）

**AI 参考实现：**

```css
.btn-primary {
  width: 100%;
  height: 96rpx;
  background: linear-gradient(135deg, #9B2335 0%, #C44569 100%);
  border-radius: 28rpx;
  color: #FFFFFF;
  font-size: 32rpx;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4rpx 16rpx rgba(155, 35, 53, 0.3);
  border: none;
  transition: all 0.2s ease;
}

.btn-primary::after {
  border: none;
}

.btn-primary:active {
  transform: scale(0.98);
  opacity: 0.9;
}

.btn-primary[disabled] {
  background: #D9C9B6;
  box-shadow: none;
  color: #FFFFFF;
}
```

```html
<button class="btn-primary" bindtap="onPick">开始取色</button>
<button class="btn-primary" disabled="{{!canSave}}">保存海报</button>
```

### 3.2 次级按钮（边框）

**AI 参考实现：**

```css
.btn-secondary {
  width: 100%;
  height: 96rpx;
  background: #FFFFFF;
  border: 2rpx solid #9B2335;
  border-radius: 28rpx;
  color: #9B2335;
  font-size: 32rpx;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.btn-secondary::after {
  border: none;
}

.btn-secondary:active {
  background: #FBF1ED;
}
```

### 3.3 轮廓按钮

**AI 参考实现：**

```css
.btn-outline {
  height: 72rpx;
  padding: 0 32rpx;
  background: transparent;
  border: 2rpx solid #D9C9B6;
  border-radius: 999rpx;
  color: #2B1810;
  font-size: 28rpx;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.btn-outline::after {
  border: none;
}

.btn-outline:active {
  background: #F7F3EE;
  border-color: #8B7355;
}
```

**AI 执行约束：** 所有按钮必须有 `:active` 反馈（缩放或透明度变化）；`::after { border: none }` 必加（去掉小程序按钮默认边框）。

---

## 四、卡片组件

### 4.1 通用卡片

**AI 参考实现：**

```css
.card {
  background: #FFFFFF;
  border-radius: 24rpx;
  padding: 32rpx;
  box-shadow: 0 4rpx 24rpx rgba(60, 30, 10, 0.06);
  margin-bottom: 24rpx;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 24rpx;
}

.card-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #2B1810;
}

.card-body {
  font-size: 28rpx;
  color: #2B1810;
  line-height: 1.6;
}

.card-footer {
  margin-top: 24rpx;
  padding-top: 24rpx;
  border-top: 2rpx solid #E8DDC9;
  display: flex;
  justify-content: flex-end;
  gap: 16rpx;
}
```

```html
<view class="card">
  <view class="card-header">
    <text class="card-title">故宫红</text>
    <text class="text-caption">#9B2335</text>
  </view>
  <view class="card-body">
    <text>取自紫禁城宫墙，沉稳而不失张力。</text>
  </view>
  <view class="card-footer">
    <button class="btn-outline">复制色值</button>
    <button class="btn-primary" style="width:auto;padding:0 48rpx;">生成海报</button>
  </view>
</view>
```

### 4.2 上传卡片

**AI 参考实现：**

```css
.upload-card {
  background: #FFFFFF;
  border-radius: 24rpx;
  border: 4rpx dashed #D9C9B6;
  padding: 48rpx 32rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.upload-card:active {
  border-color: #9B2335;
  background: #FBF1ED;
}

.upload-icon {
  width: 80rpx;
  height: 80rpx;
  margin-bottom: 16rpx;
}

.upload-text {
  font-size: 28rpx;
  color: #2B1810;
  font-weight: 500;
}

.upload-hint {
  font-size: 24rpx;
  color: #8B7355;
  margin-top: 8rpx;
}
```

```html
<view class="upload-card" bindtap="onChooseFile">
  <image class="upload-icon" src="/assets/upload-icon.png" mode="aspectFit" />
  <text class="upload-text">从相册选图取色</text>
  <text class="upload-hint">支持 JPG/PNG，最大 10MB</text>
</view>
```

---

## 五、Hero Banner

### 5.1 渐变背景 + 装饰圆圈 + 浮动动画

**AI 参考实现：**

```css
.hero-banner {
  position: relative;
  overflow: hidden;
  border-radius: 32rpx;
  padding: 48rpx 40rpx;
  background: linear-gradient(135deg, #9B2335 0%, #6B1018 100%);
  margin-bottom: 32rpx;
}

.hero-circle {
  position: absolute;
  border-radius: 50%;
  background: rgba(201, 169, 97, 0.15);
}

.hero-circle-1 {
  width: 300rpx;
  height: 300rpx;
  top: -100rpx;
  right: -80rpx;
}

.hero-circle-2 {
  width: 200rpx;
  height: 200rpx;
  bottom: -60rpx;
  left: -40rpx;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-20rpx); }
}

.hero-circle-1 {
  animation: float 4s ease-in-out infinite;
}

.hero-circle-2 {
  animation: float 5s ease-in-out infinite reverse;
}

.hero-content {
  position: relative;
  z-index: 1;
}

.hero-title {
  font-size: 48rpx;
  font-weight: 700;
  color: #FFFFFF;
  margin-bottom: 12rpx;
}

.hero-subtitle {
  font-size: 28rpx;
  color: rgba(255, 255, 255, 0.85);
  line-height: 1.5;
}
```

```html
<view class="hero-banner">
  <view class="hero-circle hero-circle-1"></view>
  <view class="hero-circle hero-circle-2"></view>
  <view class="hero-content">
    <text class="hero-title">小象取色</text>
    <text class="hero-subtitle">对准万物，一取即得中式色名</text>
  </view>
</view>
```

---

## 六、进度指示器与状态指示器

### 6.1 进度指示器

**AI 参考实现：**

```css
.progress-bar {
  width: 100%;
  height: 12rpx;
  background: #E8DDC9;
  border-radius: 999rpx;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #9B2335, #C44569);
  border-radius: 999rpx;
  transition: width 0.3s ease;
}
```

```html
<view class="progress-bar">
  <view class="progress-fill" style="width: {{progress}}%;"></view>
</view>
```

### 6.2 状态指示器

**AI 参考实现：**

```css
.status-indicator {
  display: inline-flex;
  align-items: center;
  gap: 8rpx;
  font-size: 24rpx;
  padding: 8rpx 16rpx;
  border-radius: 999rpx;
}

.status-dot {
  width: 12rpx;
  height: 12rpx;
  border-radius: 50%;
}

.status-success { background: rgba(93, 109, 61, 0.1); color: #5D6D3D; }
.status-success .status-dot { background: #5D6D3D; }

.status-warning { background: rgba(214, 137, 16, 0.1); color: #D68910; }
.status-warning .status-dot { background: #D68910; }

.status-danger { background: rgba(192, 57, 43, 0.1); color: #C0392B; }
.status-danger .status-dot { background: #C0392B; }

.status-info { background: rgba(155, 35, 53, 0.1); color: #9B2335; }
.status-info .status-dot { background: #9B2335; }
```

```html
<view class="status-indicator status-success">
  <view class="status-dot"></view>
  <text>已命名</text>
</view>
```

---

## 七、底部悬浮操作栏（safe-area 适配）

**AI 参考实现：**

```css
.bottom-bar {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  padding: 16rpx 32rpx;
  padding-bottom: calc(16rpx + constant(safe-area-inset-bottom));
  padding-bottom: calc(16rpx + env(safe-area-inset-bottom));
  box-shadow: 0 -4rpx 24rpx rgba(60, 30, 10, 0.06);
  display: flex;
  gap: 16rpx;
  z-index: 100;
}

.bottom-bar .btn-primary {
  flex: 1;
}

.has-bottom-bar {
  padding-bottom: 200rpx;
}
```

```html
<view class="page-container has-bottom-bar">
  <view class="page-title">配色海报</view>
</view>

<view class="bottom-bar">
  <button class="btn-secondary" style="width:auto;padding:0 48rpx;">取消</button>
  <button class="btn-primary" bindtap="onConfirm">保存到相册</button>
</view>
```

**AI 执行约束：** 有底部操作栏的页面，内容区必须加 `has-bottom-bar` 类，否则内容被遮挡。

---

## 八、空状态与错误卡片

### 8.1 空状态

**AI 参考实现：**

```css
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 120rpx 64rpx;
}

.empty-icon {
  width: 160rpx;
  height: 160rpx;
  margin-bottom: 32rpx;
  opacity: 0.5;
}

.empty-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #2B1810;
  margin-bottom: 12rpx;
}

.empty-desc {
  font-size: 26rpx;
  color: #8B7355;
  text-align: center;
}
```

```html
<view class="empty-state">
  <image class="empty-icon" src="/assets/empty.png" mode="aspectFit" />
  <text class="empty-title">还没有取色记录</text>
  <text class="empty-desc">对准任意颜色，开始你的第一次取色</text>
</view>
```

### 8.2 错误卡片

**AI 参考实现：**

```css
.error-card {
  background: rgba(192, 57, 43, 0.05);
  border: 2rpx solid rgba(192, 57, 43, 0.2);
  border-radius: 24rpx;
  padding: 32rpx;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.error-icon {
  width: 80rpx;
  height: 80rpx;
  margin-bottom: 16rpx;
}

.error-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #C0392B;
  margin-bottom: 8rpx;
}

.error-desc {
  font-size: 26rpx;
  color: #8B7355;
  text-align: center;
  margin-bottom: 24rpx;
}
```

```html
<view class="error-card">
  <image class="error-icon" src="/assets/error.png" mode="aspectFit" />
  <text class="error-title">命名失败</text>
  <text class="error-desc">AI 服务暂时不可用，已为你显示色值</text>
  <button class="btn-outline" bindtap="onRetry">重新命名</button>
</view>
```

---

## 九、品牌资产三件套

**每个小程序项目必须包含品牌资产三件套：**

```
miniprogram/
├── BRAND_GUIDELINES.md    # 品牌规范文档
├── config/
│   └── brand.js           # 品牌Token（程序可读）
└── assets/
    ├── logo-48.png        # 48x48 导航栏图标
    ├── logo-80.png        # 80x80 分享图标
    ├── logo-144.png       # 144x144 小图标
    ├── logo-240.png       # 240x240 大图标
    └── logo-1024.png      # 1024x1024 高清素材
```

### 9.1 BRAND_GUIDELINES.md

**AI 参考实现（小象取色）：**

```markdown
# 小象取色 品牌规范

## 品牌色
- 主色：#9B2335（故宫红）
- 辅助色：#4A90A4（天青）
- 点缀色：#C9A961（鎏金）

## 字体
- 标题：PingFang SC Semibold / Songti SC
- 正文：PingFang SC Regular

## LOGO 使用规范
- 最小尺寸：48x48px
- 安全边距：LOGO 四周留不小于高度 1/4 的空白
- 背景要求：浅色背景用深色 LOGO，深色背景用白色 LOGO

## 不可做
- 不要拉伸 LOGO
- 不要改变品牌色色值
- 不要在 LOGO 上叠加其他元素
```

### 9.2 brand.js 结构

**五层分类：brand / ui / neutral / semantic / alpha**

**AI 参考实现：**

```js
const brand = {
  brand: {
    primary: '#9B2335',
    primaryHover: '#C44569',
    primaryActive: '#6B1018',
    secondary: '#4A90A4',
    accent: '#C9A961'
  },

  ui: {
    pageBg: '#F7F3EE',
    cardBg: '#FFFFFF',
    inputBg: '#F2EDE5',
    mask: 'rgba(43, 24, 16, 0.5)',
    overlay: 'rgba(255, 255, 255, 0.95)'
  },

  neutral: {
    textPrimary: '#2B1810',
    textSecondary: '#8B7355',
    textTertiary: '#BFAE99',
    textInverse: '#FFFFFF',
    border: '#D9C9B6',
    divider: '#E8DDC9',
    bgPrimary: '#FFFFFF',
    bgSecondary: '#F7F3EE',
    bgTertiary: '#E8DDC9'
  },

  semantic: {
    success: '#5D6D3D',
    successBg: 'rgba(93, 109, 61, 0.1)',
    warning: '#D68910',
    warningBg: 'rgba(214, 137, 16, 0.1)',
    danger: '#C0392B',
    dangerBg: 'rgba(192, 57, 43, 0.1)',
    info: '#4A90A4',
    infoBg: 'rgba(74, 144, 164, 0.1)'
  },

  alpha: {
    primary10: 'rgba(155, 35, 53, 0.1)',
    primary20: 'rgba(155, 35, 53, 0.2)',
    primary30: 'rgba(155, 35, 53, 0.3)',
    black05: 'rgba(43, 24, 16, 0.05)',
    black10: 'rgba(43, 24, 16, 0.1)',
    black20: 'rgba(43, 24, 16, 0.2)',
    white10: 'rgba(255, 255, 255, 0.1)',
    white20: 'rgba(255, 255, 255, 0.2)'
  }
}

module.exports = brand
```

### 9.3 多尺寸 LOGO

| 尺寸 | 用途 | 文件 |
|------|------|------|
| 48x48 | 导航栏图标、TabBar 图标 | `logo-48.png` |
| 80x80 | 分享封面小图、头像 | `logo-80.png` |
| 144x144 | 小程序图标、列表图标 | `logo-144.png` |
| 240x240 | 大列表图标、弹窗图标 | `logo-240.png` |
| 1024x1024 | 高清素材、分享封面 | `logo-1024.png` |

> 所有 LOGO 使用 PNG 格式，背景透明。同时建议导出一份 SVG 源文件备份。

---

## 十、品牌色统一性重构5步法

当发现多个页面品牌色不一致时，按此5步重构：

```
第1步：审计现状
  └─ 搜索所有硬编码的色值（#开头、rgb），列出每个页面用到的颜色

第2步：映射到 Token
  └─ 将每个硬编码色值映射到 brand.js 中对应的 Token

第3步：替换硬编码
  └─ 逐文件将 #9B2335 替换为引用 brand.brand.primary
  └─ WXSS 中无法直接 require JS，用 CSS 变量或统一用 app.wxss 定义的 class

第4步：验证一致性
  └─ 逐页检查，确保同语义的元素使用同一 Token

第5步：建立约束
  └─ 在 BRAND_GUIDELINES.md 中记录最终 Token
  └─ 后续新页面只允许引用 Token，不允许硬编码色值
```

### CSS 变量方案（推荐用于 WXSS）

**AI 参考实现：**

```css
page {
  --color-primary: #9B2335;
  --color-primary-light: #C44569;
  --color-bg: #F7F3EE;
  --color-card: #FFFFFF;
  --color-text: #2B1810;
  --color-text-sec: #8B7355;
  --color-border: #D9C9B6;
  --color-danger: #C0392B;
  --color-success: #5D6D3D;
  --color-warning: #D68910;
  --radius: 24rpx;
  --radius-btn: 28rpx;
}

.card {
  background: var(--color-card);
  border-radius: var(--radius);
  color: var(--color-text);
}

.btn-primary {
  background: var(--color-primary);
  border-radius: var(--radius-btn);
}
```

**AI 执行约束：** 重构必须按 5 步顺序，不能跳过"审计现状"直接替换，否则会漏掉隐蔽的硬编码。

---

## 十一、Canvas 渲染场景的 Token 应用

Canvas（如海报生成、图表绘制）中无法直接使用 CSS 变量，需要从 `brand.js` 派生短别名。

### 11.1 从 brand.js 派生短别名 C

**AI 参考实现（小象取色配色海报专用）：**

```js
const brand = require('./brand')

const C = {
  p: brand.brand.primary,
  pl: brand.brand.primaryHover,
  pd: brand.brand.primaryActive,
  s: brand.brand.secondary,
  a: brand.brand.accent,

  bg: brand.ui.pageBg,
  card: brand.ui.cardBg,
  mask: brand.ui.mask,

  t: brand.neutral.textPrimary,
  ts: brand.neutral.textSecondary,
  tt: brand.neutral.textTertiary,
  ti: brand.neutral.textInverse,

  bd: brand.neutral.border,
  dv: brand.neutral.divider,

  ok: brand.semantic.success,
  warn: brand.semantic.warning,
  err: brand.semantic.danger,
  info: brand.semantic.info,
  okBg: brand.semantic.successBg,
  warnBg: brand.semantic.warningBg,
  errBg: brand.semantic.dangerBg,

  p10: brand.alpha.primary10,
  b5: brand.alpha.black05,
  b10: brand.alpha.black10,
  w10: brand.alpha.white10
}

module.exports = C
```

### 11.2 Canvas 中使用

**AI 参考实现（小象取色配色海报绘制）：**

```js
const C = require('../config/canvasTheme')

function drawPoster(ctx, data) {
  ctx.setFillStyle(C.card)
  ctx.fillRect(0, 0, 750, 1334)

  ctx.setFillStyle(C.t)
  ctx.setFontSize(52)
  ctx.fillText(data.title, 40, 100)

  ctx.setFillStyle(C.p)
  ctx.fillRect(40, 1200, 670, 96)

  ctx.setFillStyle(C.ti)
  ctx.setFontSize(32)
  ctx.fillText('小象取色', 280, 1256)

  ctx.draw()
}
```

> **短别名优势**：Canvas 代码中色值引用频繁，短别名可读性更好，且修改时只需改 `brand.js` 一处，所有 Canvas 自动同步。

**AI 执行约束：** `canvasTheme.js` 必须从 `brand.js` 派生，禁止在 Canvas 里重复定义色值。

---

## 十二、品牌一致性自查清单

发布前逐项检查：

### Token 使用
- [ ] 所有颜色来自 `brand.js` 或 CSS 变量，无硬编码色值
- [ ] 所有圆角使用 Token（`--radius` / `--radius-btn`）
- [ ] 所有字体大小来自规范表（52/36/32/28/26/24/22rpx）

### 品牌资产
- [ ] `BRAND_GUIDELINES.md` 已创建并记录最终 Token
- [ ] `brand.js` 五层分类完整（brand/ui/neutral/semantic/alpha）
- [ ] `assets/` 目录包含5个尺寸的 LOGO（48/80/144/240/1024）
- [ ] 所有 LOGO 为透明背景 PNG

### Canvas 场景
- [ ] Canvas 渲染使用 `C` 短别名，不直接写色值
- [ ] `canvasTheme.js` 从 `brand.js` 派生，非重复定义

### 一致性
- [ ] 所有页面的主按钮样式一致
- [ ] 所有卡片的圆角、阴影一致
- [ ] 所有页面背景色一致（#F7F3EE）
- [ ] 文字颜色层级一致（主 #2B1810 / 次 #8B7355 / 占位 #BFAE99）
- [ ] 底部操作栏 safe-area 适配一致

**AI 执行约束：** 任一项未达标不能发布；硬编码色值必须全部替换为 Token 引用。

---

## 使用方式

当你需要建立设计系统或统一品牌视觉时，告诉我你的需求。

**对话示例：**
- "小象取色要国风感，帮我配设计 Token" → 我生成故宫红主题的 theme.js 和 app.wxss
- "多个页面颜色不统一，怎么重构" → 我用品牌色统一性5步法帮你梳理
- "Canvas 画配色海报时怎么用品牌色" → 我帮你派生 canvasTheme.js 短别名
- "需要准备品牌资产" → 我帮你生成 BRAND_GUIDELINES.md 和 brand.js

---

## 与其他技能的关系

- **前置：** xx-setup 项目骨架（config/theme.js 的位置）
- **并行：** xx-blocks 组件库直接引用本 skill 的 Token 和样式 class
- **下游：** xx-ai 的 Canvas 海报生成用本 skill 的 canvasTheme.js 短别名
- **回溯：** xx-track 上线后监控"品牌色不一致"类反馈，反向校准 Token

## 方法论来源

- **Apple Human Interface Guidelines** — 色彩、排版、圆角规范
- **Design Tokens 规范** — W3C Design Tokens Format Module
- **Atomic Design** — Brad Frost，组件分层方法论
- **品牌系统设计** — 品牌资产三层结构（规范/Token/素材）
