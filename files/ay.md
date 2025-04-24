说明：
我希望用c#的form实现叠叠乐的游戏，玩家需要堆叠方块来建造高塔。
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/829bea70f9374448bdf9206e0e9b7af0.png#pic_center)

step1:游戏规则

```bash
游戏实现步骤：
a. 处理事件，玩家可以释放摆动的方块，方块会下落。
b. 更新摆动方块的位移，根据角度计算位置。
c. 当方块下落后，检测其与下方方块的碰撞，判断是否成功堆叠。
d. 成功则增加分数，失败则减少生命值并重置方块。
e. 绘制背景、所有已堆叠的方块、摆动的方块、分数和生命值。
f. 当堆叠方块超过一定数量时，移除底部的方块并调整位置，同时移动背景。
g. 检查游戏结束条件，显示胜利或失败画面，关闭窗口。
```

step2:游戏步骤

```bash
步骤分解：
a. 创建主窗体，设置合适的尺寸，比如950x950。
b. 添加必要的控件：Timer用于游戏循环，PictureBox或直接在窗体上绘图。
c. 处理用户输入：鼠标点击触发方块下落。
d. 摆动逻辑：使用角度和角速度，每个Timer tick更新角度，计算方块位置。
e. 方块下落：当玩家点击时，方块开始下落，可能需要另一个Timer或使用游戏循环中的状态来处理下落过程。
f. 碰撞检测：当下落的方块到达堆叠位置时，检测其与最上方方块的位置是否重叠。
g. 分数和生命值管理：成功堆叠加分，失败减生命，生命为0时游戏结束。
h. 绘制元素：包括背景、所有堆叠的方块、摆动的方块、分数和生命值显示。
i. 背景移动：当堆叠到一定高度时，背景需要上移，可能需要调整绘制的位置。
j. 开始菜单和难度选择：可以通过显示不同的面板或窗体来实现。
```

step3:
C:\Users\wangrusheng\RiderProjects\WinFormsApp31\WinFormsApp31\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Windows.Forms;

namespace WinFormsApp31
{
    public partial class Form1 : Form
    {
        private System.Windows.Forms.Timer gameTimer;
        private float angle = 0;
        private float angularVelocity = 0.05f;
        private float maxAngle = (float)(Math.PI / 4);
        private List<RectangleF> stackedBlocks = new List<RectangleF>();
        private RectangleF swingingBlock;
        private bool isFalling = false;
        private int score = 0;
        private int lives = 3;
        private const int BlockSize = 70;
        private PointF pivot;
        private float backgroundOffset = 0;
        private Random random = new Random();
        private Color currentColor;

        public Form1()
        {
            InitializeComponent();
            // 设置默认窗体客户区大小为 800x600
            this.ClientSize = new System.Drawing.Size(600, 800);
            InitializeGame();
            SetupTimer();
            this.DoubleBuffered = true;
            this.Paint += MainForm_Paint;
            this.MouseClick += MainForm_MouseClick;
        }

        private void InitializeGame()
        {
            currentColor = GenerateRandomColor();
            pivot = new PointF(ClientSize.Width / 2, 50);
            swingingBlock = new RectangleF(
                pivot.X - BlockSize / 2,
                pivot.Y + 100,
                BlockSize,
                BlockSize);
        }

        private void SetupTimer()
        {
            gameTimer = new System.Windows.Forms.Timer();
            gameTimer.Interval = 16; // ~60 FPS
            gameTimer.Tick += GameTimer_Tick;
            gameTimer.Start();
        }

        private void GameTimer_Tick(object sender, EventArgs e)
        {
            if (!isFalling)
            {
                // Update swinging motion
                angle += angularVelocity;
                if (Math.Abs(angle) > maxAngle)
                {
                    angularVelocity *= -1;
                }

                float x = pivot.X + 150 * (float)Math.Sin(angle);
                float y = pivot.Y + 150 * (float)Math.Cos(angle);
                swingingBlock.Location = new PointF(x - BlockSize / 2, y - BlockSize / 2);
            }
            else
            {
                // Update falling block
                swingingBlock.Y += 15;
                CheckCollision();
            }

            Invalidate();
        }

        private void CheckCollision()
        {
            float targetY = ClientSize.Height - BlockSize * (stackedBlocks.Count + 1) - backgroundOffset;
            
            if (swingingBlock.Y >= targetY)
            {
                if (stackedBlocks.Count == 0 || CheckAlignment())
                {
                    // Successful stack
                    score++;
                    stackedBlocks.Add(new RectangleF(swingingBlock.Location, swingingBlock.Size));
                    currentColor = GenerateRandomColor();
                    isFalling = false;
                    
                    // Adjust background offset after 5 blocks
                    if (stackedBlocks.Count > 5)
                    {
                        backgroundOffset += BlockSize;
                        RemoveBottomBlock();
                    }
                }
                else
                {
                    // Failed stack
                    lives--;
                    if (lives <= 0)
                    {
                        GameOver();
                        return;
                    }
                    isFalling = false;
                }
                
                // Reset swinging block position
                swingingBlock.Location = new PointF(
                    pivot.X - BlockSize / 2,
                    pivot.Y + 100);
            }
        }

        private bool CheckAlignment()
        {
            var lastBlock = stackedBlocks[stackedBlocks.Count - 1];
            return swingingBlock.X < lastBlock.Right && 
                   swingingBlock.Right > lastBlock.Left;
        }

        private void RemoveBottomBlock()
        {
            if (stackedBlocks.Count > 0)
            {
                stackedBlocks.RemoveAt(0);
                // Adjust all blocks positions
                for (int i = 0; i < stackedBlocks.Count; i++)
                {
                    stackedBlocks[i] = new RectangleF(
                        stackedBlocks[i].X,
                        ClientSize.Height - BlockSize * (i + 1) - backgroundOffset,
                        stackedBlocks[i].Width,
                        stackedBlocks[i].Height);
                }
            }
        }

        private void GameOver()
        {
            gameTimer.Stop();
            MessageBox.Show($"Game Over! Final Score: {score}");
            InitializeGame();
            score = 0;
            lives = 3;
            backgroundOffset = 0;
            stackedBlocks.Clear();
            gameTimer.Start();
        }

        private void MainForm_MouseClick(object sender, MouseEventArgs e)
        {
            if (!isFalling)
            {
                isFalling = true;
            }
        }

        private void MainForm_Paint(object sender, PaintEventArgs e)
        {
            var g = e.Graphics;
            g.Clear(Color.LightSkyBlue);

            // Draw stacked blocks
            foreach (var block in stackedBlocks)
            {
                DrawBlock(g, block, currentColor);
            }

            // Draw swinging/falling block
            DrawBlock(g, swingingBlock, currentColor);

            // Draw pivot line
            if (!isFalling)
            {
                g.DrawLine(Pens.Black, pivot, new PointF(
                    swingingBlock.Left + swingingBlock.Width / 2,
                    swingingBlock.Top + swingingBlock.Height / 2));
            }

            // Draw UI
            DrawUI(g);
        }

        private void DrawBlock(Graphics g, RectangleF rect, Color color)
        {
            using (var brush = new SolidBrush(color))
            {
                g.FillRectangle(brush, rect);
            }
            g.DrawRectangle(Pens.Black, rect.X, rect.Y, rect.Width, rect.Height);

            // Draw windows
            float windowSize = 15;
            float spacing = 10;
            for (float y = rect.Y + spacing; y < rect.Bottom - windowSize; y += windowSize + spacing)
            {
                for (float x = rect.X + spacing; x < rect.Right - windowSize; x += windowSize + spacing)
                {
                    g.FillRectangle(Brushes.Black, x, y, windowSize, windowSize);
                }
            }
        }

        private void DrawUI(Graphics g)
        {
            string scoreText = $"Score: {score}";
            string livesText = $"Lives: {lives}";
            
            var scoreSize = g.MeasureString(scoreText, this.Font);
            var livesSize = g.MeasureString(livesText, this.Font);

            g.DrawString(scoreText, this.Font, Brushes.Black, 
                ClientSize.Width - scoreSize.Width - 10, 10);
            g.DrawString(livesText, this.Font, Brushes.Black, 10, 10);
        }

        private Color GenerateRandomColor()
        {
            return Color.FromArgb(random.Next(150, 255), 
                random.Next(150, 255), 
                random.Next(150, 255));
        }

        protected override void OnResize(EventArgs e)
        {
            base.OnResize(e);
            InitializeGame();
            Invalidate();
        }
    }
}
```

end