说明：我计划用angular做一款打地鼠的小游戏，

# 打地鼠游戏实现文档

## 🎮 游戏逻辑
- **游戏场景**  
  采用 `3x3` 网格布局的 9 个地鼠洞
- **核心机制**  
  - 地鼠随机从洞口弹出
  - 点击有效目标获得积分
  - 30 秒倒计时游戏模式
- **难度系统**
  - `简单模式`：生成间隔 1.5s / 停留 1s
  - `普通模式`：生成间隔 1s / 停留 0.8s  
  - `困难模式`：生成间隔 0.8s / 停留 0.6s

## ⚙️ 主要功能
- **游戏控制**
  - 开始/结束游戏按钮
  - 游戏进行中禁止重复启动
- **状态显示**
  - 实时分数更新
  - 动态倒计时显示
- **交互系统**
  - 可视化难度选择按钮
  - 点击命中即时反馈
- **动画效果**
  - 地鼠弹出/收回动画
  - 按钮激活状态指示

## 🔧 实现细节
### 效果图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/dc5545574f8b4a75b440093d90ff243f.png#pic_center)

 ### 技术实现 

step1:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\mouse\mouse.component.ts

```javascript
import { Component } from '@angular/core';
import {NgForOf, NgIf, TitleCasePipe} from '@angular/common';

@Component({
  selector: 'app-mouse',
  imports: [
    NgForOf,
    NgIf,
    TitleCasePipe
  ],
  templateUrl: './mouse.component.html',
  styleUrl: './mouse.component.css'
})
export class MouseComponent {
  score = 0;
  timeLeft = 30;
  isPlaying = false;
  holes = Array(9).fill(false);
  private gameTimer: any;
  private moleTimer: any;
  // 新增类型安全难度列表
  readonly difficultyLevels = ['easy', 'medium', 'hard'] as const;

  difficulty: 'easy' | 'medium' | 'hard' = 'medium';

  // 根据难度调整参数
  get intervalTime() {
    return {
      easy: 1500,
      medium: 1000,
      hard: 800
    }[this.difficulty];
  }

  get activeTime() {
    return {
      easy: 1000,
      medium: 800,
      hard: 600
    }[this.difficulty];
  }

  startGame() {
    if (this.isPlaying) return;

    this.isPlaying = true;
    this.score = 0;
    this.timeLeft = 30;

    // 游戏倒计时
    this.gameTimer = setInterval(() => {
      this.timeLeft--;
      if (this.timeLeft <= 0) {
        this.endGame();
      }
    }, 1000);

    // 地鼠生成
    this.moleTimer = setInterval(() => {
      this.popUpMole();
    }, this.intervalTime);
  }

  endGame() {
    clearInterval(this.gameTimer);
    clearInterval(this.moleTimer);
    this.isPlaying = false;
    this.holes = this.holes.map(() => false);
  }

  popUpMole() {
    if (!this.isPlaying) return;

    const index = Math.floor(Math.random() * 9);
    this.holes[index] = true;

    setTimeout(() => {
      this.holes[index] = false;
    }, this.activeTime);
  }

  whackMole(index: number) {
    if (this.holes[index] && this.isPlaying) {
      this.score += 10;
      this.holes[index] = false;
    }
  }

  setDifficulty(level: 'easy' | 'medium' | 'hard') {
    this.difficulty = level;
  }
}

```

step2:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\mouse\mouse.component.html

```xml
<!-- mouse.component.html -->
<div class="game-card">
  <div class="game-container">
    <div class="game-info">
      <div class="info-group">
        <div class="info-item">
          <span class="label">Score:</span>
          <span class="value">{{ score }}</span>
        </div>
        <div class="info-item">
          <span class="label">Time:</span>
          <span class="value">{{ timeLeft }}</span>
        </div>
      </div>

      <div class="difficulty-group">
        <span class="label">Difficulty:</span>
        <div class="button-group">
          <button
            *ngFor="let diff of difficultyLevels"
            [class.active]="difficulty === diff"
            (click)="setDifficulty(diff)"
            class="difficulty-btn">
            {{ diff | titlecase }}
          </button>
        </div>
      </div>

      <button
        class="start-btn"
        (click)="startGame()"
        [disabled]="isPlaying">
        {{ isPlaying ? 'Playing...' : 'Start Game' }}
      </button>
    </div>

    <div class="game-board">
      <div
        *ngFor="let hole of holes; let i = index"
        class="hole"
        [class.active]="hole"
        (click)="whackMole(i)"
      >
        <div class="mole" *ngIf="hole"></div>
      </div>
    </div>
  </div>
</div>

```

step3:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\mouse\mouse.component.css

```css
/* mouse.component.css */
.game-card {
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.1);
  padding: 24px;
  max-width: 800px;
  margin: 2rem auto;
  transition: transform 0.3s ease;
}

.game-card:hover {
  transform: translateY(-2px);
}

.game-container {
  text-align: center;
}

.game-info {
  margin-bottom: 32px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.info-group {
  display: flex;
  justify-content: center;
  gap: 32px;
  margin-bottom: 16px;
}

.info-item {
  background: #f8f9fa;
  padding: 12px 24px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.label {
  font-weight: 600;
  color: #6c757d;
  margin-right: 8px;
}

.value {
  font-weight: 700;
  color: #495057;
}

.difficulty-group {
  display: flex;
  flex-direction: column;
  gap: 12px;
  align-items: center;
}

.button-group {
  display: flex;
  gap: 12px;
}

button {
  border: none;
  border-radius: 8px;
  padding: 10px 20px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.difficulty-btn {
  background: #e9ecef;
  color: #495057;
  min-width: 90px;
}

.difficulty-btn.active {
  background: #4e66f8;
  color: white;
  box-shadow: 0 4px 6px rgba(78,102,248,0.2);
}

.start-btn {
  background: #20c997;
  color: white;
  padding: 14px 32px;
  font-size: 1.1rem;
  margin-top: 16px;
}

.start-btn:disabled {
  background: #ced4da;
  box-shadow: none;
}

.start-btn:not(:disabled):hover {
  background: #1aa179;
  box-shadow: 0 4px 14px rgba(26,161,121,0.3);
}

.game-board {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 12px;
}

.hole {
  width: 150px;
  height: 150px;
  background: #654321;
  border-radius: 50%;
  position: relative;
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.2s, background 0.3s;
}

.hole:hover {
  transform: scale(1.02);
}

.hole.active {
  background: #8b4513;
  box-shadow: 0 8px 16px rgba(0,0,0,0.2);
}

.mole {
  position: absolute;
  width: 80%;
  height: 80%;
  background: #808080;
  border-radius: 50%;
  bottom: -30%;
  left: 10%;
  transition: bottom 0.3s;
  animation: pop-up 0.3s forwards;
}

@keyframes pop-up {
  from { bottom: -30%; }
  to { bottom: 10%; }
}

```

end