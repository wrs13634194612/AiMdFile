说明：我计划使用fastapi +angular，实现迷宫路径生成与求解

后端功能包括：
1.FastAPI搭建RESTful接口。写两个接口，
   1.1生成迷宫，
   1.2求解路径

前端功能包括
1.根据给定的长宽值，生成迷宫
2.点击按钮，刷新迷宫，生成新迷宫
3.点击按钮，可以给出正确的迷宫路径
4.增加开始按钮 和计时功能
5.鼠标拖拽绘制迷宫路径
6.绘制错误，弹窗提示失败，重新开始
7.拖拽绘制迷宫成功，弹窗提示成功，并给出分数

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/18ac8260b47d41558406790fc601093d.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List
import random
from collections import deque

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


class MazeRequest(BaseModel):
    width: int
    height: int


class SolveRequest(BaseModel):
    maze: List[List[int]]
    start: List[int]
    end: List[int]


def generate_maze_dfs(width: int, height: int) -> List[List[int]]:
    # 确保奇数列和行
    width = width // 2 * 2 + 1
    height = height // 2 * 2 + 1

    maze = [[1 for _ in range(width)] for _ in range(height)]
    stack = [(0, 0)]
    maze[0][0] = 0

    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]

    while stack:
        current = stack[-1]
        neighbors = []

        for dx, dy in directions:
            nx = current[0] + dx * 2
            ny = current[1] + dy * 2

            if 0 <= nx < height and 0 <= ny < width and maze[nx][ny] == 1:
                neighbors.append((nx, ny))

        if neighbors:
            next_cell = random.choice(neighbors)
            mx = current[0] + (next_cell[0] - current[0]) // 2
            my = current[1] + (next_cell[1] - current[1]) // 2

            maze[mx][my] = 0
            maze[next_cell[0]][next_cell[1]] = 0
            stack.append(next_cell)
        else:
            stack.pop()

    return maze


def solve_bfs(maze: List[List[int]], start: tuple, end: tuple) -> List[List[int]]:
    rows = len(maze)
    cols = len(maze[0]) if rows > 0 else 0
    visited = [[False for _ in range(cols)] for _ in range(rows)]
    queue = deque([(start[0], start[1], [])])
    directions = [(0, 1), (1, 0), (0, -1), (-1, 0)]

    while queue:
        x, y, path = queue.popleft()
        current_path = path + [[x, y]]

        if (x, y) == end:
            return current_path

        for dx, dy in directions:
            nx, ny = x + dx, y + dy
            if 0 <= nx < rows and 0 <= ny < cols:
                if maze[nx][ny] == 0 and not visited[nx][ny]:
                    visited[nx][ny] = True
                    queue.append((nx, ny, current_path))

    return []


@app.post("/generate-maze/")
async def generate_maze(request: MazeRequest):
    try:
        maze = generate_maze_dfs(request.width, request.height)
        return {
            "maze": maze,
            "start": [0, 0],
            "end": [len(maze) - 1, len(maze[0]) - 1]
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@app.post("/solve-maze/")
async def solve_maze(request: SolveRequest):
    try:
        path = solve_bfs(request.maze, tuple(request.start), tuple(request.end))
        return {"path": path}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step2:后端部分写完了，一定要记得去postman里面测试，确保后端接口正确，然后再写前端，
这种能正常请求返回json就表示接口没问题，避免500，404，或者其他error问题

```bash
post请求：http://127.0.0.1:8000/solve-maze/

{
    "detail": [
        {
            "type": "missing",
            "loc": [
                "body"
            ],
            "msg": "Field required",
            "input": null
        }
    ]
}

post请求：http://127.0.0.1:8000/generate-maze/

{
    "detail": [
        {
            "type": "missing",
            "loc": [
                "body"
            ],
            "msg": "Field required",
            "input": null
        }
    ]
}




```

step3:后端部分写完了，postman测试完成，接下来写前端angular
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\maze\maze.service.ts

```javascript
// maze.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class MazeService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  generateMaze(width: number, height: number): Observable<any> {
    return this.http.post(`${this.apiUrl}/generate-maze/`, { width, height });
  }

  solveMaze(maze: number[][], start: number[], end: number[]): Observable<any> {
    return this.http.post(`${this.apiUrl}/solve-maze/`, { maze, start, end });
  }
}

```

step4: C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\maze\maze.component.ts 

```javascript
// maze.component.ts
import { Component, OnInit } from '@angular/core';
import { MazeService } from './maze.service';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-maze',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './maze.component.html',
  styleUrls: ['./maze.component.css']
})
export class MazeComponent implements OnInit {
  maze: number[][] = [];
  path: number[][] = [];
  width = 15;
  height = 15;
  start: number[] = [0, 0];
  end: number[] = [0, 0];
  isDrawing = false;
  currentPath: number[][] = [];
  gameStarted = false;
  startTime = 0;
  elapsedTime = 0;
  score = 0;
  showSuccess = false;
  showError = false;
  errorMessage = '';
  private timerInterval: any;

  constructor(private mazeService: MazeService) {}

  ngOnInit() {
    this.generateNewMaze();
  }

  startGame() {
    this.gameStarted = true;
    this.startTime = Date.now();
    this.timerInterval = setInterval(() => {
      this.elapsedTime = Math.floor((Date.now() - this.startTime) / 1000);
    }, 1000);
  }

  restartGame() {
    clearInterval(this.timerInterval);
    this.gameStarted = false;
    this.elapsedTime = 0;
    this.showSuccess = false;
    this.showError = false;
    this.generateNewMaze();
  }

  generateNewMaze() {
    this.mazeService.generateMaze(this.width, this.height).subscribe({
      next: (res) => {
        this.maze = res.maze;
        this.start = res.start;
        this.end = res.end;
        this.path = [];
        this.currentPath = [];
      },
      error: (err) => console.error(err)
    });
  }

  solveMaze() {
    this.mazeService.solveMaze(this.maze, this.start, this.end).subscribe({
      next: (res) => {
        this.path = res.path;
      },
      error: (err) => console.error(err)
    });
  }

  startDrawing(event: MouseEvent) {
    if (!this.gameStarted) return;

    const cell = this.getCellFromEvent(event);
    if (cell && this.isValidStart(cell)) {
      this.isDrawing = true;
      this.currentPath = [cell];
    }
  }

  draw(event: MouseEvent) {
    if (!this.isDrawing) return;

    const cell = this.getCellFromEvent(event);
    if (cell && this.isValidMove(cell)) {
      this.currentPath = [...this.currentPath, cell];
    }
  }

  stopDrawing() {
    if (!this.isDrawing) return;

    this.isDrawing = false;
    this.verifyPath();
  }

  private verifyPath() {
    const isValid = this.checkPathValidity();

    if (isValid) {
      this.score = Math.round((this.width * this.height * 100) / this.elapsedTime);
      this.showSuccess = true;
      clearInterval(this.timerInterval);
    } else {
      this.errorMessage = 'Invalid path or not reaching the end';
      this.showError = true;
    }
  }

  private checkPathValidity(): boolean {
    // Check if path starts at start point
    if (!this.arePointsEqual(this.currentPath[0], this.start)) return false;

    // Check if path ends at end point
    if (!this.arePointsEqual(this.currentPath[this.currentPath.length - 1], this.end)) return false;

    // Check path continuity and validity
    for (let i = 1; i < this.currentPath.length; i++) {
      const prev = this.currentPath[i - 1];
      const curr = this.currentPath[i];

      // Check if adjacent cells
      if (Math.abs(prev[0] - curr[0]) + Math.abs(prev[1] - curr[1]) !== 1) return false;

      // Check if path is on valid cells
      if (this.maze[curr[0]][curr[1]] === 1) return false;
    }
    return true;
  }

  private getCellFromEvent(event: MouseEvent): number[] | null {
    const target = event.target as HTMLElement;
    const row = target.parentElement?.parentElement?.children;
    const cell = target.parentElement?.children;

    if (!row || !cell) return null;

    const i = Array.from(row).indexOf(target.parentElement!);
    const j = Array.from(cell).indexOf(target);

    return i >= 0 && j >= 0 ? [i, j] : null;
  }

  private isValidStart(cell: number[]): boolean {
    return this.arePointsEqual(cell, this.start);
  }

  private isValidMove(cell: number[]): boolean {
    const lastCell = this.currentPath[this.currentPath.length - 1];
    return this.maze[cell[0]][cell[1]] === 0 &&
      !this.currentPath.some(p => this.arePointsEqual(p, cell)) &&
      Math.abs(lastCell[0] - cell[0]) + Math.abs(lastCell[1] - cell[1]) === 1;
  }

  private arePointsEqual(a: number[], b: number[]): boolean {
    return a[0] === b[0] && a[1] === b[1];
  }

  isPath(i: number, j: number): boolean {
    return this.path.some(p => p[0] === i && p[1] === j);
  }

  isDrawingPath(i: number, j: number): boolean {
    return this.currentPath.some(p => p[0] === i && p[1] === j);
  }
}

```

step5: 
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\maze\maze.component.html

```xml
<div class="controls">
  <input type="number" [(ngModel)]="width" placeholder="Width" [disabled]="gameStarted">
  <input type="number" [(ngModel)]="height" placeholder="Height" [disabled]="gameStarted">
  <button (click)="generateNewMaze()" [disabled]="gameStarted">Generate Maze</button>
  <button (click)="solveMaze()" [disabled]="!maze.length">Solve Maze</button>
  <button (click)="startGame()" [disabled]="!maze.length || gameStarted">Start Game</button>
  <div class="timer">Time: {{ elapsedTime }}s</div>
</div>

<div class="maze-container"
     (mousedown)="startDrawing($event)"
     (mousemove)="draw($event)"
     (mouseup)="stopDrawing()"
     (mouseleave)="stopDrawing()">
  <div *ngFor="let row of maze; let i = index" class="row">
    <div
      *ngFor="let cell of row; let j = index"
      class="cell"
      [class.wall]="cell === 1"
      [class.path]="isPath(i, j)"
      [class.start]="i === start[0] && j === start[1]"
      [class.end]="i === end[0] && j === end[1]"
      [class.drawing]="isDrawingPath(i, j)">
    </div>
  </div>
</div>

<div class="overlay" *ngIf="showSuccess || showError">
  <div class="dialog">
    <div *ngIf="showSuccess">
      <h3>Success!</h3>
      <p>Time: {{ elapsedTime }}s</p>
      <p>Score: {{ score }}</p>
      <button (click)="restartGame()">Play Again</button>
    </div>
    <div *ngIf="showError">
      <h3>Wrong Path!</h3>
      <p>{{ errorMessage }}</p>
      <button (click)="restartGame()">Try Again</button>
    </div>
  </div>
</div>

```

step6: C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\maze\maze.component.css

```css
/* 控件区域 */
.controls {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
  align-items: center;
}

/* 迷宫容器 */
.maze-container {
  border: 1px solid #ccc;
  user-select: none;
}

/* 行布局 */
.row {
  display: flex;
}

/* 单元格样式 */
.cell {
  width: 20px;
  height: 20px;
  border: 1px solid #eee;
  position: relative;
  cursor: crosshair;
}

/* 墙壁样式 */
.wall {
  background-color: #333;
  cursor: not-allowed;
}

/* 路径样式 */
.path {
  background-color: #ffd700;
}

/* 起点样式 */
.start {
  background-color: #00ff00;
}

/* 终点样式 */
.end {
  background-color: #ff0000;
}

/* 绘制中的路径 */
.drawing {
  background-color: #4a90e2 !important;
}

/* 计时器 */
.timer {
  font-weight: bold;
  color: #2c3e50;
}

/* 遮罩层 */
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 对话框 */
.dialog {
  background: white;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
}

```

end