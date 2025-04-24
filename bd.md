说明：
在五子棋领域，人类输给了ai，
c++算法实现ai五子棋
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3ec03265e60543f3a301b1a09953007f.png#pic_center)

step1:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeLists.txt

```bash
# CMakeList.txt: CMakeProject1 的 CMake 项目，在此处包括源代码并定义
# 项目特定的逻辑。
#

# 将源代码添加到此项目的可执行文件。
add_executable (CMakeProject1 "CMakeProject1.cpp" "CMakeProject1.h")

if (CMAKE_VERSION VERSION_GREATER 3.12)
  set_property(TARGET CMakeProject1 PROPERTY CXX_STANDARD 20)
endif()

# TODO: 如有需要，请添加测试并安装目标。

```

step2:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

const int BOARD_SIZE =12;
enum Piece { EMPTY, BLACK, WHITE };

Piece board[BOARD_SIZE][BOARD_SIZE] = { EMPTY };

struct Move {
    int x, y, score;
    Move(int x = -1, int y = -1, int score = 0) : x(x), y(y), score(score) {}
};

void printBoard() {
    cout << "  ";
    for (int x = 0; x < BOARD_SIZE; x++) {
        cout << x % 10 << " "; // 用取模处理两位数坐标
    }
    cout << endl;

    for (int y = BOARD_SIZE - 1; y >= 0; y--) {
        cout << y % 10 << " ";
        for (int x = 0; x < BOARD_SIZE; x++) {
            if (board[x][y] == BLACK) cout << "B ";
            else if (board[x][y] == WHITE) cout << "W ";
            else cout << ". ";
        }
        cout << y % 10 << endl;
    }

    cout << "  ";
    for (int x = 0; x < BOARD_SIZE; x++) {
        cout << x % 10 << " ";
    }
    cout << "\n" << endl;
}

bool checkWin(int x, int y, Piece player) {
    const int directions[4][2] = { {1,0}, {0,1}, {1,1}, {1,-1} };

    for (auto [dx, dy] : directions) {
        int count = 1;

        // 正方向检查
        int px = x + dx, py = y + dy;
        while (px >= 0 && px < BOARD_SIZE && py >= 0 && py < BOARD_SIZE && board[px][py] == player) {
            count++;
            px += dx;
            py += dy;
        }

        // 负方向检查
        px = x - dx;
        py = y - dy;
        while (px >= 0 && px < BOARD_SIZE && py >= 0 && py < BOARD_SIZE && board[px][py] == player) {
            count++;
            px -= dx;
            py -= dy;
        }

        if (count >= 5) return true;
    }
    return false;
}

int evaluateDirection(int x, int y, int dx, int dy, Piece player) {
    int total = 1;
    bool leftOpen = false, rightOpen = false;

    // 正方向检查
    int px = x + dx, py = y + dy;
    int consecutive = 0;
    while (px >= 0 && px < BOARD_SIZE && py >= 0 && py < BOARD_SIZE && board[px][py] == player) {
        consecutive++;
        px += dx;
        py += dy;
    }
    total += consecutive;
    rightOpen = (px >= 0 && px < BOARD_SIZE && py >= 0 && py < BOARD_SIZE && board[px][py] == EMPTY);

    // 负方向检查
    px = x - dx;
    py = y - dy;
    consecutive = 0;
    while (px >= 0 && px < BOARD_SIZE && py >= 0 && py < BOARD_SIZE && board[px][py] == player) {
        consecutive++;
        px -= dx;
        py -= dy;
    }
    total += consecutive;
    leftOpen = (px >= 0 && px < BOARD_SIZE && py >= 0 && py < BOARD_SIZE && board[px][py] == EMPTY);

    if (total >= 5) return 1000000;
    if (total == 4) {
        if (leftOpen && rightOpen) return 100000; // 活四
        if (leftOpen || rightOpen) return 10000;  // 冲四
        return 0;
    }
    if (total == 3) {
        if (leftOpen && rightOpen) return 5000;    // 活三
        if (leftOpen || rightOpen) return 1000;    // 冲三
        return 0;
    }
    if (total == 2) {
        if (leftOpen && rightOpen) return 1000;    // 活二
        if (leftOpen || rightOpen) return 500;     // 冲二
        return 0;
    }
    return 0;
}

int evaluatePosition(int x, int y, Piece player) {
    if (board[x][y] != EMPTY) return -1;

    int score = 0;
    const int directions[4][2] = { {1,0}, {0,1}, {1,1}, {1,-1} };

    // 进攻得分（当前玩家下这里的价值）
    int attackScore = 0;
    for (auto [dx, dy] : directions) {
        attackScore += evaluateDirection(x, y, dx, dy, player);
    }

    // 防守得分（对手下这里的威胁）
    Piece opponent = (player == BLACK) ? WHITE : BLACK;
    int defendScore = 0;
    for (auto [dx, dy] : directions) {
        defendScore += evaluateDirection(x, y, dx, dy, opponent);
    }

    return attackScore + defendScore * 1.2; // 防守权重略高
}

Move findBestMove(Piece aiPlayer) {
    Move bestMove;
    int maxScore = -1;

    for (int x = 0; x < BOARD_SIZE; x++) {
        for (int y = 0; y < BOARD_SIZE; y++) {
            if (board[x][y] == EMPTY) {
                int score = evaluatePosition(x, y, aiPlayer);
                if (score > maxScore || (score == maxScore && rand() % 2)) {
                    maxScore = score;
                    bestMove = Move(x, y, score);
                }
            }
        }
    }
    return bestMove;
}

int main() {
    srand(time(0));
    bool blackTurn = true;

    while (true) {
        printBoard();

        if (blackTurn) {
            int x, y;
            cout << "请输入坐标（x y）：";
            if (!(cin >> x >> y)) {
                cin.clear();
                cin.ignore(INT_MAX, '\n');
                cout << "无效输入，请重新输入！" << endl;
                continue;
            }

            if (x < 0 || x >= BOARD_SIZE || y < 0 || y >= BOARD_SIZE) {
                cout << "坐标超出范围！" << endl;
                continue;
            }

            if (board[x][y] != EMPTY) {
                cout << "该位置已有棋子！" << endl;
                continue;
            }

            board[x][y] = BLACK;
            if (checkWin(x, y, BLACK)) {
                printBoard();
                cout << "黑棋胜利！" << endl;
                break;
            }
        }
        else {
            cout << "AI正在思考..." << endl;
            Move aiMove = findBestMove(WHITE);
            board[aiMove.x][aiMove.y] = WHITE;
            cout << "AI下在 (" << aiMove.x << ", " << aiMove.y << ")" << endl;

            if (checkWin(aiMove.x, aiMove.y, WHITE)) {
                printBoard();
                cout << "白棋胜利！" << endl;
                break;
            }
        }

        blackTurn = !blackTurn;
    }

    return 0;
}
```

step3:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\five\range.pipe.ts

```typescript
// range.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'range'
})
export class RangePipe implements PipeTransform {
  transform(value: number): number[] {
    return Array.from({length: value}, (_, i) => i);
  }
}

```

step4C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\five\five.component.ts:

```typescript
import { Component } from '@angular/core';
import {FormsModule} from '@angular/forms';
import {NgForOf, NgIf} from '@angular/common';
import {RangePipe} from './range.pipe';

type CellValue = 'black' | 'white' | null;

@Component({
  selector: 'app-five',
  imports: [
    FormsModule,
    NgForOf,
    NgIf,
    RangePipe
  ],
  templateUrl: './five.component.html',
  styleUrl: './five.component.css'
})
export class FiveComponent {
  gridSize = 12;
  board: CellValue[][] = [];
  currentPlayer: 'black' | 'white' = 'black';
  gameOver = false;

  constructor() {
    this.initBoard();
  }

  initBoard() {
    this.board = Array.from({ length: this.gridSize }, () =>
      Array(this.gridSize).fill(null)
    );
  }

  getYValues() {
    return Array.from({ length: this.gridSize }, (_, i) => this.gridSize - 1 - i);
  }

  placeStone(x: number, y: number) {
    if (this.gameOver) {
      alert('游戏已结束，请重新开始！');
      return;
    }

    if (this.board[y][x] !== null) {
      alert('该位置已有棋子！');
      return;
    }

    this.board[y][x] = this.currentPlayer;

    if (this.checkWin(x, y)) {
      setTimeout(() => {
        alert(`${this.currentPlayer === 'black' ? '黑棋' : '白棋'} 获胜！`);
        this.gameOver = true;
      });
    } else {
      this.currentPlayer = this.currentPlayer === 'black' ? 'white' : 'black';
    }
  }

  checkWin(x: number, y: number): boolean {
    const directions = [
      { dx: 1, dy: 0 },  // 水平
      { dx: 0, dy: 1 },  // 垂直
      { dx: 1, dy: 1 },  // 右下对角线
      { dx: 1, dy: -1 }  // 右上对角线
    ];

    for (const dir of directions) {
      let count = 1;

      // 正向检查
      let [xi, yi] = [x + dir.dx, y + dir.dy];
      while (this.isValidCell(xi, yi) && this.board[yi][xi] === this.currentPlayer) {
        count++;
        xi += dir.dx;
        yi += dir.dy;
      }

      // 反向检查
      [xi, yi] = [x - dir.dx, y - dir.dy];
      while (this.isValidCell(xi, yi) && this.board[yi][xi] === this.currentPlayer) {
        count++;
        xi -= dir.dx;
        yi -= dir.dy;
      }

      if (count >= 5) return true;
    }
    return false;
  }

  isValidCell(x: number, y: number): boolean {
    return x >= 0 && x < this.gridSize && y >= 0 && y < this.gridSize;
  }

  restart() {
    this.initBoard();
    this.currentPlayer = 'black';
    this.gameOver = false;
  }

  onGridSizeChange() {
    this.gridSize = Math.min(Math.max(this.gridSize, 5), 20);
    this.restart();
  }
}

```

step5:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\five\five.component.html

```xml
<!-- app.component.html -->
<div class="container">
  <h1>五子棋游戏</h1>

  <div class="controls">
    <label>
      网格大小：
      <input type="number" [(ngModel)]="gridSize"
             (change)="onGridSizeChange()" min="5" max="20">
    </label>

    <button (click)="restart()">重新开始</button>

    <div class="status">
      当前玩家：<span [class]="currentPlayer">{{ currentPlayer === 'black' ? '黑棋' : '白棋' }}</span>
    </div>
  </div>

  <div class="board">
    <div *ngFor="let y of getYValues()" class="row">
      <div *ngFor="let x of gridSize | range"
           class="cell"
           [class.has-stone]="board[y][x]"
           (click)="placeStone(x, y)">
        <div class="coords">{{x}},{{y}}</div>
        <div *ngIf="board[y][x]" class="stone" [class.black]="board[y][x] === 'black'"
             [class.white]="board[y][x] === 'white'"></div>
      </div>
    </div>
  </div>
</div>

```

step6:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\five\five.component.css

```css
/* app.component.css */
.container {
  padding: 20px;
  font-family: Arial, sans-serif;
}

.controls {
  margin-bottom: 20px;
  display: flex;
  gap: 20px;
  align-items: center;
}

input[type="number"] {
  width: 60px;
  padding: 5px;
}

button {
  padding: 5px 15px;
  cursor: pointer;
}

.status span {
  padding: 2px 10px;
  border-radius: 3px;
  color: white;
}

.status .black { background: black; }
.status .white { background: #666; border: 1px solid #333; }

.board {
  border: 2px solid #333;
  display: inline-block;
}

.row {
  display: flex;
}

.cell {
  width: 40px;
  height: 40px;
  border: 1px solid #ddd;
  position: relative;
  cursor: pointer;
  background: #fff3e0;
}

.coords {
  position: absolute;
  font-size: 10px;
  color: #666;
  top: 1px;
  left: 1px;
  user-select: none;
}

.stone {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 1px 1px 3px rgba(0,0,0,0.3);
}

.black { background: #000; }
.white { background: #fff; border: 1px solid #333; }

```

end