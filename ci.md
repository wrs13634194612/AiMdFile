说明：
我希望用forms实现俄罗斯方块
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/335c2b67e9da4cd8b238f390011933f1.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp2\WinFormsApp2\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Windows.Forms;

namespace WinFormsApp2
{
    public partial class Form1 : Form
    {
        // 游戏常量
        private readonly int cellSize = 25;
        private readonly int cols = 10;
        private readonly int rows = 20;
        
        // 游戏状态
        private int[,] board;
        private int[,] currentBlock;
        private Point blockPosition;
        private int score;
        private System.Windows.Forms.Timer gameTimer;
        private bool isPlaying;
        private bool isPaused;

        // UI控件
        private Button startButton, pauseButton, stopButton;
        private Button leftButton, rightButton, downButton, rotateButton;
        private Label scoreLabel;

        // 方块形状
        private readonly List<int[][,]> shapes = new List<int[][,]> 
        {
            new int[][,] { new int[,] {{1,1,1,1}} },    // I
            new int[][,] { new int[,] {{1,1}, {1,1}} }, // O
            new int[][,] { new int[,] {{1,1,1}, {0,1,0}} }, // T
            new int[][,] { new int[,] {{1,1,0}, {0,1,1}} }, // Z
            new int[][,] { new int[,] {{0,1,1}, {1,1,0}} }, // S
            new int[][,] { new int[,] {{1,1,1}, {0,0,1}} }, // L
            new int[][,] { new int[,] {{1,1,1}, {1,0,0}} }  // J
        };

        public Form1()
        {
            InitializeComponent();
            InitializeGame();
            InitializeControls();
            this.DoubleBuffered = true;
            this.Paint += Form1_Paint;
            this.KeyDown += Form1_KeyDown;
        }

        private void InitializeGame()
        {
            board = new int[rows, cols];
            gameTimer = new System.Windows.Forms.Timer { Interval = 600 };
            gameTimer.Tick += GameLoop;
        }

        private void InitializeControls()
        {
            // 分数显示
            scoreLabel = new Label
            {
                Text = "得分: 0",
                Location = new Point(cols * cellSize + 20, 370),
                Size = new Size(150, 20),
                Font = new Font("微软雅黑", 10)
            };

            // 游戏控制按钮
            startButton = new Button
            {
                Text = "开始游戏",
                Location = new Point(cols * cellSize + 20, 20),
                Size = new Size(120, 40),
                Font = new Font("微软雅黑", 10)
            };
            startButton.Click += StartButton_Click;

            pauseButton = new Button
            {
                Text = "暂停游戏",
                Location = new Point(cols * cellSize + 20, 70),
                Size = new Size(120, 40),
                Enabled = false,
                Font = new Font("微软雅黑", 10)
            };
            pauseButton.Click += PauseButton_Click;

            stopButton = new Button
            {
                Text = "结束游戏",
                Location = new Point(cols * cellSize + 20, 120),
                Size = new Size(120, 40),
                Enabled = false,
                Font = new Font("微软雅黑", 10)
            };
            stopButton.Click += StopButton_Click;

            // 方向控制按钮（垂直布局）
            int buttonY = 170;
            leftButton = new Button
            {
                Text = "左移",
                Location = new Point(cols * cellSize + 20, buttonY),
                Size = new Size(120, 40),
                Font = new Font("微软雅黑", 10),
                Enabled = false
            };
            leftButton.Click += (s, e) => MoveBlock(-1, 0);

            buttonY += 50;
            rightButton = new Button
            {
                Text = "右移",
                Location = new Point(cols * cellSize + 20, buttonY),
                Size = new Size(120, 40),
                Font = new Font("微软雅黑", 10),
                Enabled = false
            };
            rightButton.Click += (s, e) => MoveBlock(1, 0);

            buttonY += 50;
            downButton = new Button
            {
                Text = "下移",
                Location = new Point(cols * cellSize + 20, buttonY),
                Size = new Size(120, 40),
                Font = new Font("微软雅黑", 10),
                Enabled = false
            };
            downButton.Click += (s, e) => MoveBlock(0, 1);

            buttonY += 50;
            rotateButton = new Button
            {
                Text = "旋转",
                Location = new Point(cols * cellSize + 20, buttonY),
                Size = new Size(120, 40),
                Font = new Font("微软雅黑", 10),
                Enabled = false
            };
            rotateButton.Click += (s, e) => { if (isPlaying && !isPaused) RotateBlock(); };

            // 添加控件
            Controls.AddRange(new Control[] {
                scoreLabel,
                startButton, pauseButton, stopButton,
                leftButton, rightButton, downButton, rotateButton
            });

            // 设置窗体属性
            Text = "俄罗斯方块";
            ClientSize = new Size(cols * cellSize + 220, Math.Max(rows * cellSize + 50, 450));
            FormBorderStyle = FormBorderStyle.FixedSingle;
            MaximizeBox = false;
        }

        private void CreateBlock()
        {
            var random = new Random();
            var shape = shapes[random.Next(shapes.Count)];
            currentBlock = shape[0];
            blockPosition = new Point(
                (cols - currentBlock.GetLength(1)) / 2,
                0
            );
        }

        private bool CheckCollision(int dx, int dy)
        {
            for (int i = 0; i < currentBlock.GetLength(0); i++)
            {
                for (int j = 0; j < currentBlock.GetLength(1); j++)
                {
                    if (currentBlock[i, j] == 1)
                    {
                        int nx = blockPosition.X + j + dx;
                        int ny = blockPosition.Y + i + dy;

                        if (nx < 0 || nx >= cols || ny >= rows)
                            return true;

                        if (ny >= 0 && board[ny, nx] != 0)
                            return true;
                    }
                }
            }
            return false;
        }

        private void MergeBlock()
        {
            for (int i = 0; i < currentBlock.GetLength(0); i++)
            {
                for (int j = 0; j < currentBlock.GetLength(1); j++)
                {
                    if (currentBlock[i, j] == 1)
                    {
                        int y = blockPosition.Y + i;
                        int x = blockPosition.X + j;
                        if (y >= 0) board[y, x] = 1;
                    }
                }
            }
        }

        private void ClearLines()
        {
            int lines = 0;
            for (int y = rows - 1; y >= 0; y--)
            {
                bool full = true;
                for (int x = 0; x < cols; x++)
                {
                    if (board[y, x] == 0)
                    {
                        full = false;
                        break;
                    }
                }

                if (full)
                {
                    lines++;
                    for (int yy = y; yy > 0; yy--)
                    {
                        for (int x = 0; x < cols; x++)
                        {
                            board[yy, x] = board[yy - 1, x];
                        }
                    }
                    y++;
                }
            }
            score += lines * 100;
            scoreLabel.Text = $"得分: {score}";
        }

        private void GameLoop(object sender, EventArgs e)
        {
            if (!CheckCollision(0, 1))
            {
                blockPosition.Y++;
            }
            else
            {
                MergeBlock();
                ClearLines();
                CreateBlock();
                if (CheckCollision(0, 0))
                {
                    GameOver();
                }
            }
            Invalidate();
        }

        private void GameOver()
        {
            gameTimer.Stop();
            isPlaying = false;
            MessageBox.Show("游戏结束！最终得分: " + score);
            ToggleControls(false);
        }

        private void StartGame()
        {
            board = new int[rows, cols];
            score = 0;
            scoreLabel.Text = "得分: 0";
            isPlaying = true;
            isPaused = false;
            CreateBlock();
            gameTimer.Start();
            ToggleControls(true);
        }

        private void ToggleControls(bool enable)
        {
            pauseButton.Enabled = enable;
            stopButton.Enabled = enable;
            leftButton.Enabled = enable;
            rightButton.Enabled = enable;
            downButton.Enabled = enable;
            rotateButton.Enabled = enable;
        }

        private void MoveBlock(int dx, int dy)
        {
            if (isPlaying && !isPaused && !CheckCollision(dx, dy))
            {
                blockPosition.X += dx;
                blockPosition.Y += dy;
                Invalidate();
            }
        }

        private void RotateBlock()
        {
            var rotated = new int[currentBlock.GetLength(1), currentBlock.GetLength(0)];
            for (int i = 0; i < currentBlock.GetLength(0); i++)
            {
                for (int j = 0; j < currentBlock.GetLength(1); j++)
                {
                    rotated[j, currentBlock.GetLength(0) - 1 - i] = currentBlock[i, j];
                }
            }

            var original = currentBlock;
            currentBlock = rotated;

            int offset = 0;
            while (CheckCollision(0, 0))
            {
                offset++;
                blockPosition.X += blockPosition.X >= cols / 2 ? -1 : 1;
                if (offset > 2)
                {
                    currentBlock = original;
                    return;
                }
            }
        }

        private void Form1_Paint(object sender, PaintEventArgs e)
        {
            var g = e.Graphics;
            
            // 绘制游戏板
            for (int y = 0; y < rows; y++)
            {
                for (int x = 0; x < cols; x++)
                {
                    var brush = board[y, x] != 0 ? Brushes.Red : Brushes.Black;
                    g.FillRectangle(brush, x * cellSize, y * cellSize, cellSize - 1, cellSize - 1);
                }
            }

            // 绘制当前方块
            if (currentBlock != null)
            {
                for (int i = 0; i < currentBlock.GetLength(0); i++)
                {
                    for (int j = 0; j < currentBlock.GetLength(1); j++)
                    {
                        if (currentBlock[i, j] == 1)
                        {
                            int x = (blockPosition.X + j) * cellSize;
                            int y = (blockPosition.Y + i) * cellSize;
                            g.FillRectangle(Brushes.Green, x, y, cellSize - 1, cellSize - 1);
                        }
                    }
                }
            }
        }

        private void Form1_KeyDown(object sender, KeyEventArgs e)
        {
            if (!isPlaying || isPaused) return;

            switch (e.KeyCode)
            {
                case Keys.Left:
                    if (!CheckCollision(-1, 0)) blockPosition.X--;
                    break;
                case Keys.Right:
                    if (!CheckCollision(1, 0)) blockPosition.X++;
                    break;
                case Keys.Down:
                    if (!CheckCollision(0, 1)) blockPosition.Y++;
                    break;
                case Keys.Up:
                    RotateBlock();
                    break;
            }
            Invalidate();
        }

        private void StartButton_Click(object sender, EventArgs e) => StartGame();

        private void PauseButton_Click(object sender, EventArgs e)
        {
            isPaused = !isPaused;
            if (isPaused)
            {
                gameTimer.Stop();
                pauseButton.Text = "继续游戏";
            }
            else
            {
                gameTimer.Start();
                pauseButton.Text = "暂停游戏";
            }
        }

        private void StopButton_Click(object sender, EventArgs e)
        {
            gameTimer.Stop();
            isPlaying = false;
            ToggleControls(false);
            board = new int[rows, cols];
            Invalidate();
        }

  
    }
}
```

end