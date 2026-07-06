---
name: xx-blocks
description: 可复用组件模式库技能。当用户需要编写小程序页面结构、上传卡片、进度步骤、AI流式输出、提示条目、免责声明、错误提示、功能亮点网格、列表项、数据统计卡片、标签页切换、加载骨架屏等组件，或需要通用工具函数时使用。提供可直接复用的 WXML/JS/WXSS 模板。AI 按本规范生成组件，用户按检查清单验收。
---

# 可复用组件模式库

> 不要重复造轮子。每个组件模板来自10+上线产品，AI 参考实现，开箱即用。

## 这个技能做什么

提供一套**开箱即用的组件模式模板**，覆盖小程序开发中 90% 的常见 UI 场景。每个模板包含 WXML + JS + WXSS，AI 直接参考实现。

**读者说明：** 本 skill 主要给 AI 执行规范（组件模板代码），次要给人判断标准（组件使用检查清单）。用户不需要自己写组件，只需要会描述"我要一个取色历史列表"、能截图说"这个列表间距不对"。

贯穿案例：**小象取色**，组件示例用取色卡/颜色历史/海报场景。色值沿用 xx-brand 的故宫红主题。

---

## 一、标准页面结构模板

### 1.1 WXML

**AI 参考实现（取色页）：**

```html
<view class="page-container">
  <view class="page-title">取色</view>

  <view class="content-section">
    <view class="section-title">对准颜色取样</view>
  </view>
</view>
```

### 1.2 JS（含 theme 加载）

**AI 参考实现：**

```js
const theme = require('../../config/theme')

Page({
  data: {
    theme: theme,
    loading: true,
    error: false,
    list: []
  },

  onLoad(options) {
    this.setData({ pageId: options.id || '' })
    this.loadData()
  },

  onShow() {
  },

  onShareAppMessage() {
    return {
      title: '小象取色 - 一键取中式色名',
      path: '/pages/picker/picker'
    }
  },

  onShareTimeline() {
    return {
      title: '小象取色 - 一键取中式色名'
    }
  },

  async loadData() {
    this.setData({ loading: true, error: false })
    try {
      const res = await wx.cloud.callFunction({
        name: 'aiNameColor',
        data: { action: 'getList' }
      })
      if (res.result.code === 0) {
        this.setData({ list: res.result.data })
      } else {
        throw new Error(res.result.message)
      }
    } catch (err) {
      console.error('加载失败:', err)
      this.setData({ error: true })
    } finally {
      this.setData({ loading: false })
    }
  },

  onRetry() {
    this.loadData()
  }
})
```

### 1.3 WXSS

**AI 参考实现：**

```css
.page-container {
  padding: 32rpx;
  min-height: 100vh;
  box-sizing: border-box;
}

.page-title {
  font-size: 52rpx;
  font-weight: 700;
  color: #2B1810;
  margin-bottom: 32rpx;
}

.content-section {
  margin-bottom: 32rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #2B1810;
  margin-bottom: 16rpx;
}
```

**AI 执行约束：** 每个页面必须定义 `onShareAppMessage` 和 `onShareTimeline`；`onLoad` 必须接收 options 参数。

---

## 二、带悬浮操作栏的页面模板

**AI 参考实现（海报生成页）：**

```html
<view class="page-container has-bottom-bar">
  <view class="page-title">配色海报</view>

  <view class="content-section">
    <view class="card">
      <text class="card-title">故宫红</text>
      <text class="text-body">#9B2335</text>
    </view>
  </view>
</view>

<view class="bottom-bar">
  <button class="btn-secondary bar-btn" bindtap="onCancel">取消</button>
  <button class="btn-primary bar-btn" bindtap="onConfirm" disabled="{{!canConfirm}}">保存到相册</button>
</view>
```

```js
Page({
  data: {
    canConfirm: false
  },

  onCancel() {
    wx.navigateBack()
  },

  onConfirm() {
    if (!this.data.canConfirm) return
    wx.showLoading({ title: '保存中...' })
    setTimeout(() => {
      wx.hideLoading()
      wx.showToast({ title: '已保存到相册', icon: 'success' })
      setTimeout(() => wx.navigateBack(), 1500)
    }, 1000)
  }
})
```

```css
.has-bottom-bar {
  padding-bottom: 200rpx;
}

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

.bar-btn {
  flex: 1;
}
```

---

## 三、上传卡片组件

### 3.1 WXML

**AI 参考实现（从相册选图取色）：**

```html
<view class="upload-card {{disabled ? 'disabled' : ''}}" bindtap="onChooseFile">
  <block wx:if="{{file}}">
    <view class="file-preview">
      <image wx:if="{{isImage}}" class="file-thumb" src="{{file.path}}" mode="aspectFill" />
      <view wx:else class="file-icon-wrap">
        <text class="file-icon">🎨</text>
      </view>
      <view class="file-info">
        <text class="file-name">{{file.name}}</text>
        <text class="file-size">{{fileSizeText}}</text>
      </view>
      <view class="file-remove" catchtap="onRemove">✕</view>
    </view>
  </block>
  <block wx:else>
    <view class="upload-placeholder">
      <view class="upload-icon-circle">
        <text class="upload-plus">+</text>
      </view>
      <text class="upload-text">从相册选图取色</text>
      <text class="upload-hint">{{acceptText}}，最大 {{maxSizeText}}</text>
    </view>
  </block>
</view>
```

### 3.2 JS（含文件选择、大小校验、格式校验）

**AI 参考实现：**

```js
Component({
  properties: {
    accept: {
      type: Array,
      value: ['jpg', 'jpeg', 'png']
    },
    maxSize: {
      type: Number,
      value: 10 * 1024 * 1024
    },
    disabled: {
      type: Boolean,
      value: false
    }
  },

  data: {
    file: null,
    isImage: false,
    fileSizeText: '',
    maxSizeText: '',
    acceptText: ''
  },

  lifetimes: {
    attached() {
      this.setData({
        maxSizeText: this.formatSize(this.data.maxSize),
        acceptText: this.data.accept.join('、').toUpperCase()
      })
    }
  },

  methods: {
    onChooseFile() {
      if (this.data.disabled) return
      this.chooseImage()
    },

    chooseImage() {
      wx.chooseMedia({
        count: 1,
        mediaType: ['image'],
        sourceType: ['album', 'camera'],
        success: (res) => {
          const file = res.tempFiles[0]
          this.validateAndSet(file, 'image')
        }
      })
    },

    validateAndSet(file, type) {
      const ext = file.name ? file.name.split('.').pop().toLowerCase() : 'jpg'
      if (!this.data.accept.includes(ext)) {
        wx.showToast({
          title: `不支持的格式，仅支持 ${this.data.acceptText}`,
          icon: 'none'
        })
        return
      }

      if (file.size > this.data.maxSize) {
        wx.showToast({
          title: `文件不能超过 ${this.data.maxSizeText}`,
          icon: 'none'
        })
        return
      }

      const isImage = ['jpg', 'jpeg', 'png', 'gif', 'webp'].includes(ext)
      this.setData({
        file: file,
        isImage: isImage,
        fileSizeText: this.formatSize(file.size)
      })

      this.triggerEvent('change', { file })
    },

    onRemove() {
      this.setData({ file: null, isImage: false, fileSizeText: '' })
      this.triggerEvent('change', { file: null })
    },

    formatSize(bytes) {
      if (bytes < 1024) return bytes + 'B'
      if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + 'KB'
      return (bytes / 1024 / 1024).toFixed(1) + 'MB'
    }
  }
})
```

### 3.3 WXSS

**AI 参考实现：**

```css
.upload-card {
  background: #FFFFFF;
  border-radius: 24rpx;
  border: 4rpx dashed #D9C9B6;
  padding: 48rpx 32rpx;
  transition: all 0.2s ease;
}

.upload-card:active {
  border-color: #9B2335;
  background: #FBF1ED;
}

.upload-card.disabled {
  opacity: 0.5;
}

.upload-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.upload-icon-circle {
  width: 96rpx;
  height: 96rpx;
  border-radius: 50%;
  background: #FBF1ED;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 16rpx;
}

.upload-plus {
  font-size: 48rpx;
  color: #9B2335;
  font-weight: 300;
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

.file-preview {
  display: flex;
  align-items: center;
  gap: 16rpx;
}

.file-thumb {
  width: 96rpx;
  height: 96rpx;
  border-radius: 16rpx;
}

.file-info {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.file-name {
  font-size: 28rpx;
  color: #2B1810;
  word-break: break-all;
}

.file-size {
  font-size: 24rpx;
  color: #8B7355;
  margin-top: 4rpx;
}

.file-remove {
  width: 48rpx;
  height: 48rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #8B7355;
  font-size: 28rpx;
}
```

---

## 四、进度步骤组件

**AI 参考实现（取色→命名→海报）：**

```html
<view class="progress-steps">
  <block wx:for="{{steps}}" wx:key="index">
    <view class="step-item">
      <view wx:if="{{index > 0}}" class="step-line {{index <= current ? 'active' : ''}}"></view>
      <view class="step-dot {{index < current ? 'done' : ''}} {{index === current ? 'current' : ''}}">
        <text wx:if="{{index < current}}" class="step-check">✓</text>
        <text wx:elif="{{index === current}}" class="step-num">{{index + 1}}</text>
        <text wx:else class="step-num">{{index + 1}}</text>
      </view>
      <view class="step-content">
        <text class="step-title {{index <= current ? 'active' : ''}}">{{item.title}}</text>
        <text wx:if="{{item.desc}}" class="step-desc">{{item.desc}}</text>
      </view>
    </view>
  </block>
</view>
```

```js
Component({
  properties: {
    steps: {
      type: Array,
      value: []
    },
    current: {
      type: Number,
      value: 0
    }
  }
})
```

```css
.progress-steps {
  display: flex;
  flex-direction: column;
}

.step-item {
  display: flex;
  align-items: flex-start;
  position: relative;
  padding-bottom: 40rpx;
}

.step-item:last-child {
  padding-bottom: 0;
}

.step-line {
  position: absolute;
  left: 24rpx;
  top: -40rpx;
  width: 4rpx;
  height: 40rpx;
  background: #D9C9B6;
}

.step-line.active {
  background: #9B2335;
}

.step-dot {
  width: 48rpx;
  height: 48rpx;
  border-radius: 50%;
  background: #E8DDC9;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  margin-right: 24rpx;
  transition: all 0.3s ease;
}

.step-dot.done {
  background: #5D6D3D;
}

.step-dot.current {
  background: #9B2335;
  box-shadow: 0 0 0 8rpx rgba(155, 35, 53, 0.15);
}

.step-check {
  color: #FFFFFF;
  font-size: 24rpx;
}

.step-num {
  color: #8B7355;
  font-size: 24rpx;
  font-weight: 600;
}

.step-dot.current .step-num {
  color: #FFFFFF;
}

.step-content {
  flex: 1;
  padding-top: 4rpx;
}

.step-title {
  font-size: 28rpx;
  color: #8B7355;
  font-weight: 500;
}

.step-title.active {
  color: #2B1810;
}

.step-desc {
  display: block;
  font-size: 24rpx;
  color: #8B7355;
  margin-top: 4rpx;
}
```

**使用示例：**

```html
<progress-steps
  steps="{{[
    { title: '摄像头取色', desc: '对准颜色取样' },
    { title: 'AI 命名中', desc: '生成中式色名' },
    { title: '生成海报', desc: '配色方案与海报' }
  ]}}"
  current="{{1}}"
/>
```

---

## 五、AI 流式输出组件

### 5.1 WXML（streamText 显示 + 光标动画）

**AI 参考实现（AI 命名结果流式显示）：**

```html
<view class="ai-stream-container">
  <view class="ai-header">
    <view class="ai-avatar">
      <image src="/assets/logo-80.png" mode="aspectFit" class="ai-avatar-img" />
    </view>
    <text class="ai-name">小象取色</text>
    <view wx:if="{{thinking}}" class="ai-thinking">
      <view class="dot"></view>
      <view class="dot"></view>
      <view class="dot"></view>
    </view>
  </view>

  <view class="ai-content">
    <text class="ai-text">{{displayText}}</text>
    <text wx:if="{{streaming}}" class="ai-cursor">▋</text>
  </view>
</view>
```

### 5.2 JS

**AI 参考实现：**

```js
Component({
  properties: {
    fullText: {
      type: String,
      value: ''
    },
    speed: {
      type: Number,
      value: 30
    }
  },

  data: {
    displayText: '',
    streaming: false,
    thinking: false,
    _timer: null,
    _index: 0
  },

  observers: {
    'fullText': function(text) {
      if (!text) return
      this.startStream(text)
    }
  },

  lifetimes: {
    detached() {
      this.clearTimer()
    }
  },

  methods: {
    startStream(text) {
      this.clearTimer()
      this.setData({
        displayText: '',
        streaming: true,
        thinking: true,
        _index: 0
      })

      setTimeout(() => {
        this.setData({ thinking: false })
        this.streamNext(text)
      }, 500)
    },

    streamNext(text) {
      const index = this.data._index
      if (index >= text.length) {
        this.setData({ streaming: false })
        this.triggerEvent('done')
        return
      }

      const chunkSize = Math.min(
        Math.floor(Math.random() * 3) + 1,
        text.length - index
      )
      const newDisplay = text.substring(0, index + chunkSize)

      this.setData({
        displayText: newDisplay,
        _index: index + chunkSize
      })

      this.data._timer = setTimeout(() => {
        this.streamNext(text)
      }, this.data.speed)
    },

    clearTimer() {
      if (this.data._timer) {
        clearTimeout(this.data._timer)
        this.data._timer = null
      }
    },

    skip() {
      this.clearTimer()
      this.setData({
        displayText: this.data.fullText,
        _index: this.data.fullText.length,
        streaming: false,
        thinking: false
      })
      this.triggerEvent('done')
    }
  }
})
```

### 5.3 WXSS（光标动画）

**AI 参考实现：**

```css
.ai-stream-container {
  background: #FFFFFF;
  border-radius: 24rpx;
  padding: 32rpx;
}

.ai-header {
  display: flex;
  align-items: center;
  gap: 16rpx;
  margin-bottom: 24rpx;
}

.ai-avatar {
  width: 64rpx;
  height: 64rpx;
  border-radius: 50%;
  overflow: hidden;
  background: #FBF1ED;
}

.ai-avatar-img {
  width: 100%;
  height: 100%;
}

.ai-name {
  font-size: 28rpx;
  font-weight: 600;
  color: #2B1810;
}

.ai-thinking {
  display: flex;
  gap: 8rpx;
}

.ai-thinking .dot {
  width: 12rpx;
  height: 12rpx;
  border-radius: 50%;
  background: #8B7355;
  animation: dotBounce 1.4s infinite ease-in-out;
}

.ai-thinking .dot:nth-child(2) { animation-delay: 0.2s; }
.ai-thinking .dot:nth-child(3) { animation-delay: 0.4s; }

@keyframes dotBounce {
  0%, 80%, 100% { transform: scale(0.6); opacity: 0.4; }
  40% { transform: scale(1); opacity: 1; }
}

.ai-content {
  font-size: 28rpx;
  color: #2B1810;
  line-height: 1.8;
}

.ai-text {
  word-break: break-all;
}

.ai-cursor {
  color: #9B2335;
  font-weight: 300;
  animation: cursorBlink 1s infinite;
}

@keyframes cursorBlink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}
```

**AI 执行约束：** 组件 detached 时必须 clearTimer，否则页面销毁后定时器还在跑会报错。

---

## 六、提示条目组件

> 通用提示条目，可用于颜色命名置信度提示、配色冲突提示等。

### 6.1 WXML

**AI 参考实现：**

```html
<view class="risk-item risk-{{level}}">
  <view class="risk-header">
    <view class="risk-tag risk-tag-{{level}}">
      <view class="risk-dot"></view>
      <text>{{levelText}}</text>
    </view>
    <text class="risk-title">{{title}}</text>
  </view>

  <view class="risk-section">
    <text class="risk-label">说明</text>
    <view class="risk-clause">
      <text>{{clause}}</text>
    </view>
  </view>

  <view wx:if="{{suggestion}}" class="risk-section">
    <text class="risk-label">建议</text>
    <view class="risk-suggestion">
      <text class="suggestion-icon">💡</text>
      <text class="suggestion-text">{{suggestion}}</text>
    </view>
  </view>
</view>
```

### 6.2 JS

**AI 参考实现：**

```js
Component({
  properties: {
    level: {
      type: String,
      value: 'medium'
    },
    title: {
      type: String,
      value: ''
    },
    clause: {
      type: String,
      value: ''
    },
    suggestion: {
      type: String,
      value: ''
    }
  },

  data: {
    levelText: ''
  },

  observers: {
    'level': function(level) {
      const map = {
        high: '高置信',
        medium: '中置信',
        low: '低置信'
      }
      this.setData({ levelText: map[level] || '提示' })
    }
  }
})
```

### 6.3 WXSS

**AI 参考实现：**

```css
.risk-item {
  background: #FFFFFF;
  border-radius: 24rpx;
  padding: 32rpx;
  margin-bottom: 24rpx;
  border-left: 8rpx solid #D9C9B6;
}

.risk-high { border-left-color: #C0392B; }
.risk-medium { border-left-color: #D68910; }
.risk-low { border-left-color: #5D6D3D; }

.risk-header {
  display: flex;
  align-items: center;
  gap: 16rpx;
  margin-bottom: 24rpx;
}

.risk-tag {
  display: inline-flex;
  align-items: center;
  gap: 8rpx;
  padding: 6rpx 16rpx;
  border-radius: 999rpx;
  font-size: 22rpx;
  font-weight: 500;
}

.risk-tag-high { background: rgba(192, 57, 43, 0.1); color: #C0392B; }
.risk-tag-medium { background: rgba(214, 137, 16, 0.1); color: #D68910; }
.risk-tag-low { background: rgba(93, 109, 61, 0.1); color: #5D6D3D; }

.risk-dot {
  width: 12rpx;
  height: 12rpx;
  border-radius: 50%;
}

.risk-tag-high .risk-dot { background: #C0392B; }
.risk-tag-medium .risk-dot { background: #D68910; }
.risk-tag-low .risk-dot { background: #5D6D3D; }

.risk-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #2B1810;
  flex: 1;
}

.risk-section {
  margin-top: 20rpx;
}

.risk-label {
  font-size: 24rpx;
  color: #8B7355;
  font-weight: 500;
  display: block;
  margin-bottom: 8rpx;
}

.risk-clause {
  background: #F2EDE5;
  border-radius: 16rpx;
  padding: 20rpx;
  font-size: 26rpx;
  color: #2B1810;
  line-height: 1.6;
}

.risk-suggestion {
  display: flex;
  align-items: flex-start;
  gap: 12rpx;
  background: rgba(155, 35, 53, 0.05);
  border-radius: 16rpx;
  padding: 20rpx;
}

.suggestion-icon {
  font-size: 28rpx;
}

.suggestion-text {
  font-size: 26rpx;
  color: #2B1810;
  line-height: 1.6;
  flex: 1;
}
```

**使用示例（颜色命名置信度提示）：**

```html
<risk-item
  level="low"
  title="此色命名置信度较低"
  clause="取样色 #F5F5F5 饱和度过低，接近纯白，中式命名可能不唯一。"
  suggestion="建议在自然光下重新取样，或手动选择'月白'。"
/>
```

---

## 七、免责声明组件

**AI 参考实现（AI 命名结果免责）：**

```html
<view class="disclaimer">
  <view class="disclaimer-header">
    <text class="disclaimer-icon">⚠️</text>
    <text class="disclaimer-title">免责声明</text>
  </view>
  <view class="disclaimer-body">
    <text wx:for="{{items}}" wx:key="index" class="disclaimer-item">
      {{index + 1}}. {{item}}
    </text>
  </view>
  <view wx:if="{{canClose}}" class="disclaimer-close" bindtap="onClose">
    <text>我知道了</text>
  </view>
</view>
```

```js
Component({
  properties: {
    items: {
      type: Array,
      value: [
        'AI 生成的色名仅供参考，不构成专业色彩鉴定。',
        '命名可能存在误差，正式设计请以标准色卡为准。',
        '用户应自行判断并承担使用风险。'
      ]
    },
    canClose: {
      type: Boolean,
      value: false
    }
  },

  methods: {
    onClose() {
      this.triggerEvent('close')
    }
  }
})
```

```css
.disclaimer {
  background: rgba(214, 137, 16, 0.05);
  border: 2rpx solid rgba(214, 137, 16, 0.2);
  border-radius: 24rpx;
  padding: 32rpx;
  margin: 24rpx 0;
}

.disclaimer-header {
  display: flex;
  align-items: center;
  gap: 12rpx;
  margin-bottom: 16rpx;
}

.disclaimer-icon {
  font-size: 32rpx;
}

.disclaimer-title {
  font-size: 28rpx;
  font-weight: 600;
  color: #D68910;
}

.disclaimer-body {
  display: flex;
  flex-direction: column;
  gap: 12rpx;
}

.disclaimer-item {
  font-size: 24rpx;
  color: #8B7355;
  line-height: 1.6;
}

.disclaimer-close {
  text-align: center;
  margin-top: 24rpx;
  padding-top: 24rpx;
  border-top: 2rpx solid rgba(214, 137, 16, 0.15);
  font-size: 28rpx;
  color: #D68910;
  font-weight: 500;
}
```

---

## 八、错误提示组件（含重试按钮）

**AI 参考实现（命名失败）：**

```html
<view class="error-view">
  <view class="error-icon-wrap">
    <text class="error-emoji">😵</text>
  </view>
  <text class="error-title">{{title}}</text>
  <text class="error-desc">{{desc}}</text>
  <button wx:if="{{showRetry}}" class="btn-outline error-retry" bindtap="onRetry">
    {{retryText}}
  </button>
</view>
```

```js
Component({
  properties: {
    title: { type: String, value: '命名失败' },
    desc: { type: String, value: 'AI 服务暂时不可用，已为你显示色值' },
    showRetry: { type: Boolean, value: true },
    retryText: { type: String, value: '重新命名' }
  },

  methods: {
    onRetry() {
      this.triggerEvent('retry')
    }
  }
})
```

```css
.error-view {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 120rpx 64rpx;
}

.error-icon-wrap {
  margin-bottom: 32rpx;
}

.error-emoji {
  font-size: 96rpx;
}

.error-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #2B1810;
  margin-bottom: 12rpx;
}

.error-desc {
  font-size: 26rpx;
  color: #8B7355;
  text-align: center;
  margin-bottom: 32rpx;
}

.error-retry {
  width: auto;
  padding: 0 48rpx;
}
```

---

## 九、功能亮点网格

**AI 参考实现（小象取色功能）：**

```html
<view class="feature-grid">
  <view wx:for="{{features}}" wx:key="index" class="feature-card">
    <view class="feature-icon-wrap" style="background: {{item.bgColor || '#FBF1ED'}};">
      <text class="feature-icon">{{item.icon}}</text>
    </view>
    <text class="feature-title">{{item.title}}</text>
    <text class="feature-desc">{{item.desc}}</text>
  </view>
</view>
```

```js
data: {
  features: [
    { icon: '📷', title: '实时取色', desc: '摄像头一取即得', bgColor: 'rgba(155, 35, 53, 0.1)' },
    { icon: '🎨', title: '中式命名', desc: '故宫红/天青色', bgColor: 'rgba(74, 144, 164, 0.1)' },
    { icon: '🎯', title: 'AI 配色', desc: '一键生成方案', bgColor: 'rgba(201, 169, 97, 0.1)' },
    { icon: '🖼️', title: '配色海报', desc: '一键保存分享', bgColor: 'rgba(93, 109, 61, 0.1)' }
  ]
}
```

```css
.feature-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 24rpx;
}

.feature-card {
  width: calc(50% - 12rpx);
  background: #FFFFFF;
  border-radius: 24rpx;
  padding: 32rpx;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  box-shadow: 0 4rpx 24rpx rgba(60, 30, 10, 0.04);
}

.feature-icon-wrap {
  width: 80rpx;
  height: 80rpx;
  border-radius: 20rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 16rpx;
}

.feature-icon {
  font-size: 40rpx;
}

.feature-title {
  font-size: 30rpx;
  font-weight: 600;
  color: #2B1810;
  margin-bottom: 4rpx;
}

.feature-desc {
  font-size: 24rpx;
  color: #8B7355;
}
```

---

## 十、列表项组件

**AI 参考实现（颜色历史列表）：**

```html
<view class="list-item" bindtap="onTap">
  <view wx:if="{{icon}}" class="list-item-icon" style="background: {{iconBg}};">
    <text class="list-emoji">{{icon}}</text>
  </view>
  <view class="list-item-content">
    <text class="list-item-title">{{title}}</text>
    <text wx:if="{{subtitle}}" class="list-item-subtitle">{{subtitle}}</text>
  </view>
  <view wx:if="{{badge}}" class="list-item-badge">{{badge}}</view>
  <view wx:if="{{showArrow}}" class="list-item-arrow">›</view>
</view>
```

```js
Component({
  properties: {
    icon: { type: String, value: '' },
    iconBg: { type: String, value: '#F2EDE5' },
    title: { type: String, value: '' },
    subtitle: { type: String, value: '' },
    badge: { type: String, value: '' },
    showArrow: { type: Boolean, value: true }
  },

  methods: {
    onTap() {
      this.triggerEvent('tap')
    }
  }
})
```

```css
.list-item {
  display: flex;
  align-items: center;
  padding: 24rpx 32rpx;
  background: #FFFFFF;
  border-bottom: 2rpx solid #F7F3EE;
}

.list-item:active {
  background: #F2EDE5;
}

.list-item-icon {
  width: 64rpx;
  height: 64rpx;
  margin-right: 24rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 16rpx;
}

.list-emoji {
  font-size: 32rpx;
}

.list-item-content {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.list-item-title {
  font-size: 30rpx;
  color: #2B1810;
}

.list-item-subtitle {
  font-size: 24rpx;
  color: #8B7355;
  margin-top: 4rpx;
}

.list-item-badge {
  background: #C0392B;
  color: #FFFFFF;
  font-size: 20rpx;
  min-width: 32rpx;
  height: 32rpx;
  border-radius: 999rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 8rpx;
  margin-right: 16rpx;
}

.list-item-arrow {
  color: #BFAE99;
  font-size: 36rpx;
}
```

**使用示例（颜色历史）：**

```html
<list-item
  icon="🟥"
  iconBg="#FBF1ED"
  title="故宫红"
  subtitle="#9B2335 · 07-06 14:30"
  badge="新"
/>
```

---

## 十一、底部悬浮操作栏

> 完整实现见本技能"二、带悬浮操作栏的页面模板"。此处补充多按钮变体。

**AI 参考实现：**

```html
<view class="bottom-bar">
  <button class="btn-outline bar-btn" bindtap="onSaveDraft">存草稿</button>
  <button class="btn-secondary bar-btn" bindtap="onPreview">预览</button>
  <button class="btn-primary bar-btn" bindtap="onSubmit">保存相册</button>
</view>
```

```css
.bottom-bar .bar-btn-primary {
  flex: 1;
}
```

---

## 十二、数据统计卡片

**AI 参考实现（取色统计）：**

```html
<view class="stat-grid">
  <view wx:for="{{stats}}" wx:key="key" class="stat-card">
    <text class="stat-value" style="color: {{item.color || '#2B1810'}};">{{item.value}}</text>
    <text class="stat-label">{{item.label}}</text>
  </view>
</view>
```

```js
data: {
  stats: [
    { key: 'total', value: '128', label: '取色总数', color: '#9B2335' },
    { key: 'named', value: '120', label: '已命名', color: '#5D6D3D' },
    { key: 'poster', value: '15', label: '海报数', color: '#C9A961' }
  ]
}
```

```css
.stat-grid {
  display: flex;
  gap: 16rpx;
}

.stat-card {
  flex: 1;
  background: #FFFFFF;
  border-radius: 24rpx;
  padding: 32rpx 16rpx;
  text-align: center;
  box-shadow: 0 4rpx 24rpx rgba(60, 30, 10, 0.04);
}

.stat-value {
  font-size: 48rpx;
  font-weight: 700;
  display: block;
  margin-bottom: 8rpx;
}

.stat-label {
  font-size: 24rpx;
  color: #8B7355;
}
```

---

## 十三、标签页切换

**AI 参考实现（颜色分类）：**

```html
<view class="tabs">
  <scroll-view scroll-x class="tabs-scroll" enhanced show-scrollbar="{{false}}">
    <view class="tabs-list">
      <view
        wx:for="{{tabs}}"
        wx:key="key"
        class="tab-item {{activeKey === item.key ? 'active' : ''}}"
        data-key="{{item.key}}"
        bindtap="onTabChange"
      >
        <text class="tab-text">{{item.label}}</text>
        <view wx:if="{{item.count !== undefined}}" class="tab-count">{{item.count}}</view>
        <view wx:if="{{activeKey === item.key}}" class="tab-indicator"></view>
      </view>
    </view>
  </scroll-view>
</view>
```

```js
Component({
  properties: {
    tabs: { type: Array, value: [] },
    activeKey: { type: String, value: '' }
  },

  methods: {
    onTabChange(e) {
      const key = e.currentTarget.dataset.key
      this.setData({ activeKey: key })
      this.triggerEvent('change', { key })
    }
  }
})
```

```css
.tabs {
  background: #FFFFFF;
}

.tabs-scroll {
  width: 100%;
}

.tabs-list {
  display: flex;
  white-space: nowrap;
  padding: 0 16rpx;
}

.tab-item {
  position: relative;
  padding: 24rpx 32rpx;
  display: inline-flex;
  align-items: center;
  gap: 8rpx;
}

.tab-text {
  font-size: 30rpx;
  color: #8B7355;
  font-weight: 500;
}

.tab-item.active .tab-text {
  color: #2B1810;
  font-weight: 600;
}

.tab-count {
  font-size: 22rpx;
  background: #C0392B;
  color: #FFFFFF;
  min-width: 32rpx;
  height: 32rpx;
  border-radius: 999rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0 8rpx;
}

.tab-indicator {
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 48rpx;
  height: 6rpx;
  border-radius: 999rpx;
  background: #9B2335;
}
```

**使用示例：**

```html
<tabs
  tabs="{{[
    { key: 'all', label: '全部', count: 128 },
    { key: 'red', label: '红', count: 45 },
    { key: 'cyan', label: '青', count: 30 },
    { key: 'yellow', label: '黄', count: 53 }
  ]}}"
  activeKey="all"
/>
```

---

## 十四、加载骨架屏

**AI 参考实现（历史页加载）：**

```html
<view class="skeleton-container" wx:if="{{loading}}">
  <view wx:for="{{repeatCount}}" wx:key="index" class="skeleton-card">
    <view class="skeleton-line skeleton-title"></view>
    <view class="skeleton-line skeleton-text"></view>
    <view class="skeleton-line skeleton-text short"></view>
  </view>
</view>
```

```js
Component({
  properties: {
    loading: { type: Boolean, value: true },
    repeatCount: { type: Number, value: 3 }
  }
})
```

```css
.skeleton-container {
  padding: 32rpx;
}

.skeleton-card {
  background: #FFFFFF;
  border-radius: 24rpx;
  padding: 32rpx;
  margin-bottom: 24rpx;
}

.skeleton-line {
  border-radius: 8rpx;
  background: linear-gradient(
    90deg,
    #E8DDC9 25%,
    #F7F3EE 50%,
    #E8DDC9 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

.skeleton-title {
  width: 40%;
  height: 36rpx;
  margin-bottom: 24rpx;
}

.skeleton-text {
  width: 100%;
  height: 28rpx;
  margin-bottom: 16rpx;
}

.skeleton-text.short {
  width: 60%;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

---

## 十五、通用工具函数

**AI 参考实现（utils/util.js）：**

```js
function formatSize(bytes) {
  if (!bytes) return '0B'
  if (bytes < 1024) return bytes + 'B'
  if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(1) + 'KB'
  if (bytes < 1024 * 1024 * 1024) return (bytes / 1024 / 1024).toFixed(1) + 'MB'
  return (bytes / 1024 / 1024 / 1024).toFixed(1) + 'GB'
}

function formatDate(date, fmt = 'YYYY-MM-DD HH:mm') {
  if (!date) return ''
  const d = new Date(date)
  if (isNaN(d.getTime())) return ''

  const pad = (n) => String(n).padStart(2, '0')
  const map = {
    YYYY: d.getFullYear(),
    MM: pad(d.getMonth() + 1),
    DD: pad(d.getDate()),
    HH: pad(d.getHours()),
    mm: pad(d.getMinutes()),
    ss: pad(d.getSeconds())
  }

  return fmt.replace(/YYYY|MM|DD|HH|mm|ss/g, (match) => map[match])
}

function debounce(fn, delay = 300) {
  let timer = null
  return function (...args) {
    if (timer) clearTimeout(timer)
    timer = setTimeout(() => {
      fn.apply(this, args)
      timer = null
    }, delay)
  }
}

function copyToClipboard(text) {
  return new Promise((resolve, reject) => {
    wx.setClipboardData({
      data: text,
      success: () => {
        wx.showToast({ title: '已复制', icon: 'success' })
        resolve(true)
      },
      fail: (err) => {
        wx.showToast({ title: '复制失败', icon: 'none' })
        reject(err)
      }
    })
  })
}

module.exports = {
  formatSize,
  formatDate,
  debounce,
  copyToClipboard
}
```

**使用示例：**

```js
const { formatSize, formatDate, debounce, copyToClipboard } = require('../../utils/util')

Page({
  data: {
    fileSize: formatSize(1048576),
    createDate: formatDate(Date.now()),
    inputValue: ''
  },

  onSearch: debounce(function (e) {
    this.searchData(e.detail.value)
  }, 500),

  async onCopyHex() {
    await copyToClipboard('#9B2335')
  }
})
```

---

## 十六、组件使用检查清单

开发新页面/组件时逐项检查：

### 结构规范
- [ ] 页面 JS 中加载了 `theme`（如需使用 Token）
- [ ] 页面定义了 `onShareAppMessage` 和 `onShareTimeline`
- [ ] `onLoad` 接收了页面参数（options）
- [ ] 错误状态使用 `error-view` 组件，而非自定义
- [ ] 加载状态使用 `skeleton` 组件
- [ ] 列表项使用 `list-item` 组件

### 样式规范
- [ ] 所有颜色使用 Token 或 CSS 变量，无硬编码色值
- [ ] 字体大小来自规范表（52/36/32/28/26/24/22rpx）
- [ ] 卡片圆角统一 24rpx，按钮圆角 28rpx
- [ ] 有底部操作栏的页面加了 `has-bottom-bar` 类
- [ ] 底部操作栏适配了 safe-area

### 交互规范
- [ ] 按钮有 `:active` 反馈（缩放或透明度变化）
- [ ] 表单提交有 loading 状态
- [ ] 操作成功有 toast 反馈
- [ ] AI 输出使用 `ai-stream` 组件，有光标动画
- [ ] 提示信息使用 `risk-item` 组件

### 数据规范
- [ ] 文件大小显示用 `formatSize`
- [ ] 日期显示用 `formatDate`
- [ ] 搜索输入用 `debounce` 防抖
- [ ] 复制功能用 `copyToClipboard`

**AI 执行约束：** 任一项未达标不能交付；禁止重复造已有组件的轮子。

---

## 使用方式

当你需要开发某个组件或页面时，告诉我具体需求，我帮你匹配最合适的模板。

**对话示例：**
- "我要做一个颜色历史列表" → 我给你 list-item 完整模板 + 颜色历史示例
- "AI 命名结果要打字效果" → 我给你 ai-stream 组件
- "取色置信度低要提示用户" → 我给你 risk-item 提示条目
- "历史页加载要骨架屏" → 我给你 skeleton 组件
- "要按颜色分类切换" → 我给你 tabs 组件

---

## 与其他技能的关系

- **前置：** xx-setup 项目骨架（components/ 目录位置）
- **并行：** xx-brand 提供 Token 和样式 class，本 skill 组件直接引用
- **下游：** xx-ai 的 AI 命名结果用本 skill 的 ai-stream 组件展示
- **回溯：** xx-track 监控组件加载耗时，反向优化骨架屏策略

## 方法论来源

- **Apple Human Interface Guidelines** — 组件交互与视觉规范
- **Atomic Design** — Brad Frost，原子化组件设计
- **微信小程序自定义组件** — Component 构造器与生命周期
- **设计系统实践** — 组件复用与一致性保障策略
