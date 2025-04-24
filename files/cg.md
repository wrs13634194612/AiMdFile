说明：
forms实现连连看
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5b0c7547808544c98028999c3bcb6e7e.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp2\WinFormsApp2\Form1.cs

```csharp
  using System;
using System.Collections.Generic;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;

namespace WinFormsApp2
{
    public partial class Form1 : Form
    {
        private const int ROWS = 12;
        private const int COLS = 12;
        private const int CELL_SIZE = 40;
        private const int BORDER = 1;
        
        private readonly string[] EMOJI_LIST = {
            "🐶", "🐱", "🐭", "🐹", "🐰", "🦊", "🐻", "🐼",
            "🐨", "🐯", "🦁", "🐮", "🐷", "🐸", "🐵", "🐧"
        };

        private Button[,] grid = new Button[ROWS, COLS];
        private int[,] board = new int[ROWS, COLS];
        private Point? firstSelection = null;
        private List<Point> currentPath = null;
        private System.Windows.Forms.Timer pathTimer = new System.Windows.Forms.Timer();
        private int score = 0;
        private List<Point> hintPair = new List<Point>();
        private System.Windows.Forms.Timer hintTimer = new System.Windows.Forms.Timer();

        public Form1()
        {
            InitializeComponent();
            InitializeGameBoard();
            SetupControls();
            
            pathTimer.Interval = 500;
            pathTimer.Tick += (s, e) => ClearPath();
            hintTimer.Interval = 2000;
            hintTimer.Tick += (s, e) => ClearHint();
            
            this.DoubleBuffered = true;
            this.ClientSize = new Size(COLS * CELL_SIZE, ROWS * CELL_SIZE + 60);
        }

        private void SetupControls()
        {
            // Score Label
            var lblScore = new Label
            {
                Name = "lblScore",
                Location = new Point(10, ROWS * CELL_SIZE + 10),
                AutoSize = true,
                Text = "得分: 0"
            };

            // Restart Button
            var btnRestart = new Button
            {
                Text = "重新开始",
                Location = new Point(100, ROWS * CELL_SIZE + 10),
                Size = new Size(80, 30)
            };
            btnRestart.Click += (s, e) => RestartGame();

            // Hint Button
            var btnHint = new Button
            {
                Text = "提示",
                Location = new Point(200, ROWS * CELL_SIZE + 10),
                Size = new Size(80, 30)
            };
            btnHint.Click += (s, e) => ShowHint();

            Controls.AddRange(new Control[] { lblScore, btnRestart, btnHint });
        }

        private void InitializeGameBoard()
        {
            int innerRows = ROWS - 2 * BORDER;
            int innerCols = COLS - 2 * BORDER;
            int totalInner = innerRows * innerCols;

            List<int> tileList = new List<int>();
            for (int i = 0; i < totalInner / 2; i++)
            {
                int index = i % EMOJI_LIST.Length;
                tileList.Add(index);
                tileList.Add(index);
            }

            var rand = new Random();
            tileList = tileList.OrderBy(x => rand.Next()).ToList();

            int tileIndex = 0;
            for (int i = 0; i < ROWS; i++)
            {
                for (int j = 0; j < COLS; j++)
                {
                    var btn = new Button
                    {
                        Size = new Size(CELL_SIZE, CELL_SIZE),
                        Location = new Point(j * CELL_SIZE, i * CELL_SIZE),
                        Tag = new Point(i, j),
                        Font = new Font("Segoe UI Emoji", 12),
                        FlatStyle = FlatStyle.Flat
                    };
                    btn.Click += Button_Click;

                    if (IsBorder(i, j))
                    {
                        board[i, j] = -1;
                        btn.Enabled = false;
                        btn.BackColor = Color.Gray;
                    }
                    else
                    {
                        board[i, j] = tileList[tileIndex++];
                        btn.Text = EMOJI_LIST[board[i, j]];
                    }

                    grid[i, j] = btn;
                    Controls.Add(btn);
                }
            }
        }

        private void Button_Click(object sender, EventArgs e)
        {
            var btn = (Button)sender;
            var pos = (Point)btn.Tag;
            
            if (board[pos.X, pos.Y] == -1) return;

            if (firstSelection == null)
            {
                firstSelection = pos;
                btn.BackColor = Color.LightBlue;
            }
            else
            {
                var second = pos;
                if (firstSelection == second)
                {
                    ClearSelection();
                    return;
                }

                var p1 = firstSelection.Value;
                var p2 = second;

                if (board[p1.X, p1.Y] == board[p2.X, p2.Y] && CheckConnection(p1, p2))
                {
                    currentPath = GetConnectionPath(p1, p2);
                    pathTimer.Start();
                    Invalidate();

                    board[p1.X, p1.Y] = -1;
                    board[p2.X, p2.Y] = -1;
                    grid[p1.X, p1.Y].Text = "";
                    grid[p2.X, p2.Y].Text = "";
                    score += 10;
                    UpdateScore();
                }

                ClearSelection();
            }
        }

        private bool CheckConnection(Point p1, Point p2)
        {
            if (DirectConnection(p1, p2)) return true;
            if (OneTurnConnection(p1, p2)) return true;
            return TwoTurnConnection(p1, p2);
        }

        private bool DirectConnection(Point a, Point b)
        {
            if (a.X == b.X)
            {
                int minY = Math.Min(a.Y, b.Y);
                int maxY = Math.Max(a.Y, b.Y);
                for (int y = minY + 1; y < maxY; y++)
                {
                    if (board[a.X, y] != -1) return false;
                }
                return true;
            }

            if (a.Y == b.Y)
            {
                int minX = Math.Min(a.X, b.X);
                int maxX = Math.Max(a.X, b.X);
                for (int x = minX + 1; x < maxX; x++)
                {
                    if (board[x, a.Y] != -1) return false;
                }
                return true;
            }

            return false;
        }

        private bool OneTurnConnection(Point a, Point b)
        {
            var corner1 = new Point(a.X, b.Y);
            if (board[corner1.X, corner1.Y] == -1)
            {
                if (DirectConnection(a, corner1) && DirectConnection(corner1, b)) return true;
            }

            var corner2 = new Point(b.X, a.Y);
            if (board[corner2.X, corner2.Y] == -1)
            {
                if (DirectConnection(a, corner2) && DirectConnection(corner2, b)) return true;
            }

            return false;
        }

        private bool TwoTurnConnection(Point a, Point b)
        {
            // Horizontal scan
            for (int x = 0; x < ROWS; x++)
            {
                var p1 = new Point(x, a.Y);
                var p2 = new Point(x, b.Y);
                if (board[p1.X, p1.Y] == -1 && board[p2.X, p2.Y] == -1)
                {
                    if (DirectConnection(a, p1) && DirectConnection(p1, p2) && DirectConnection(p2, b))
                        return true;
                }
            }

            // Vertical scan
            for (int y = 0; y < COLS; y++)
            {
                var p1 = new Point(a.X, y);
                var p2 = new Point(b.X, y);
                if (board[p1.X, p1.Y] == -1 && board[p2.X, p2.Y] == -1)
                {
                    if (DirectConnection(a, p1) && DirectConnection(p1, p2) && DirectConnection(p2, b))
                        return true;
                }
            }

            return false;
        }

        private List<Point> GetConnectionPath(Point a, Point b)
        {
            var path = new List<Point> { a };

            if (DirectConnection(a, b))
            {
                path.Add(b);
                return path;
            }

            if (OneTurnConnection(a, b))
            {
                if (board[a.X, b.Y] == -1)
                {
                    path.Add(new Point(a.X, b.Y));
                }
                else
                {
                    path.Add(new Point(b.X, a.Y));
                }
                path.Add(b);
                return path;
            }

            // For two-turn connection (simplified path)
            path.Add(a);
            path.Add(b);
            return path;
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            if (currentPath != null && currentPath.Count >= 2)
            {
                using (var pen = new Pen(Color.Blue, 4))
                {
                    var points = currentPath.Select(p => 
                        new Point(p.Y * CELL_SIZE + CELL_SIZE/2, p.X * CELL_SIZE + CELL_SIZE/2)).ToArray();
                    e.Graphics.DrawLines(pen, points);
                }
            }

            // Draw hint
            if (hintPair.Count == 2)
            {
                using (var pen = new Pen(Color.Gold, 4))
                {
                    var points = hintPair.Select(p => 
                        new Point(p.Y * CELL_SIZE + CELL_SIZE/2, p.X * CELL_SIZE + CELL_SIZE/2)).ToArray();
                    e.Graphics.DrawLines(pen, points);
                }
            }
        }

        private void ShowHint()
        {
            for (int i = BORDER; i < ROWS - BORDER; i++)
            {
                for (int j = BORDER; j < COLS - BORDER; j++)
                {
                    if (board[i, j] == -1) continue;

                    for (int x = i; x < ROWS - BORDER; x++)
                    {
                        for (int y = (x == i ? j + 1 : BORDER); y < COLS - BORDER; y++)
                        {
                            if (board[x, y] == board[i, j] && CheckConnection(new Point(i, j), new Point(x, y)))
                            {
                                hintPair = new List<Point> { new Point(i, j), new Point(x, y) };
                                hintTimer.Start();
                                Invalidate();
                                return;
                            }
                        }
                    }
                }
            }
        }

        private void ClearPath() => currentPath = null;
        private void ClearHint() => hintPair.Clear();
        private void UpdateScore() => Controls["lblScore"].Text = $"得分: {score}";
        private bool IsBorder(int x, int y) => x < BORDER || y < BORDER || x >= ROWS - BORDER || y >= COLS - BORDER;
        private void ClearSelection()
        {
            if (firstSelection != null)
                grid[firstSelection.Value.X, firstSelection.Value.Y].BackColor = DefaultBackColor;
            firstSelection = null;
        }

        private void RestartGame()
        {
            Controls.OfType<Button>().Where(b => b.Tag != null).ToList().ForEach(b => Controls.Remove(b));
            board = new int[ROWS, COLS];
            grid = new Button[ROWS, COLS];
            score = 0;
            UpdateScore();
            InitializeGameBoard();
            ClearPath();
            ClearHint();
        }
    }
}
```
备选方案： 纯数字版

```csharp
 namespace WinFormsApp2
{
public partial class Form1 : Form
{
// Board dimensions (include a border to simplify connection checking)
private int rows = 12; // for example, 12 rows (including border)
private int cols = 12; // 12 columns (including border)
private int cellSize = 40; // pixel size for each button

// Game grid of buttons and underlying board values.
    private Button[,] grid;
    private int[,] board;

    // To store the first and second selected positions
    private Point? firstSelection = null;

    public Form1()
    {
        InitializeComponent();
        InitializeGameBoard();
    }

    // Initializes the game board with a border and fills the inner cells with matching pairs.
    private void InitializeGameBoard()
    {
        grid = new Button[rows, cols];
        board = new int[rows, cols];

        // Calculate total inner cells (excluding borders) and make sure they form pairs.
        int innerRows = rows - 2;
        int innerCols = cols - 2;
        int totalInner = innerRows * innerCols;
        if(totalInner % 2 != 0)
        {
            MessageBox.Show("Total number of inner cells must be even.");
            return;
        }

        // Prepare list of pairs (here we simply use numbers to represent different tiles)
        List<int> tileList = new List<int>();
        for (int i = 0; i < totalInner / 2; i++)
        {
            tileList.Add(i);
            tileList.Add(i);
        }

        // Shuffle the tile list randomly
        Random rand = new Random();
        tileList = tileList.OrderBy(x => rand.Next()).ToList();

        // Create buttons and assign values
        int index = 0;
        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                Button btn = new Button();
                btn.Width = cellSize;
                btn.Height = cellSize;
                btn.Location = new Point(j * cellSize, i * cellSize);
                btn.Tag = new Point(i, j);
                btn.Click += Button_Click;
                grid[i, j] = btn;
                this.Controls.Add(btn);

                // Create a border: outside the inner grid, set value to -1 (indicating an always-empty cell)
                if (i == 0 || j == 0 || i == rows - 1 || j == cols - 1)
                {
                    board[i, j] = -1;
                    btn.Enabled = false; // disable border buttons
                    btn.BackColor = Color.Gray;
                }
                else
                {
                    board[i, j] = tileList[index];
                    index++;
                    // Display the tile value (in a final game you would show an image/icon)
                    btn.Text = board[i, j].ToString();
                }
            }
        }
    }

    // Handles button click events for selecting tiles.
    private void Button_Click(object sender, EventArgs e)
    {
        Button btn = sender as Button;
        Point pos = (Point)btn.Tag;
        int x = pos.X;
        int y = pos.Y;

        // Ignore clicks on empty cells (or border cells)
        if (board[x, y] == -1 || string.IsNullOrEmpty(btn.Text))
            return;

        if (firstSelection == null)
        {
            // First tile selected
            firstSelection = pos;
            btn.BackColor = Color.Yellow;
        }
        else
        {
            // Second tile selected
            Point secondSelection = pos;

            // If user clicked the same tile, just ignore
            if (firstSelection.Value == secondSelection)
                return;

            // Check if the two tiles have the same value.
            Point p1 = firstSelection.Value;
            Point p2 = secondSelection;
            if (board[p1.X, p1.Y] != board[p2.X, p2.Y])
            {
                // Not a matching pair. Reset first tile's highlight and choose the new one.
                grid[p1.X, p1.Y].BackColor = default(Color);
                firstSelection = secondSelection;
                btn.BackColor = Color.Yellow;
            }
            else
            {
                // Same value: check if they can be connected
                if (CheckConnectivity(p1, p2))
                {
                    // Connection is valid: remove the tiles.
                    board[p1.X, p1.Y] = -1;
                    board[p2.X, p2.Y] = -1;
                    grid[p1.X, p1.Y].Text = "";
                    grid[p2.X, p2.Y].Text = "";
                    grid[p1.X, p1.Y].BackColor = default(Color);
                    grid[p2.X, p2.Y].BackColor = default(Color);
                }
                else
                {
                    // Cannot connect: reset the highlight.
                    grid[p1.X, p1.Y].BackColor = default(Color);
                }
                // Reset selection for next pair.
                firstSelection = null;
            }
        }
    }

    // Checks whether two positions can be connected using at most two turns.
    private bool CheckConnectivity(Point p1, Point p2)
    {
        // 1. Direct connection check: same row or same column.
        if (DirectConnection(p1, p2))
            return true;
        // 2. Check connection with one turn.
        if (OneTurnConnection(p1, p2))
            return true;
        // 3. Check connection with two turns.
        if (TwoTurnConnection(p1, p2))
            return true;
        return false;
    }

    // Direct connection: both positions on the same row or column and the cells between them are empty.
    private bool DirectConnection(Point p1, Point p2)
    {
        if (p1.X == p2.X)
        {
            int minY = Math.Min(p1.Y, p2.Y);
            int maxY = Math.Max(p1.Y, p2.Y);
            for (int j = minY + 1; j < maxY; j++)
            {
                if (board[p1.X, j] != -1)
                    return false;
            }
            return true;
        }
        if (p1.Y == p2.Y)
        {
            int minX = Math.Min(p1.X, p2.X);
            int maxX = Math.Max(p1.X, p2.X);
            for (int i = minX + 1; i < maxX; i++)
            {
                if (board[i, p1.Y] != -1)
                    return false;
            }
            return true;
        }
        return false;
    }

    // One-turn connection: Try turning at (p1.X, p2.Y) and (p2.X, p1.Y)
    private bool OneTurnConnection(Point p1, Point p2)
    {
        // First turning point: (p1.X, p2.Y)
        Point p3 = new Point(p1.X, p2.Y);
        if (board[p3.X, p3.Y] == -1)
        {
            if (DirectConnection(p1, p3) && DirectConnection(p3, p2))
                return true;
        }
        // Second turning point: (p2.X, p1.Y)
        Point p4 = new Point(p2.X, p1.Y);
        if (board[p4.X, p4.Y] == -1)
        {
            if (DirectConnection(p1, p4) && DirectConnection(p4, p2))
                return true;
        }
        return false;
    }

    // Two-turn connection: try connecting by scanning through possible rows and columns.
    private bool TwoTurnConnection(Point p1, Point p2)
    {
        // Try vertical scan: for each row, check if we can connect via two turns.
        for (int i = 0; i < rows; i++)
        {
            Point p3 = new Point(i, p1.Y);
            Point p4 = new Point(i, p2.Y);
            if (board[p3.X, p3.Y] == -1 && board[p4.X, p4.Y] == -1)
            {
                if (DirectConnection(p1, p3) && OneTurnConnection(p3, p2))
                    return true;
            }
        }
        // Try horizontal scan: for each column, check similarly.
        for (int j = 0; j < cols; j++)
        {
            Point p3 = new Point(p1.X, j);
            Point p4 = new Point(p2.X, j);
            if (board[p3.X, p3.Y] == -1 && board[p4.X, p4.Y] == -1)
            {
                if (DirectConnection(p1, p3) && OneTurnConnection(p3, p2))
                    return true;
            }
        }
        return false;
    }
}

}


```

end