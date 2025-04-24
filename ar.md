vue实现在线进制转换

主要功能包括：

1.支持2-36进制之间的转换。
2.支持整数和浮点数的转换。
3.输入验证（虽然可能存在不严格的情况）。
4.错误提示。
5.结果展示，包括大写字母。
6.用户友好的界面，包括下拉菜单、输入框、按钮和结果区域。
7.小数部分处理，限制精度为10位。
8.即时转换（通过按钮触发，而非实时响应）。

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9928927d00884261b2347188c7c92148.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled18\src\views\Home.vue

```typescript
<template>
  <div class="converter-container">
    <h1>在线进制转换</h1>
    <p class="description">支持在2~36进制之间进行任意转换，支持浮点型</p>

    <div class="converter-wrapper">
      <div class="converter-row">
        <div class="select-group">
          <select v-model="fromBase" class="base-select">
            <option v-for="n in bases" :value="n">{{ n }}进制</option>
          </select>
        </div>
        <div class="input-group">
          <input
            type="text"
            v-model="inputNumber"
            placeholder="转换数字"
            class="number-input"
          >
        </div>
      </div>

      <div class="converter-row">
        <div class="select-group">
          <select v-model="toBase" class="base-select">
            <option v-for="n in bases" :value="n">{{ n }}进制</option>
          </select>
        </div>
        <div class="result-group">
          <div class="result-display">{{ result }}</div>
        </div>
      </div>
    </div>

    <button @click="convert" class="convert-btn">立即转换</button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const fromBase = ref(16)
const toBase = ref(10)
const inputNumber = ref('3c')
const result = ref('')
const bases = Array.from({ length: 35 }, (_, i) => i + 2); // 生成 2 到 36 的进制数组

const convert = () => {
  try {
    // Handle empty input
    if (!inputNumber.value) {
      result.value = '';
      return;
    }

    // Check if the input number is valid for the selected base
    const isValid = /^[0-9a-z.]+$/i.test(inputNumber.value);
    if (!isValid) {
      result.value = '输入包含无效字符';
      return;
    }

    // Separate integer and fractional parts
    const [integerPartStr, fractionalPartStr = ''] = inputNumber.value.split('.');

    // Convert integer part
    const integerPartDecimal = parseInt(integerPartStr, fromBase.value);
    if (isNaN(integerPartDecimal)) {
      result.value = '无效的输入数字';
      return;
    }
    const integerPartResult = integerPartDecimal.toString(toBase.value).toUpperCase();

    // Convert fractional part if it exists
    let fractionalPartResult = '';
    if (fractionalPartStr) {
      let decimalFraction = 0;
      for (let i = 0; i < fractionalPartStr.length; i++) {
        const digit = parseInt(fractionalPartStr[i], fromBase.value);
        if (isNaN(digit) || digit >= fromBase.value) {
          result.value = '无效的小数部分';
          return;
        }
        decimalFraction += digit * Math.pow(fromBase.value, -(i + 1));
      }

      let tempFractionalResult = '';
      let tempDecimal = decimalFraction;
      for (let i = 0; i < 10; i++) { // Limit precision to 10 digits
        tempDecimal *= toBase.value;
        const integerPart = Math.floor(tempDecimal);
        tempFractionalResult += integerPart.toString(toBase.value).toUpperCase();
        tempDecimal -= integerPart;
        if (tempDecimal === 0) {
          break;
        }
      }
      fractionalPartResult = '.' + tempFractionalResult;
    }

    result.value = integerPartResult + fractionalPartResult;

  } catch (error) {
    result.value = '转换出错';
    console.error("Conversion error:", error);
  }
}
</script>

<style scoped>
.converter-container {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

h1 {
  text-align: center;
  color: #333;
  margin-bottom: 10px;
}

.description {
  text-align: center;
  color: #666;
  margin-bottom: 30px;
}

.converter-wrapper {
  margin: 20px 0;
}

.converter-row {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.select-group, .input-group, .result-group {
  flex: 1;
}

.base-select, .number-input {
  width: 100%;
  padding: 12px;
  border: 1px solid #fff;
  border-radius: 4px;
  font-size: 16px;
}

.result-display {
  padding: 12px;
  background: #f8f9fa;
  border: 1px solid #eee;
  border-radius: 4px;
  min-height: 46px;
}

.convert-btn {
  width: 100%;
  padding: 12px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background 0.3s;
}

.convert-btn:hover {
  background: #0056b3;
}
</style>
```

end