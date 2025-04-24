说明：
vue和angular实现飞机大战
1.用表情符合代替图片资源（火箭是自己战机，黄色是子弹，其他是敌人战机）
2.用鼠标移动飞机左右方向
3.空格键发射子弹
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f7eac944248a46068519f3c36d1a4bde.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Flay.vue

```typescript
<template>
  <div class="game-container">
    <canvas ref="canvas" :width="game.SWIDTH" :height="game.SHEIGHT" @mousemove="onMouseMove"></canvas>

    <div v-if="game.gameState === 'menu'" class="menu overlay">
      <h2>飞机大战</h2>
      <button @click="game.startGame()">开始游戏</button>
      <button @click="exitGame">退出游戏</button>
    </div>

    <div v-if="game.gameState === 'gameover'" class="gameover overlay">
      <h2>游戏结束! 分数: {{ game.score }}</h2>
      <button @click="game.startGame()">重新开始</button>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted, onUnmounted } from 'vue';

// 转换GameService为响应式对象
const game = reactive({
  ENEMY_EMOJIS: ["🦄","🐺","🐗","🐴","🦋","🐌","🐞","🐝","🦀","🐡","🐬","🦑","🐊","🦓","🐇"],
  SWIDTH: 600,
  SHEIGHT: 900,
  HERO_SIZE: 40,
  ENEMY_SIZE: 30,
  BULLET_SPEED: 8,
  heroX: 300,
  heroY: 800,
  bullets: [],
  enemies: [],
  score: 0,
  gameState: 'menu',
  lastTime: 0,

  startGame() {
    this.gameState = 'playing';
    this.heroX = this.SWIDTH / 2;
    this.heroY = this.SHEIGHT - 100;
    this.bullets = [];
    this.enemies = [];
    this.score = 0;
    this.lastTime = 0;
    this.gameLoop(0);
  },

  gameLoop(timestamp) {
    if (this.gameState !== 'playing') return;

    const deltaTime = timestamp - this.lastTime;
    this.lastTime = timestamp;

    if (Math.random() < 0.05 && this.enemies.length < 10) {
      this.enemies.push(new Enemy(
        this.ENEMY_EMOJIS[Math.floor(Math.random() * this.ENEMY_EMOJIS.length)],
        Math.random() * (this.SWIDTH - this.ENEMY_SIZE),
        -this.ENEMY_SIZE
      ));
    }

    this.bullets = this.bullets.filter(bullet => {
      bullet.y -= this.BULLET_SPEED;
      return bullet.y > -this.BULLET_SPEED;
    });

    this.enemies = this.enemies.filter(enemy => {
      enemy.y += 4;

      const hit = this.bullets.some(bullet =>
        bullet.x < enemy.x + this.ENEMY_SIZE &&
        bullet.x + 10 > enemy.x &&
        bullet.y < enemy.y + this.ENEMY_SIZE &&
        bullet.y + 20 > enemy.y
      );

      if (hit) this.score++;

      const crash =
        this.heroX < enemy.x + this.ENEMY_SIZE &&
        this.heroX + this.HERO_SIZE > enemy.x &&
        this.heroY < enemy.y + this.ENEMY_SIZE &&
        this.heroY + this.HERO_SIZE > enemy.y;

      if (crash) this.gameState = 'gameover';

      return !hit && enemy.y < this.SHEIGHT && !crash;
    });

    requestAnimationFrame((t) => this.gameLoop(t));
  }
});

class Enemy {
  constructor(emoji, x, y) {
    this.emoji = emoji;
    this.x = x;
    this.y = y;
  }
}

const canvas = ref(null);
let animationFrameId;

const onMouseMove = (e) => {
  if (game.gameState === 'playing') {
    const rect = canvas.value.getBoundingClientRect();
    game.heroX = e.clientX - rect.left - game.HERO_SIZE / 2;
    game.heroY = e.clientY - rect.top - game.HERO_SIZE / 2;
    keepInBounds();
  }
};

const keepInBounds = () => {
  game.heroX = Math.max(0, Math.min(game.SWIDTH - game.HERO_SIZE, game.heroX));
  game.heroY = Math.max(0, Math.min(game.SHEIGHT - game.HERO_SIZE, game.heroY));
};

const handleKeyDown = (e) => {
  if (e.code === 'Space' && game.gameState === 'playing') {
    game.bullets.push({
      x: game.heroX + game.HERO_SIZE / 2 - 5,
      y: game.heroY - 10
    });
  }
};

const draw = () => {
  const ctx = canvas.value.getContext('2d');
  ctx.clearRect(0, 0, game.SWIDTH, game.SHEIGHT);

  ctx.font = '40px Segoe UI Emoji';
  ctx.fillText('🚀', game.heroX, game.heroY);

  ctx.fillStyle = 'gold';
  game.bullets.forEach(b => ctx.fillRect(b.x, b.y, 10, 20));

  ctx.fillStyle = 'red';
  game.enemies.forEach(enemy => {
    ctx.font = '30px Segoe UI Emoji';
    ctx.fillText(enemy.emoji, enemy.x, enemy.y);
  });

  ctx.fillStyle = 'black';
  ctx.font = '20px Arial';
  ctx.fillText(`Score: ${game.score}`, 10, 30);

  animationFrameId = requestAnimationFrame(draw);
};

const exitGame = () => {
  // 退出逻辑
};

onMounted(() => {
  document.addEventListener('keydown', handleKeyDown);
  draw();
});

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeyDown);
  cancelAnimationFrame(animationFrameId);
});
</script>

<style scoped>
.game-container {
  position: relative;
  background: linear-gradient(135deg, #1a1a1a 0%, #4a4a4a 100%);
  border-radius: 15px;
  box-shadow: 0 0 20px rgba(0,0,0,0.5);
  overflow: hidden;
  margin: 20px;
}

canvas {
  display: block;
  border-radius: 10px;
  box-shadow: inset 0 0 10px rgba(255,255,255,0.1);
}

.overlay {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: rgba(0, 0, 0, 0.8);
  padding: 2rem;
  border-radius: 15px;
  text-align: center;
  box-shadow: 0 0 20px rgba(0,0,0,0.5);
  color: white;
}

button {
  background: #4CAF50;
  border: none;
  padding: 10px 20px;
  margin: 10px;
  border-radius: 5px;
  color: white;
  cursor: pointer;
  transition: background 0.3s;
}

button:hover {
  background: #45a049;
}

h2 {
  margin: 0 0 1rem 0;
  color: #fff;
}
</style>
```

手动分割线
以下是angular代码
step101:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\flay\flay.component.ts

```typescript
import { Component, HostListener, ViewChild, ElementRef } from '@angular/core';
import { GameService } from './game.service';
import {NgIf} from '@angular/common';
@Component({
  selector: 'app-flay',
  imports: [
    NgIf
  ],
  template: `
    <canvas #canvas [width]="game.SWIDTH" [height]="game.SHEIGHT"
            (mousemove)="onMouseMove($event)"></canvas>

    <div *ngIf="game.gameState === 'menu'" class="menu">
      <h2>飞机大战</h2>
      <button (click)="game.startGame()">开始游戏</button>
      <button (click)="exitGame()">退出游戏</button>
    </div>

    <div *ngIf="game.gameState === 'gameover'" class="gameover">
      <h2>游戏结束! 分数: {{game.score}}</h2>
      <button (click)="game.startGame()">重新开始</button>
    </div>
  `,
  styles: [`
    canvas { border: 1px solid #ccc; }
    .menu, .gameover {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      text-align: center;
    }
  `]
})

export class FlayComponent {
  @ViewChild('canvas') canvas!: ElementRef<HTMLCanvasElement>;

  constructor(public game: GameService) {}

  ngAfterViewInit() {
    this.draw();
  }

  onMouseMove(e: MouseEvent) {
    if (this.game.gameState === 'playing') {
      const rect = this.canvas.nativeElement.getBoundingClientRect();
      this.game.heroX = e.clientX - rect.left - this.game.HERO_SIZE/2;
      this.game.heroY = e.clientY - rect.top - this.game.HERO_SIZE/2;
      this.keepInBounds();
    }
  }

  @HostListener('document:keydown', ['$event'])
  onKeyDown(e: KeyboardEvent) {
    if (e.code === 'Space' && this.game.gameState === 'playing') {
      this.game.bullets.push({
        x: this.game.heroX + this.game.HERO_SIZE/2 - 5,
        y: this.game.heroY - 10
      });
    }
  }

  private keepInBounds() {
    this.game.heroX = Math.max(0, Math.min(
      this.game.SWIDTH - this.game.HERO_SIZE,
      this.game.heroX
    ));
    this.game.heroY = Math.max(0, Math.min(
      this.game.SHEIGHT - this.game.HERO_SIZE,
      this.game.heroY
    ));
  }

  private draw() {
    requestAnimationFrame(() => this.draw());
    const ctx = this.canvas.nativeElement.getContext('2d')!;

    ctx.clearRect(0, 0, this.game.SWIDTH, this.game.SHEIGHT);

    // 绘制玩家
    ctx.font = '40px Segoe UI Emoji';
    ctx.fillText('🚀', this.game.heroX, this.game.heroY);

    // 绘制子弹
    ctx.fillStyle = 'gold';
    this.game.bullets.forEach(b => ctx.fillRect(b.x, b.y, 10, 20));

    // 绘制敌人
    ctx.fillStyle = 'red';
    this.game.enemies.forEach(enemy => {
      ctx.font = '30px Segoe UI Emoji';
      ctx.fillText(enemy.emoji, enemy.x, enemy.y);
    });

    // 绘制分数
    ctx.fillStyle = 'black';
    ctx.font = '20px Arial';
    ctx.fillText(`Score: ${this.game.score}`, 10, 30);
  }

  exitGame() {
    // 退出逻辑（根据实际需求实现）
  }
}

```

step102:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\flay\game.service.ts

```typescript
// game.service.ts
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class GameService {
  readonly ENEMY_EMOJIS = ["🦄","🐺","🐗","🐴","🦋","🐌","🐞","🐝","🦀","🐡","🐬","🦑","🐊","🦓","🐇"];
  readonly SWIDTH = 600;
  readonly SHEIGHT = 900;
  readonly HERO_SIZE = 40;
  readonly ENEMY_SIZE = 30;
  readonly BULLET_SPEED = 8;

  heroX = this.SWIDTH / 2;
  heroY = this.SHEIGHT - 100;
  bullets: { x: number, y: number }[] = [];
  enemies: Enemy[] = [];
  score = 0;
  gameState: 'menu' | 'playing' | 'gameover' = 'menu';

  private lastTime = 0;

  startGame() {
    this.gameState = 'playing';
    this.heroX = this.SWIDTH / 2;
    this.heroY = this.SHEIGHT - 100;
    this.bullets = [];
    this.enemies = [];
    this.score = 0;
    this.gameLoop(0);
  }

  private gameLoop(timestamp: number) {
    if (this.gameState !== 'playing') return;

    const deltaTime = timestamp - this.lastTime;
    this.lastTime = timestamp;

    // 生成敌人
    if (Math.random() < 0.05 && this.enemies.length < 10) {
      this.enemies.push(new Enemy(
        this.ENEMY_EMOJIS[Math.floor(Math.random() * this.ENEMY_EMOJIS.length)],
        Math.random() * (this.SWIDTH - this.ENEMY_SIZE),
        -this.ENEMY_SIZE
      ));
    }

    // 更新子弹
    this.bullets = this.bullets.filter(bullet => {
      bullet.y -= this.BULLET_SPEED;
      return bullet.y > -this.BULLET_SPEED;
    });

    // 更新敌人并检测碰撞
    this.enemies = this.enemies.filter(enemy => {
      enemy.y += 4;

      // 子弹碰撞
      const hit = this.bullets.some(bullet =>
        bullet.x < enemy.x + this.ENEMY_SIZE &&
        bullet.x + 10 > enemy.x &&
        bullet.y < enemy.y + this.ENEMY_SIZE &&
        bullet.y + 20 > enemy.y
      );

      if (hit) this.score++;

      // 玩家碰撞
      const crash =
        this.heroX < enemy.x + this.ENEMY_SIZE &&
        this.heroX + this.HERO_SIZE > enemy.x &&
        this.heroY < enemy.y + this.ENEMY_SIZE &&
        this.heroY + this.HERO_SIZE > enemy.y;

      if (crash) this.gameState = 'gameover';

      return !hit && enemy.y < this.SHEIGHT && !crash;
    });

    requestAnimationFrame((t) => this.gameLoop(t));
  }
}

class Enemy {
  constructor(
    public emoji: string,
    public x: number,
    public y: number
  ) {}
}

```

end