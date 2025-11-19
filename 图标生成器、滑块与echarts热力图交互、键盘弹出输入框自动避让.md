# 核心功能实现学习文档

本文档汇总了三个重要的核心功能实现逻辑，供日后学习和参考使用。

---

## 目录

1. [滑块调整 ECharts 视图显示](#一滑块调整-echarts-视图显示)
2. [键盘弹出时输入框自动避让](#二键盘弹出时输入框自动避让)
3. [TabBar 图标生成器](#三tabbar-图标生成器)

---

## 一、滑块调整 ECharts 视图显示

### 1.1 功能说明

通过外部滑块（slider）控制 ECharts 热力图的温度显示范围，实现数据筛选和可视化交互。

### 1.2 核心原理

1. **双滑块控制**：最低温度滑块 + 最高温度滑块
2. **实时更新**：滑块拖动时实时更新 visualMap 配置
3. **联动显示**：滑块值、visualMap 标签、热力图同步更新

### 1.3 实现步骤

#### 步骤 1: 定义响应式变量

```typescript
// 全局温度范围（数据的实际范围）
const globalMinTemp = ref(15)
const globalMaxTemp = ref(50)

// 用户选择的温度范围
const tempRangeMin = ref(15)
const tempRangeMax = ref(50)
```

#### 步骤 2: 模板结构

```vue
<template>
  <view class="temp-range-control">
    <!-- 标题显示当前范围 -->
    <view class="range-header">
      <text class="range-title">温度范围筛选</text>
      <text class="range-value">
        {{ tempRangeMin.toFixed(1) }}℃ - {{ tempRangeMax.toFixed(1) }}℃
      </text>
    </view>
    
    <!-- 滑块组 -->
    <view class="range-sliders">
      <!-- 最低温度滑块 -->
      <view class="slider-group">
        <text class="slider-label">最低温度</text>
        <slider 
          :min="globalMinTemp" 
          :max="globalMaxTemp" 
          :value="tempRangeMin"
          :step="0.1"
          activeColor="#6366f1"
          backgroundColor="#334155"
          block-size="20"
          @change="handleMinTempChange"
          @changing="handleMinTempChanging"
        />
        <text class="slider-value">{{ tempRangeMin.toFixed(1) }}℃</text>
      </view>
      
      <!-- 最高温度滑块 -->
      <view class="slider-group">
        <text class="slider-label">最高温度</text>
        <slider 
          :min="globalMinTemp" 
          :max="globalMaxTemp" 
          :value="tempRangeMax"
          :step="0.1"
          activeColor="#ef4444"
          backgroundColor="#334155"
          block-size="20"
          @change="handleMaxTempChange"
          @changing="handleMaxTempChanging"
        />
        <text class="slider-value">{{ tempRangeMax.toFixed(1) }}℃</text>
      </view>
    </view>
    
    <!-- 重置按钮 -->
    <button class="reset-btn" @tap="resetTempRange">重置范围</button>
  </view>
</template>
```

#### 步骤 3: 事件处理逻辑

```typescript
// 最低温度变化事件（拖动结束）
const handleMinTempChange = (e: any) => {
  const newMin = parseFloat(e.detail.value.toFixed(1))
  // 确保最低温度 < 最高温度
  if (newMin < tempRangeMax.value) {
    tempRangeMin.value = newMin
    updateChartVisualMap() // 更新图表
  }
}

// 最低温度实时跟随（拖动中）
const handleMinTempChanging = (e: any) => {
  const newMin = parseFloat(e.detail.value.toFixed(1))
  if (newMin < tempRangeMax.value) {
    tempRangeMin.value = newMin
    // 注意：这里不调用 updateChartVisualMap，避免频繁更新影响性能
  }
}

// 最高温度变化事件（拖动结束）
const handleMaxTempChange = (e: any) => {
  const newMax = parseFloat(e.detail.value.toFixed(1))
  // 确保最高温度 > 最低温度
  if (newMax > tempRangeMin.value) {
    tempRangeMax.value = newMax
    updateChartVisualMap() // 更新图表
  }
}

// 最高温度实时跟随（拖动中）
const handleMaxTempChanging = (e: any) => {
  const newMax = parseFloat(e.detail.value.toFixed(1))
  if (newMax > tempRangeMin.value) {
    tempRangeMax.value = newMax
  }
}

// 重置温度范围
const resetTempRange = () => {
  tempRangeMin.value = globalMinTemp.value
  tempRangeMax.value = globalMaxTemp.value
  updateChartVisualMap()
}
```

#### 步骤 4: 更新 ECharts 配置

```typescript
// 更新图表的 visualMap 配置
const updateChartVisualMap = () => {
  if (!chartInstance) return
  
  chartInstance.setOption({
    visualMap: {
      min: tempRangeMin.value,
      max: tempRangeMax.value,
      text: [
        `高温 ${tempRangeMax.value.toFixed(1)}℃`,
        `低温 ${tempRangeMin.value.toFixed(1)}℃`
      ],
    }
  })
}
```

#### 步骤 5: 初始化时更新全局范围

```typescript
// 在 updateChart 函数中，计算出实际温度范围后
const minTemp = Math.floor((calculatedMin - padding) * 10) / 10
const maxTemp = Math.ceil((calculatedMax + padding) * 10) / 10

// 更新全局温度范围
globalMinTemp.value = minTemp
globalMaxTemp.value = maxTemp

// 如果当前筛选范围超出新的全局范围，重置筛选范围
if (tempRangeMin.value < minTemp || tempRangeMin.value > maxTemp) {
  tempRangeMin.value = minTemp
}
if (tempRangeMax.value > maxTemp || tempRangeMax.value < minTemp) {
  tempRangeMax.value = maxTemp
}
```

### 1.4 样式参考

```scss
.temp-range-control {
  background: rgba(10, 32, 68, 0.6);
  border: 1px solid rgba(30, 144, 255, 0.3);
  border-radius: 16rpx;
  padding: 24rpx;
  margin-bottom: 24rpx;

  .range-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 24rpx;

    .range-title {
      color: #fff;
      font-size: 28rpx;
      font-weight: bold;
    }

    .range-value {
      color: #1e90ff;
      font-size: 26rpx;
      font-weight: bold;
    }
  }

  .range-sliders {
    .slider-group {
      margin-bottom: 24rpx;
      display: flex;
      flex-direction: column;
      gap: 8rpx;

      .slider-label {
        color: #9ca3af;
        font-size: 24rpx;
      }

      .slider-value {
        align-self: flex-end;
        color: #fff;
        font-size: 24rpx;
        font-weight: bold;
      }
    }
  }

  .reset-btn {
    width: 100%;
    padding: 16rpx 0;
    background: rgba(30, 144, 255, 0.2);
    border: 1px solid rgba(30, 144, 255, 0.4);
    color: #1e90ff;
    font-size: 26rpx;
    border-radius: 8rpx;
  }
}
```

### 1.5 关键点总结

1. **@change vs @changing**
   - `@change`: 拖动结束时触发，用于更新图表
   - `@changing`: 拖动中实时触发，用于更新显示值

2. **性能优化**
   - 拖动中不频繁更新图表，只更新显示值
   - 拖动结束后才更新图表配置

3. **数据验证**
   - 始终确保 `minTemp < maxTemp`
   - 防止无效的温度范围设置

4. **精度控制**
   - 使用 `toFixed(1)` 保留一位小数
   - `parseFloat` 转换确保数值类型

---

## 二、键盘弹出时输入框自动避让

### 2.1 功能说明

在小程序登录页面，当键盘弹出时，自动调整页面布局，确保输入框不被键盘遮挡。

### 2.2 核心原理

1. **监听键盘高度变化**：使用 `uni.onKeyboardHeightChange`
2. **动态调整布局**：根据键盘高度设置 `paddingBottom`
3. **页面滚动支持**：使用 `scroll-view` 和 `page-meta` 配合

### 2.3 完整实现

#### 步骤 1: 模板结构

```vue
<template>
  <!-- 动态控制页面溢出属性 -->
  <page-meta 
    :page-style="'overflow: ' + (keyboardHeight > 0 ? 'visible' : 'hidden')" 
  />
  
  <!-- 可滚动容器 -->
  <scroll-view 
    class="login-container" 
    scroll-y 
    :scroll-with-animation="true"
  >
    <!-- 登录内容容器，底部 padding 根据键盘高度动态调整 -->
    <view 
      class="login-wrapper" 
      :style="{ paddingBottom: keyboardHeight + 'px' }"
    >
      <view class="login-card">
        <!-- 输入框 -->
        <view class="input-wrapper">
          <input
            v-model="formData.username"
            type="text"
            placeholder="请输入用户名"
            :adjust-position="true"
            :hold-keyboard="false"
            @focus="focusedInput = 'username'"
            @blur="focusedInput = ''"
          />
        </view>
        
        <view class="input-wrapper">
          <input
            v-model="formData.password"
            type="password"
            placeholder="请输入密码"
            :adjust-position="true"
            :hold-keyboard="false"
            @focus="focusedInput = 'password'"
            @blur="focusedInput = ''"
          />
        </view>
      </view>
    </view>
  </scroll-view>
</template>
```

#### 步骤 2: 响应式变量

```typescript
import { ref, onMounted, onUnmounted } from 'vue'

// 键盘高度（单位：px）
const keyboardHeight = ref(0)

// 当前聚焦的输入框
const focusedInput = ref('')

// 表单数据
const formData = reactive({
  username: '',
  password: '',
})
```

#### 步骤 3: 监听键盘事件

```typescript
// 页面挂载时，监听键盘高度变化
onMounted(() => {
  uni.onKeyboardHeightChange((res) => {
    keyboardHeight.value = res.height
    console.log('键盘高度:', res.height)
  })
})

// 页面卸载时，移除监听
onUnmounted(() => {
  uni.offKeyboardHeightChange(() => {})
})
```

#### 步骤 4: Input 配置说明

```vue
<input
  :adjust-position="true"   
  <!-- 键盘弹起时，自动调整页面滚动位置 -->
  
  :hold-keyboard="false"     
  <!-- 点击页面其他区域时，自动收起键盘 -->
  
  @focus="focusedInput = 'username'"  
  <!-- 聚焦时记录当前输入框 -->
  
  @blur="focusedInput = ''"   
  <!-- 失焦时清除记录 -->
/>
```

### 2.4 工作流程

```
1. 用户点击输入框
   ↓
2. 键盘弹出，触发 onKeyboardHeightChange
   ↓
3. 更新 keyboardHeight 的值
   ↓
4. 触发模板重新渲染
   ↓
5. paddingBottom 动态增加，内容区域上移
   ↓
6. 输入框完全可见，不被键盘遮挡
   ↓
7. 用户完成输入，键盘收起
   ↓
8. keyboardHeight 变为 0，布局恢复正常
```

### 2.5 样式配置

```scss
.login-container {
  width: 100%;
  height: 100vh;
  overflow-y: auto; // 允许滚动
  background: #0a0e1a;
}

.login-wrapper {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 40rpx;
  box-sizing: border-box;
  // paddingBottom 通过 :style 动态设置
}

.login-card {
  width: 100%;
  max-width: 680rpx;
  background: rgba(26, 34, 53, 0.8);
  border-radius: 32rpx;
  padding: 60rpx 50rpx;
  box-shadow: 0 8rpx 32rpx rgba(0, 0, 0, 0.5);
}
```

### 2.6 关键点总结

| 配置项 | 作用 | 重要性 |
|--------|------|--------|
| `page-meta` | 控制页面溢出属性 | ⭐⭐⭐ |
| `scroll-view` | 提供滚动能力 | ⭐⭐⭐ |
| `paddingBottom` | 动态调整底部空间 | ⭐⭐⭐⭐⭐ |
| `adjust-position` | 自动滚动到可见区域 | ⭐⭐⭐⭐ |
| `hold-keyboard` | 控制键盘收起时机 | ⭐⭐ |
| `onKeyboardHeightChange` | 监听键盘高度 | ⭐⭐⭐⭐⭐ |

### 2.7 常见问题

**Q1: 为什么需要 page-meta？**
> 键盘弹起时，如果页面 overflow 为 hidden，可能导致滚动失效。动态设置为 visible 可以确保正常滚动。

**Q2: 为什么用 paddingBottom 而不是 marginBottom？**
> padding 是内边距，包含在元素内部，不会导致布局抖动。margin 是外边距，可能引起其他元素位置变化。

**Q3: 如何处理不同键盘高度？**
> `onKeyboardHeightChange` 会自动返回实际键盘高度，无需手动适配不同设备。

**Q4: iOS 和 Android 表现一致吗？**
> 基本一致，但 iOS 键盘弹出动画更流畅。建议添加 `scroll-with-animation="true"` 优化体验。

---

## 三、TabBar 图标生成器

### 3.1 功能说明

使用 Node.js Canvas 库自动批量生成小程序 TabBar 所需的图标（包括普通状态和激活状态）。

### 3.2 技术栈

- **Node.js**: 运行环境
- **canvas**: 绘图库（需要 `npm install canvas`）
- **fs**: 文件系统操作
- **path**: 路径处理

### 3.3 核心代码结构

```javascript
const fs = require('fs');
const path = require('path');
const { Canvas } = require('canvas');

// ========== 1. 配置部分 ==========

const iconDir = path.join(__dirname, '../src/static/tabbar');
const size = 162; // 2倍图，实际显示 81x81

// 图标配置数组
const icons = [
  {
    name: 'home',           // 文件名
    type: 'house',          // 绘制类型
    color: '#9ca3af',       // 普通状态颜色
    activeColor: '#818cf8', // 激活状态颜色
  },
  {
    name: 'heatmap',
    type: 'thermometer',
    color: '#9ca3af',
    activeColor: '#818cf8',
  },
  {
    name: 'chart4',
    type: 'warning',
    color: '#9ca3af',
    activeColor: '#818cf8',
  },
];

// ========== 2. 绘制函数集合 ==========

const drawers = {
  // 房子图标
  house: (ctx, size, color) => {
    ctx.strokeStyle = color;
    ctx.fillStyle = color;
    ctx.lineWidth = size * 0.08;
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    
    const centerX = size / 2;
    const centerY = size / 2;
    const houseSize = size * 0.5;
    
    // 绘制屋顶（三角形）
    ctx.beginPath();
    ctx.moveTo(centerX, centerY - houseSize * 0.4);
    ctx.lineTo(centerX - houseSize * 0.3, centerY - houseSize * 0.05);
    ctx.lineTo(centerX + houseSize * 0.3, centerY - houseSize * 0.05);
    ctx.closePath();
    ctx.fill();
    
    // 绘制墙体（矩形）
    ctx.fillRect(
      centerX - houseSize * 0.25, 
      centerY - houseSize * 0.05, 
      houseSize * 0.5, 
      houseSize * 0.4
    );
  },
  
  // 温度计图标
  thermometer: (ctx, size, color) => {
    ctx.strokeStyle = color;
    ctx.fillStyle = color;
    ctx.lineWidth = size * 0.06;
    ctx.lineCap = 'round';
    
    const centerX = size / 2;
    const topY = size * 0.2;
    const bottomY = size * 0.75;
    
    // 温度计杆（竖线）
    ctx.beginPath();
    ctx.moveTo(centerX, topY);
    ctx.lineTo(centerX, bottomY);
    ctx.stroke();
    
    // 底部圆（水银球）
    ctx.beginPath();
    ctx.arc(centerX, bottomY, size * 0.08, 0, Math.PI * 2);
    ctx.fill();
    
    // 液体填充（矩形）
    ctx.fillRect(
      centerX - size * 0.04, 
      bottomY - size * 0.25, 
      size * 0.08, 
      size * 0.25
    );
  },
  
  // 警告图标
  warning: (ctx, size, color) => {
    ctx.fillStyle = color;
    
    const centerX = size / 2;
    const topY = size * 0.25;
    const bottomY = size * 0.7;
    const width = size * 0.4;
    
    // 三角形外框
    ctx.beginPath();
    ctx.moveTo(centerX, topY);
    ctx.lineTo(centerX - width / 2, bottomY);
    ctx.lineTo(centerX + width / 2, bottomY);
    ctx.closePath();
    ctx.fill();
    
    // 感叹号（背景色）
    ctx.fillStyle = '#0a0e1a';
    
    // 感叹号竖线
    ctx.fillRect(
      centerX - size * 0.03, 
      topY + size * 0.15, 
      size * 0.06, 
      size * 0.2
    );
    
    // 感叹号点
    ctx.beginPath();
    ctx.arc(centerX, bottomY - size * 0.1, size * 0.03, 0, Math.PI * 2);
    ctx.fill();
  },
};

// ========== 3. 图标生成函数 ==========

function generateIcon(iconConfig, isActive) {
  // 创建 Canvas
  const canvas = new Canvas(size, size);
  const ctx = canvas.getContext('2d');
  
  // 清空画布，设置透明背景
  ctx.clearRect(0, 0, size, size);
  
  // 获取对应的绘制函数
  const drawer = drawers[iconConfig.type];
  if (!drawer) {
    console.error(`未知的图标类型: ${iconConfig.type}`);
    return null;
  }
  
  // 选择颜色（普通或激活）
  const color = isActive ? iconConfig.activeColor : iconConfig.color;
  
  // 执行绘制
  drawer(ctx, size, color);
  
  // 生成文件名
  const filename = isActive 
    ? `${iconConfig.name}-active.png` 
    : `${iconConfig.name}.png`;
  const filepath = path.join(iconDir, filename);
  
  // 保存为 PNG 文件
  const buffer = canvas.toBuffer('image/png');
  fs.writeFileSync(filepath, buffer);
  
  return filename;
}

// ========== 4. 主函数 ==========

function main() {
  console.log('开始生成 TabBar 图标...\n');
  
  // 确保目录存在
  if (!fs.existsSync(iconDir)) {
    fs.mkdirSync(iconDir, { recursive: true });
  }
  
  // 批量生成所有图标
  icons.forEach((icon) => {
    const normalFile = generateIcon(icon, false);  // 普通状态
    const activeFile = generateIcon(icon, true);   // 激活状态
    
    if (normalFile && activeFile) {
      console.log(`✅ 生成 ${icon.name}: ${normalFile}, ${activeFile}`);
    } else {
      console.error(`❌ 生成 ${icon.name} 失败`);
    }
  });
  
  console.log('\n✨ 所有图标生成完成！');
  console.log(`图标位置: ${iconDir}`);
}

// ========== 5. 执行 ==========

main();
```

### 3.4 使用方法

#### 方法 1: 安装依赖并运行

```bash
# 1. 安装 canvas 依赖
npm install canvas

# 2. 运行脚本
node scripts/generate-tabbar-png-icons.js

# 3. 查看生成的图标
# 位置：src/static/tabbar/
```

#### 方法 2: 添加到 package.json

```json
{
  "scripts": {
    "generate:icons": "node scripts/generate-tabbar-png-icons.js"
  }
}
```

```bash
# 运行
npm run generate:icons
```

### 3.5 添加新图标

#### 步骤 1: 定义绘制函数

```javascript
const drawers = {
  // ... 其他绘制函数 ...
  
  // 新增：用户图标
  user: (ctx, size, color) => {
    ctx.fillStyle = color;
    const centerX = size / 2;
    const centerY = size / 2;
    
    // 头部（圆形）
    ctx.beginPath();
    ctx.arc(centerX, centerY - size * 0.1, size * 0.15, 0, Math.PI * 2);
    ctx.fill();
    
    // 身体（梯形，用路径绘制）
    ctx.beginPath();
    ctx.moveTo(centerX - size * 0.25, centerY + size * 0.3);
    ctx.lineTo(centerX - size * 0.15, centerY + size * 0.05);
    ctx.lineTo(centerX + size * 0.15, centerY + size * 0.05);
    ctx.lineTo(centerX + size * 0.25, centerY + size * 0.3);
    ctx.closePath();
    ctx.fill();
  },
};
```

#### 步骤 2: 添加配置

```javascript
const icons = [
  // ... 其他图标配置 ...
  
  {
    name: 'user',
    type: 'user',
    color: '#9ca3af',
    activeColor: '#818cf8',
  },
];
```

#### 步骤 3: 运行脚本

```bash
npm run generate:icons
```

### 3.6 Canvas 绘图 API 速查

| API | 说明 | 示例 |
|-----|------|------|
| `ctx.fillStyle` | 设置填充颜色 | `ctx.fillStyle = '#ff0000'` |
| `ctx.strokeStyle` | 设置描边颜色 | `ctx.strokeStyle = '#00ff00'` |
| `ctx.lineWidth` | 设置线宽 | `ctx.lineWidth = 2` |
| `ctx.lineCap` | 设置线端样式 | `ctx.lineCap = 'round'` |
| `ctx.lineJoin` | 设置线连接样式 | `ctx.lineJoin = 'round'` |
| `ctx.beginPath()` | 开始路径 | - |
| `ctx.closePath()` | 闭合路径 | - |
| `ctx.moveTo(x, y)` | 移动到起点 | `ctx.moveTo(10, 10)` |
| `ctx.lineTo(x, y)` | 画线到某点 | `ctx.lineTo(100, 100)` |
| `ctx.arc(x, y, r, start, end)` | 画圆弧 | `ctx.arc(50, 50, 20, 0, Math.PI*2)` |
| `ctx.fill()` | 填充路径 | - |
| `ctx.stroke()` | 描边路径 | - |
| `ctx.fillRect(x, y, w, h)` | 填充矩形 | `ctx.fillRect(10, 10, 50, 50)` |
| `ctx.strokeRect(x, y, w, h)` | 描边矩形 | `ctx.strokeRect(10, 10, 50, 50)` |
| `ctx.clearRect(x, y, w, h)` | 清除矩形区域 | `ctx.clearRect(0, 0, 200, 200)` |

### 3.7 图标设计建议

1. **尺寸比例**
   - 图标主体占 canvas 的 50%-60%
   - 留出适当边距，避免贴边

2. **线宽设置**
   - 推荐：`size * 0.05` 至 `size * 0.08`
   - 太细不清晰，太粗显笨重

3. **颜色选择**
   - 普通状态：灰色调 `#9ca3af`
   - 激活状态：主题色 `#818cf8`、`#6366f1`

4. **细节处理**
   - 使用 `lineCap: 'round'` 让线端圆润
   - 使用 `lineJoin: 'round'` 让转角平滑
   - 避免过于复杂的图形，保持简洁

5. **测试验证**
   - 生成后在实际设备查看效果
   - 检查普通和激活状态的视觉区分度
   - 确保在不同背景下都清晰可见

### 3.8 故障排除

**Q1: 提示 "canvas 模块未安装"**
```bash
# 解决：安装 canvas
npm install canvas

# 如果安装失败（需要编译），尝试
npm install canvas --legacy-peer-deps
```

**Q2: macOS 安装 canvas 失败**
```bash
# 安装依赖
brew install pkg-config cairo pango libpng jpeg giflib librsvg

# 再安装 canvas
npm install canvas
```

**Q3: Windows 安装 canvas 失败**
```bash
# 使用预编译版本
npm install canvas --canvas_binary_host_mirror=https://registry.npmmirror.com/-/binary/canvas
```

**Q4: 生成的图标是空白的**
> 检查绘制函数是否正确调用了 `fill()` 或 `stroke()`

**Q5: 图标颜色不对**
> 确保在调用绘制函数前设置了 `ctx.fillStyle` 或 `ctx.strokeStyle`

---

## 四、综合应用场景

### 4.1 完整的小程序登录页

结合键盘避让 + 精美 UI：

```vue
<template>
  <page-meta :page-style="'overflow: ' + (keyboardHeight > 0 ? 'visible' : 'hidden')" />
  <scroll-view class="login-container" scroll-y :scroll-with-animation="true">
    <view class="login-wrapper" :style="{ paddingBottom: keyboardHeight + 'px' }">
      <!-- 登录表单内容 -->
    </view>
  </scroll-view>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const keyboardHeight = ref(0)

onMounted(() => {
  uni.onKeyboardHeightChange((res) => {
    keyboardHeight.value = res.height
  })
})

onUnmounted(() => {
  uni.offKeyboardHeightChange(() => {})
})
</script>
```

### 4.2 带温度筛选的热力图页面

结合滑块控制 + ECharts 交互：

```vue
<template>
  <view class="page">
    <!-- 温度范围控制 -->
    <view class="temp-range-control">
      <slider 
        :min="globalMin" 
        :max="globalMax" 
        :value="rangeMin"
        @change="handleMinChange"
      />
      <slider 
        :min="globalMin" 
        :max="globalMax" 
        :value="rangeMax"
        @change="handleMaxChange"
      />
    </view>
    
    <!-- 热力图 -->
    <canvas id="chart" type="2d"></canvas>
  </view>
</template>

<script setup>
const updateChartRange = () => {
  chartInstance.setOption({
    visualMap: {
      min: rangeMin.value,
      max: rangeMax.value,
    }
  })
}
</script>
```

### 4.3 自动化图标生成流程

在 CI/CD 中集成：

```json
// package.json
{
  "scripts": {
    "prepare": "npm run generate:icons",
    "generate:icons": "node scripts/generate-tabbar-png-icons.js",
    "build": "npm run generate:icons && uni build"
  }
}
```

---

## 五、学习资源

### 5.1 官方文档

- [uni-app 官方文档](https://uniapp.dcloud.net.cn/)
- [微信小程序文档](https://developers.weixin.qq.com/miniprogram/dev/framework/)
- [ECharts 官方文档](https://echarts.apache.org/zh/index.html)
- [Node.js Canvas](https://github.com/Automattic/node-canvas)

### 5.2 延伸阅读

- Canvas 2D API 教程
- 小程序键盘交互最佳实践
- ECharts 在小程序中的性能优化

### 5.3 工具推荐

- **调试工具**: 微信开发者工具
- **设计工具**: Figma / Sketch（设计图标原型）
- **图标库**: Ionicons / Font Awesome（参考图标设计）

---

## 六、总结

本文档汇总了三个核心功能的实现逻辑：

1. **滑块调整 ECharts 视图**：实现数据可视化的交互筛选
2. **键盘自动避让**：提升移动端输入体验
3. **图标自动生成**：提高开发效率，保持设计一致性

这些功能都遵循以下原则：
- ✅ **响应式设计**：适配不同设备和场景
- ✅ **性能优化**：避免频繁更新和不必要的计算
- ✅ **用户体验**：流畅的交互和清晰的视觉反馈
- ✅ **可维护性**：代码结构清晰，易于扩展

---

**文档版本**: 1.0.0  
**创建日期**: 2025-11-04  
**适用项目**: 三峡东岳庙数据中心小程序


