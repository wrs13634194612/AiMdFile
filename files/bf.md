说明：
angular实现和ai玩五子棋
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4b4bc21cd3944e68bb700b564e01313b.png#pic_center)

step0:游戏规则

```bash
# 五子棋游戏规则

## 基本规则

### 🎯 游戏目标
- **先形成连续五子**：
- 玩家通过轮流落子，
- 率先在横、竖、斜任一方向形成连续五个同色棋子的一方获胜

AI特性
🤖 计算机玩家


决策逻辑

使用改进版Minimax算法
包含Alpha-Beta剪枝优化
搜索深度默认3层



行为特征

500ms模拟思考延迟
优先选择中心区域
自动防御玩家四连

```

step1:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\wordle\game-ai.service.ts

```typescript
import { Injectable } from '@angular/core';

type PieceType = 'empty' | 'black' | 'white';
type Board = PieceType[][];

const SCORE_TABLE: { [key: string]: number } = {
  '11111': 500000,   // 连五
  '011110': 10000,   // 活四
  '011112': 5000,    // 冲四
  '211110': 5000,
  '01110': 4000,     // 活三
  '01112': 2000,     // 眠三
  '21110': 2000,
  '001110': 1000,
};

@Injectable({
  providedIn: 'root'
})
export class GameAIService {
  private aiPlayer: PieceType = 'white';
  private humanPlayer: PieceType = 'black';
  private searchDepth = 3;

  getBestMove(board: Board): [number, number] {
    const moves = this.generateMoves(board);
    let bestScore = -Infinity;
    let bestMove: [number, number] = [7, 7]; // 默认中心位置

    for (const [x, y] of moves) {
      const newBoard = this.cloneBoard(board);
      newBoard[x][y] = this.aiPlayer;
      const score = this.minimax(newBoard, this.searchDepth - 1, -Infinity, Infinity, false);

      if (score > bestScore) {
        bestScore = score;
        bestMove = [x, y];
      }
    }
    return bestMove;
  }

  private minimax(
    board: Board,
    depth: number,
    alpha: number,
    beta: number,
    isMaximizing: boolean
  ): number {
    if (depth === 0 || this.isGameOver(board)) {
      return this.evaluateBoard(board);
    }

    const moves = this.generateMoves(board);

    if (isMaximizing) {
      let maxEvaluation = -Infinity;
      for (const [x, y] of moves) {
        const newBoard = this.cloneBoard(board);
        newBoard[x][y] = this.aiPlayer;
        const evaluation = this.minimax(newBoard, depth - 1, alpha, beta, false);
        maxEvaluation = Math.max(maxEvaluation, evaluation);
        alpha = Math.max(alpha, evaluation);
        if (beta <= alpha) break;
      }
      return maxEvaluation;
    } else {
      let minEvaluation = Infinity;
      for (const [x, y] of moves) {
        const newBoard = this.cloneBoard(board);
        newBoard[x][y] = this.humanPlayer;
        const evaluation = this.minimax(newBoard, depth - 1, alpha, beta, true);
        minEvaluation = Math.min(minEvaluation, evaluation);
        beta = Math.min(beta, evaluation);
        if (beta <= alpha) break;
      }
      return minEvaluation;
    }
  }

  private evaluateBoard(board: Board): number {
    let totalScore = 0;
    const size = board.length;

    // 检查所有行、列和对角线
    for (let x = 0; x < size; x++) {
      for (let y = 0; y < size; y++) {
        if (board[x][y] !== 'empty') {
          const score = this.evaluatePosition(board, x, y);
          totalScore += board[x][y] === this.aiPlayer ? score : -score * 0.8;
        }
      }
    }
    return totalScore;
  }

  private evaluatePosition(board: Board, x: number, y: number): number {
    let score = 0;
    const directions = [
      [1, 0],  // 水平
      [0, 1],  // 垂直
      [1, 1],  // 主对角线
      [1, -1], // 副对角线
    ];

    for (const [dx, dy] of directions) {
      let pattern = '';
      for (let i = -4; i <= 4; i++) {
        const nx = x + dx * i;
        const ny = y + dy * i;

        if (nx < 0 || nx >= board.length || ny < 0 || ny >= board.length) {
          pattern += '2';
        } else {
          pattern += this.pieceToChar(board[nx][ny], board[x][y]);
        }
      }

      for (const [key, value] of Object.entries(SCORE_TABLE)) {
        if (pattern.includes(key)) {
          score += value;
          break;
        }
      }
    }
    return score;
  }

  private pieceToChar(piece: PieceType, current: PieceType): string {
    if (piece === 'empty') return '0';
    return piece === current ? '1' : '2';
  }

  private generateMoves(board: Board): [number, number][] {
    const moves: [number, number][] = [];
    const size = board.length;

    for (let x = 0; x < size; x++) {
      for (let y = 0; y < size; y++) {
        if (board[x][y] === 'empty' && this.hasNeighbor(board, x, y, 3)) {
          moves.push([x, y]);
        }
      }
    }

    // 根据启发式评分排序
    return moves.sort((a, b) => {
      return this.moveHeuristic(board, b[0], b[1]) - this.moveHeuristic(board, a[0], a[1]);
    });
  }

  private moveHeuristic(board: Board, x: number, y: number): number {
    let score = 0;
    const tempBoard = this.cloneBoard(board);
    tempBoard[x][y] = this.aiPlayer;
    score += this.evaluatePosition(tempBoard, x, y);
    tempBoard[x][y] = this.humanPlayer;
    score += this.evaluatePosition(tempBoard, x, y) * 0.8;
    return score;
  }

  private hasNeighbor(board: Board, x: number, y: number, distance: number): boolean {
    const size = board.length;
    for (let dx = -distance; dx <= distance; dx++) {
      for (let dy = -distance; dy <= distance; dy++) {
        const nx = x + dx;
        const ny = y + dy;
        if (nx >= 0 && nx < size && ny >= 0 && ny < size && board[nx][ny] !== 'empty') {
          return true;
        }
      }
    }
    return false;
  }

  private isGameOver(board: Board): boolean {
    // 这里可以添加具体的胜利检查逻辑
    return false;
  }

  private cloneBoard(board: Board): Board {
    return board.map(row => [...row]);
  }
}

```

step2：C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\wordle\wordle.component.ts

```typescript
import { CommonModule } from '@angular/common';
import { Component, Injectable, OnInit } from '@angular/core';
import { GameAIService } from './game-ai.service';

type PieceType = 'empty' | 'black' | 'white';
const BOARD_SIZE = 7;

@Component({
  selector: 'app-gomoku',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="game-container">
      <h2>{{ gameStatus }}</h2>
      <div class="board">
        <div *ngFor="let row of board; let x = index" class="row">
          <div
            *ngFor="let cell of row; let y = index"
            class="cell"
            [class.black]="cell === 'black'"
            [class.white]="cell === 'white'"
            (click)="handleClick(x, y)">
          </div>
        </div>
      </div>
      <button class="restart-btn" (click)="initGame()">New Game</button>
    </div>
  `,
  styles: [`
    .game-container {
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 20px;
    }

    .board {
      border: 2px solid #333;
      margin: 20px 0;
    }

    .row {
      display: flex;
    }

    .cell {
      width: 30px;
      height: 30px;
      border: 1px solid #ddd;
      background: #feb;
      cursor: pointer;
      position: relative;
      transition: background-color 0.3s;

      &:hover {
        background-color: #fed;
      }

      &.black::after {
        content: '';
        position: absolute;
        top: 2px;
        left: 2px;
        width: 24px;
        height: 24px;
        background: #333;
        border-radius: 50%;
      }

      &.white::after {
        content: '';
        position: absolute;
        top: 2px;
        left: 2px;
        width: 24px;
        height: 24px;
        background: #fff;
        border: 2px solid #333;
        border-radius: 50%;
      }
    }

    .restart-btn {
      padding: 10px 20px;
      background: #4CAF50;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 16px;

      &:hover {
        background: #45a049;
      }
    }
  `]
})
export class WordleComponent implements OnInit {
  board: PieceType[][] = [];
  currentPlayer: PieceType = 'black';
  gameActive = true;

  constructor(private aiService: GameAIService) {}

  ngOnInit(): void {
    this.initGame();
  }

  initGame(): void {
    this.board = Array(BOARD_SIZE).fill(null)
      .map(() => Array(BOARD_SIZE).fill('empty'));
    this.currentPlayer = 'black';
    this.gameActive = true;
  }

  get gameStatus(): string {
    if (!this.gameActive) return 'Game Over!';
    return this.currentPlayer === 'black'
      ? 'Your Turn (Black)'
      : 'AI Thinking...';
  }

  handleClick(x: number, y: number): void {
    if (!this.gameActive || this.currentPlayer !== 'black') return;

    if (this.placePiece(x, y, 'black')) {
      if (this.checkWin(x, y)) {
        this.gameActive = false;
        return;
      }
      this.currentPlayer = 'white';
      this.aiMove();
    }
  }

  private aiMove(): void {
    setTimeout(() => {  // 模拟AI思考时间
      const [x, y] = this.aiService.getBestMove(this.board);
      this.placePiece(x, y, 'white');

      if (this.checkWin(x, y)) {
        this.gameActive = false;
      } else {
        this.currentPlayer = 'black';
      }
    }, 500);
  }

  private placePiece(x: number, y: number, piece: PieceType): boolean {
    if (x < 0 || x >= BOARD_SIZE || y < 0 || y >= BOARD_SIZE) return false;
    if (this.board[x][y] !== 'empty') return false;

    this.board = this.board.map((row, i) =>
      i === x ? row.map((cell, j) => j === y ? piece : cell) : row
    );
    return true;
  }

  private checkWin(x: number, y: number): boolean {
    const directions = [
      [1, 0],  // 水平
      [0, 1],  // 垂直
      [1, 1],  // 主对角线
      [1, -1], // 副对角线
    ];

    for (const [dx, dy] of directions) {
      let count = 1;

      // 正向检查
      for (let i = 1; i < 5; i++) {
        const nx = x + dx * i;
        const ny = y + dy * i;
        if (this.isSamePiece(nx, ny, this.board[x][y])) count++;
        else break;
      }

      // 反向检查
      for (let i = 1; i < 5; i++) {
        const nx = x - dx * i;
        const ny = y - dy * i;
        if (this.isSamePiece(nx, ny, this.board[x][y])) count++;
        else break;
      }

      if (count >= 5) return true;
    }
    return false;
  }

  private isSamePiece(x: number, y: number, piece: PieceType): boolean {
    return x >= 0 && x < BOARD_SIZE &&
           y >= 0 && y < BOARD_SIZE &&
           this.board[x][y] === piece;
  }
}

```

end