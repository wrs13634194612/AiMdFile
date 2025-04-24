说明：
vue+form实现flappybird
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/cf2e0d3c861f4099b1badd08cc510c7f.png#pic_center)


form实现：
step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp3\WinFormsApp3\Form1.cs

```csharp
namespace WinFormsApp3;

public partial class Form1 : Form
{
    private const int GRAVITY = 1;
    private const int JUMP_FORCE = -15;
    private const int PIPE_SPEED = 5;
    private const int PIPE_SPACING = 200;
    private const int PIPE_WIDTH = 50;

    private readonly Random random = new Random();
    private readonly Font gameFont = new Font("Arial", 24);
    private readonly StringFormat centerFormat = new StringFormat() 
        { Alignment = StringAlignment.Center, LineAlignment = StringAlignment.Center };

    private int score;
    private int birdY;
    private int velocity;
    private bool isGameOver;
    private List<Rectangle> pipes = new List<Rectangle>();
    private int frameCounter;
    private System.Windows.Forms.Timer gameTimer; // 添加这一行
    
    public Form1()
    {
        InitializeComponent();
    
        DoubleBuffered = true;
        ClientSize = new Size(800, 600);
        
        gameTimer = new System.Windows.Forms.Timer(); // 初始化 Timer
        gameTimer.Interval = 16; // ~60 FPS
        gameTimer.Tick += GameLoop;
        KeyDown += OnKeyPress;
        
        InitializeGame();
    }
    
     private void InitializeGame()
    {
        score = 0;
        birdY = ClientSize.Height / 2;
        velocity = 0;
        pipes.Clear();
        isGameOver = false;
        frameCounter = 0;
        
        gameTimer.Start();
        Invalidate();
    }

    private void GameLoop(object sender, EventArgs e)
    {
        if (!isGameOver)
        {
            UpdateBird();
            UpdatePipes();
            CheckCollisions();
            frameCounter++;
        }
        Invalidate();
    }

    private void UpdateBird()
    {
        velocity += GRAVITY;
        birdY += velocity;
        birdY = Math.Clamp(birdY, 0, ClientSize.Height - 50);
    }

    private void UpdatePipes()
    {
        // 生成新管道
        if (frameCounter % 100 == 0)
        {
            int gapPosition = random.Next(150, ClientSize.Height - 150);
            pipes.Add(new Rectangle(ClientSize.Width, 0, PIPE_WIDTH, gapPosition - PIPE_SPACING/2));
            pipes.Add(new Rectangle(ClientSize.Width, gapPosition + PIPE_SPACING/2, 
                PIPE_WIDTH, ClientSize.Height - gapPosition - PIPE_SPACING/2));
        }

        // 移动管道
        for (int i = pipes.Count - 1; i >= 0; i--)
        {
            pipes[i] = new Rectangle(pipes[i].X - PIPE_SPEED, pipes[i].Y, pipes[i].Width, pipes[i].Height);
            if (pipes[i].Right < 0) pipes.RemoveAt(i);
        }
    }

    private void CheckCollisions()
    {
        Rectangle birdRect = new Rectangle(100, birdY, 50, 50);
        
        foreach (var pipe in pipes)
        {
            if (pipe.IntersectsWith(birdRect))
            {
                isGameOver = true;
                return;
            }

            // 计分逻辑：当管道通过小鸟时加分
            if (pipe.X + pipe.Width == 100 && !isGameOver) score++;
        }
    }

    private void OnKeyPress(object sender, KeyEventArgs e)
    {
        if (e.KeyCode == Keys.Space)
        {
            if (isGameOver) InitializeGame();
            else velocity = JUMP_FORCE;
        }
    }

    protected override void OnPaint(PaintEventArgs e)
    {
        base.OnPaint(e);
        var g = e.Graphics;
        g.Clear(Color.SkyBlue);

        // 绘制小鸟
        g.DrawString("🐦", gameFont, Brushes.Yellow, new Rectangle(100, birdY, 50, 50), centerFormat);

    
        // 绘制管道
        foreach (var pipe in pipes)
        {
            g.FillRectangle(Brushes.Green, pipe); // 修正 Brush.Green 为 Brushes.Green
            g.DrawString(pipe.Height > 200 ? "🌿" : "🪵", gameFont, Brushes.Green, pipe, centerFormat);
        }

        // 绘制分数
        g.DrawString($"Score: {score}", gameFont, Brushes.White, 10, 10);

        // 游戏结束提示
        if (isGameOver)
        {
            g.DrawString("游戏结束!\n按空格键重玩", gameFont, Brushes.Red, 
                ClientRectangle, centerFormat);
        }
    }
}
```

end


///我是分割线

vue实现：
step1:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Bird.vue

```typescript
<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { useGame } from './use.game';

const canvasEl = ref<HTMLCanvasElement | null>(null);
const game = useGame();

onMounted(() => {
  const canvas = canvasEl.value;
  if (!canvas) return;
  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  let animationFrameId: number;
  const gameLoop = () => {
    game.update();
    draw(ctx);
    animationFrameId = requestAnimationFrame(gameLoop);
  };
  gameLoop();

  const handleKeyPress = (event: KeyboardEvent) => {
    if (event.code === 'Space') {
      game.jump();
    }
  };
  window.addEventListener('keydown', handleKeyPress);

  onUnmounted(() => {
    cancelAnimationFrame(animationFrameId);
    window.removeEventListener('keydown', handleKeyPress);
  });
});

function draw(ctx: CanvasRenderingContext2D) {
  ctx.clearRect(0, 0, 800, 600);

  // 绘制小鸟
  ctx.fillStyle = 'yellow';
  ctx.font = '50px Arial';
  ctx.fillText('🐦', 100, game.getBirdY());

  // 绘制管道
  ctx.fillStyle = 'green';
  game.getPipes().forEach(pipe => {
    ctx.fillRect(pipe.x, pipe.y, pipe.width, pipe.height);
    ctx.fillText(
      pipe.height > 200 ? '🌿' : '🪵',
      pipe.x + pipe.width / 2,
      pipe.y + pipe.height / 2
    );
  });
}
</script>

<template>
  <div class="container">
    <canvas ref="canvasEl" width="800" height="600"></canvas>
    <div class="score">Score: {{ game.score }}</div>
    <div v-if="game.isGameOver" class="game-over">
      游戏结束!<br>按空格键重玩
    </div>
  </div>
</template>

<style scoped>
.container {
  position: relative;
  display: block;
}

canvas {
  background: skyblue;
}

.score {
  position: absolute;
  top: 10px;
  left: 10px;
  color: white;
  font: 24px Arial;
}

.game-over {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: red;
  font: 24px Arial;
  text-align: center;
}
</style>
```

step2:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\use.game.ts

```typescript
import { ref } from 'vue';

class Rectangle {
  constructor(
    public x: number,
    public y: number,
    public width: number,
    public height: number
  ) {}
}

export function useGame() {
  const score = ref(0);
  const isGameOver = ref(false);

  let birdY = 300;
  let velocity = 0;
  let pipes: Rectangle[] = [];
  let frameCounter = 0;

  // 游戏常量
  const GRAVITY = 1;
  const JUMP_FORCE = -15;
  const PIPE_SPEED = 5;
  const PIPE_SPACING = 200;
  const PIPE_WIDTH = 50;

  function update() {
    if (!isGameOver.value) {
      updateBird();
      updatePipes();
      checkCollisions();
      frameCounter++;
    }
  }

  function updateBird() {
    velocity += GRAVITY;
    birdY += velocity;
    birdY = Math.max(0, Math.min(birdY, 600 - 50));
  }

  function updatePipes() {
    // 生成新管道
    if (frameCounter % 100 === 0) {
      const gapPosition = 150 + Math.random() * 300;
      pipes.push(
        new Rectangle(800, 0, PIPE_WIDTH, gapPosition - PIPE_SPACING / 2),
        new Rectangle(
          800,
          gapPosition + PIPE_SPACING / 2,
          PIPE_WIDTH,
          600 - gapPosition - PIPE_SPACING / 2
        )
      );
    }

    // 移动并过滤管道
    pipes = pipes
      .map(pipe => new Rectangle(
        pipe.x - PIPE_SPEED,
        pipe.y,
        pipe.width,
        pipe.height
      ))
      .filter(pipe => pipe.x + pipe.width > 0);
  }

  function checkCollisions() {
    const birdRect = new Rectangle(100, birdY, 50, 50);

    for (const pipe of pipes) {
      if (checkCollision(birdRect, pipe)) {
        isGameOver.value = true;
        return;
      }

      if (pipe.x + pipe.width === 100 && !isGameOver.value) {
        score.value++;
      }
    }
  }

  function checkCollision(a: Rectangle, b: Rectangle): boolean {
    return a.x < b.x + b.width &&
           a.x + a.width > b.x &&
           a.y < b.y + b.height &&
           a.y + a.height > b.y;
  }

  function jump() {
    if (isGameOver.value) {
      resetGame();
    } else {
      velocity = JUMP_FORCE;
    }
  }

  function resetGame() {
    score.value = 0;
    birdY = 300;
    velocity = 0;
    pipes = [];
    isGameOver.value = false;
    frameCounter = 0;
  }

  return {
    score,
    isGameOver,
    update,
    jump,
    getBirdY: () => birdY,
    getPipes: () => pipes,
  };
}
```

end