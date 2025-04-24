说明：使用angular技术，material控件，ngfor循环，img网络图片展示，分页组件

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5ad5b1a6ce9e4005a5f16ca628df2095.png#pic_center)

step1: C:\Users\Administrator\WebstormProjects\untitled4\src\app\home\home.component.ts

```javascript
import { Component, ViewChild, AfterViewInit } from '@angular/core';
import {MatListModule} from '@angular/material/list';
import {NgForOf, SlicePipe} from '@angular/common';
import {MatPaginator} from '@angular/material/paginator';

// 定义列表项的数据结构
interface ListItem {
  id: number;
  title: string;
  description: string;
  avatarUrl: string;
}


@Component({
  selector: 'app-home',
  imports: [MatListModule, NgForOf, MatPaginator, SlicePipe],
  templateUrl: './home.component.html',
  styleUrl: './home.component.css'
})
export class HomeComponent implements AfterViewInit {
  // 列表项数据数组
  listItems: ListItem[] = [
    {
      id: 1,
      title: 'Item 1',
      description: 'Item 1 description goes here',
      avatarUrl: 'https://randomuser.me/api/portraits/men/1.jpg'
    },
    {
      id: 2,
      title: 'Item 2',
      description: 'Item 2 description goes here',
      avatarUrl: 'https://randomuser.me/api/portraits/men/2.jpg'
    },
    {
      id: 3,
      title: 'Item 3',
      description: 'Item 3 description goes here',
      avatarUrl: 'https://randomuser.me/api/portraits/men/3.jpg'
    },
    {
      id: 4,
      title: 'Item 4',
      description: 'Item 4 description goes here',
      avatarUrl: 'https://randomuser.me/api/portraits/men/4.jpg'
    },
    {
      id: 5,
      title: 'Item 5',
      description: 'Item 5 description goes here',
      avatarUrl: 'https://randomuser.me/api/portraits/men/5.jpg'
    },
    {
      id: 6,
      title: 'Item 6',
      description: 'Item 6 description goes here',
      avatarUrl: 'https://randomuser.me/api/portraits/men/6.jpg'
    }
  ];

  // 分页相关变量
  @ViewChild(MatPaginator) paginator!: MatPaginator;
  pageSize = 2; // 每页显示的条数
  currentPage = 0; // 当前页码

  constructor() { }

  // 处理点击事件
  handleClick(event: Event, itemId: number): void {
    console.log(`Clicked on item ${itemId}`);
    // 在这里可以添加其他逻辑，比如导航到详情页面
  }

  ngAfterViewInit(): void {
    // 将数据源连接到分页器
    this.paginator.length = this.listItems.length;
    this.paginator.pageSize = this.pageSize;
  }

  // 处理分页事件
  handlePageEvent(event: any): void {
    this.pageSize = event.pageSize;
    this.currentPage = event.pageIndex;
    console.log(`Page size: ${this.pageSize}, Page index: ${this.currentPage}`);
  }
}


```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\home\home.component.html

```xml
<mat-list role="list">
  <mat-list-item role="listitem" *ngFor="let item of listItems | slice: currentPage * pageSize : (currentPage + 1) * pageSize">
    <div class="item-container" (click)="handleClick($event, item.id)">
      <img
        [src]="item.avatarUrl"
        [alt]="item.title"
        class="item-avatar"
      />
      <div class="item-content">
        <h3 class="item-title">{{ item.title }}</h3>
        <p class="item-description">{{ item.description }}</p>
      </div>
    </div>
  </mat-list-item>
</mat-list>

<!-- 分页组件 -->
<mat-paginator
  [pageSize]="pageSize"
  [pageSizeOptions]="[2, 4, 6]"
  (page)="handlePageEvent($event)"
  [length]="listItems.length"
  [hidePageSize]="false"
  showFirstLastButtons="true">
</mat-paginator>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\home\home.component.css

```css
/* 在你的 CSS 文件中添加以下样式 */
.custom-list {
  background-color: #f8f9fa; /* 列表背景色 */
  padding: 20px; /* 列表的内边距 */
}

.list-item {
  background-color: #ffffff; /* 列表项背景色 */
  margin-bottom: 10px; /* 列表项之间的间距 */
  border-radius: 8px; /* 列表项的圆角 */
  box-shadow: 0 2px 4px rgba(0,0,0,0.1); /* 添加阴影效果 */
  transition: transform 0.3s ease; /* 添加悬停动画 */
}

.list-item:hover {
  transform: translateX(5px); /* 悬停时向右移动5px */
  box-shadow: 0 4px 6px rgba(0,0,0,0.15); /* 悬停时增加阴影 */
}

.item-container {
  padding: 15px; /* 内容的内边距 */
  display: flex;
  align-items: center; /* 垂直居中 */
}

.item-avatar {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: #e0e0e0;
  object-fit: cover;
  margin-right: 15px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.15);
  transition: transform 0.3s ease;
}

.item-avatar:hover {
  transform: scale(1.05);
}


.item-content {
  flex: 1; /* 文本内容占满剩余空间 */
}

.item-title {
  color: #2c3e50; /* 标题颜色 */
  margin: 0; /* 移除标题的默认外边距 */
  font-size: 18px; /* 标题字体大小 */
  font-weight: 600; /* 标题加粗 */
}

.item-description {
  color: #7f8c8d; /* 描述文字颜色 */
  margin: 5px 0 0 0; /* 描述文字的外边距 */
  font-size: 14px; /* 描述文字大小 */
  line-height: 1.5; /* 行高 */
}

```

end