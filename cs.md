说明：
c#使用forms实现屏幕截图
step101：

```csharp
using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp2
{
    public partial class Form1 : Form
    {
        // 项目路径常量
        private const string ProjectPath = @"C:\Users\wangrusheng\RiderProjects\WinFormsApp2\WinFormsApp2\";
        
        public Form1()
        {
            InitializeComponent();
            InitializeUIComponents();
        }

        private void InitializeUIComponents()
        {
            // 窗体设置
            this.Text = "自动截图工具";
            this.Size = new Size(300, 150);
            
            // 添加提示标签
            Label label = new Label
            {
                Text = "点击按钮开始区域截图",
                AutoSize = true,
                Location = new Point(20, 20)
            };
            
            // 添加截图按钮
            Button screenshotButton = new Button
            {
                Text = "开始截图",
                Size = new Size(100, 30),
                Location = new Point(20, 50)
            };
            screenshotButton.Click += ScreenshotButton_Click;

            // 添加控件
            this.Controls.Add(label);
            this.Controls.Add(screenshotButton);
        }

        private void ScreenshotButton_Click(object sender, EventArgs e)
        {
            this.Hide();
            using (var selector = new RegionSelector())
            {
                if (selector.ShowDialog() == DialogResult.OK)
                {
                    SaveScreenshot(selector.SelectedRegion);
                }
            }
            this.Show();
        }

        private void SaveScreenshot(Rectangle region)
        {
            try
            {
                using (Bitmap bmp = new Bitmap(region.Width, region.Height))
                {
                    // 截图操作
                    using (Graphics g = Graphics.FromImage(bmp))
                    {
                        g.CopyFromScreen(region.Location, Point.Empty, region.Size);
                    }

                    // 生成唯一文件名
                    /*string timestamp = DateTime.Now.ToString("yyyyMMddHHmmss");
                    string fileName = $"screenshot_{timestamp}.png";*/
                    string fileName = $"screenshot.png";
                    string fullPath = Path.Combine(ProjectPath, fileName);

                    // 确保目录存在
                    Directory.CreateDirectory(ProjectPath);
                    
                    // 保存文件
                    bmp.Save(fullPath, ImageFormat.Png);
                    
                    // 提示用户
                    /*MessageBox.Show($"截图已自动保存至：\n{fullPath}", "操作成功", 
                                  MessageBoxButtons.OK, 
                                  MessageBoxIcon.Information);*/
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"保存失败：{ex.Message}", "错误", 
                              MessageBoxButtons.OK, 
                              MessageBoxIcon.Error);
            }
        }
    }

    public class RegionSelector : Form
    {
        private Point start;
        private Rectangle selection;
        private bool selecting;

        public Rectangle SelectedRegion => selection;

        public RegionSelector()
        {
            this.FormBorderStyle = FormBorderStyle.None;
            this.WindowState = FormWindowState.Maximized;
            this.Opacity = 0.3;
            this.DoubleBuffered = true;
            this.TopMost = true;
            this.Cursor = Cursors.Cross;
            this.BackColor = Color.Black;
            this.MouseDown += (s, e) => StartSelection(e);
            this.MouseMove += (s, e) => UpdateSelection(e);
            this.MouseUp += (s, e) => EndSelection();
            this.KeyDown += (s, e) => { if (e.KeyCode == Keys.Escape) CancelSelection(); };
        }

        private void StartSelection(MouseEventArgs e)
        {
            if (e.Button == MouseButtons.Left)
            {
                start = Control.MousePosition;
                selecting = true;
            }
        }

        private void UpdateSelection(MouseEventArgs e)
        {
            if (selecting)
            {
                Point current = Control.MousePosition;
                selection = new Rectangle(
                    Math.Min(start.X, current.X),
                    Math.Min(start.Y, current.Y),
                    Math.Abs(start.X - current.X),
                    Math.Abs(start.Y - current.Y));
                Invalidate();
            }
        }

        private void EndSelection()
        {
            if (selection.Width > 0 && selection.Height > 0)
            {
                DialogResult = DialogResult.OK;
            }
            Close();
        }

        private void CancelSelection()
        {
            selection = Rectangle.Empty;
            Close();
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            if (selecting && !selection.IsEmpty)
            {
                Rectangle screenRect = RectangleToClient(RectangleToScreen(selection));
                using (Pen pen = new Pen(Color.Red, 2))
                {
                    e.Graphics.DrawRectangle(pen, screenRect);
                }
            }
        }

        protected override void OnFormClosed(FormClosedEventArgs e)
        {
            base.OnFormClosed(e);
            Cursor.Current = Cursors.Default;
        }
    }
}
```

step0:自动保存

```csharp
using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp2
{
    public partial class Form1 : Form
    {
        // 项目路径常量
        private const string ProjectPath = @"C:\Users\wangrusheng\RiderProjects\WinFormsApp2\WinFormsApp2\";
        
        public Form1()
        {
            InitializeComponent();
            InitializeUIComponents();
        }

        private void InitializeUIComponents()
        {
            // 窗体设置
            this.Text = "自动截图工具";
            this.Size = new Size(300, 150);
            
            // 添加提示标签
            Label label = new Label
            {
                Text = "点击按钮开始区域截图",
                AutoSize = true,
                Location = new Point(20, 20)
            };
            
            // 添加截图按钮
            Button screenshotButton = new Button
            {
                Text = "开始截图",
                Size = new Size(100, 30),
                Location = new Point(20, 50)
            };
            screenshotButton.Click += ScreenshotButton_Click;

            // 添加控件
            this.Controls.Add(label);
            this.Controls.Add(screenshotButton);
        }

        private void ScreenshotButton_Click(object sender, EventArgs e)
        {
            this.Hide();
            using (var selector = new RegionSelector())
            {
                if (selector.ShowDialog() == DialogResult.OK)
                {
                    SaveScreenshot(selector.SelectedRegion);
                }
            }
            this.Show();
        }

        private void SaveScreenshot(Rectangle region)
        {
            try
            {
                using (Bitmap bmp = new Bitmap(region.Width, region.Height))
                {
                    // 截图操作
                    using (Graphics g = Graphics.FromImage(bmp))
                    {
                        g.CopyFromScreen(region.Location, Point.Empty, region.Size);
                    }

                    // 生成唯一文件名
                    string timestamp = DateTime.Now.ToString("yyyyMMddHHmmss");
                    string fileName = $"screenshot_{timestamp}.png";
                    string fullPath = Path.Combine(ProjectPath, fileName);

                    // 确保目录存在
                    Directory.CreateDirectory(ProjectPath);
                    
                    // 保存文件
                    bmp.Save(fullPath, ImageFormat.Png);
                    
                    // 提示用户
                    MessageBox.Show($"截图已自动保存至：\n{fullPath}", "操作成功", 
                                  MessageBoxButtons.OK, 
                                  MessageBoxIcon.Information);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"保存失败：{ex.Message}", "错误", 
                              MessageBoxButtons.OK, 
                              MessageBoxIcon.Error);
            }
        }
    }

    public class RegionSelector : Form
    {
        private Point start;
        private Rectangle selection;
        private bool selecting;

        public Rectangle SelectedRegion => selection;

        public RegionSelector()
        {
            this.FormBorderStyle = FormBorderStyle.None;
            this.WindowState = FormWindowState.Maximized;
            this.Opacity = 0.3;
            this.DoubleBuffered = true;
            this.TopMost = true;
            this.Cursor = Cursors.Cross;
            this.BackColor = Color.Black;
            this.MouseDown += (s, e) => StartSelection(e);
            this.MouseMove += (s, e) => UpdateSelection(e);
            this.MouseUp += (s, e) => EndSelection();
            this.KeyDown += (s, e) => { if (e.KeyCode == Keys.Escape) CancelSelection(); };
        }

        private void StartSelection(MouseEventArgs e)
        {
            if (e.Button == MouseButtons.Left)
            {
                start = Control.MousePosition;
                selecting = true;
            }
        }

        private void UpdateSelection(MouseEventArgs e)
        {
            if (selecting)
            {
                Point current = Control.MousePosition;
                selection = new Rectangle(
                    Math.Min(start.X, current.X),
                    Math.Min(start.Y, current.Y),
                    Math.Abs(start.X - current.X),
                    Math.Abs(start.Y - current.Y));
                Invalidate();
            }
        }

        private void EndSelection()
        {
            if (selection.Width > 0 && selection.Height > 0)
            {
                DialogResult = DialogResult.OK;
            }
            Close();
        }

        private void CancelSelection()
        {
            selection = Rectangle.Empty;
            Close();
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            if (selecting && !selection.IsEmpty)
            {
                Rectangle screenRect = RectangleToClient(RectangleToScreen(selection));
                using (Pen pen = new Pen(Color.Red, 2))
                {
                    e.Graphics.DrawRectangle(pen, screenRect);
                }
            }
        }

        protected override void OnFormClosed(FormClosedEventArgs e)
        {
            base.OnFormClosed(e);
            Cursor.Current = Cursors.Default;
        }
    }
}
```

step1: 点击按钮，拖拽，截图，保存本地
C:\Users\wangrusheng\RiderProjects\WinFormsApp1\WinFormsApp1\Form1.cs

```csharp
using System;
using System.Drawing;
using System.Drawing.Imaging;
using System.Windows.Forms;

namespace WinFormsApp1
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();

            // 添加Label控件
            Label label = new Label();
            label.Text = "拖拽选择区域截图";
            label.AutoSize = true;
            label.Location = new Point(20, 20);
            this.Controls.Add(label);

            // 添加截图按钮
            Button screenshotButton = new Button();
            screenshotButton.Text = "开始截图";
            screenshotButton.Location = new Point(20, 50);
            screenshotButton.Click += ScreenshotButton_Click;
            this.Controls.Add(screenshotButton);
        }

        private void ScreenshotButton_Click(object sender, EventArgs e)
        {
            this.Hide();
            using (var selector = new RegionSelector())
            {
                if (selector.ShowDialog() == DialogResult.OK)
                {
                    SaveScreenshot(selector.SelectedRegion);
                }
            }
            this.Show();
        }

        private void SaveScreenshot(Rectangle region)
        {
            using (Bitmap bmp = new Bitmap(region.Width, region.Height))
            {
                using (Graphics g = Graphics.FromImage(bmp))
                {
                    g.CopyFromScreen(region.Location, Point.Empty, region.Size);
                }

                SaveFileDialog sfd = new SaveFileDialog();
                sfd.Filter = "PNG 图片|*.png|JPEG 图片|*.jpg";
                sfd.Title = "保存截图";
                
                if (sfd.ShowDialog() == DialogResult.OK)
                {
                    ImageFormat format = sfd.FilterIndex switch
                    {
                        1 => ImageFormat.Png,
                        2 => ImageFormat.Jpeg,
                        _ => ImageFormat.Png
                    };
                    bmp.Save(sfd.FileName, format);
                    MessageBox.Show($"截图已保存至：{sfd.FileName}", "操作成功");
                }
            }
        }
    }

    public class RegionSelector : Form
    {
        private Point start;
        private Rectangle selection;
        private bool selecting;

        public Rectangle SelectedRegion => selection;

        public RegionSelector()
        {
            this.FormBorderStyle = FormBorderStyle.None;
            this.WindowState = FormWindowState.Maximized;
            this.Opacity = 0.3;
            this.DoubleBuffered = true;
            this.TopMost = true;
            this.Cursor = Cursors.Cross;
            this.BackColor = Color.Black;
            this.MouseDown += (s, e) => StartSelection(e);
            this.MouseMove += (s, e) => UpdateSelection(e);
            this.MouseUp += (s, e) => EndSelection();
            this.KeyDown += (s, e) => { if (e.KeyCode == Keys.Escape) CancelSelection(); };
        }

        private void StartSelection(MouseEventArgs e)
        {
            if (e.Button == MouseButtons.Left)
            {
                start = Control.MousePosition;
                selecting = true;
            }
        }

        private void UpdateSelection(MouseEventArgs e)
        {
            if (selecting)
            {
                Point current = Control.MousePosition;
                selection = new Rectangle(
                    Math.Min(start.X, current.X),
                    Math.Min(start.Y, current.Y),
                    Math.Abs(start.X - current.X),
                    Math.Abs(start.Y - current.Y));
                Invalidate();
            }
        }

        private void EndSelection()
        {
            if (selection.Width > 0 && selection.Height > 0)
            {
                DialogResult = DialogResult.OK;
            }
            Close();
        }

        private void CancelSelection()
        {
            selection = Rectangle.Empty;
            Close();
        }

        protected override void OnPaint(PaintEventArgs e)
        {
            base.OnPaint(e);
            if (selecting && !selection.IsEmpty)
            {
                Rectangle screenRect = RectangleToClient(RectangleToScreen(selection));
                using (Pen pen = new Pen(Color.Red, 2))
                {
                    e.Graphics.DrawRectangle(pen, screenRect);
                }
            }
        }

        protected override void OnFormClosed(FormClosedEventArgs e)
        {
            base.OnFormClosed(e);
            Cursor.Current = Cursors.Default;
        }
    }
}
```

step2:简单截取整个屏幕 C:\Users\wangrusheng\RiderProjects\WinFormsApp1\WinFormsApp1\Form1.cs

```csharp
using System.Drawing;
using System.Drawing.Imaging;
using System.Windows.Forms;

namespace WinFormsApp1;

public partial class Form1 : Form
{
    public Form1()
    {
        InitializeComponent();

        // 添加Label控件
        Label label = new Label();
        label.Text = "Hello World!";
        label.AutoSize = true;
        label.Location = new Point(20, 20);
        this.Controls.Add(label);

        // 添加截图按钮
        Button screenshotButton = new Button();
        screenshotButton.Text = "截图";
        screenshotButton.Location = new Point(20, 50);
        screenshotButton.Click += ScreenshotButton_Click; // 绑定点击事件
        this.Controls.Add(screenshotButton);
    }

    private void ScreenshotButton_Click(object sender, EventArgs e)
    {
        // 获取主屏幕的尺寸
        Rectangle screenSize = Screen.PrimaryScreen.Bounds;
        
        // 创建位图对象
        using (Bitmap screenshot = new Bitmap(screenSize.Width, screenSize.Height))
        {
            // 使用Graphics对象捕获屏幕
            using (Graphics graphics = Graphics.FromImage(screenshot))
            {
                graphics.CopyFromScreen(Point.Empty, Point.Empty, screenSize.Size);
            }

            // 显示保存文件对话框
            SaveFileDialog saveDialog = new SaveFileDialog();
            saveDialog.Filter = "PNG 图片|*.png|JPEG 图片|*.jpg";
            saveDialog.Title = "保存截图";
            
            if (saveDialog.ShowDialog() == DialogResult.OK)
            {
                // 根据用户选择的格式保存
                ImageFormat format = saveDialog.FilterIndex switch
                {
                    1 => ImageFormat.Png,
                    2 => ImageFormat.Jpeg,
                    _ => ImageFormat.Png
                };
                
                screenshot.Save(saveDialog.FileName, format);
                MessageBox.Show($"截图已保存至：{saveDialog.FileName}", "操作成功");
            }
        }
    }
}
```
end