说明：
我希望用angular实现连连看
1.生成方格
2.相同图案连接
3.已连接图标隐藏
4.得分系统
5.提示，当你实在找不到的时候，点击提示，可以帮你设置图标高亮
5.重新开始 刷新
6.支持多套图标数组，可以用来设计关卡
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f78ebcaa768a4882ab5855a60a17c189.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\links\links.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { NgForOf,NgIf } from '@angular/common';

interface BoardCell {
  value: number;
  cleared: boolean;
  selected: boolean;
  x: number;
  y: number;
}

@Component({
  selector: 'app-links',
  templateUrl: './links.component.html',
  imports: [NgForOf,NgIf],
  styleUrls: ['./links.component.css']
})
export class LinksComponent implements OnInit {
  rows = 12;
  cols = 12;
  cellSize = 40;
  board: BoardCell[][] = [];
  firstSelection: { x: number, y: number } | null = null;
  score = 0;
  hintPair: { x: number, y: number }[] | null = null;
  currentPath: { x: number, y: number }[] | null = null;
  readonly EMOJI_LIST = [
  '🐶', '🐱', '🐭', '🐹', '🐰', '🦊', '🐻', '🐼',
  '🐨', '🐯', '🦁', '🐮', '🐷', '🐸', '🐵', '🐧'

  ];
  readonly EMOJI_LIST2 = [
  '🐤', '🦄', '🐺', '🐗', '🐴', '🦋', '🐌', '🐞',
  '🐝', '🦀', '🐡', '🐬', '🦑', '🐊', '🦓', '🐇'
  ];

  ngOnInit(): void {
    this.initializeGameBoard();
  }

  initializeGameBoard(): void {
    const innerRows = this.rows - 2;
    const innerCols = this.cols - 2;
    const totalInner = innerRows * innerCols;

    if (totalInner % 2 !== 0) {
      console.error('Total number of inner cells must be even.');
      return;
    }

    const tileList: number[] = [];
    const emojiCount = this.EMOJI_LIST.length;
    for (let i = 0; i < totalInner / 2; i++) {
      const emojiIndex = i % emojiCount;
      tileList.push(emojiIndex, emojiIndex);
    }

    for (let i = tileList.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [tileList[i], tileList[j]] = [tileList[j], tileList[i]];
    }

    let index = 0;
    this.board = [];
    for (let i = 0; i < this.rows; i++) {
      this.board[i] = [];
      for (let j = 0; j < this.cols; j++) {
        const isBorder = i === 0 || j === 0 || i === this.rows - 1 || j === this.cols - 1;
        this.board[i][j] = {
          value: isBorder ? -1 : tileList[index++],
          cleared: isBorder,
          selected: false,
          x: i,
          y: j
        };
      }
    }
  }

  handleCellClick(cell: BoardCell): void {
    if (cell.cleared || cell.value === -1) return;

    if (!this.firstSelection) {
      this.firstSelection = { x: cell.x, y: cell.y };
      cell.selected = true;
    } else {
      const secondSelection = { x: cell.x, y: cell.y };
      const firstCell = this.board[this.firstSelection.x][this.firstSelection.y];

      if (this.firstSelection.x === secondSelection.x &&
          this.firstSelection.y === secondSelection.y) {
        firstCell.selected = false;
        this.firstSelection = null;
        return;
      }

      if (firstCell.value === cell.value) {
        const path = this.checkConnectivityWithPath(this.firstSelection, secondSelection);
        if (path) {
          firstCell.cleared = true;
          cell.cleared = true;
          this.score += 10;
          this.currentPath = path;
          setTimeout(() => this.currentPath = null, 500);
        }
      }

      firstCell.selected = false;
      cell.selected = false;
      this.firstSelection = null;
    }
  }

  restart(): void {
    this.score = 0;
    this.hintPair = null;
    this.currentPath = null;
    this.initializeGameBoard();
  }

  hint(): void {
    for (let i = 0; i < this.rows; i++) {
      for (let j = 0; j < this.cols; j++) {
        const cell = this.board[i][j];
        if (cell.cleared || cell.value === -1) continue;

        for (let x = i; x < this.rows; x++) {
          for (let y = (x === i ? j + 1 : 0); y < this.cols; y++) {
            const other = this.board[x][y];
            if (!other.cleared && other.value === cell.value &&
                this.checkConnectivityWithPath({x: i, y: j}, {x, y})) {
              this.hintPair = [{x: i, y: j}, {x, y}];
              setTimeout(() => this.hintPair = null, 2000);
              return;
            }
          }
        }
      }
    }
  }

  isHintCell(cell: BoardCell): boolean {
    return !!this.hintPair?.some(p => p.x === cell.x && p.y === cell.y);
  }

  generatePathD(path: {x: number, y: number}[]): string {
    return path.map((p, i) => {
      const x = p.y * this.cellSize + this.cellSize / 2;
      const y = p.x * this.cellSize + this.cellSize / 2;
      return `${i === 0 ? 'M' : 'L'} ${x} ${y}`;
    }).join(' ');
  }

  private checkConnectivityWithPath(p1: { x: number, y: number }, p2: { x: number, y: number }) {
    const direct = this.directConnection(p1, p2);
    if (direct.result) return direct.path;

    const oneTurn = this.oneTurnConnection(p1, p2);
    if (oneTurn.result) return oneTurn.path;

    const twoTurn = this.twoTurnConnection(p1, p2);
    if (twoTurn.result) return twoTurn.path;

    return null;
  }

  private directConnection(p1: { x: number, y: number }, p2: { x: number, y: number }) {
    // ...保持原有逻辑，返回{ result: boolean, path: [...] }
    // 示例实现：
    if (p1.x === p2.x) {
      const minY = Math.min(p1.y, p2.y);
      const maxY = Math.max(p1.y, p2.y);
      for (let y = minY + 1; y < maxY; y++) {
        if (!this.board[p1.x][y].cleared) return { result: false, path: [] };
      }
      return { result: true, path: [p1, p2] };
    }

    if (p1.y === p2.y) {
      const minX = Math.min(p1.x, p2.x);
      const maxX = Math.max(p1.x, p2.x);
      for (let x = minX + 1; x < maxX; x++) {
        if (!this.board[x][p1.y].cleared) return { result: false, path: [] };
      }
      return { result: true, path: [p1, p2] };
    }

    return { result: false, path: [] };
  }

  private oneTurnConnection(p1: { x: number, y: number }, p2: { x: number, y: number }) {
    // ...保持原有逻辑，返回路径
    // 示例实现：
    const corner1 = { x: p1.x, y: p2.y };
    if (this.board[corner1.x][corner1.y].cleared) {
      const path1 = this.directConnection(p1, corner1);
      const path2 = this.directConnection(corner1, p2);
      if (path1.result && path2.result) {
        return { result: true, path: [...path1.path, ...path2.path.slice(1)] };
      }
    }

    const corner2 = { x: p2.x, y: p1.y };
    if (this.board[corner2.x][corner2.y].cleared) {
      const path1 = this.directConnection(p1, corner2);
      const path2 = this.directConnection(corner2, p2);
      if (path1.result && path2.result) {
        return { result: true, path: [...path1.path, ...path2.path.slice(1)] };
      }
    }

    return { result: false, path: [] };
  }

  private twoTurnConnection(p1: { x: number, y: number }, p2: { x: number, y: number }) {
    // ...保持原有逻辑，返回路径
    // 示例实现：
    for (let x = 0; x < this.rows; x++) {
      const p3 = { x, y: p1.y };
      const p4 = { x, y: p2.y };
      if (this.board[p3.x][p3.y].cleared && this.board[p4.x][p4.y].cleared) {
        const path1 = this.directConnection(p1, p3);
        const path2 = this.oneTurnConnection(p3, p2);
        if (path1.result && path2.result) {
          return { result: true, path: [...path1.path, ...path2.path.slice(1)] };
        }
      }
    }

    for (let y = 0; y < this.cols; y++) {
      const p3 = { x: p1.x, y };
      const p4 = { x: p2.x, y };
      if (this.board[p3.x][p3.y].cleared && this.board[p4.x][p4.y].cleared) {
        const path1 = this.directConnection(p1, p3);
        const path2 = this.oneTurnConnection(p3, p2);
        if (path1.result && path2.result) {
          return { result: true, path: [...path1.path, ...path2.path.slice(1)] };
        }
      }
    }

    return { result: false, path: [] };
  }
}

```

step2:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\links\links.component.html

```xml
<!-- links.component.html -->
<div class="controls">
  <button (click)="hint()">提示</button>
  <button (click)="restart()">重新开始</button>
  <div class="score">得分: {{ score }}</div>
</div>

<div class="board">
  <div class="row" *ngFor="let row of board">
     <div *ngFor="let cell of row"
         class="cell"
         [class.cleared]="cell.cleared"
         [class.selected]="cell.selected"
         [class.hint]="isHintCell(cell)"
         (click)="handleCellClick(cell)">

      <!-- 修改这里：清除后不显示内容 -->
      <span *ngIf="!cell.cleared && cell.value !== -1">
        {{ EMOJI_LIST[cell.value] }}
      </span>

      <div *ngIf="cell.value === -1" class="border-cell"></div>
    </div>

  </div>
  <svg class="path-overlay">
    <path *ngIf="currentPath" [attr.d]="generatePathD(currentPath)" stroke="blue" stroke-width="4" fill="none"/>
  </svg>
</div>

```

step3:C:\Users\wangrusheng\PycharmProjects\untitled15\src\app\links\links.component.css

```css
/* links.component.css */
.controls {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
  align-items: center;
}

.score {
  font-weight: bold;
  color: #333;
}

.board {
  position: relative;
  border: 2px solid #333;
}

.row {
  display: flex;
}

.cell {
  width: 40px;
  height: 40px;
  border: 1px solid #ddd;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  background-color: #fff;
  cursor: pointer;
  transition: background-color 0.2s;
}

.cell.cleared {
  background-color: #f0f0f0;
  opacity: 0.5;
}

.cell.selected {
  background-color: #a0d8ef !important;
}

.cell.hint {
  background-color: #ffeb3b;
}

.path-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

```

end