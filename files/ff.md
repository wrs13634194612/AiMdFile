说明：
我计划用angular，实现一个好友列表，仿照微信好友列表的样式，左边是联系人列表，右边是字母导航，可以滑动列表，当滑动左边联系人列表时，需要对应显示右侧的字母导航
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c51efe1f683244e5bed8f14caddbf1c8.png#pic_center)

step1: C:\Users\Administrator\WebstormProjects\untitled4\src\app\contact\contact.component.ts

```javascript
import {Component, OnInit, AfterViewInit, ViewChild, ElementRef, ViewChildren, QueryList } from '@angular/core';
import {NgForOf} from '@angular/common';

interface Contact {
  name: string;
  phone: string;
  avatar: string;
}

@Component({
  selector: 'app-contact',
  imports: [
    NgForOf
  ],
  templateUrl: './contact.component.html',
  styleUrl: './contact.component.css'
})
export class ContactComponent  implements OnInit, AfterViewInit {
  contacts: { [key: string]: Contact[] } = {};
  letters: string[] = [];
  currentLetter: string = 'A';
  private letterPositions: { [key: string]: number } = {};
  private touchStartY: number = 0;
  private itemHeight: number = 48; // 默认高度

  @ViewChild('scrollContainer') scrollContainer!: ElementRef<HTMLElement>;
  @ViewChildren('letterItem') letterItems!: QueryList<ElementRef>;

  mockData: Contact[] = [
    { name: 'Alice', phone: '13800138000', avatar: '👩' },
    { name: 'Bob', phone: '13912345678', avatar: '👨' },
    { name: 'Charlie', phone: '13698765432', avatar: '👨' },
    { name: 'dabing', phone: '13698765432', avatar: '👨' },
    { name: 'gongming', phone: '13698765432', avatar: '👨' },

    { name: 'meiguo', phone: '13800138000', avatar: '👩' },
    { name: 'songguo', phone: '13912345678', avatar: '👨' },
    { name: 'zhaoguo', phone: '13698765432', avatar: '👨' },
    { name: 'iboy', phone: '13800138000', avatar: '👩' },
    { name: 'jack', phone: '13698765432', avatar: '👨' },

    { name: 'kfc', phone: '13912345678', avatar: '👨' },
    { name: 'long', phone: '13912345678', avatar: '👨' },
    { name: 'nan', phone: '13912345678', avatar: '👨' },
    { name: 'oppo', phone: '13698765432', avatar: '👨' },
    { name: 'fuyu', phone: '13800138000', avatar: '👩' },
    { name: 'yelang', phone: '13912345678', avatar: '👨' },
    { name: 'tianqi', phone: '13698765432', avatar: '👨' },
    { name: 'rile', phone: '13698765432', avatar: '👨' },

    { name: 'uyun', phone: '13698765432', avatar: '👨' },
    { name: 'vivo', phone: '13698765432', avatar: '👨' },
    { name: 'wow', phone: '13698765432', avatar: '👨' },


    { name: 'qiming', phone: '13800138000', avatar: '👩' },
    { name: 'eguo', phone: '13912345678', avatar: '👨' },
    { name: 'yingjili', phone: '13698765432', avatar: '👨' },


    { name: 'falanxi', phone: '13698765432', avatar: '👨' },
    { name: 'xibanya', phone: '13800138000', avatar: '👩' },
    { name: 'putaoya', phone: '13912345678', avatar: '👨' },
    { name: 'xiongyali', phone: '13698765432', avatar: '👨' },

    { name: 'saierweiya', phone: '13800138000', avatar: '👩' },
    { name: 'suomali', phone: '13912345678', avatar: '👨' },
    { name: 'aiji', phone: '13698765432', avatar: '👨' },

    { name: 'sudan', phone: '13800138000', avatar: '👩' },
    { name: 'hasake', phone: '13912345678', avatar: '👨' },
    { name: 'yilang', phone: '13698765432', avatar: '👨' },
  ];

  ngOnInit() {
    this.groupContacts();
  }

  ngAfterViewInit(): void {
    this.calculateLetterPositions();
    this.calcItemHeight();
    this.setupScrollListener();
  }

  // 新增滚动监听
  private setupScrollListener() {
    this.scrollContainer.nativeElement.addEventListener('scroll', () => {
      this.updateCurrentLetterByScroll();
    });
  }

  // 根据滚动位置更新当前字母
  private updateCurrentLetterByScroll() {
    const scrollTop = this.scrollContainer.nativeElement.scrollTop;
    const containerHeight = this.scrollContainer.nativeElement.clientHeight;

    // 查找当前可见的第一个完整分组
    const currentGroup = this.letters.find(letter => {
      const position = this.letterPositions[letter];
      return position >= scrollTop && position < scrollTop + containerHeight;
    });

    if (currentGroup && currentGroup !== this.currentLetter) {
      this.currentLetter = currentGroup;
    }
  }

  private groupContacts() {
    this.contacts = this.mockData.reduce((acc, contact) => {
      const initial = contact.name[0].toUpperCase();
      if (!acc[initial]) acc[initial] = [];
      acc[initial].push(contact);
      return acc;
    }, {} as { [key: string]: Contact[] });

    this.letters = Object.keys(this.contacts).sort();
  }

  private calcItemHeight() {
    if (this.letterItems?.first) {
      this.itemHeight = this.letterItems.first.nativeElement.offsetHeight;
    }
  }

  // 优化后的字母位置计算方法
  private calculateLetterPositions() {
    setTimeout(() => {
      let cumulativeHeight = 0;
      this.letters.forEach(letter => {
        const element = document.getElementById(`group-${letter}`);
        if (element) {
          this.letterPositions[letter] = cumulativeHeight;
          cumulativeHeight += element.offsetHeight;
        }
      });
    }, 200); // 增加延迟确保DOM完全渲染
  }


  handleTouchStart(event: TouchEvent) {
    this.touchStartY = event.touches[0].clientY;
    this.updateCurrentLetter(event);
  }

  handleTouchMove(event: TouchEvent) {
    this.updateCurrentLetter(event);
  }

  handleTouchEnd() {
    // 添加触摸结束处理
    this.updateCurrentLetterByScroll();
  }

  // 优化后的触摸处理
  private updateCurrentLetter(event: TouchEvent) {
    const containerRect = this.scrollContainer.nativeElement.getBoundingClientRect();
    const touchY = event.touches[0].clientY - containerRect.top;
    const scrollTop = this.scrollContainer.nativeElement.scrollTop;
    const adjustedTouchY = touchY + scrollTop;  // 考虑当前滚动位置

    // 二分查找优化性能
    let low = 0;
    let high = this.letters.length - 1;
    let targetIndex = 0;

    while (low <= high) {
      const mid = Math.floor((low + high) / 2);
      const midPosition = this.letterPositions[this.letters[mid]];

      if (midPosition < adjustedTouchY) {
        low = mid + 1;
        targetIndex = mid;
      } else {
        high = mid - 1;
      }
    }

    const targetLetter = this.letters[targetIndex];
    if (targetLetter !== this.currentLetter) {
      this.currentLetter = targetLetter;
      this.smoothScrollTo(targetLetter);
    }
  }

  // 优化后的滚动方法
  private smoothScrollTo(letter: string) {
    const position = this.letterPositions[letter];
    if (position !== undefined) {
      requestAnimationFrame(() => {
        this.scrollContainer.nativeElement.scrollTo({
          top: position,
          behavior: 'smooth'
        });
      });
    }
  }

  scrollTo(letter: string) {
    this.currentLetter = letter;
    this.smoothScrollTo(letter);
  }
}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\contact\contact.component.html

```xml
<!-- contact.component.html -->
<div class="contact-container">
  <!-- 主滚动区域 -->
  <div #scrollContainer class="scroll-container">
    <ng-container *ngFor="let letter of letters">
      <!-- 字母分组 -->
      <div [id]="'group-' + letter" class="letter-group">
        <div class="group-title">{{ letter }}</div>
        <!-- 联系人列表 -->
        <div class="contact-item" *ngFor="let contact of contacts[letter]">
          <div class="avatar">{{ contact.avatar }}</div>
          <div class="info">
            <div class="name">{{ contact.name }}</div>
            <div class="phone">{{ contact.phone }}</div>
          </div>
        </div>
      </div>
    </ng-container>
  </div>

  <!-- 右侧字母导航 -->
  <div class="index-bar"
       (touchstart)="handleTouchStart($event)"
       (touchmove)="handleTouchMove($event)"
       (touchend)="handleTouchEnd()">
    <div #letterItem
         *ngFor="let letter of letters"
         class="index-letter"
         [class.active]="letter === currentLetter"
         (click)="scrollTo(letter)">
      {{ letter }}
    </div>
  </div>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\contact\contact.component.css

```css
/* contact.component.css */
.contact-container {
  position: relative;
  height: 100vh;
  overflow: hidden;
  font-family: -apple-system, BlinkMacSystemFont, 'Helvetica Neue', sans-serif;
}

/* 主滚动区域 */
.scroll-container {
  height: 100vh;
  overflow-y: auto;
  padding: 0 16px 20px;
  padding-right: 60px; /* 为右侧导航留出空间 */
  scrollbar-width: none; /* Firefox */
  -ms-overflow-style: none; /* IE/Edge */
  box-sizing: border-box;
}

/* 隐藏滚动条 */
.scroll-container::-webkit-scrollbar {
  display: none; /* Chrome/Safari */
}

/* 右侧字母导航 */
.index-bar {
  position: fixed;
  right: 16px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 9999; /* 最高层级 */
  background: rgba(255, 255, 255, 0.96);
  border-radius: 20px;
  padding: 12px 4px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.12);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px); /* Safari */
}

/* 字母分组标题 */
.group-title {
  padding: 12px 0;
  color: #666;
  font-size: 15px;
  font-weight: 500;
  background: linear-gradient(to bottom, #f8f9fa, #ffffff);
  position: sticky;
  top: 0;
  z-index: 10;
}

/* 联系人卡片 */
.contact-item {
  display: flex;
  align-items: center;
  padding: 14px;
  background: #fff;
  border-radius: 12px;
  margin-bottom: 8px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.04);
  transition: transform 0.2s;
  position: relative;
  z-index: 1; /* 保持内容在底层 */
}

/* 头像样式 */
.avatar {
  width: 42px;
  height: 42px;
  border-radius: 10px;
  background: #f0f2f5;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  margin-right: 16px;
}

/* 信息区域 */
.info {
  flex: 1;
  min-width: 0; /* 防止文本溢出 */
}

.name {
  font-size: 16px;
  color: #1a1a1a;
  font-weight: 500;
  margin-bottom: 4px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.phone {
  font-size: 13px;
  color: #666;
}

/* 字母导航项 */
.index-letter {
  color: #666;
  font-size: 13px;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 4px 0;
  border-radius: 14px;
  cursor: pointer;
  transition: all 0.2s;
  touch-action: none; /* 禁用默认触摸行为 */
}

.index-letter.active {
  color: #ffffff;
  background: #007AFF;
  transform: scale(1.12);
  font-weight: 500;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .scroll-container {
    padding-right: 52px;
  }

  .index-bar {
    right: 12px;
    padding: 8px 2px;
  }

  .index-letter {
    width: 24px;
    height: 24px;
    font-size: 12px;
  }

  .contact-item {
    padding: 12px;
  }

  .avatar {
    width: 38px;
    height: 38px;
    font-size: 18px;
  }
}

@media (max-width: 480px) {
  .scroll-container {
    padding-right: 46px;
  }

  .index-bar {
    right: 8px;
    border-radius: 16px;
    padding: 6px 1px;
  }

  .index-letter {
    width: 22px;
    height: 22px;
    font-size: 11px;
  }

  .group-title {
    font-size: 14px;
  }

  .name {
    font-size: 15px;
  }

  .phone {
    font-size: 12px;
  }
}

```

end