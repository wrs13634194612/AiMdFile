说明：
form实现康威生命游戏
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6d9adf6088504c65aa9186bd6bbda08d.png#pic_center)

step1:
C:\Users\wangrusheng\RiderProjects\WinFormsApp22\WinFormsApp22\Form1.cs

```csharp
namespace WinFormsApp22;

public partial class Form1 : Form
{
    private const int ROWS = 20;
    private const int COLS = 20;
    private bool[,] currentBoard = new bool[ROWS, COLS];
    private bool isSimulating = false;
    private bool isSettingCells = false; // 禁用用户交互

    // 预定义三种模式的初始活细胞坐标
    private readonly int[,] initialLiveCells0 = new int[,]
    {
        // 滑翔机（左上角）
        {1, 2}, {2, 3}, {3, 1}, {3, 2}, {3, 3},
        
        // 脉冲星（中心区域）
        {5,7}, {5,8}, {5,9}, {5,10}, {5,11}, {5,12},
        {7,5}, {7,6}, {7,10}, {7,11}, {7,15}, {7,16},
        {8,5}, {8,6}, {8,10}, {8,11}, {8,15}, {8,16},
        {9,5}, {9,6}, {9,10}, {9,11}, {9,15}, {9,16},
        {10,5}, {10,6}, {10,7}, {10,8}, {10,12}, {10,13}, {10,14}, {10,15}, {10,16},
        {11,5}, {11,6}, {11,10}, {11,11}, {11,15}, {11,16},
        {12,5}, {12,6}, {12,10}, {12,11}, {12,15}, {12,16},
        {13,5}, {13,6}, {13,10}, {13,11}, {13,15}, {13,16},
        {15,7}, {15,8}, {15,9}, {15,10}, {15,11}, {15,12},
        
        // 轻型太空船（右下角）
        {15,16}, {15,19}, {16,15}, {16,19}, {17,15}, {17,19}, {18,16}, {18,17}, {18,18}
    };
    
    private readonly int[,] initialLiveCells1 = new int[,]
    {
        // 滑翔机（左上角）
        {1, 2}, {2, 3}, {3, 1}, {3, 2}, {3, 3}
    };
    
    private readonly int[,] initialLiveCells2 = new int[,]
    {
        // 脉冲星（中心区域）
        {5,7}, {5,8}, {5,9}, {5,10}, {5,11}, {5,12},
        {7,5}, {7,6}, {7,10}, {7,11}, {7,15}, {7,16},
        {8,5}, {8,6}, {8,10}, {8,11}, {8,15}, {8,16},
        {9,5}, {9,6}, {9,10}, {9,11}, {9,15}, {9,16},
        {10,5}, {10,6}, {10,7}, {10,8}, {10,12}, {10,13}, {10,14}, {10,15}, {10,16},
        {11,5}, {11,6}, {11,10}, {11,11}, {11,15}, {11,16},
        {12,5}, {12,6}, {12,10}, {12,11}, {12,15}, {12,16},
        {13,5}, {13,6}, {13,10}, {13,11}, {13,15}, {13,16},
        {15,7}, {15,8}, {15,9}, {15,10}, {15,11}, {15,12}
    };
    
    
    private readonly int[,] initialLiveCells = new int[,]
    {
        // 轻型太空船（右下角）
        {15,16}, {15,19}, {16,15}, {16,19}, {17,15}, {17,19}, {18,16}, {18,17}, {18,18}
    };

    // 控件声明
    private Button btnStart;
    private Button btnReset;
    private DataGridView dataGridView;
    private System.Windows.Forms.Timer timer1;

    public Form1()
    {
        InitializeControls();
        InitializeGridView();
        InitializeLiveCells(); // 加载预设细胞
        btnStart.Enabled = HasLiveCells();
        this.DoubleBuffered = true;
    }

    private void InitializeLiveCells()
    {
        currentBoard = new bool[ROWS, COLS];
        for (int i = 0; i < initialLiveCells.GetLength(0); i++)
        {
            int row = initialLiveCells[i, 0];
            int col = initialLiveCells[i, 1];
            if (row < ROWS && col < COLS)
                currentBoard[row, col] = true;
        }
        UpdateGridDisplay();
    }

    private void InitializeControls()
    {
        // 初始化 DataGridView
        dataGridView = new DataGridView
        {
            Name = "dataGridView",
            Location = new Point(10, 10),
            Size = new Size(400, 400),
            AllowUserToAddRows = false,
            RowHeadersVisible = false,
            ColumnHeadersVisible = false,
            ReadOnly = true,          // 禁止编辑
            GridColor = Color.Silver,
            SelectionMode = DataGridViewSelectionMode.CellSelect,
            DefaultCellStyle = new DataGridViewCellStyle
            {
                SelectionBackColor = Color.Transparent,
                SelectionForeColor = Color.Transparent
            },
            ScrollBars = ScrollBars.None
        };

        // 初始化按钮
        btnStart = new Button
        {
            Text = "Start",
            Location = new Point(420, 10),
            Size = new Size(80, 30),
            BackColor = Color.SteelBlue,
            ForeColor = Color.White
        };
        btnStart.Click += btnStart_Click;

        btnReset = new Button
        {
            Text = "Reset",
            Location = new Point(420, 50),
            Size = new Size(80, 30),
            BackColor = Color.Firebrick,
            ForeColor = Color.White
        };
        btnReset.Click += btnReset_Click;

        // 初始化定时器
        timer1 = new System.Windows.Forms.Timer
        {
            Interval = 200
        };
        timer1.Tick += timer1_Tick;

        // 添加控件
        Controls.Add(dataGridView);
        Controls.Add(btnStart);
        Controls.Add(btnReset);

        // 窗体设置
        ClientSize = new Size(510, 420);
        Text = "Game of Life (Preset Mode)";
        FormBorderStyle = FormBorderStyle.FixedSingle;
        MaximizeBox = false;
    }

    private void InitializeGridView()
    {
        dataGridView.Columns.Clear();
        for (int i = 0; i < COLS; i++)
        {
            dataGridView.Columns.Add(new DataGridViewTextBoxColumn
            {
                Width = 20,
                SortMode = DataGridViewColumnSortMode.NotSortable
            });
        }

        dataGridView.Rows.Add(ROWS);
        foreach (DataGridViewRow row in dataGridView.Rows)
        {
            row.Height = 20;
        }
        UpdateGridDisplay();
    }

    private void UpdateGridDisplay()
    {
        for (int i = 0; i < ROWS; i++)
        {
            for (int j = 0; j < COLS; j++)
            {
                dataGridView[j, i].Style.BackColor = currentBoard[i, j] ? 
                    Color.Gold : Color.FromArgb(64, 64, 64);
            }
        }
        dataGridView.Refresh();
    }

    private int CountNeighbors(int x, int y)
    {
        int count = 0;
        for (int i = -1; i <= 1; i++)
        {
            for (int j = -1; j <= 1; j++)
            {
                if (i == 0 && j == 0) continue;

                int nx = x + i;
                int ny = y + j;

                if (nx >= 0 && nx < ROWS && ny >= 0 && ny < COLS)
                {
                    if (currentBoard[nx, ny]) count++;
                }
            }
        }
        return count;
    }

    private void NextGeneration()
    {
        bool[,] newBoard = new bool[ROWS, COLS];
        for (int i = 0; i < ROWS; i++)
        {
            for (int j = 0; j < COLS; j++)
            {
                int neighbors = CountNeighbors(i, j);
                newBoard[i, j] = currentBoard[i, j] ? 
                    (neighbors == 2 || neighbors == 3) : 
                    (neighbors == 3);
            }
        }
        currentBoard = newBoard;
        if (!HasLiveCells()) StopSimulation();
    }

    private bool HasLiveCells()
    {
        foreach (bool cell in currentBoard)
        {
            if (cell) return true;
        }
        return false;
    }

    private void StopSimulation()
    {
        timer1.Stop();
        isSimulating = false;
        btnStart.Text = "Start";
        btnStart.Enabled = HasLiveCells();
    }

    // 事件处理
    private void btnStart_Click(object sender, EventArgs e)
    {
        if (!isSimulating)
        {
            isSimulating = true;
            btnStart.Text = "Stop";
            timer1.Start();
        }
        else
        {
            StopSimulation();
        }
    }

    private void timer1_Tick(object sender, EventArgs e)
    {
        NextGeneration();
        UpdateGridDisplay();
    }

    private void btnReset_Click(object sender, EventArgs e)
    {
        InitializeLiveCells(); // 重置到初始配置
        StopSimulation();
    }
}
```

end