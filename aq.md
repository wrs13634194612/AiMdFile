vue自定义颜色选择器

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/82edca81eb024028a0e700dee4bbf98e.png#pic_center)

step0: 默认写法 调用系统自带的颜色选择器

```bash
       <input type="color">

```

step1:C:\Users\wangrusheng\PycharmProjects\untitled18\src\views\Home.vue

```typescript
<template>
  <div class="container">
    <!-- 颜色选择器组件 -->
    <ColorPicker v-model="selectedColor" />

    <!-- 新增的动态背景按钮 -->
    <div>

    <button
      class="dynamic-button"
      :style="{ backgroundColor: selectedColor }"
    >
      我的背景色会变化！
    </button>

       <input type="color">

    <p>当前选中颜色: {{ selectedColor }}</p>

    </div>
     </div>
</template>

<script>
import ColorPicker from './ColorPicker.vue'

export default {
  components: { ColorPicker },
  data() {
    return {
      selectedColor: '#ff0000' // 默认颜色
    }
  }
}
</script>



```

step2:C:\Users\wangrusheng\PycharmProjects\untitled18\src\views\ColorPicker.vue

```typescript
<template>
  <div class="color-picker">
    <!-- 饱和度/明度选择区域 -->
    <div
      class="saturation"
      :style="{ backgroundColor: `hsl(${hsv.h}, 100%, 50%)` }"
      @mousedown="startDrag"
    >
      <div
        class="selector"
        :style="{
          left: `${hsv.s * 100}%`,
          top: `${(1 - hsv.v) * 100}%`,
          backgroundColor: currentColor
        }"
      ></div>
    </div>

    <!-- 色相滑块 -->
    <div class="hue-slider" @mousedown="startHueDrag">
      <div
        class="hue-pointer"
        :style="{ left: `${(hsv.h / 360) * 100}%` }"
      ></div>
    </div>

    <!-- 颜色显示和输入 -->
    <div class="color-preview" :style="{ backgroundColor: currentColor }"></div>
    <input
      v-model="hexColor"
      class="hex-input"
      placeholder="#FFFFFF"
      @input="handleHexInput"
    >
  </div>
</template>

<script>
export default {
  props: {
    modelValue: String
  },
  emits: ['update:modelValue'],
  data() {
    return {
      hsv: { h: 0, s: 1, v: 1 },
      hexColor: '#ff0000',
      isDragging: false,
      isHueDragging: false
    }
  },
  computed: {
    currentColor() {
      return this.hsvToHex(this.hsv)
    }
  },
  methods: {
    startDrag(e) {
      this.isDragging = true
      this.handleDrag(e)
      window.addEventListener('mousemove', this.handleDrag)
      window.addEventListener('mouseup', this.stopDrag)
    },
    startHueDrag(e) {
      this.isHueDragging = true
      this.handleHueDrag(e)
      window.addEventListener('mousemove', this.handleHueDrag)
      window.addEventListener('mouseup', this.stopHueDrag)
    },
    handleDrag(e) {
      if (!this.isDragging) return
      const rect = e.target.getBoundingClientRect()
      const x = Math.max(0, Math.min(1, (e.clientX - rect.left) / rect.width))
      const y = Math.max(0, Math.min(1, (e.clientY - rect.top) / rect.height))

      this.hsv.s = x
      this.hsv.v = 1 - y
      this.updateHex()
    },
    handleHueDrag(e) {
      if (!this.isHueDragging) return
      const rect = e.target.getBoundingClientRect()
      const x = Math.max(0, Math.min(1, (e.clientX - rect.left) / rect.width))
      this.hsv.h = x * 360
      this.updateHex()
    },
    stopDrag() {
      this.isDragging = false
      window.removeEventListener('mousemove', this.handleDrag)
      window.removeEventListener('mouseup', this.stopDrag)
    },
    stopHueDrag() {
      this.isHueDragging = false
      window.removeEventListener('mousemove', this.handleHueDrag)
      window.removeEventListener('mouseup', this.stopHueDrag)
    },
    updateHex() {
      this.hexColor = this.hsvToHex(this.hsv)
      this.$emit('update:modelValue', this.hexColor)
    },
    handleHexInput() {
      if (/^#([0-9A-F]{3}){1,2}$/i.test(this.hexColor)) {
        this.hsv = this.hexToHsv(this.hexColor)
      }
    },
    // 颜色转换函数
    hsvToHex(hsv) {
      const h = hsv.h / 360
      let r, g, b

      const i = Math.floor(h * 6)
      const f = h * 6 - i
      const p = hsv.v * (1 - hsv.s)
      const q = hsv.v * (1 - f * hsv.s)
      const t = hsv.v * (1 - (1 - f) * hsv.s)

      switch (i % 6) {
        case 0: r = hsv.v, g = t, b = p; break
        case 1: r = q, g = hsv.v, b = p; break
        case 2: r = p, g = hsv.v, b = t; break
        case 3: r = p, g = q, b = hsv.v; break
        case 4: r = t, g = p, b = hsv.v; break
        case 5: r = hsv.v, g = p, b = q; break
      }

      return `#${[r, g, b]
        .map(x => Math.round(x * 255)
        .toString(16)
        .padStart(2, '0'))
        .join('')}`
    },
    hexToHsv(hex) {
      // 转换逻辑（此处省略具体实现）
      // 返回类似 {h: 0, s: 1, v: 1} 的HSV对象
    }
  }
}
</script>

<style>
.color-picker {
  width: 300px;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.saturation {
  position: relative;
  width: 100%;
  height: 200px;
  border-radius: 4px;
  background: linear-gradient(to top, #000, transparent),
              linear-gradient(to right, #fff, transparent);
}

.selector {
  position: absolute;
  width: 16px;
  height: 16px;
  border: 2px solid white;
  border-radius: 50%;
  transform: translate(-8px, -8px);
  box-shadow: 0 1px 3px rgba(0,0,0,0.3);
}

.hue-slider {
  position: relative;
  height: 12px;
  margin: 15px 0;
  background: linear-gradient(to right,
    #ff0000 0%,
    #ffff00 17%,
    #00ff00 33%,
    #00ffff 50%,
    #0000ff 67%,
    #ff00ff 83%,
    #ff0000 100%);
  border-radius: 6px;
}

.hue-pointer {
  position: absolute;
  width: 16px;
  height: 16px;
  background: white;
  border-radius: 50%;
  transform: translate(-8px, -2px);
  box-shadow: 0 1px 3px rgba(0,0,0,0.3);
}

.color-preview {
  width: 40px;
  height: 40px;
  border-radius: 4px;
  border: 1px solid #ddd;
}

.hex-input {
  margin-left: 10px;
  padding: 8px;
  width: 100px;
  border: 1px solid #ddd;
  border-radius: 4px;
}
</style>
```

end