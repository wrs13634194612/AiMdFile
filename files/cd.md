说明：
我希望用c#的form实现飞机大战
效果图:
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/021584be9d1d4be48bc68ffe788a44f0.png#pic_center)

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
        private readonly string[] ENEMY_EMOJIS = { "🦄", "🐺", "🐗", "🐴", "🦋", "🐌", "🐞", "🐝", "🦀", "🐡", "🐬", "🦑", "🐊", "🦓", "🐇" };
        private const int Swidth = 600;
        private const int Sheight = 900;
        private const int HeroSize = 40;
        private const int EnemySize = 30;
        private const int BulletSpeed = 8;
        private System.Windows.Forms.Timer gameTimer; // 添加这一行
        
        private Rectangle heroRect;
        private List<Enemy> enemies = new List<Enemy>();
        private List<Rectangle> bullets = new List<Rectangle>();
        private int score;
        private GameState gameState = GameState.Menu;
        private Font emojiFont = new Font("Segoe UI Emoji", 20);

        public Form1()
        {
            InitializeComponent();
            this.DoubleBuffered = true;
            this.ClientSize = new Size(Swidth, Sheight);
            this.Paint += MainForm_Paint;
            this.MouseMove += MainForm_MouseMove;
            this.KeyDown += MainForm_KeyDown;
            
            gameTimer = new System.Windows.Forms.Timer(); // 初始化 Timer
            gameTimer.Interval = 16;
            gameTimer.Tick += GameTimer_Tick;
        }

        private enum GameState { Menu, Playing, GameOver }

        private void GameTimer_Tick(object sender, EventArgs e)
        {
            if (gameState != GameState.Playing) return;

            // 生成敌人
            if (new Random().Next(0, 100) < 5 && enemies.Count < 10)
            {
                enemies.Add(new Enemy(
                    ENEMY_EMOJIS[new Random().Next(ENEMY_EMOJIS.Length)],
                    new Point(new Random().Next(0, Swidth - EnemySize), -EnemySize)));
            }

            // 更新子弹位置
            for (int i = bullets.Count - 1; i >= 0; i--)
            {
                var bullet = bullets[i];
                bullet.Y -= BulletSpeed;
                bullets[i] = bullet;

                if (bullet.Y < -BulletSpeed)
                    bullets.RemoveAt(i);
            }

            // 更新敌人位置并检测碰撞
            for (int i = enemies.Count - 1; i >= 0; i--)
            {
                var enemy = enemies[i];
                enemy.Update();

                // 检测子弹碰撞
                foreach (var bullet in bullets)
                {
                    if (enemy.Bounds.IntersectsWith(bullet))
                    {
                        enemies.RemoveAt(i);
                        score++;
                        break;
                    }
                }

                // 检测玩家碰撞
                if (enemy.Bounds.IntersectsWith(heroRect))
                {
                    gameState = GameState.GameOver;
                    gameTimer.Stop();
                }

                if (enemy.Bounds.Y > Sheight)
                    enemies.RemoveAt(i);
            }

            Invalidate();
        }

        private void MainForm_KeyDown(object sender, KeyEventArgs e)
        {
            if (gameState == GameState.Playing && e.KeyCode == Keys.Space)
            {
                bullets.Add(new Rectangle(
                    heroRect.X + heroRect.Width / 2 - 5,
                    heroRect.Y - 10,
                    10, 20));
            }

            if (gameState == GameState.GameOver && e.KeyCode == Keys.Enter)
            {
                gameState = GameState.Menu;
                Invalidate();
            }
        }

        private void MainForm_MouseMove(object sender, MouseEventArgs e)
        {
            if (gameState == GameState.Playing)
            {
                heroRect.X = e.X - HeroSize / 2;
                heroRect.Y = e.Y - HeroSize / 2;
                heroRect = KeepInBounds(heroRect);
            }
        }

        private void MainForm_Paint(object sender, PaintEventArgs e)
        {
            var g = e.Graphics;
            g.Clear(Color.White);

            switch (gameState)
            {
                case GameState.Menu:
                    DrawMenu(g);
                    break;
                case GameState.Playing:
                    DrawGame(g);
                    break;
                case GameState.GameOver:
                    DrawGameOver(g);
                    break;
            }
        }

        private void DrawMenu(Graphics g)
        {
            var title = "飞机大战";
            var start = "开始游戏 (点击)";
            var exit = "退出游戏 (按ESC)";

            var titleSize = g.MeasureString(title, Font);
            g.DrawString(title, Font, Brushes.Black, Swidth / 2 - titleSize.Width / 2, 100);

            var startRect = new RectangleF(Swidth / 2 - 100, 300, 200, 40);
            g.DrawString(start, Font, Brushes.Blue, startRect);

            var exitRect = new RectangleF(Swidth / 2 - 100, 400, 200, 40);
            g.DrawString(exit, Font, Brushes.Red, exitRect);

            // 检测鼠标点击
            this.MouseClick += (s, e) =>
            {
                if (startRect.Contains(e.Location))
                {
                    StartGame();
                }
            };
        }

        private void DrawGame(Graphics g)
        {
            // 绘制玩家
            g.DrawString("🚀", emojiFont, Brushes.Black, heroRect);

            // 绘制敌人
            foreach (var enemy in enemies)
            {
                g.DrawString(enemy.Emoji, emojiFont, Brushes.Red, enemy.Bounds);
            }

            // 绘制子弹
            foreach (var bullet in bullets)
            {
                g.FillRectangle(Brushes.Gold, bullet);
            }

            // 绘制分数
            g.DrawString($"Score: {score}", Font, Brushes.Black, 10, 10);
        }

        private void DrawGameOver(Graphics g)
        {
            g.DrawString($"游戏结束! 分数: {score}", Font, Brushes.Red, Swidth / 2 - 100, Sheight / 2);
            g.DrawString("按 Enter 返回菜单", Font, Brushes.Black, Swidth / 2 - 100, Sheight / 2 + 50);
        }

        private void StartGame()
        {
            gameState = GameState.Playing;
            heroRect = new Rectangle(Swidth / 2 - HeroSize / 2, Sheight - 100, HeroSize, HeroSize);
            enemies.Clear();
            bullets.Clear();
            score = 0;
            gameTimer.Start();
        }

        private Rectangle KeepInBounds(Rectangle rect)
        {
            rect.X = Math.Max(0, Math.Min(Swidth - rect.Width, rect.X));
            rect.Y = Math.Max(0, Math.Min(Sheight - rect.Height, rect.Y));
            return rect;
        }

        private class Enemy
        {
            public string Emoji { get; }
            public Rectangle Bounds { get; private set; }

            public Enemy(string emoji, Point position)
            {
                Emoji = emoji;
                Bounds = new Rectangle(position, new Size(EnemySize, EnemySize));
            }

            public void Update()
            {
                Bounds = new Rectangle(Bounds.X, Bounds.Y + 4, Bounds.Width, Bounds.Height);
            }
        }
    }
}
```

end