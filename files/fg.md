说明：我计划用angular实现一个轮播图的效果，必须设置图片切换动画，图片自动轮播，设置轮播的间隔时间，然后还有两个按钮 收到控制图片的上一页，下一页的切换
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/036b8fb576394e428181b89fbaccbc7a.png#pic_center)

step1: C:\Users\Administrator\WebstormProjects\untitled4\src\app\banner\banner.component.ts

```javascript
import { Component , Input, OnInit, OnDestroy, OnChanges, SimpleChanges} from '@angular/core';
import { trigger, transition, style, animate, state } from '@angular/animations';
import {NgForOf, NgIf} from '@angular/common';

@Component({
  selector: 'app-banner',
  imports: [
    NgForOf,
    NgIf
  ],
  templateUrl: './banner.component.html',
  styleUrl: './banner.component.css',
  animations: [
    trigger('slide', [
      state('void', style({ opacity: 0 })),
      transition('void => *', [
        style({ transform: 'translateX(100%)', opacity: 0 }),
        animate('600ms cubic-bezier(0.25, 0.8, 0.25, 1)',
          style({ transform: 'translateX(0)', opacity: 1 }))
      ]),
      transition('* => void', [
        animate('600ms cubic-bezier(0.25, 0.8, 0.25, 1)',
          style({ transform: 'translateX(-100%)', opacity: 0 }))
      ])
    ])
  ]
})
export class BannerComponent implements OnInit, OnDestroy, OnChanges  {
  @Input() images: string[] = [];
  currentIndex = 0;
  autoPlayInterval: any;
  isDragging = false;
  startX = 0;
  threshold = 50;
  defaultImages = [
    'https://randomuser.me/api/portraits/men/1.jpg',
    'https://randomuser.me/api/portraits/men/2.jpg',
    'https://randomuser.me/api/portraits/men/3.jpg',
    'https://randomuser.me/api/portraits/men/4.jpg',
    'https://randomuser.me/api/portraits/men/5.jpg',
    'https://randomuser.me/api/portraits/men/6.jpg'
  ];

  ngOnInit() {
    this.initImages();
    this.startAutoPlay();
  }

  ngOnChanges(changes: SimpleChanges) {
    if (changes['images']) {
      this.initImages();
    }
  }

  private initImages() {
    if (this.images.length === 0) {
      this.images = [...this.defaultImages];
    }
  }

  ngOnDestroy() {
    clearInterval(this.autoPlayInterval);
  }

  startAutoPlay() {
    this.autoPlayInterval = setInterval(() => {
      this.next();
    }, 3000);
  }

  next() {
    this.currentIndex = (this.currentIndex + 1) % this.images.length;
  }

  prev() {
    this.currentIndex = (this.currentIndex - 1 + this.images.length) % this.images.length;
  }

  goTo(index: number) {
    this.currentIndex = index;
  }

  pauseAutoPlay() {
    clearInterval(this.autoPlayInterval);
  }

  resumeAutoPlay() {
    this.startAutoPlay();
  }

  onTouchStart(e: TouchEvent) {
    this.startX = e.touches[0].clientX;
    this.isDragging = true;
    this.pauseAutoPlay();
  }

  onTouchMove(e: TouchEvent) {
    if (!this.isDragging) return;
    const currentX = e.touches[0].clientX;
    const diff = this.startX - currentX;

    if (Math.abs(diff) > this.threshold) {
      diff > 0 ? this.next() : this.prev();
      this.isDragging = false;
    }
  }

  onTouchEnd() {
    this.isDragging = false;
    this.resumeAutoPlay();
  }

}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\banner\banner.component.html

```xml
<!-- banner.component.html -->
<div class="carousel-container"
     (mouseenter)="pauseAutoPlay()"
     (mouseleave)="resumeAutoPlay()"
     (touchstart)="onTouchStart($event)"
     (touchmove)="onTouchMove($event)"
     (touchend)="onTouchEnd()">

  <div class="slide-wrapper">
    <ng-container *ngFor="let img of images; let i = index">
      <div class="slide" @slide *ngIf="currentIndex === i"> <!-- 分离条件判断 -->
        <img [src]="img" alt="Slide {{i + 1}}">
      </div>
    </ng-container>
  </div>


  <button class="nav-btn prev-btn" (click)="prev()">
    <svg viewBox="0 0 24 24">
      <path d="M15.41 16.59L10.83 12l4.58-4.59L14 6l-6 6 6 6 1.41-1.41z"/>
    </svg>
  </button>

  <button class="nav-btn next-btn" (click)="next()">
    <svg viewBox="0 0 24 24">
      <path d="M8.59 16.59L13.17 12 8.59 7.41 10 6l6 6-6 6-1.41-1.41z"/>
    </svg>
  </button>

  <div class="indicators">
    <button *ngFor="let img of images; let i = index"
            class="indicator"
            [class.active]="i === currentIndex"
            (click)="goTo(i)">
    </button>
  </div>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\banner\banner.component.css

```css
/* banner.component.css */
.carousel-container {
  position: relative;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.slide-wrapper {
  position: relative;
  width: 100%;
  padding-top: 56.25%; /* 16:9 aspect ratio */
}

.slide {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.slide img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.nav-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 48px;
  height: 48px;
  background: rgba(255, 255, 255, 0.9);
  border: none;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.nav-btn:hover {
  background: white;
  transform: translateY(-50%) scale(1.1);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.nav-btn svg {
  width: 24px;
  height: 24px;
  fill: #2d3748;
}

.prev-btn {
  left: 20px;
}

.next-btn {
  right: 20px;
}

.indicators {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
}

.indicator {
  width: 12px;
  height: 12px;
  border: none;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: all 0.3s ease;
}

.indicator.active {
  background: white;
  width: 24px;
}

.indicator:not(.active):hover {
  background: rgba(255, 255, 255, 0.8);
  transform: scale(1.2);
}

/* 移动端优化 */
@media (max-width: 768px) {
  .nav-btn {
    width: 36px;
    height: 36px;
  }

  .nav-btn svg {
    width: 20px;
    height: 20px;
  }

  .indicators {
    bottom: 10px;
  }

  .indicator {
    width: 8px;
    height: 8px;
  }

  .indicator.active {
    width: 16px;
  }
}

```

end