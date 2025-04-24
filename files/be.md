说明:
forms实现推箱子小游戏
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/fe5a0505765a4f02be76e730a0e6bc36.png#pic_center)

step0:游戏规则

```bash
# 推箱子游戏规则说明

## 🎯 游戏目标
- 通过控制角色移动，将所有**棕色箱子(3)**推到**红色目标点(4)**上
- 当所有箱子都变为**绿色(7)**时，即完成当前关卡
- 完成全部关卡后触发通关动画

## 🕹️ 操作方式
- 点击右侧方向按钮控制移动：
- 通过输入框直接跳转关卡：
1. 在右侧文本框中输入关卡编号(0开始)
2. 点击"加载关卡"按钮

## ⚙️ 基本规则
1. **角色移动**
 - 可自由穿过空地和目标点
 - 无法穿越墙体(灰色格子)
 - 在目标点上时显示为浅蓝色(6)

2. **箱子推动**
 - 仅能向前直线推动
 - 需满足推动条件：
   - 箱子前方是空地(0)或目标点(4)
   - 前方无其他箱子或墙体
 - 箱子到达目标点时变为绿色(7)

3. **地形限制**
 - 墙体形成游戏区域边界
 - 目标点可重复使用
 - 允许多个箱子叠放在相邻目标点

## 🏆 胜利条件
- 完成条件：当前关卡中**所有普通箱子(3)消失**，仅保留绿色箱子(7)
- 通关反馈：
- 弹出对话框提示关卡完成
- 可选择直接进入下一关
- 最终关卡完成后显示全屏庆祝动画

## 🗺️ 关卡系统
- 关卡总数：`mapData.GetLength(0)` 个预设关卡
- 进度机制：
- 支持任意关卡重复挑战
- 自动记录当前所在关卡
- 通关后解锁下一关卡

## 🎨 界面元素
| 颜色         | 图标 | 说明                |
|--------------|------|---------------------|
| 🟦 浅蓝色     | ⑥    | 角色位于目标点       |
| 🟦 蓝色       | ②    | 普通角色状态         |
| 🟫 棕色       | ③    | 可移动的箱子         |
| 🟩 绿色       | ⑦    | 已完成的箱子         |
| 🟥 红色       | ④    | 空目标点             |
| 🟪 紫色(示例)| ①    | 不可穿越的墙体       |
| ⬜ 白色       | ⓪    | 可通行的空地         |

## 💡 操作提示
1. **路径规划**  
 移动前观察多个箱子的位置关系，优先处理"卡死位"的箱子

2. **死角规避**  
 避免将箱子推入以下位置：
 - 两面墙体形成的直角
 - 地图边缘非目标点区域
 - 其他箱子背后的封闭空间

3. **复位技巧**  
 当箱子被卡死时：
 - 点击加载关卡按钮
 - 输入当前关卡编号
 - 重新开始本关

## ⚠️ 注意事项
1. 输入关卡号时需满足：`0 ≤ 关卡号 < 总关卡数`
2. 最后一个箱子推到目标点时会立即触发胜利判断
3. 角色移动后即时渲染，无需手动刷新界面
4. 支持窗口自适应缩放，但建议保持默认窗口尺寸
```

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp28\WinFormsApp28\Form1.cs

```csharp
using System;
using System.Drawing;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp28
{
    public partial class Form1 : Form
    {
          private int[,,] mapData = new int[,,]
        {
            {
                {0,0,1,1,1,0,0,0,0,0,0},
                {0,0,1,4,1,0,0,0,0,0,0},
                {0,0,1,0,1,1,1,1,0,0,0},
                {1,1,1,3,0,3,4,1,0,0,0},
                {1,4,0,3,2,1,1,1,0,0,0},
                {1,1,1,1,3,1,0,0,0,0,0},
                {0,0,0,1,4,1,0,0,0,0,0},
                {0,0,0,1,1,1,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0}
            },
            {
                {0,1,1,1,1,1,0,0,0,0,0},
                {0,1,2,0,1,1,1,0,0,0,0},
                {0,1,0,3,0,0,1,0,0,0,0},
                {1,1,1,0,1,0,1,1,0,0,0},
                {1,4,1,0,1,0,0,1,0,0,0},
                {1,4,3,0,0,1,0,1,0,0,0},
                {1,4,0,0,0,3,0,1,0,0,0},
                {1,1,1,1,1,1,1,1,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0}
            },
            {
                {0,1,1,1,1,0,0,0,0,0,0},
                {1,1,0,0,1,0,0,0,0,0,0},
                {1,2,3,0,1,0,0,0,0,0,0},
                {1,1,3,0,1,1,0,0,0,0,0},
                {1,1,0,3,0,1,0,0,0,0,0},
                {1,4,3,0,0,1,0,0,0,0,0},
                {1,4,4,7,4,1,0,0,0,0,0},
                {1,1,1,1,1,1,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0}
            },
            {
                {1,1,1,1,1,0,0,0,0,0,0},
                {1,2,0,0,1,0,0,0,0,0,0},
                {1,0,3,3,1,0,1,1,1,0,0},
                {1,0,3,0,1,0,1,4,1,0,0},
                {1,1,1,0,1,1,1,4,1,0,0},
                {0,1,1,0,0,0,0,4,1,0,0},
                {0,1,0,0,0,1,0,0,1,0,0},
                {0,1,0,0,0,1,1,1,1,0,0},
                {0,1,1,1,1,1,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0}
            },
            {
                {0,1,1,1,1,1,1,1,0,0,0},
                {0,1,0,0,0,0,0,1,1,1,0},
                {1,1,3,1,1,1,0,0,0,1,0},
                {1,0,2,0,3,0,0,3,0,1,0},
                {1,0,4,4,1,0,3,0,1,1,0},
                {1,1,4,4,1,0,0,0,1,0,0},
                {0,1,1,1,1,1,1,1,1,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0}
            },
            {
                {0,0,0,1,1,1,1,1,1,1,0},
                {0,0,1,1,0,0,1,0,2,1,0},
                {0,0,1,0,0,0,1,0,0,1,0},
                {0,0,1,3,0,3,0,3,0,1,0},
                {0,0,1,0,3,1,1,0,0,1,0},
                {1,1,1,0,3,0,1,0,1,1,0},
                {1,4,4,4,4,4,0,0,1,0,0},
                {1,1,1,1,1,1,1,1,1,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0},
                {0,0,0,0,0,0,0,0,0,0,0}
            }
        };

        private int[,] currentMap = new int[11, 11];
        private Panel[,] panels = new Panel[11, 11];
        private int currentLevel = 0;
        private const int CellSize = 40;
        private TextBox levelTextBox;

        public Form1()
        {
            InitializeComponent();
            InitializeUI();
            InitializeGame();

        }

        private void InitializeGame()
        {
            LoadLevel(0);
            InitializePanels();
        }

        private void InitializeUI()
        {
            // 窗体设置
            this.Text = "推箱子游戏";
            this.ClientSize = new Size(11 * CellSize + 150, 11 * CellSize);
            this.DoubleBuffered = true;

            // 关卡输入控件
            levelTextBox = new TextBox
            {
                Location = new Point(11 * CellSize + 20, 20),
                Size = new Size(60, 20),
                Text = currentLevel.ToString()
            };
            this.Controls.Add(levelTextBox);

            // 加载关卡按钮
            var loadButton = new Button
            {
                Text = "加载关卡",
                Location = new Point(11 * CellSize + 20, 50),
                Size = new Size(80, 30)
            };
            loadButton.Click += LoadLevelButton_Click;
            this.Controls.Add(loadButton);

            // 方向控制按钮
            CreateControlButton("↑", 60, 100, (s, e) => MovePlayer(-1, 0));
            CreateControlButton("↓", 60, 180, (s, e) => MovePlayer(1, 0));
            CreateControlButton("←", 20, 140, (s, e) => MovePlayer(0, -1));
            CreateControlButton("→", 100, 140, (s, e) => MovePlayer(0, 1));
        }

        private void InitializePanels()
        {
            for (int i = 0; i < 11; i++)
            {
                for (int j = 0; j < 11; j++)
                {
                    panels[i, j] = new Panel
                    {
                        Size = new Size(CellSize, CellSize),
                        Location = new Point(j * CellSize, i * CellSize),
                        BorderStyle = BorderStyle.FixedSingle
                    };
                    this.Controls.Add(panels[i, j]);
                    panels[i, j].BringToFront();
                }
            }
            UpdateMapDisplay();
        }

        private void CreateControlButton(string text, int x, int y, EventHandler handler)
        {
            var btn = new Button
            {
                Text = text,
                Size = new Size(40, 40),
                Location = new Point(11 * CellSize + x, y),
                Font = new Font("Microsoft YaHei", 12f)
            };
            btn.Click += handler;
            this.Controls.Add(btn);
        }

        private void LoadLevel(int level)
        {
            if (level < 0 || level >= mapData.GetLength(0)) return;
            
            for (int i = 0; i < 11; i++)
            {
                for (int j = 0; j < 11; j++)
                {
                    currentMap[i, j] = mapData[level, i, j];
                }
            }
            currentLevel = level;
            levelTextBox.Text = level.ToString();
        }

        private void UpdateMapDisplay()
        {
            for (int i = 0; i < 11; i++)
            {
                for (int j = 0; j < 11; j++)
                {
                    SetCellAppearance(panels[i, j], currentMap[i, j]);
                }
            }
        }

        private void SetCellAppearance(Panel panel, int cellType)
        {
            panel.BackColor = cellType switch
            {
                0 => Color.White,    // 空地
                1 => Color.Gray,     // 墙
                2 => Color.Blue,     // 玩家
                3 => Color.Brown,    // 箱子
                4 => Color.Red,      // 目标点
                6 => Color.LightBlue,// 玩家在目标点
                7 => Color.Green,    // 箱子在目标点
                _ => Color.White
            };
        }

        private void MovePlayer(int dx, int dy)
        {
            if (TryMovePlayer(dx, dy))
            {
                UpdateMapDisplay();
                if (CheckWin()) HandleLevelCompletion();
            }
        }

        private bool TryMovePlayer(int dx, int dy)
        {
            var (x, y) = FindPlayer();
            int newX = x + dx;
            int newY = y + dy;

            if (!IsValidPosition(newX, newY)) return false;

            int targetCell = currentMap[newX, newY];
            if (targetCell == 1) return false; // 撞墙

            // 处理箱子移动
            if (targetCell == 3 || targetCell == 7)
            {
                int boxX = newX + dx;
                int boxY = newY + dy;
                if (!IsValidPosition(boxX, boxY) || 
                    currentMap[boxX, boxY] == 1 || 
                    currentMap[boxX, boxY] == 3 || 
                    currentMap[boxX, boxY] == 7) return false;

                currentMap[boxX, boxY] = currentMap[boxX, boxY] == 4 ? 7 : 3;
            }

            // 更新玩家位置
            currentMap[newX, newY] = currentMap[newX, newY] switch
            {
                4 => 6,
                7 => 6,
                _ => 2
            };

            // 清除原位置
            currentMap[x, y] = currentMap[x, y] switch
            {
                6 => 4,
                _ => 0
            };

            return true;
        }

        private (int X, int Y) FindPlayer()
        {
            for (int i = 0; i < 11; i++)
                for (int j = 0; j < 11; j++)
                    if (currentMap[i, j] == 2 || currentMap[i, j] == 6)
                        return (i, j);
            return (-1, -1);
        }

        private bool IsValidPosition(int x, int y) => x >= 0 && x < 11 && y >= 0 && y < 11;

        private bool CheckWin()
        {
            foreach (int cell in currentMap)
                if (cell == 3) return false;
            return true;
        }

        private void HandleLevelCompletion()
        {
            if (currentLevel == mapData.GetLength(0) - 1)
            {
                MessageBox.Show("恭喜通关所有关卡！");
                return;
            }

            if (MessageBox.Show("进入下一关？", "关卡完成", MessageBoxButtons.YesNo) == DialogResult.Yes)
            {
                LoadLevel(currentLevel + 1);
                UpdateMapDisplay();
            }
        }

        private void LoadLevelButton_Click(object sender, EventArgs e)
        {
            if (int.TryParse(levelTextBox.Text, out int level))
            {
                if (level >= 0 && level < mapData.GetLength(0))
                {
                    LoadLevel(level);
                    UpdateMapDisplay();
                }
                else
                {
                    MessageBox.Show($"请输入0-{mapData.GetLength(0) - 1}之间的关卡号");
                }
            }
            else
            {
                MessageBox.Show("请输入有效的数字");
            }
        }
    }
}
```

end