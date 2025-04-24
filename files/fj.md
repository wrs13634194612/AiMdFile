说明：写一个画板程序，实现
1.鼠标绘制图案，
2.绘制矩形，
3.橡皮擦，
4.改变画笔颜色，
5.清除所有
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6433801d03ea402cbc4ab9d9d55545cd.png#pic_center)

step1: C:\Users\Administrator\WebstormProjects\untitled4\src\app\canvas\canvas.component.ts

```javascript
import { Component, ViewChild, ElementRef, AfterViewInit } from '@angular/core';
import {NgForOf} from '@angular/common';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-canvas',
  imports: [NgForOf,CommonModule],
  templateUrl: './canvas.component.html',
  styleUrl: './canvas.component.css'
})
export class CanvasComponent implements AfterViewInit  {



  @ViewChild('canvas', { static: true }) canvasRef!: ElementRef;
  private ctx!: CanvasRenderingContext2D;
  private isDrawing = false;
  private startX = 0;
  private startY = 0;
  private lastX = 0;
  private lastY = 0;
  private initialImageData?: ImageData;

  tools = ['pencil', 'eraser', 'rectangle'];
  selectedTool = 'pencil';
  colors = ['#2c2c2c', '#e63946', '#457b9d', '#588157'];
  selectedColor = '#2c2c2c';
  lineWidth = 4;

  ngAfterViewInit() {
    this.initializeCanvas();
  }

  private initializeCanvas(): void {
    const canvas = this.canvasRef.nativeElement;
    this.ctx = canvas.getContext('2d');
    this.ctx.fillStyle = '#ffffff';
    this.ctx.fillRect(0, 0, canvas.width, canvas.height);
  }

  startDrawing(event: MouseEvent) {
    this.isDrawing = true;
    const rect = this.canvasRef.nativeElement.getBoundingClientRect();
    this.startX = this.lastX = event.clientX - rect.left;
    this.startY = this.lastY = event.clientY - rect.top;

    if (this.selectedTool === 'rectangle') {
      this.initialImageData = this.ctx.getImageData(0, 0,
        this.canvasRef.nativeElement.width,
        this.canvasRef.nativeElement.height
      );
    }

    if (this.selectedTool === 'eraser') {
      this.ctx.globalCompositeOperation = 'destination-out';
      this.ctx.strokeStyle = 'rgba(0,0,0,1)';
    }
  }

  draw(event: MouseEvent) {
    if (!this.isDrawing) return;

    const rect = this.canvasRef.nativeElement.getBoundingClientRect();
    const currentX = event.clientX - rect.left;
    const currentY = event.clientY - rect.top;

    this.ctx.beginPath();
    this.ctx.lineCap = 'round';

    if (this.selectedTool === 'pencil') {
      this.ctx.strokeStyle = this.selectedColor;
      this.ctx.lineWidth = this.lineWidth;
      this.ctx.moveTo(this.lastX, this.lastY);
      this.ctx.lineTo(currentX, currentY);
      this.ctx.stroke();
      [this.lastX, this.lastY] = [currentX, currentY];
    }
    else if (this.selectedTool === 'eraser') {
      this.ctx.lineWidth = this.lineWidth;
      this.ctx.moveTo(this.lastX, this.lastY);
      this.ctx.lineTo(currentX, currentY);
      this.ctx.stroke();
      [this.lastX, this.lastY] = [currentX, currentY];
    }
    else if (this.selectedTool === 'rectangle') {
      if (this.initialImageData) {
        this.ctx.putImageData(this.initialImageData, 0, 0);
      }
      this.ctx.strokeStyle = this.selectedColor;
      this.ctx.lineWidth = this.lineWidth;
      const width = currentX - this.startX;
      const height = currentY - this.startY;
      this.ctx.strokeRect(this.startX, this.startY, width, height);
    }
  }

  stopDrawing() {
    this.isDrawing = false;
    if (this.selectedTool === 'eraser') {
      this.ctx.globalCompositeOperation = 'source-over';
    }
  }

  clearCanvas() {
    this.ctx.clearRect(0, 0,
      this.canvasRef.nativeElement.width,
      this.canvasRef.nativeElement.height
    );
    this.ctx.fillRect(0, 0,
      this.canvasRef.nativeElement.width,
      this.canvasRef.nativeElement.height
    );
  }






}

```

step2:  C:\Users\Administrator\WebstormProjects\untitled4\src\app\canvas\canvas.component.html

```javascript
<!-- canvas.component.html -->
<div class="toolbar">
  <button *ngFor="let tool of tools"
          [class.active]="tool === selectedTool"
          (click)="selectedTool = tool">
    {{ tool }}
  </button>

  <div class="color-picker">
    <button *ngFor="let color of colors"
            [style.background]="color"
            [class.active]="color === selectedColor"
            (click)="selectedColor = color"></button>
  </div>

  <button (click)="clearCanvas()">Clear</button>
</div>

<canvas #canvas
        width="800"
        height="600"
        (mousedown)="startDrawing($event)"
        (mousemove)="draw($event)"
        (mouseup)="stopDrawing()"
        (mouseout)="stopDrawing()"></canvas>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\canvas\canvas.component.css

```css
/* canvas.component.css */
.toolbar {
  background: #f8f9fa;
  padding: 1rem;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  margin-bottom: 1.5rem;
  display: flex;
  gap: 1rem;
  align-items: center;
}

button {
  padding: 0.5rem 1rem;
  border: 2px solid #dee2e6;
  border-radius: 6px;
  background: white;
  cursor: pointer;
  transition: all 0.2s;
}

button.active {
  border-color: #4dabf7;
  background: #e7f5ff;
}

.color-picker button {
  width: 32px;
  height: 32px;
  padding: 0;
  border-radius: 50%;
}

canvas {
  border: 2px solid #dee2e6;
  border-radius: 8px;
  background: white;
  cursor: crosshair;
}

```

end