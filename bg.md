说明：
forms实现地铁跑酷小游戏

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a2726022f9484f07b56e9a5fea7b92f4.png#pic_center)

step0:游戏规则

```bash
# 游戏规则文档

## 游戏目标
通过操控角色躲避障碍、收集金币，获得高分并生存更长时间。生命值耗尽时游戏结束。

---

## 基本操作
### 空格键
- 开始游戏（主菜单或游戏结束时）
- 暂停/继续游戏（游戏中）

### 方向键
- **↑（上箭头）**：跳跃（仅站立时可触发）
- **↓（下箭头）**：下蹲（仅站立时可触发，持续0.6秒后自动恢复）
- **←（左箭头）**：移动到下方轨道
- **→（右箭头）**：移动到上方轨道

---

## 游戏机制
### 角色状态
- **站立**：可自由移动或触发跳跃/下蹲
- **跳跃**：垂直速度受重力影响逐渐下落，落地后恢复站立
- **下蹲**：角色高度缩短，期间无法跳跃或再次下蹲

### 轨道系统
- 3条水平轨道（垂直位置）：
  - 上方轨道：Y=180
  - 中间轨道：Y=410
  - 下方轨道：Y=640
- **瞬时切换**：左右键可在任意状态（空中/地面）切换轨道

### 游戏对象
| 对象类型       | 外观         | 效果              | 生成概率 |
|----------------|--------------|-------------------|----------|
| 金币（金色方块）| 金色         | 触碰+10分         | 1/5      |
| 障碍物（深灰方块）| 深灰色      | 触碰-1生命值      | 4/5      |

- 生成规则：从屏幕左侧生成并向右移动，移出右侧边界后消失

---

## 生命系统
- **初始生命值**：2点
- **失败条件**：生命值归零时游戏结束，显示最终得分

---

## 难度曲线
| 达成分数 | 移动速度 |
|----------|----------|
| 100分    | 12       |
| 300分    | 15       |
| 500分    | 20       |

---

## 界面说明
### HUD显示
- 实时显示：
  - 当前分数（右上角）
  - 剩余生命值（左上角）
- 特殊状态提示：
  - 暂停时显示"PAUSED"
  - 游戏结束显示"GAME OVER"及最终得分

### 轨道可视化
- 三条水平分隔线（Y坐标）：
  - 240（上/中轨道分界）
  - 470（中/下轨道分界）
  - 700（下边界）

### 角色外观
- 颜色随生命值变化：
  - 2点：绿色
  - 1点：橙色
  - 0点：红色

---

## 进阶策略
1. **轨道优先级**：优先收集金币轨道，灵活切换躲避障碍
2. **动作组合**：
   - 跳跃规避高空障碍
   - 下蹲躲避低空障碍
3. **速度预判**：随分数提升加速后，需提前0.5-1秒预判对象位置
4. **生命管理**：保持至少1点生命应对突发障碍

> 游戏要求玩家在高速移动场景中快速决策，通过精准操作和路线规划挑战生存极限。祝您游戏愉快！
```


step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp27\WinFormsApp27\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Windows.Forms;

namespace WinFormsApp27
{
    public partial class Form1 : Form
    {
        // 游戏常量
        private const int PlayerSize = 60;
        private const int Gravity = 4;
        private const int JumpVelocity = 32;
        private const int CrouchDuration = 600;
        private const int MaxHealth = 2;

        // 游戏状态
        private Player player;
        private List<GameObject> objects = new List<GameObject>();
        private int score = 0;
        private int gameSpeed = 10;
        private bool isPaused = false;
        private bool isGameRunning = false;
        private bool gameOver = false;

        // 计时器
        private System.Windows.Forms.Timer gameTimer;
        private DateTime lastGenerateTime = DateTime.Now;
        private int generateInterval = 1000;

        public Form1()
        {
            InitializeComponent();
            InitializeGame();
            this.DoubleBuffered = true;
            this.KeyDown += MainForm_KeyDown;
        }

        private void InitializeGame()
        {
            player = new Player
            {
                Position = 2,
                State = PlayerState.Stand,
                Bounds = new Rectangle(1000, 410, PlayerSize, PlayerSize),
                Health = MaxHealth,
                VerticalSpeed = 0
            };

            gameTimer = new System.Windows.Forms.Timer { Interval = 30 };
            gameTimer.Tick += GameLoop;
        }

        private void GameLoop(object sender, EventArgs e)
        {
            if (!isPaused && isGameRunning)
            {
                UpdateGame();
                GenerateObjects();
            }
            Invalidate();
        }

        private void UpdateGame()
        {
            UpdatePlayer();
            UpdateObjects();
            UpdateDifficulty();

            if (player.Health <= 0)
            {
                gameOver = true;
                isGameRunning = false;
            }
        }

        private void UpdatePlayer()
        {
            // 处理跳跃
            if (player.State == PlayerState.Jump)
            {
                var bounds = player.Bounds;
                bounds.Y += player.VerticalSpeed;
                player.Bounds = bounds;

                player.VerticalSpeed += Gravity;

                // 落地检测
                if (player.Bounds.Y > GetLaneY(player.Position))
                {
                    bounds = player.Bounds;
                    bounds.Y = GetLaneY(player.Position);
                    player.Bounds = bounds;
                    player.State = PlayerState.Stand;
                    player.VerticalSpeed = 0;
                }
            }

            // 处理下蹲
            if (player.State == PlayerState.Crouch && 
                (DateTime.Now - player.CrouchStartTime).TotalMilliseconds > CrouchDuration)
            {
                player.State = PlayerState.Stand;
                var bounds = player.Bounds;
                bounds.Height = PlayerSize;
                player.Bounds = bounds;
            }
        }

        private void GenerateObjects()
        {
            if ((DateTime.Now - lastGenerateTime).TotalMilliseconds > generateInterval)
            {
                objects.Add(new GameObject
                {
                    Type = new Random().Next(0, 5),
                    Bounds = new Rectangle(-40, GetLaneY(new Random().Next(1, 4)), 30, 30)
                });
                lastGenerateTime = DateTime.Now;
            }
        }

        private void UpdateObjects()
        {
            for (int i = objects.Count - 1; i >= 0; i--)
            {
                // 移动物体（使用临时变量修改）
                var obj = objects[i];
                var bounds = obj.Bounds;
                bounds.X += gameSpeed;
                obj.Bounds = bounds;
                objects[i] = obj;

                // 碰撞检测
                if (obj.Bounds.IntersectsWith(player.Bounds))
                {
                    HandleCollision(obj);
                    objects.RemoveAt(i);
                }

                // 移出屏幕
                if (obj.Bounds.X > this.ClientSize.Width)
                {
                    objects.RemoveAt(i);
                }
            }
        }

        private void HandleCollision(GameObject obj)
        {
            if (obj.Type == 0) score += 10;
            else player.Health--;
        }

        private void UpdateDifficulty()
        {
            if (score > 500) gameSpeed = 20;
            else if (score > 300) gameSpeed = 15;
            else if (score > 100) gameSpeed = 12;
        }

        private int GetLaneY(int lane)
        {
            return lane switch
            {
                1 => 180,
                2 => 410,
                3 => 640,
                _ => 410
            };
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            var g = e.Graphics;

            if (isGameRunning)
            {
                DrawLanes(g);
                DrawPlayer(g);
                DrawObjects(g);
                DrawHUD(g);
            }
            else
            {
                DrawMenu(g);
            }
        }

        private void DrawLanes(Graphics g)
        {
            using var pen = new Pen(Brushes.Black, 3);
            g.DrawLine(pen, 0, 240, ClientSize.Width, 240);
            g.DrawLine(pen, 0, 470, ClientSize.Width, 470);
            g.DrawLine(pen, 0, 700, ClientSize.Width, 700);
        }

        private void DrawPlayer(Graphics g)
        {
            var color = player.Health switch
            {
                1 => Brushes.Orange,
                0 => Brushes.Red,
                _ => Brushes.Green
            };
            g.FillRectangle(color, player.Bounds);
        }

        private void DrawObjects(Graphics g)
        {
            foreach (var obj in objects)
            {
                g.FillRectangle(obj.Type == 0 ? Brushes.Gold : Brushes.DarkGray, obj.Bounds);
            }
        }

        private void DrawHUD(Graphics g)
        {
            g.DrawString($"Score: {score}", Font, Brushes.Black, 10, 10);
            g.DrawString($"Health: {player.Health}", Font, Brushes.Red, 10, 30);
            if (isPaused)
            {
                g.DrawString("PAUSED", new Font("Arial", 40), Brushes.Gray, 
                    ClientSize.Width/2 - 80, ClientSize.Height/2 - 20);
            }
        }

        private void DrawMenu(Graphics g)
        {
            g.DrawString("PARKOUR GAME", new Font("Arial", 40), Brushes.Black, 300, 200);
            g.DrawString("Press SPACE to Start", new Font("Arial", 20), Brushes.Gray, 400, 300);
            if (gameOver)
            {
                g.DrawString("GAME OVER", new Font("Arial", 40), Brushes.Red, 350, 400);
                g.DrawString($"Final Score: {score}", new Font("Arial", 20), Brushes.Black, 420, 450);
            }
        }

        private void MainForm_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.Space)
            {
                if (!isGameRunning) StartGame();
                else isPaused = !isPaused;
            }

            if (isGameRunning && !isPaused) HandleGameInput(e.KeyCode);
        }

        private void StartGame()
        {
            isGameRunning = true;
            gameOver = false;
            score = 0;
            player.Health = MaxHealth;
            objects.Clear();
            gameTimer.Start();
        }

        private void HandleGameInput(Keys key)
        {
            switch (key)
            {
                case Keys.Up:
                    if (player.State == PlayerState.Stand)
                    {
                        player.State = PlayerState.Jump;
                        player.VerticalSpeed = -JumpVelocity;
                    }
                    break;
                
                case Keys.Down:
                    if (player.State == PlayerState.Stand)
                    {
                        player.State = PlayerState.Crouch;
                        var bounds = player.Bounds;
                        bounds.Height = PlayerSize - 30;
                        player.Bounds = bounds;
                        player.CrouchStartTime = DateTime.Now;
                    }
                    break;
                
                case Keys.Left:
                    if (player.Position < 3) 
                    {
                        player.Position++;
                        var bounds = player.Bounds;
                        bounds.Y = GetLaneY(player.Position);
                        player.Bounds = bounds;
                    }
                    break;
                
                case Keys.Right:
                    if (player.Position > 1)
                    {
                        player.Position--;
                        var bounds = player.Bounds;
                        bounds.Y = GetLaneY(player.Position);
                        player.Bounds = bounds;
                    }
                    break;
            }
        }
    }

    public enum PlayerState { Stand, Crouch, Jump }

    public class Player
    {
        public int Position { get; set; }
        public PlayerState State { get; set; }
        public Rectangle Bounds { get; set; }
        public int Health { get; set; }
        public int VerticalSpeed { get; set; }
        public DateTime CrouchStartTime { get; set; }
    }

    public class GameObject
    {
        public int Type { get; set; }
        public Rectangle Bounds { get; set; }
    }
}
```

end