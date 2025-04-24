说明：
用angular实现计算器效果，ui风格为暗黑
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f8b1def7f4c24205ab620aa05c0585be.png#pic_center)

step1: C:\Users\Administrator\WebstormProjects\untitled4\src\app\calnum\calnum.component.ts

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-calnum',
  imports: [],
  templateUrl: './calnum.component.html',
  styleUrl: './calnum.component.css'
})
export class CalnumComponent {

  mainDisplay = '0';
  subDisplay = '';
  currentValue = '';
  operator = '';
  firstOperand: number | null = null;

  handleInput(value: string): void {
    if (value === '.' && this.currentValue.includes('.')) return;

    this.currentValue = this.currentValue === '0' ? value : this.currentValue + value;
    this.mainDisplay = this.currentValue;
  }

  setOperator(op: string): void {
    if (this.currentValue === '') return;

    this.operator = op;
    this.firstOperand = parseFloat(this.currentValue);
    this.subDisplay = `${this.currentValue} ${op}`;
    this.currentValue = '';
  }

  calculate(): void {
    if (!this.firstOperand || !this.operator) return;

    const secondOperand = parseFloat(this.currentValue);
    let result: number;

    switch (this.operator) {
      case '+':
        result = this.firstOperand + secondOperand;
        break;
      case '-':
        result = this.firstOperand - secondOperand;
        break;
      case '×':
        result = this.firstOperand * secondOperand;
        break;
      case '÷':
        result = this.firstOperand / secondOperand;
        break;
      default:
        return;
    }

    this.mainDisplay = result.toString();
    this.subDisplay = `${this.firstOperand} ${this.operator} ${secondOperand} =`;
    this.currentValue = result.toString();
    this.firstOperand = null;
    this.operator = '';
  }

  clear(): void {
    this.mainDisplay = '0';
    this.subDisplay = '';
    this.currentValue = '';
    this.firstOperand = null;
    this.operator = '';
  }

}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\calnum\calnum.component.html

```xml
<div class="calculator">
  <div class="display">
    <div class="sub-display">{{ subDisplay }}</div>
    <div class="main-display">{{ mainDisplay }}</div>
  </div>

  <div class="buttons">
    <button (click)="clear()">AC</button>
    <button (click)="handleInput('7')">7</button>
    <button (click)="handleInput('8')">8</button>
    <button (click)="handleInput('9')">9</button>
    <button class="operator" (click)="setOperator('÷')">÷</button>

    <button (click)="handleInput('4')">4</button>
    <button (click)="handleInput('5')">5</button>
    <button (click)="handleInput('6')">6</button>
    <button class="operator" (click)="setOperator('×')">×</button>

    <button (click)="handleInput('1')">1</button>
    <button (click)="handleInput('2')">2</button>
    <button (click)="handleInput('3')">3</button>
    <button class="operator" (click)="setOperator('-')">-</button>

    <button (click)="handleInput('0')">0</button>
    <button (click)="handleInput('.')">.</button>
    <button (click)="calculate()">=</button>
    <button class="operator" (click)="setOperator('+')">+</button>
  </div>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\calnum\calnum.component.css

```css
/* calculator.component.css */
.calculator {
  background: #1a1a1a;
  border-radius: 16px;
  padding: 24px;
  width: 320px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
  border: 1px solid #333;
  margin: 20px auto;
}

.display {
  background: #000;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 20px;
  text-align: right;
  border: 1px solid #333;
}

.sub-display {
  color: #666;
  font-size: 14px;
  min-height: 20px;
  margin-bottom: 8px;
  font-family: 'Courier New', monospace;
}

.main-display {
  color: #fff;
  font-size: 36px;
  font-weight: 300;
  letter-spacing: -1px;
  font-family: 'Segoe UI', sans-serif;
}

.buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}

button {
  border: none;
  border-radius: 8px;
  padding: 18px;
  font-size: 20px;
  cursor: pointer;
  transition: all 0.15s ease;
  background: #2d2d2d;
  color: #fff;
  font-weight: 500;
}

button:hover {
  background: #3d3d3d;
  transform: translateY(-1px);
}

button:active {
  transform: translateY(1px);
}

button.operator {
  background: #ff9500;
  color: #fff;
}

button.operator:hover {
  background: #ffaa33;
}

button:nth-child(1) { /* AC 按钮 */
  background: #ff3b30;
  color: #fff;
}

button:nth-child(1):hover {
  background: #ff453a;
}

button:nth-child(19) { /* = 按钮 */
  background: #34c759;
  color: #fff;
  grid-column: span 2;
}

button:nth-child(19):hover {
  background: #30d158;
}

/* 数字按钮特殊效果 */
button:not(.operator):not(:first-child):not(:last-child) {
  background: #262626;
}

/* 按钮文字阴影 */
button {
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.3);
}

/* 输入动画 */
button {
  transition:
    background 0.2s ease,
    transform 0.1s cubic-bezier(0.4, 0, 0.2, 1),
    box-shadow 0.2s ease;
}

/* 键盘聚焦效果 */
button:focus-visible {
  outline: 2px solid #007AFF;
  outline-offset: 2px;
}

/* 显示区域渐变效果 */
.display {
  background: linear-gradient(145deg, #0a0a0a, #000);
}

/* 操作符按钮激活状态 */
button.operator.active {
  background: #ffaa33;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.2);
}

```

end