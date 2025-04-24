vue实现二维码生成器和解码器

1.生成基本二维码：根据输入的value生成二维码。
2.可定制尺寸：通过size调整大小。
3.颜色和背景色：设置二维码颜色和背景。
4.静区（quiet zone）支持：通过quietZone调整周围的空白区域。
5.错误纠正级别：ecl参数控制容错级别。
6.渐变效果：启用线性渐变，可以自定义方向和颜色。
7.Logo嵌入：在二维码中间添加logo，可调整大小、边距、背景色和圆角。
8.错误处理：生成失败时显示错误信息。
9.响应式更新：当props变化时自动重新生成二维码。
10.视图框调整：根据静区计算viewBox，确保正确显示。

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6f49b31b09674157ba86f3c3dacf758e.png#pic_center)

step1: 添加依赖 npm install qrcode @types/qrcode


step2:核心组件 实现二维码
C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\QrCode.vue

```typescript
<template>
  <!-- SVG 容器 -->
  <svg
    v-if="!error"
    :viewBox="viewBox"
    :width="size"
    :height="size"
  >
    <!-- 背景 -->
    <rect
      :x="-quietZone"
      :y="-quietZone"
      :width="size + quietZone * 2"
      :height="size + quietZone * 2"
      :fill="backgroundColor"
    />

    <!-- 渐变定义 -->
    <defs v-if="enableLinearGradient">
      <linearGradient
        id="grad"
        :x1="gradientDirection[0]"
        :y1="gradientDirection[1]"
        :x2="gradientDirection[2]"
        :y2="gradientDirection[3]"
      >
        <stop offset="0" :stop-color="linearGradient[0]" stop-opacity="1"/>
        <stop offset="1" :stop-color="linearGradient[1]" stop-opacity="1"/>
      </linearGradient>
    </defs>

    <!-- 二维码路径 -->
    <path
      :d="path"
      stroke-linecap="butt"
      :stroke="enableLinearGradient ? 'url(#grad)' : color"
      :stroke-width="cellSize"
    />

    <!-- Logo 容器 -->
    <svg
      v-if="logo"
      :x="(size - logoSize - logoMargin * 2) / 2"
      :y="(size - logoSize - logoMargin * 2) / 2"
    >
      <!-- Logo 背景 -->
      <rect
        :width="logoSize + logoMargin * 2"
        :height="logoSize + logoMargin * 2"
        :fill="logoBackgroundColor"
        :rx="logoBorderRadius + (logoMargin / logoSize) * logoBorderRadius"
        :ry="logoBorderRadius + (logoMargin / logoSize) * logoBorderRadius"
      />

      <!-- Logo 图片 -->
      <image
        :x="logoMargin"
        :y="logoMargin"
        :width="logoSize"
        :height="logoSize"
        :href="logo"
        preserveAspectRatio="xMidYMid slice"
        :rx="logoBorderRadius"
        :ry="logoBorderRadius"
      />
    </svg>
  </svg>

  <!-- 错误显示 -->
  <div v-if="error" class="error">
    {{ error.message }}
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect, computed } from 'vue'
import QRCode from 'qrcode'

// Props 定义
const props = defineProps({
  value: { type: String, default: 'this is a QR code' },
  size: { type: Number, default: 100 },
  color: { type: String, default: 'black' },
  backgroundColor: { type: String, default: 'white' },
  logo: { type: String, default: undefined },
  logoSize: { type: Number, default: 100 * 0.2 }, // 默认基于 size 计算
  logoBackgroundColor: { type: String, default: 'transparent' },
  logoMargin: { type: Number, default: 2 },
  logoBorderRadius: { type: Number, default: 0 },
  quietZone: { type: Number, default: 0 },
  enableLinearGradient: { type: Boolean, default: false },
  gradientDirection: {
    type: Array as () => string[],
    default: () => ['0%', '0%', '100%', '100%']
  },
  linearGradient: {
    type: Array as () => string[],
    default: () => ['rgb(255,0,0)', 'rgb(0,255,255)']
  },
  ecl: { type: String as () => 'L'|'M'|'Q'|'H', default: 'M' }
})

// 响应式数据
const path = ref('')
const cellSize = ref(0)
const error = ref<Error | null>(null)

// 生成二维码矩阵
const genMatrix = (value: string, errorCorrectionLevel: string): boolean[][] => {
  const arr = Array.prototype.slice.call(
    QRCode.create(value, { errorCorrectionLevel }).modules.data, 0
  )
  const sqrt = Math.sqrt(arr.length)
  return arr.reduce((rows: boolean[][], key: boolean, index: number) => {
    (index % sqrt === 0) ? rows.push([key]) : rows[rows.length - 1].push(key)
    return rows
  }, [])
}

// 转换矩阵为 SVG 路径
const transformMatrixIntoPath = (matrix: boolean[][], size: number) => {
  const cellSize = size / matrix.length
  let path = ''
  matrix.forEach((row, i) => {
    let needDraw = false
    row.forEach((column, j) => {
      if (column) {
        if (!needDraw) {
          path += `M${cellSize * j} ${cellSize / 2 + cellSize * i} `
          needDraw = true
        }
        if (needDraw && j === matrix.length - 1) {
          path += `L${cellSize * (j + 1)} ${cellSize / 2 + cellSize * i} `
        }
      } else {
        if (needDraw) {
          path += `L${cellSize * j} ${cellSize / 2 + cellSize * i} `
          needDraw = false
        }
      }
    })
  })
  return { cellSize, path }
}

// 计算 viewBox
const viewBox = computed(() => [
  -props.quietZone,
  -props.quietZone,
  props.size + props.quietZone * 2,
  props.size + props.quietZone * 2
].join(' '))

// 监听 props 变化
watchEffect(() => {
  try {
    const matrix = genMatrix(props.value, props.ecl)
    const result = transformMatrixIntoPath(matrix, props.size)
    path.value = result.path
    cellSize.value = result.cellSize
    error.value = null
  } catch (err) {
    error.value = err instanceof Error ? err : new Error('QR Code generation failed')
  }
})
</script>

<style scoped>
.error {
  color: red;
  border: 1px solid red;
  padding: 1rem;
}
</style>
```

step3: 调用二维码组件 
C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\ImgCode.vue

```typescript
<template>
  <QrCode
    :value="qrCodeValue"
    :size="200"
    color="#333"
    logo="/logo.png"
    :logo-size="40"
    logo-background-color="white"
    :enable-linear-gradient="true"
  />
</template>

<script setup>
import QrCode from './QrCode.vue'
import { ref } from 'vue'
// 图片资源存放在 C:\Users\wangrusheng\PycharmProjects\untitled3\public\logo.png
// JSON.stringify() 是 JavaScript 中用于将对象或值转换为 JSON 格式字符串的核心方法，
//value="你们好，这里是二维码识别器，很高兴认识你们"
const qrCodeValue = ref(JSON.stringify({
  "data": [
    {
      "case_id": 3,
      "category_id": 4,
      "title": "as",
      "description": "ad",
      "price": 4500.0,
      "client_id": 5,
      "lawyer_id": 6,
      "status": "pending",
      "created_at": "2025-03-21T06:11:59",
      "updated_at": "2025-03-21T06:11:59",
      "category_name": "af"
    },
    {
      "case_id": 8,
      "category_id": 8,
      "title": "bs",
      "description": "bd",
      "price": 6000.0,
      "client_id": 5,
      "lawyer_id": 6,
      "status": "in_progress",
      "created_at": "2025-03-21T06:11:59",
      "updated_at": "2025-03-21T06:11:59",
      "category_name": "bf"
    }
  ]
}))
</script>
```
step4:路由

```bash
C:\Users\wangrusheng\PycharmProjects\untitled3\src\router\index.js
import ImgCode from '../views/ImgCode.vue'
{ path: '/imgcode', name: 'imgcode', component: ImgCode },

C:\Users\wangrusheng\PycharmProjects\untitled3\src\App.vue
<router-link to="/imgcode" active-class="active-link">二维码</router-link>
```
///我是分割线
解码部分
step101:安装依赖 pip install pyzbar Pillow，你去网站上，随便找个二维码生成器，然后将生成的二维码保存本地

step102:解析C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py

```python
from pyzbar.pyzbar import decode
from PIL import Image

# 二维码图片路径 C:\Users\wangrusheng\Downloads
old_file = r'C:\Users\wangrusheng\Downloads\logoz.png'

try:
    # 打开图片文件
    with Image.open(old_file) as img:
        # 解析二维码
        decoded_objects = decode(img)

        if not decoded_objects:
            print("未检测到二维码内容。")
        else:
            for obj in decoded_objects:
                # 解码数据（假设内容为UTF-8文本）
                data = obj.data.decode('utf-8')
                print("解析结果:", data)

except FileNotFoundError:
    print(f"错误：文件 '{old_file}' 不存在。")
except Exception as e:
    print(f"解析时发生错误: {str(e)}")
```

step103:运行

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py 
解析结果: 能不能点点赞  点点关注，我谢谢大家
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> 

```

end