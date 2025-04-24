说明：我计划用angular实现舒尔特方格的功能，必须是动态的，比如3*3，5*5，9*9，而且无论是什么样式的，都必须保持正方形，然后还有时间监听，计算用户完成方格的时间，

另外，我还需要优化ui样式，让它变得好看，增加背景，圆角和悬停效果，还可以加一点小动画，让它看起来更加美观，下面给出完整代码
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3a72f0ef1e3f489a88df633639324d2c.png#pic_center)

step1: C:\Users\Administrator\WebstormProjects\untitled4\src\app\schulte\schulte.component.ts

```javascript
import { Component ,OnDestroy,OnInit} from '@angular/core';
import {NgForOf, NgIf} from '@angular/common';

@Component({
  selector: 'app-schulte',
  imports: [NgForOf, NgIf],
  templateUrl: './schulte.component.html',
  styleUrl: './schulte.component.css'
})
export class SchulteComponent  implements OnDestroy,OnInit{

  gridSize = 4;
  numbers: number[] = [];
  currentNumber = 1;
  timeUsed = 0;
  isStarted = false;
  private timer: number | undefined;
  gridStyle = {};

  ngOnInit() {
    this.initializeGame();
    this.updateGridStyle();
  }

  private initializeGame() {
    const size =this.gridSize ** 2;
    this.numbers = Array.from({ length: size }, (_, i) => i + 1);

    // Fisher-Yates shuffle
    for (let i = this.numbers.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [this.numbers[i], this.numbers[j]] = [this.numbers[j], this.numbers[i]];
    }
  }

  updateGridStyle() {
    const cellSizePercentage = 100 / this.gridSize;
    const fontSize = Math.min(4 / this.gridSize, 1.2);

    this.gridStyle = {
      'gridTemplateColumns': `repeat(${this.gridSize}, 1fr)`,
      'fontSize': `${fontSize}rem`
    };
  }

  startGame() {
    this.clearTimer();
    this.isStarted = true;
    this.currentNumber = 1;
    this.timeUsed = 0;

    this.timer = window.setInterval(() => {
      this.timeUsed++;
    }, 1000);
  }

  handleClick(num: number) {
    if (!this.isStarted) return;

    if (num === this.currentNumber) {
      this.currentNumber++;
      if (this.currentNumber > this.gridSize ** 2) {
        this.gameComplete();
      }
    } else {
      this.handleError();
    }
  }

  private gameComplete() {
    this.clearTimer();
    alert(`🎉 完成时间: ${this.timeUsed}秒`);
    this.isStarted = false;
    this.initializeGame();
  }

  private handleError() {
    this.clearTimer();
    this.isStarted = false;
    alert('❌ 点错数字，重新开始!');
    this.initializeGame();
  }

  private clearTimer() {
    if (this.timer) {
      clearInterval(this.timer);
      this.timer = undefined;
    }
  }

  ngOnDestroy() {
    this.clearTimer();
  }


}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\schulte\schulte.component.html

```xml
<div class="game-container">
  <button *ngIf="!isStarted"
          class="start-btn"
          (click)="startGame()">
    🚀 开始挑战
  </button>

  <div class="status-box">
    <div class="status-item">
      <span class="label">当前数字</span>
      <span class="value">{{ currentNumber }}</span>
    </div>
    <div class="status-item">
      <span class="label">用时</span>
      <span class="value">{{ timeUsed }}s</span>
    </div>
  </div>

  <div class="grid" [style]="gridStyle">
    <button
      *ngFor="let num of numbers"
      (click)="handleClick(num)"
      [disabled]="!isStarted"
      class="grid-btn"
      [class.active]="num === currentNumber"
    >
      {{ num }}
    </button>
  </div>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\schulte\schulte.component.css

```css
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  padding: 2rem;
  background: #f8f9fa;
  border-radius: 1.5rem;
  box-shadow: 0 8px 30px rgba(0,0,0,0.1);
  max-width: 95vw;
  margin: 2rem auto;
}

.start-btn {
  padding: 1rem 2.5rem;
  font-size: 1.2rem;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  color: white;
  border: none;
  border-radius: 0.75rem;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.start-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(99, 102, 241, 0.3);
}

.status-box {
  display: flex;
  gap: 2rem;
  padding: 0.8rem 1.5rem;
  background: white;
  border-radius: 0.75rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}

.status-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.label {
  font-size: 0.9rem;
  color: #6b7280;
}

.value {
  font-size: 1.2rem;
  font-weight: 600;
  color: #1f2937;
}

.grid {
  display: grid;
  gap: 0.5rem;
  width: 80vmin;
  height: 80vmin;
  max-width: 600px;
  max-height: 600px;
  padding: 1rem;
  background: white;
  border-radius: 1rem;
  box-shadow: inset 0 2px 8px rgba(0,0,0,0.05);
}

.grid-btn {
  aspect-ratio: 1;
  border: none;
  border-radius: 0.5rem;
  background: linear-gradient(145deg, #f3f4f6, #e5e7eb);
  color: #1f2937;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.grid-btn:hover:not(:disabled) {
  background: linear-gradient(145deg, #e5e7eb, #d1d5db);
  transform: scale(1.03);
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

.grid-btn.active {
  background: linear-gradient(145deg, #c7d2fe, #a5b4fc);
  color: #4338ca;
  transform: scale(0.98);
}

.grid-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

```

end