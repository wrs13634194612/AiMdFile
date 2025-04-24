说明：
我希望用angular+form实现2048小游戏
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/66dc2326fa5b45e787c176396bc5f6a0.png#pic_center)

angular：
step1:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\num-game\num-game.component.ts

```typescript
import { Component, HostListener, OnInit } from '@angular/core';
import {NgForOf, NgIf, NgStyle} from '@angular/common';

interface Tile {
  value: number;
  merged: boolean;
}


@Component({
  selector: 'app-num-game',
  imports: [
    NgForOf,
    NgStyle,
    NgIf
  ],
  templateUrl: './num-game.component.html',
  styleUrl: './num-game.component.css'
})
export class NumGameComponent implements OnInit {
  gridSize = 4;
  grid: Tile[][] = [];
  score = 0;
  gameOver = false;
  bestScore = 0;

  private colorMap = new Map<number, string>([
    [0, '#ccc0b3'],
    [2, '#eee4da'],
    [4, '#ede0c8'],
    [8, '#f2b179'],
    [16, '#f59563'],
    [32, '#f67c5f'],
    [64, '#f65e3b'],
    [128, '#edcf72'],
    [256, '#edcc61'],
    [512, '#edc850'],
    [1024, '#edc53f'],
    [2048, '#edc22e']
  ]);

  ngOnInit(): void {
    this.initializeGame();
  }

  initializeGame(): void {
    this.grid = Array.from({ length: this.gridSize }, () =>
      Array.from({ length: this.gridSize }, () => ({ value: 0, merged: false }))
    );
    this.score = 0;
    this.gameOver = false;
    this.addNewTile();
    this.addNewTile();
  }

  addNewTile(): void {
    const emptyCells: { x: number; y: number }[] = [];
    for (let i = 0; i < this.gridSize; i++) {
      for (let j = 0; j < this.gridSize; j++) {
        if (this.grid[i][j].value === 0) {
          emptyCells.push({ x: i, y: j });
        }
      }
    }

    if (emptyCells.length > 0) {
      const { x, y } = emptyCells[Math.floor(Math.random() * emptyCells.length)];
      this.grid[x][y] = {
        value: Math.random() < 0.9 ? 2 : 4,
        merged: false
      };
    }
  }

  @HostListener('window:keydown', ['$event'])
  handleKeyboardEvent(event: KeyboardEvent): void {
    if (this.gameOver) return;

    let moved = false;
    switch (event.key) {
      case 'ArrowLeft':
        moved = this.moveLeft();
        break;
      case 'ArrowRight':
        moved = this.moveRight();
        break;
      case 'ArrowUp':
        moved = this.moveUp();
        break;
      case 'ArrowDown':
        moved = this.moveDown();
        break;
      default:
        return;
    }

    if (moved) {
      this.addNewTile();
      this.bestScore = Math.max(this.score, this.bestScore);
      if (this.checkGameOver()) {
        this.gameOver = true;
      }
    }
  }

  moveLeft(): boolean {
    let moved = false;
    for (const row of this.grid) {
      const result = this.mergeRow(row);
      if (!this.areRowsEqual(row, result)) {
        moved = true;
        row.splice(0, this.gridSize, ...result);
      }
    }
    return moved;
  }

  moveRight(): boolean {
    let moved = false;
    for (const row of this.grid) {
      row.reverse();
      const result = this.mergeRow(row);
      result.reverse();
      if (!this.areRowsEqual(row, result)) {
        moved = true;
        row.splice(0, this.gridSize, ...result);
      }
    }
    return moved;
  }

  moveUp(): boolean {
    this.transposeGrid();
    const moved = this.moveLeft();
    this.transposeGrid();
    return moved;
  }

  moveDown(): boolean {
    this.transposeGrid();
    const moved = this.moveRight();
    this.transposeGrid();
    return moved;
  }

  private mergeRow(row: Tile[]): Tile[] {
    const newRow: Tile[] = [];
    let previous: Tile | null = null;

    for (const current of row.filter(t => t.value !== 0)) {
      if (previous && !previous.merged && previous.value === current.value) {
        newRow[newRow.length - 1] = {
          value: previous.value * 2,
          merged: true
        };
        this.score += newRow[newRow.length - 1].value;
        previous = null;
      } else {
        newRow.push({ ...current, merged: false });
        previous = newRow[newRow.length - 1];
      }
    }

    while (newRow.length < this.gridSize) {
      newRow.push({ value: 0, merged: false });
    }
    return newRow;
  }

  private transposeGrid(): void {
    this.grid = this.grid[0].map((_, i) =>
      this.grid.map(row => row[i])
    );
  }

  private areRowsEqual(a: Tile[], b: Tile[]): boolean {
    return a.every((tile, i) => tile.value === b[i].value);
  }

  private checkGameOver(): boolean {
    // Check for empty cells
    if (this.grid.some(row => row.some(cell => cell.value === 0))) {
      return false;
    }

    // Check horizontal merges
    for (let i = 0; i < this.gridSize; i++) {
      for (let j = 0; j < this.gridSize - 1; j++) {
        if (this.grid[i][j].value === this.grid[i][j + 1].value) {
          return false;
        }
      }
    }

    // Check vertical merges
    for (let j = 0; j < this.gridSize; j++) {
      for (let i = 0; i < this.gridSize - 1; i++) {
        if (this.grid[i][j].value === this.grid[i + 1][j].value) {
          return false;
        }
      }
    }

    return true;
  }

  getTileStyle(value: number): { [key: string]: string } {
    return {
      'background-color': this.colorMap.get(value) || '#000000',
      'color': value > 4 ? '#f9f6f2' : '#776e65',
      'font-size': value >= 100 ? '20px' : '24px'
    };
  }

  restartGame(): void {
    this.initializeGame();
  }
}

```

step2:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\num-game\num-game.component.html

```xml
<!-- game.component.html -->
<div class="game-container">
  <div class="header">
    <div class="scores">
      <div class="score-box">
        <div class="score-label">SCORE</div>
        <div class="score-value">{{ score }}</div>
      </div>
      <div class="score-box">
        <div class="score-label">BEST</div>
        <div class="score-value">{{ bestScore }}</div>
      </div>
    </div>
    <button class="new-game-btn" (click)="restartGame()">New Game</button>
  </div>

  <div class="grid">
    <div class="grid-row" *ngFor="let row of grid">
      <div
        *ngFor="let cell of row"
        class="grid-cell"
        [ngStyle]="getTileStyle(cell.value)">
        {{ cell.value || '' }}
      </div>
    </div>
  </div>

  <div class="game-over-overlay" *ngIf="gameOver">
    <div class="game-over-message">
      <h2>Game Over!</h2>
      <button class="try-again-btn" (click)="restartGame()">Try Again</button>
    </div>
  </div>
</div>

```

step3:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\num-game\num-game.component.css

```css
/* game.component.css */
.game-container {
  width: 480px;
  margin: 0 auto;
  font-family: 'Arial Rounded', Arial, sans-serif;
}

.header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.scores {
  display: flex;
  gap: 10px;
}

.score-box {
  background: #bbada0;
  padding: 10px 20px;
  border-radius: 5px;
  text-align: center;
}

.score-label {
  color: #eee4da;
  font-size: 12px;
  text-transform: uppercase;
}

.score-value {
  color: white;
  font-size: 24px;
  font-weight: bold;
}

.new-game-btn {
  background: #8f7a66;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.2s;
}

.new-game-btn:hover {
  background: #9c8a7b;
}

.grid {
  background: #bbada0;
  padding: 15px;
  border-radius: 10px;
  position: relative;
}

.grid-row {
  display: flex;
  margin-bottom: 15px;
}

.grid-row:last-child {
  margin-bottom: 0;
}

.grid-cell {
  width: 100px;
  height: 100px;
  background: rgba(238, 228, 218, 0.35);
  border-radius: 5px;
  margin-right: 15px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  font-size: 24px;
  transition: all 0.15s ease;
}

.grid-cell:last-child {
  margin-right: 0;
}

.game-over-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(238, 228, 218, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 10px;
}

.game-over-message {
  background: #8f7a66;
  padding: 20px 40px;
  border-radius: 5px;
  text-align: center;
  color: white;
}

.try-again-btn {
  background: white;
  color: #8f7a66;
  border: none;
  padding: 10px 20px;
  border-radius: 3px;
  margin-top: 10px;
  cursor: pointer;
  font-weight: bold;
}

```

///分割线
form：
step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp3\WinFormsApp3\Form1.cs

```csharp
namespace WinFormsApp3;

public class Tile
{
    public int Value { get; set; }
    public bool Merged { get; set; }
}
public partial class Form1 : Form
{
    private const int GridSize = 4;
    private const int CellSize = 100;
    private const int Spacing = 15;
        
    private Tile[,] grid = new Tile[GridSize, GridSize];
    private int score = 0;
    private int bestScore = 0;
    private bool gameOver = false;
        
    private Dictionary<int, Color> colorMap = new Dictionary<int, Color>
    {
        {0, ColorTranslator.FromHtml("#ccc0b3")},
        {2, ColorTranslator.FromHtml("#eee4da")},
        {4, ColorTranslator.FromHtml("#ede0c8")},
        {8, ColorTranslator.FromHtml("#f2b179")},
        {16, ColorTranslator.FromHtml("#f59563")},
        {32, ColorTranslator.FromHtml("#f67c5f")},
        {64, ColorTranslator.FromHtml("#f65e3b")},
        {128, ColorTranslator.FromHtml("#edcf72")},
        {256, ColorTranslator.FromHtml("#edcc61")},
        {512, ColorTranslator.FromHtml("#edc850")},
        {1024, ColorTranslator.FromHtml("#edc53f")},
        {2048, ColorTranslator.FromHtml("#edc22e")}
    };

    // UI Controls
    private Panel panelGrid;
    private Label lblScore;
    private Label lblBest;
    private Button btnNewGame;
    private Panel panelGameOver;
    public Form1()
    {
        InitializeComponent();
     
            // Header
            var headerPanel = new Panel
            {
                Location = new Point(20, 20),
                Size = new Size(440, 70)
            };

            var scoresPanel = new Panel
            {
                Size = new Size(200, 70),
                Location = new Point(0, 0)
            };

            lblScore = new Label
            {
                Text = "0",
                Font = new Font("Arial", 24, FontStyle.Bold),
                ForeColor = Color.Black,
                Location = new Point(10, 30),
                Size = new Size(90, 40)
            };

            lblBest = new Label
            {
                Text = "0",
                Font = new Font("Arial", 24, FontStyle.Bold),
                ForeColor = Color.Black,
                Location = new Point(110, 30),
                Size = new Size(90, 40)
            };

            btnNewGame = new Button
            {
                Text = "New Game",
                Size = new Size(120, 40),
                Location = new Point(320, 15),
                BackColor = ColorTranslator.FromHtml("#8f7a66"),
                ForeColor = Color.Black,
                FlatStyle = FlatStyle.Flat
            };
            btnNewGame.Click += btnNewGame_Click;

            // Game Grid
            panelGrid = new Panel
            {
                Size = new Size(
                    GridSize * CellSize + (GridSize + 1) * Spacing,
                    GridSize * CellSize + (GridSize + 1) * Spacing),
                Location = new Point(20, 100),
                BackColor = ColorTranslator.FromHtml("#bbada0")
            };

            // Game Over Panel
            panelGameOver = new Panel
            {
                Size = panelGrid.Size,
                Location = panelGrid.Location,
                BackColor = Color.FromArgb(128, 238, 228, 218),
                Visible = false
            };

            var gameOverLabel = new Label
            {
                Text = "Game Over!",
                Font = new Font("Arial", 24, FontStyle.Bold),
                ForeColor = ColorTranslator.FromHtml("#8f7a66"),
                AutoSize = true,
                Location = new Point(120, 150)
            };

            var btnTryAgain = new Button
            {
                Text = "Try Again",
                Size = new Size(120, 40),
                Location = new Point(160, 200),
                BackColor = Color.Black,
                ForeColor = ColorTranslator.FromHtml("#8f7a66"),
                FlatStyle = FlatStyle.Flat
            };
            btnTryAgain.Click += btnTryAgain_Click;

            panelGameOver.Controls.Add(gameOverLabel);
            panelGameOver.Controls.Add(btnTryAgain);

            // Add controls
            headerPanel.Controls.Add(scoresPanel);
            headerPanel.Controls.Add(btnNewGame);
            scoresPanel.Controls.Add(new Label { Text = "SCORE", Location = new Point(10, 5), ForeColor = ColorTranslator.FromHtml("#eee4da") });
            scoresPanel.Controls.Add(new Label { Text = "BEST", Location = new Point(110, 5), ForeColor = ColorTranslator.FromHtml("#eee4da") });
            scoresPanel.Controls.Add(lblScore);
            scoresPanel.Controls.Add(lblBest);
            
            this.Controls.Add(headerPanel);
            this.Controls.Add(panelGrid);
            this.Controls.Add(panelGameOver);
            InitializeGame();
            this.KeyPreview = true;
            this.Text = "2048 Game";
            this.BackColor = ColorTranslator.FromHtml("#faf8ef");
            this.ClientSize = new Size(480, 600);
            // 添加以下代码确保窗体获得焦点
            this.Load += (sender, e) => this.Focus();
    }
    
    
 

        private void InitializeGame()
        {
            for (int i = 0; i < GridSize; i++)
            {
                for (int j = 0; j < GridSize; j++)
                {
                    grid[i, j] = new Tile { Value = 0, Merged = false };
                }
            }
            
            score = 0;
            gameOver = false;
            AddNewTile();
            AddNewTile();
            UpdateUI();
        }

        private void AddNewTile()
        {
            var emptyCells = new List<Point>();
            for (int i = 0; i < GridSize; i++)
            {
                for (int j = 0; j < GridSize; j++)
                {
                    if (grid[i, j].Value == 0)
                    {
                        emptyCells.Add(new Point(i, j));
                    }
                }
            }

            if (emptyCells.Count > 0)
            {
                var random = new Random();
                var cell = emptyCells[random.Next(emptyCells.Count)];
                grid[cell.X, cell.Y].Value = random.NextDouble() < 0.9 ? 2 : 4;
            }
        }

        private void UpdateUI()
        {
            lblScore.Text = score.ToString();
            lblBest.Text = bestScore.ToString();

            panelGrid.Controls.Clear();
            for (int i = 0; i < GridSize; i++)
            {
                for (int j = 0; j < GridSize; j++)
                {
                    var cell = new Panel
                    {
                        Width = CellSize,
                        Height = CellSize,
                        BackColor = colorMap[grid[i, j].Value],
                        Location = new Point(
                            j * (CellSize + Spacing) + Spacing,
                            i * (CellSize + Spacing) + Spacing),
                        Font = new Font("Arial", grid[i, j].Value >= 100 ? 20 : 24, FontStyle.Bold),
                        ForeColor = grid[i, j].Value > 4 ? ColorTranslator.FromHtml("#f9f6f2") 
                                                       : ColorTranslator.FromHtml("#776e65")
                    };

                    var label = new Label
                    {
                        Text = grid[i, j].Value > 0 ? grid[i, j].Value.ToString() : "",
                        Dock = DockStyle.Fill,
                        TextAlign = ContentAlignment.MiddleCenter
                    };

                    cell.Controls.Add(label);
                    panelGrid.Controls.Add(cell);
                }
            }

            panelGameOver.Visible = gameOver;
        }

        #region Movement Logic
        protected override void OnKeyDown(KeyEventArgs e)
        {
            base.OnKeyDown(e);
            if (gameOver) return;

            bool moved = false;
            switch (e.KeyCode)
            {
                case Keys.A:
                    moved = MoveLeft();
                    break;
                case Keys.D:
                    moved = MoveRight();
                    break;
                case Keys.W:
                    moved = MoveUp();
                    break;
                case Keys.S:
                    moved = MoveDown();
                    break;
                default:
                    return;
            }

            if (moved)
            {
                AddNewTile();
                bestScore = Math.Max(score, bestScore);
                gameOver = CheckGameOver();
                UpdateUI();
            }
        }

        private bool MoveLeft()
        {
            bool moved = false;
            for (int i = 0; i < GridSize; i++)
            {
                var row = new Tile[GridSize];
                for (int j = 0; j < GridSize; j++) row[j] = grid[i, j];
                var mergedRow = MergeRow(row);
                
                if (!AreRowsEqual(row, mergedRow))
                {
                    moved = true;
                    for (int j = 0; j < GridSize; j++) grid[i, j] = mergedRow[j];
                }
            }
            return moved;
        }

        private bool MoveRight()
        {
            bool moved = false;
            for (int i = 0; i < GridSize; i++)
            {
                var row = new Tile[GridSize];
                for (int j = 0; j < GridSize; j++) row[j] = grid[i, GridSize - 1 - j];
                var mergedRow = MergeRow(row);
                
                if (!AreRowsEqual(row, mergedRow))
                {
                    moved = true;
                    for (int j = 0; j < GridSize; j++) grid[i, GridSize - 1 - j] = mergedRow[j];
                }
            }
            return moved;
        }

        private bool MoveUp()
        {
            Transpose();
            bool moved = MoveLeft();
            Transpose();
            return moved;
        }

        private bool MoveDown()
        {
            Transpose();
            bool moved = MoveRight();
            Transpose();
            return moved;
        }

        private Tile[] MergeRow(Tile[] row)
        {
            var result = new List<Tile>();
            Tile previous = null;

            foreach (var current in row)
            {
                if (current.Value == 0) continue;

                if (previous != null && !previous.Merged && previous.Value == current.Value)
                {
                    result[result.Count - 1] = new Tile 
                    { 
                        Value = previous.Value * 2, 
                        Merged = true 
                    };
                    score += result[result.Count - 1].Value;
                    previous = null;
                }
                else
                {
                    var tile = new Tile { Value = current.Value, Merged = false };
                    result.Add(tile);
                    previous = tile;
                }
            }

            while (result.Count < GridSize)
                result.Add(new Tile { Value = 0, Merged = false });

            return result.ToArray();
        }
        #endregion

        #region Helper Methods
        private void Transpose()
        {
            var newGrid = new Tile[GridSize, GridSize];
            for (int i = 0; i < GridSize; i++)
                for (int j = 0; j < GridSize; j++)
                    newGrid[j, i] = grid[i, j];
            grid = newGrid;
        }

        private bool AreRowsEqual(Tile[] a, Tile[] b)
        {
            for (int i = 0; i < GridSize; i++)
                if (a[i].Value != b[i].Value) return false;
            return true;
        }

        private bool CheckGameOver()
        {
            // Check empty cells
            for (int i = 0; i < GridSize; i++)
                for (int j = 0; j < GridSize; j++)
                    if (grid[i, j].Value == 0) return false;

            // Check possible merges
            for (int i = 0; i < GridSize; i++)
                for (int j = 0; j < GridSize; j++)
                {
                    if (i < GridSize - 1 && grid[i, j].Value == grid[i + 1, j].Value)
                        return false;
                    if (j < GridSize - 1 && grid[i, j].Value == grid[i, j + 1].Value)
                        return false;
                }

            return true;
        }
        #endregion

        #region Event Handlers
        private void btnNewGame_Click(object sender, EventArgs e)
        {
            InitializeGame();
        }

        private void btnTryAgain_Click(object sender, EventArgs e)
        {
            InitializeGame();
        }
        #endregion


    
}
```

end