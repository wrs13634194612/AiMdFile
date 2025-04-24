说明：angular九宫格ui
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b3ee31bff018410da3eba9f3012d8090.png#pic_center)

step1:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\order\order.component.ts

```javascript
import { Component } from '@angular/core';
import {NgForOf} from '@angular/common';

interface Order {
  title: string;
  price: string;
  requirementType: string;
  referenceUrl: string;
  options: string[];
  status: string;
  publishDate: string;
  paymentMode: string;
}

@Component({
  selector: 'app-order',
  imports: [
    NgForOf
  ],
  templateUrl: './order.component.html',
  styleUrl: './order.component.css'
})
export class OrderComponent {
  orders: Order[] = [];

  constructor() {
    // 生成测试数据
    for(let i = 1; i <= 8; i++) {
      this.orders.push({
        title: `网站定制开发项目 ${i}`,
        price: `${1200 + i * 100}元`,
        requirementType: '网站定制开发',
        referenceUrl: 'https://papermk.com/案例' + i,
        options: ['整站开发', '响应式', '后台管理', '多语言'],
        status: i % 2 === 0 ? '进行中' : '已结案',
        publishDate: `2024-03-${i.toString().padStart(2, '0')}发布`,
        paymentMode: i % 3 === 0 ? '项目付费' : '招标付费'
      });
    }
  }

  selectedOption: { [key: number]: string } = {};

  selectOption(orderIndex: number, option: string) {
    this.selectedOption[orderIndex] = option;
  }
}

```

step2:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\order\order.component.html

```xml
<div class="grid-container">
  <div *ngFor="let order of orders; let i = index" class="card">
    <!-- Header Section -->
    <div class="header-section">
      <h1 class="title">{{ order.title }}</h1>
      <p class="price">{{ order.price }}</p>
    </div>

    <!-- Reference Section -->
    <div class="reference-section">
      <p class="section-label">需求类型</p>
      <p class="reference-text">{{ order.requirementType }}</p>
      <a class="reference-link" href="{{ order.referenceUrl }}" target="_blank">参考链接</a>
    </div>

    <!-- Options Section -->
    <div class="options-section">
      <button *ngFor="let option of order.options"
              class="option-button"
              [class.selected]="selectedOption[i] === option"
              (click)="selectOption(i, option)">
        {{ option }}
      </button>
    </div>

    <!-- Status Section -->
    <div class="status-section">
      <div class="status-group">
        <span class="status-badge">{{ order.status }}</span>
        <span class="publish-date">{{ order.publishDate }}</span>
        <span class="payment-mode">{{ order.paymentMode }}</span>
      </div>
    </div>
  </div>
</div>

```

step3:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\order\order.component.css

```css
/* Grid布局 */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  max-width: 1200px;
  margin: 20px auto;
  padding: 0 20px;
}

/* Card样式 */
.card {
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.08);
  padding: 20px;
  transition: transform 0.2s, box-shadow 0.2s;
  break-inside: avoid;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.12);
}

/* Header Section */
.header-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid #f0f0f0;
}

.title {
  font-size: 16px;
  line-height: 1.3;
  margin: 0;
  color: #2d3748;
  font-weight: 600;
}

.price {
  font-size: 14px;
  color: #38a169;
  margin: 0;
  font-weight: 500;
}

/* Reference Section */
.reference-section {
  margin-bottom: 16px;
}

.section-label {
  font-size: 12px;
  color: #718096;
  margin: 0 0 4px 0;
}

.reference-text {
  font-size: 14px;
  color: #2d3748;
  margin: 6px 0;
}

.reference-link {
  color: #4299e1;
  font-size: 12px;
  text-decoration: none;
  display: block;
  margin-top: 8px;
  transition: color 0.2s;
}

.reference-link:hover {
  color: #3182ce;
}

/* Options Section */
.options-section {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 8px;
  margin-bottom: 16px;
}

.option-button {
  padding: 8px 12px;
  border: none;
  border-radius: 8px;
  background: #f8f9fa;
  color: #2d3748;
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  font-weight: 500;
}

.option-button:hover {
  background: #e9ecef;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
}

.option-button.selected {
  background: #2d3748;
  color: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Status Section */
.status-section {
  padding-top: 16px;
  border-top: 1px solid #f0f0f0;
}

.status-group {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 8px;
}

.status-badge,
.publish-date,
.payment-mode {
  font-size: 12px;
  padding: 6px 12px;
  border-radius: 16px;
  background: #f8f9fa;
  color: #4a5568;
  white-space: nowrap;
  display: inline-flex;
  align-items: center;
}

.status-badge {
  background: #e9f5eb;
  color: #38a169;
}

/* 响应式布局 */
@media (max-width: 992px) {
  .grid-container {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .grid-container {
    grid-template-columns: 1fr;
  }

  .status-group {
    flex-wrap: wrap;
  }

  .options-section {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 480px) {
  .card {
    padding: 16px;
  }

  .status-badge,
  .publish-date,
  .payment-mode {
    font-size: 11px;
    padding: 4px 8px;
  }
}

```

end