 # WinForms绘图应用程序功能说明

## 1. 绘图功能

### 自由绘制
- 鼠标拖拽绘制自由曲线
- 支持抗锯齿平滑线条（`SmoothingMode.AntiAlias`）

### 画笔设置
- 颜色选择器（通过`ColorDialog`实现）
- 笔刷大小调节（支持1-20像素范围）
- 圆形笔尖设计（`LineCap.Round`端点处理）

### 橡皮擦功能
- 实质为白色画笔覆盖
- 与画笔共享大小调节系统

### 画布管理
- 一键清除功能（`Graphics.Clear`重置为灰色背景）
- 动态画布尺寸（随窗口缩放自动重置）

---

## 2. 图像处理

### 裁剪工具
- 选区操作流程：
  1. 鼠标拖拽绘制矩形选区
  2. 红色虚线框实时反馈（`DashStyle.Dash`）
  3. 释放鼠标完成裁剪
- 智能画布更新：
  - 自动替换原画布内容
  - 保持高宽比不变

### 图片粘贴
- 剪贴板集成：
  - 支持标准图像格式（`Clipboard.ContainsImage`检测）
  - 自动居中定位算法（`(容器宽-图宽)/2`计算）
- 透明通道保留：
  - 支持带Alpha通道的PNG图像
  - 使用`Graphics.DrawImage`原生绘制

---
 效果图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d010cd9bacdd4c4ea96bdb5dbb827429.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp39\WinFormsApp39\Form1.cs

```csharp
 using System;
using System.Drawing;
using System.Drawing.Drawing2D;
using System.Windows.Forms;

namespace WinFormsApp39;

public partial class Form1 : Form
{
    private Bitmap canvas;
    private Graphics graphics;
    private Point previousPoint;
    private bool isDrawing = false;
    private Color currentColor = Color.Black;
    private int penSize = 3;
    private Pen currentPen;

    // 新增裁剪相关变量
    private bool isCropping = false;
    private Rectangle cropRect;
    private Point cropStart;

    private PictureBox pictureBox;
    private TrackBar trackBarPenSize;
    private Label lblPenSize;

    public Form1()
    {
        InitializeComponent();

        // 创建顶部工具栏面板
        FlowLayoutPanel topPanel = new FlowLayoutPanel
        {
            Dock = DockStyle.Top,
            AutoSize = true,
            FlowDirection = FlowDirection.LeftToRight,
            Padding = new Padding(5),
            WrapContents = false
        };
        Controls.Add(topPanel);

        // 初始化绘图区域
        pictureBox = new PictureBox
        {
            Dock = DockStyle.Fill,
            BackColor = Color.LightGray,
            BorderStyle = BorderStyle.FixedSingle
        };
        pictureBox.MouseDown += PictureBox_MouseDown;
        pictureBox.MouseMove += PictureBox_MouseMove;
        pictureBox.MouseUp += PictureBox_MouseUp;
        pictureBox.Paint += PictureBox_Paint;
        Controls.Add(pictureBox);

        // 初始化画布
        canvas = new Bitmap(pictureBox.Width, pictureBox.Height);
        graphics = Graphics.FromImage(canvas);
        graphics.Clear(Color.Gray);
        pictureBox.Image = canvas;

        // 初始化画笔
        currentPen = new Pen(currentColor, penSize)
        {
            StartCap = LineCap.Round,
            EndCap = LineCap.Round
        };

        // 颜色选择按钮
        var btnColor = new Button
        {
            Text = "选择颜色",
            Size = new Size(100, 30)
        };
        btnColor.Click += BtnColor_Click;
        topPanel.Controls.Add(btnColor);

        // 笔刷大小调节
        trackBarPenSize = new TrackBar
        {
            Minimum = 1,
            Maximum = 20,
            Value = penSize,
            Width = 150
        };
        trackBarPenSize.Scroll += TrackBarPenSize_Scroll;
        topPanel.Controls.Add(trackBarPenSize);

        // 笔刷大小标签
        lblPenSize = new Label
        {
            Text = $"笔刷大小: {penSize}",
            AutoSize = true,
            Margin = new Padding(10, 5, 0, 0)
        };
        topPanel.Controls.Add(lblPenSize);

        // 功能按钮组
        AddToolButton(topPanel, "清除", BtnClear_Click);
        AddToolButton(topPanel, "橡皮擦", BtnEraser_Click);
        AddToolButton(topPanel, "画笔", BtnPen_Click);
        AddToolButton(topPanel, "保存", BtnSave_Click);
        AddToolButton(topPanel, "裁剪", BtnCrop_Click);
        AddToolButton(topPanel, "粘贴", BtnPaste_Click);

        // 窗口设置
        ClientSize = new Size(2400, 1200);

        // 订阅ResizeEnd事件
        this.ResizeEnd += Form1_ResizeEnd;
    }

        // 处理窗口调整大小结束事件
    private void Form1_ResizeEnd(object sender, EventArgs e)
    {
        AdjustCanvasSize();
    }


    // 新增方法：调整画布大小
    private void AdjustCanvasSize()
    {
        // 确保pictureBox的尺寸有效
        if (pictureBox.Width <= 0 || pictureBox.Height <= 0)
            return;

        // 创建新画布
        Bitmap newCanvas = new Bitmap(pictureBox.Width, pictureBox.Height);
        using (Graphics g = Graphics.FromImage(newCanvas))
        {
            g.Clear(Color.Gray);
        }

        // 释放旧资源
        if (canvas != null)
        {
            canvas.Dispose();
        }
        if (graphics != null)
        {
            graphics.Dispose();
        }

        // 更新画布和绘图对象
        canvas = newCanvas;
        graphics = Graphics.FromImage(canvas);
        pictureBox.Image = canvas;

        // 重新初始化画笔
        if (currentPen != null)
        {
            currentPen.Dispose();
        }
        currentPen = new Pen(currentColor, penSize)
        {
            StartCap = LineCap.Round,
            EndCap = LineCap.Round
        };
    }



    private void AddToolButton(FlowLayoutPanel panel, string text, EventHandler handler)
    {
        var btn = new Button
        {
            Text = text,
            Size = new Size(80, 30),
            Margin = new Padding(5)
        };
        btn.Click += handler;
        panel.Controls.Add(btn);
    }

    // 剩余的事件处理方法和辅助方法保持不变
    // （包括 PictureBox_MouseDown, PictureBox_MouseMove 等所有现有方法）
    
    #region 事件处理方法
    private void PictureBox_MouseDown(object sender, MouseEventArgs e)
    {
        if (isCropping)
        {
            cropStart = e.Location;
            cropRect = new Rectangle(e.Location, new Size(0, 0));
        }
        else
        {
            isDrawing = true;
            previousPoint = e.Location;
        }
    }

   
 

    private void PictureBox_MouseMove(object sender, MouseEventArgs e)
    {
        if (isCropping)
        {
            if (e.Button == MouseButtons.Left)
            {
                // 计算选区矩形
                int x = Math.Min(cropStart.X, e.X);
                int y = Math.Min(cropStart.Y, e.Y);
                int width = Math.Abs(e.X - cropStart.X);
                int height = Math.Abs(e.Y - cropStart.Y);
                
                cropRect = new Rectangle(x, y, width, height);
                pictureBox.Invalidate();
            }
        }
        else if (isDrawing)
        {
            using (var g = Graphics.FromImage(canvas))
            {
                g.SmoothingMode = SmoothingMode.AntiAlias;
                g.DrawLine(currentPen, previousPoint, e.Location);
            }
            previousPoint = e.Location;
            pictureBox.Invalidate();
        }
    }

    private void PictureBox_MouseUp(object sender, MouseEventArgs e)
    {
        if (isCropping)
        {
            if (cropRect.Width > 0 && cropRect.Height > 0)
            {
                // 执行裁剪操作
                Bitmap croppedImage = new Bitmap(cropRect.Width, cropRect.Height);
                using (Graphics g = Graphics.FromImage(croppedImage))
                {
                    g.DrawImage(canvas, new Rectangle(0, 0, croppedImage.Width, croppedImage.Height),
                        cropRect,
                        GraphicsUnit.Pixel);
                }
                
                // 更新画布
                canvas = croppedImage;
                graphics = Graphics.FromImage(canvas);
                pictureBox.Image = canvas;
            }
            isCropping = false;
            cropRect = Rectangle.Empty;
            pictureBox.Invalidate();
        }
        else
        {
            isDrawing = false;
        }
    }

    private void PictureBox_Paint(object sender, PaintEventArgs e)
    {
        // 绘制裁剪选区
        if (isCropping && cropRect != Rectangle.Empty)
        {
            using (Pen dashPen = new Pen(Color.Red, 1))
            {
                dashPen.DashStyle = DashStyle.Dash;
                e.Graphics.DrawRectangle(dashPen, cropRect);
            }
        }
    }

    private void BtnCrop_Click(object sender, EventArgs e)
    {
        isCropping = true;
        isDrawing = false;  // 确保退出绘图模式
    }


    private void BtnPaste_Click(object sender, EventArgs e)
    {
        if (Clipboard.ContainsImage())
        {
            Image clipboardImage = Clipboard.GetImage();
        
            // 计算居中坐标（修改部分）
            int x = (pictureBox.Width - clipboardImage.Width) / 2;
            int y = (pictureBox.Height - clipboardImage.Height) / 2;
        
            // 限制最小坐标为(0,0)
            x = Math.Max(x, 0);
            y = Math.Max(y, 0);

            using (Graphics g = Graphics.FromImage(canvas))
            {
                g.DrawImage(clipboardImage, x, y); // 在居中位置绘制
            }
            pictureBox.Invalidate();
        }
        else
        {
            MessageBox.Show("剪贴板中没有图片");
        }
    }

    // 其他原有事件处理方法保持不变
    private void BtnColor_Click(object sender, EventArgs e)
    {
        using (var colorDialog = new ColorDialog())
        {
            if (colorDialog.ShowDialog() == DialogResult.OK)
            {
                currentColor = colorDialog.Color;
                currentPen.Color = currentColor;
            }
        }
    }

    private void TrackBarPenSize_Scroll(object sender, EventArgs e)
    {
        penSize = trackBarPenSize.Value;
        currentPen.Width = penSize;
        lblPenSize.Text = $"笔刷大小: {penSize}";
    }

    private void BtnClear_Click(object sender, EventArgs e)
    {
        graphics.Clear(Color.Gray);
        pictureBox.Invalidate();
    }

    private void BtnEraser_Click(object sender, EventArgs e)
    {
        currentPen.Color = Color.Gray;
        isCropping = false;  // 退出裁剪模式
    }

    private void BtnPen_Click(object sender, EventArgs e)
    {
        currentPen.Color = currentColor;
        isCropping = false;  // 退出裁剪模式
    }

    private void BtnSave_Click(object sender, EventArgs e)
    {
        using (var saveDialog = new SaveFileDialog())
        {
            saveDialog.Filter = "PNG图片|*.png|JPEG图片|*.jpg";
            if (saveDialog.ShowDialog() == DialogResult.OK)
            {
                canvas.Save(saveDialog.FileName);
            }
        }
    }
    #endregion
}

```

end