说明：我计划用angular实现一个贪吃蛇的程序，并且有方向键去控制蛇的上下左右的移动，并且有得分系统，当蛇撞到墙壁或者自身，游戏结束
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b6e9ca1681834aff96ac9ca4c17cd67a.png#pic_center)

step1: C:\Users\Administrator\WebstormProjects\untitled4\src\app\snake\snake.component.ts

```javascript
import { Component , OnInit, OnDestroy, HostListener } from '@angular/core';


interface Position {
  x: number;
  y: number;
}

@Component({
  selector: 'app-snake',
  imports: [],
  templateUrl: './snake.component.html',
  styleUrl: './snake.component.css'
})
export class SnakeComponent implements OnInit, OnDestroy{
  private ctx!: CanvasRenderingContext2D;
  private gameLoop!: any;
  private gridSize = 20;

  snake: Position[] = [{ x: 10, y: 10 }];
  food: Position = this.generateFood();
  direction: 'UP' | 'DOWN' | 'LEFT' | 'RIGHT' = 'RIGHT';
  score = 0;
  gameSpeed = 150;
  isPaused = false;

  @HostListener('window:keydown', ['$event'])
  handleKeydown(event: KeyboardEvent) {
    this.handleDirection(event.key.replace('Arrow', '').toUpperCase());
  }

  ngOnInit() {
    const canvas = document.getElementById('gameCanvas') as HTMLCanvasElement;
    this.ctx = canvas.getContext('2d')!;
    this.startGame();
  }

  ngOnDestroy() {
    clearInterval(this.gameLoop);
  }

  handleDirection(newDirection: string) {
    const validMoves: Record<string, boolean> = {
      'UP': this.direction !== 'DOWN',
      'DOWN': this.direction !== 'UP',
      'LEFT': this.direction !== 'RIGHT',
      'RIGHT': this.direction !== 'LEFT'
    };

    if (validMoves[newDirection]) {
      this.direction = newDirection as any;
    }
  }

  private startGame() {
    this.gameLoop = setInterval(() => this.update(), this.gameSpeed);
  }

  private update() {
    if (this.isPaused) return;

    const head = { ...this.snake[0] };
    switch (this.direction) {
      case 'UP': head.y--; break;
      case 'DOWN': head.y++; break;
      case 'LEFT': head.x--; break;
      case 'RIGHT': head.x++; break;
    }

    if (this.checkCollision(head)) {
      clearInterval(this.gameLoop);
      alert(`游戏结束! 得分: ${this.score}`);
      return;
    }

    this.snake.unshift(head);

    if (head.x === this.food.x && head.y === this.food.y) {
      this.score++;
      this.food = this.generateFood();
      this.gameSpeed = Math.max(50, this.gameSpeed - 2);
    } else {
      this.snake.pop();
    }

    this.draw();
  }

  private draw() {
    // 游戏区域背景
    this.ctx.fillStyle = '#1a1b26';
    this.ctx.fillRect(0, 0, 400, 400);

    // 食物绘制
    this.ctx.fillStyle = '#f7768e';
    this.ctx.beginPath();
    this.ctx.roundRect(
      this.food.x * this.gridSize + 2,
      this.food.y * this.gridSize + 2,
      this.gridSize - 4,
      this.gridSize - 4,
      4
    );
    this.ctx.fill();

    // 蛇身绘制
    this.snake.forEach((segment, index) => {
      this.ctx.fillStyle = index === 0 ? '#9ece6a' : '#73daca';
      this.ctx.beginPath();
      this.ctx.roundRect(
        segment.x * this.gridSize + 2,
        segment.y * this.gridSize + 2,
        this.gridSize - 4,
        this.gridSize - 4,
        index === 0 ? 6 : 4
      );
      this.ctx.fill();
    });
  }

  private generateFood(): Position {
    return {
      x: Math.floor(Math.random() * 20),
      y: Math.floor(Math.random() * 20)
    };
  }

  private checkCollision(pos: Position): boolean {
    return pos.x < 0 || pos.x >= 20 || pos.y < 0 || pos.y >= 20 ||
      this.snake.some(segment => segment.x === pos.x && segment.y === pos.y);
  }

  togglePause() {
    this.isPaused = !this.isPaused;
  }

  resetGame() {
    this.snake = [{ x: 10, y: 10 }];
    this.direction = 'RIGHT';
    this.score = 0;
    this.gameSpeed = 150;
    this.food = this.generateFood();
    this.isPaused = false;
    clearInterval(this.gameLoop);
    this.startGame();
  }
}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\snake\snake.component.html

```xml
<!-- snake.component.html -->
<div class="game-container">
  <div class="game-header">
    <div class="score-box">
      <span>得分: {{ score }}</span>
    </div>
    <div class="controls">
      <button class="control-btn pause-btn" (click)="togglePause()">
        {{ isPaused ? '继续' : '暂停' }}
      </button>
      <button class="control-btn reset-btn" (click)="resetGame()">重置</button>
    </div>
  </div>

  <canvas id="gameCanvas" width="400" height="400"></canvas>

  <div class="direction-controls">
    <div class="control-row">
      <button class="direction-btn up" (click)="handleDirection('UP')">↑</button>
    </div>
    <div class="control-row">
      <button class="direction-btn left" (click)="handleDirection('LEFT')">←</button>
      <button class="direction-btn down" (click)="handleDirection('DOWN')">↓</button>
      <button class="direction-btn right" (click)="handleDirection('RIGHT')">→</button>
    </div>
  </div>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\snake\snake.component.css

```css
/* snake.component.css */
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.5rem;
  padding: 2rem;
  background: #24283b;
  border-radius: 1.5rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

.game-header {
  display: flex;
  justify-content: space-between;
  width: 100%;
  max-width: 400px;
  color: #c0caf5;
}

.score-box {
  background: #414868;
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  font-size: 1.1rem;
}

.controls {
  display: flex;
  gap: 0.5rem;
}

.control-btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 0.5rem;
  cursor: pointer;
  transition: transform 0.2s, opacity 0.2s;
}

.pause-btn {
  background: linear-gradient(135deg, #7aa2f7, #2ac3de);
  color: white;
}

.reset-btn {
  background: linear-gradient(135deg, #f7768e, #ff9e64);
  color: white;
}

.control-btn:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

.direction-controls {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.control-row {
  display: flex;
  justify-content: center;
  gap: 0.5rem;
}

.direction-btn {
  width: 3.5rem;
  height: 3.5rem;
  border: none;
  border-radius: 1rem;
  background: linear-gradient(145deg, #414868, #565f89);
  color: #c0caf5;
  font-size: 1.5rem;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.direction-btn:hover {
  background: linear-gradient(145deg, #565f89, #414868);
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.direction-btn:active {
  transform: scale(0.95);
}

/* 移动端优化 */
@media (max-width: 480px) {
  .game-container {
    padding: 1rem;
    width: 95vw;
  }

  canvas {
    width: 95vw;
    height: 95vw;
  }

  .direction-btn {
    width: 3rem;
    height: 3rem;
    font-size: 1.2rem;
  }
}

```

end