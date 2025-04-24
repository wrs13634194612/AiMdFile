说明：
用angular实现房贷计算器效果，优化布局样式，等额本金和等额本息两种还款方式，且有每月还款明细列表table
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5debefccbb704c8591bcc9a140aef6f8.png#pic_center)

step1:C:\Users\Administrator\WebstormProjects\untitled4\src\app\calculator\calculator.component.ts

```javascript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CurrencyPipe, NgFor, NgIf } from '@angular/common';

interface LoanResult {
  monthlyPayment?: number;
  firstMonthPayment?: number; // 等额本金首月还款
  monthlyDecrease?: number;   // 每月递减金额
  totalPayment: number;
  totalInterest: number;
}

interface Installment {
  period: number;
  payment: number;
  principal: number;
  interest: number;
  remaining: number;
}

@Component({
  selector: 'app-calculator',
  imports: [
    FormsModule,
    CurrencyPipe,
    NgIf,
    NgFor
  ],
  templateUrl: './calculator.component.html',
  styleUrls: ['./calculator.component.css']
})
export class CalculatorComponent {
  // 新增还款方式选择
  repaymentMethod: 'equal-installment' | 'equal-principal' = 'equal-installment';

  // 新增还款计划明细
  installments: Installment[] = [];

  // 原有属性保持不变
  loanType: 'commercial' | 'fund' = 'commercial';
  loanAmount = 100;
  loanYears = 20;
  customRate?: number;
  useCustomRate = false;
  result?: LoanResult;

  private get defaultRate() {
    const isShortTerm = this.loanYears <= 5;
    return this.loanType === 'commercial'
      ? (isShortTerm ? 4.75 : 4.90)
      : (isShortTerm ? 2.75 : 3.25);
  }

  updateRate() {
    if (!this.useCustomRate) {
      this.customRate = this.defaultRate;
    }
  }

  toggleRate() {
    this.useCustomRate = !this.useCustomRate;
    if (!this.useCustomRate) this.customRate = this.defaultRate;
  }

  calculate() {
    if (this.repaymentMethod === 'equal-installment') {
      this.calculateEqualInstallment();
    } else {
      this.calculateEqualPrincipal();
    }
  }

  private calculateEqualInstallment() {
    const rate = (this.customRate ?? this.defaultRate) / 100;
    const months = this.loanYears * 12;
    const monthlyRate = rate / 12;
    const principal = this.loanAmount * 10000;

    const factor = Math.pow(1 + monthlyRate, months);
    const monthlyPayment = principal * monthlyRate * factor / (factor - 1);

    this.result = {
      monthlyPayment: Number(monthlyPayment.toFixed(2)),
      totalPayment: Number((monthlyPayment * months).toFixed(2)),
      totalInterest: Number((monthlyPayment * months - principal).toFixed(2))
    };

    this.generateInstallmentPlan(principal, monthlyRate, monthlyPayment);
  }

  private calculateEqualPrincipal() {
    const rate = (this.customRate ?? this.defaultRate) / 100;
    const months = this.loanYears * 12;
    const monthlyRate = rate / 12;
    const principal = this.loanAmount * 10000;

    // 每月偿还本金
    const monthlyPrincipal = principal / months;
    let remaining = principal;
    let totalInterest = 0;
    const installments: Installment[] = [];

    for (let i = 1; i <= months; i++) {
      const interest = remaining * monthlyRate;
      const payment = monthlyPrincipal + interest;
      remaining -= monthlyPrincipal;
      totalInterest += interest;

      installments.push({
        period: i,
        payment: Number(payment.toFixed(2)),
        principal: Number(monthlyPrincipal.toFixed(2)),
        interest: Number(interest.toFixed(2)),
        remaining: Number(remaining > 0 ? remaining.toFixed(2) : 0)
      });
    }

    this.result = {
      firstMonthPayment: installments[0].payment,
      monthlyDecrease: Number((installments[0].payment - installments[1].payment).toFixed(2)),
      totalPayment: Number((principal + totalInterest).toFixed(2)),
      totalInterest: Number(totalInterest.toFixed(2))
    };

    this.installments = installments;
  }

  private generateInstallmentPlan(principal: number, monthlyRate: number, monthlyPayment: number) {
    let remaining = principal;
    const installments: Installment[] = [];

    for (let i = 1; i <= this.loanYears * 12; i++) {
      const interest = remaining * monthlyRate;
      const principalPaid = monthlyPayment - interest;
      remaining -= principalPaid;

      installments.push({
        period: i,
        payment: Number(monthlyPayment.toFixed(2)),
        principal: Number(principalPaid.toFixed(2)),
        interest: Number(interest.toFixed(2)),
        remaining: Number(remaining > 0 ? remaining.toFixed(2) : 0)
      });
    }

    this.installments = installments;
  }
}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\calculator\calculator.component.html

```xml
<!-- calculator.component.html -->
<div class="calculator-container">
  <div class="calculator-card">
    <!-- 输入表单区域 -->
    <div class="input-section">
      <h2 class="calculator-title">贷款计算器</h2>

      <div class="form-grid">
        <!-- 原有表单控件保持不变 -->
        <!-- 新增贷款金额输入 -->
        <div class="form-group">
          <label class="form-label">贷款金额（万元）</label>
          <input
            class="form-input"
            type="number"
            [(ngModel)]="loanAmount"
            min="1"
            step="1"
            required
          >
        </div>

        <!-- 新增贷款期限输入 -->
        <div class="form-group">
          <label class="form-label">贷款期限（年）</label>
          <input
            class="form-input"
            type="number"
            [(ngModel)]="loanYears"
            (change)="updateRate()"
            min="1"
            max="30"
            step="1"
            required
          >
        </div>

        <!-- 原有利率输入改造 -->
        <div class="form-group">
          <label class="form-label">年利率（%）</label>
          <div class="rate-input-group">
            <input
              class="form-input"
              type="number"
              [(ngModel)]="customRate"
              [disabled]="!useCustomRate"
              step="0.01"
              min="1"
              max="20"
            >
            <button
              class="toggle-rate-btn"
              (click)="toggleRate()"
              [class.active]="useCustomRate"
            >
              {{useCustomRate ? '恢复默认' : '自定义'}}
            </button>
          </div>
        </div>
        <div class="form-group">
          <label class="form-label">还款方式：</label>
          <select class="form-select" [(ngModel)]="repaymentMethod">
            <option value="equal-installment">等额本息</option>
            <option value="equal-principal">等额本金</option>
          </select>
        </div>
      </div>

      <button class="calculate-btn" (click)="calculate()">开始计算</button>
    </div>

    <!-- 计算结果展示 -->
    <div *ngIf="result" class="result-section">
      <div class="result-card">
        <h3 class="result-title">计算结果</h3>
        <div class="result-grid">
          <div *ngIf="repaymentMethod === 'equal-installment'" class="result-item">
            <span class="result-label">每月还款</span>
            <span class="result-value highlight">{{ result.monthlyPayment | currency:'CNY':'symbol':'1.2-2' }}</span>
          </div>
          <div *ngIf="repaymentMethod === 'equal-principal'" class="result-item">
            <div class="result-item">
              <span class="result-label">首月还款</span>
              <span class="result-value highlight">{{ result.firstMonthPayment | currency:'CNY':'symbol':'1.2-2' }}</span>
            </div>
            <div class="result-item">
              <span class="result-label">每月递减</span>
              <span class="result-value highlight">{{ result.monthlyDecrease | currency:'CNY':'symbol':'1.2-2' }}</span>
            </div>
          </div>
          <div class="result-item">
            <span class="result-label">总还款额</span>
            <span class="result-value">{{ result.totalPayment | currency:'CNY':'symbol':'1.2-2' }}</span>
          </div>
          <div class="result-item">
            <span class="result-label">支付利息</span>
            <span class="result-value accent">{{ result.totalInterest | currency:'CNY':'symbol':'1.2-2' }}</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 还款计划表 -->
    <div *ngIf="installments.length" class="table-section">
      <h3 class="table-title">还款计划明细</h3>
      <div class="table-container">
        <table class="installment-table">
          <thead>
          <tr>
            <th class="period">期数</th>
            <th>月供</th>
            <th>本金</th>
            <th>利息</th>
            <th class="remaining">剩余本金</th>
          </tr>
          </thead>
          <tbody>
          <tr *ngFor="let item of installments; let even = even"
              [class.even-row]="even"
              class="data-row">
            <td class="period">{{ item.period }}</td>
            <td>{{ item.payment | currency:'CNY':'symbol':'1.2-2' }}</td>
            <td>{{ item.principal | currency:'CNY':'symbol':'1.2-2' }}</td>
            <td>{{ item.interest | currency:'CNY':'symbol':'1.2-2' }}</td>
            <td class="remaining">{{ item.remaining | currency:'CNY':'symbol':'1.2-2' }}</td>
          </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\calculator\calculator.component.css

```css
/* calculator.component.css */
/* 基础样式重置 */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* 容器样式 */
.calculator-container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.calculator-card {
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
  overflow: hidden;
}

/* 输入区域样式 */
.input-section {
  padding: 2rem;
  background: linear-gradient(135deg, #f8f9ff 0%, #f1f4ff 100%);
  border-bottom: 1px solid #e0e7ff;
}

.calculator-title {
  color: #2d3954;
  font-size: 1.8rem;
  margin-bottom: 2rem;
  font-weight: 600;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.form-group {
  margin-bottom: 1rem;
}

.form-label {
  display: block;
  color: #4a5568;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-select, .form-input {
  width: 100%;
  padding: 0.75rem 1rem;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.form-select:focus, .form-input:focus {
  border-color: #6366f1;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
}



.rate-input-group {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.toggle-rate-btn {
  padding: 0.5rem 1rem;
  border: 2px solid #e0e7ff;
  border-radius: 8px;
  background: white;
  color: #4f46e5;
  font-weight: 500;
  transition: all 0.3s ease;
}

.toggle-rate-btn.active {
  background: #6366f1;
  color: white;
  border-color: #6366f1;
}

/* 按钮样式 */
.calculate-btn {
  background: #6366f1;
  color: white;
  padding: 1rem 2rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  width: 100%;
  max-width: 300px;
  display: block;
  margin: 1.5rem auto 0;
}

.calculate-btn:hover {
  background: #4f46e5;
  transform: translateY(-1px);
  box-shadow: 0 5px 15px rgba(99, 102, 241, 0.3);
}

/* 结果区域样式 */
.result-section {
  padding: 2rem;
}

.result-card {
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  padding: 1.5rem;
  margin-bottom: 2rem;
}

.result-title {
  color: #2d3954;
  font-size: 1.4rem;
  margin-bottom: 1.5rem;
}

.result-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
}

.result-item {
  background: #f8f9ff;
  padding: 1rem;
  border-radius: 8px;
  text-align: center;
}

.result-label {
  display: block;
  color: #64748b;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.result-value {
  font-size: 1.2rem;
  font-weight: 600;
  color: #2d3954;
}

.highlight {
  color: #6366f1;
  font-size: 1.4rem;
}

.accent {
  color: #ef4444;
}

/* 表格样式 */
.table-section {
  padding: 0 2rem 2rem;
}

.table-title {
  color: #2d3954;
  font-size: 1.4rem;
  margin-bottom: 1.5rem;
}

.table-container {
  border: 1px solid #e0e7ff;
  border-radius: 12px;
  overflow-x: auto;
}

.installment-table {
  width: 100%;
  border-collapse: collapse;
  min-width: 800px;
}

.installment-table th {
  background: #f8f9ff;
  color: #4a5568;
  font-weight: 600;
  padding: 1rem;
  text-align: center;
  border-bottom: 2px solid #e0e7ff;
}

.installment-table td {
  padding: 1rem;
  text-align: center;
  color: #4a5568;
  border-bottom: 1px solid #f1f5f9;
}

.period, .remaining {
  font-weight: 500;
}

.even-row {
  background-color: #f8fafc;
}

.data-row:hover {
  background-color: #f1f5f9;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .calculator-container {
    padding: 0;
  }

  .calculator-card {
    border-radius: 0;
  }

  .input-section {
    padding: 1.5rem;
  }

  .result-grid {
    grid-template-columns: 1fr;
  }

  .installment-table th,
  .installment-table td {
    padding: 0.75rem;
    font-size: 0.9rem;
  }
}

```

end