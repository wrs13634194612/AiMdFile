forms数字聚光灯实现了专注于当前活动窗口的屏幕遮罩工具
说明：
1.在非活动窗口区域创建半透明黑色遮罩
2.突出显示当前获得焦点的窗口
3.通过降低周围区域的亮度帮助用户集中注意力
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bcf2bad084e34900bd719059b83b0773.png#pic_center)

step1:
C:\Users\wangrusheng\RiderProjects\WinFormsApp32\WinFormsApp32\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Runtime.InteropServices;
using System.Windows.Forms;

namespace WinFormsApp32
{
    public partial class Form1 : Form
    {
        #region Win32 API
        [DllImport("user32.dll")]
        private static extern IntPtr GetForegroundWindow();

        [DllImport("user32.dll")]
        private static extern bool GetWindowRect(IntPtr hWnd, out RECT rect);

        [DllImport("user32.dll")]
        private static extern bool SetLayeredWindowAttributes(IntPtr hwnd, uint crKey, byte bAlpha, uint dwFlags);

        private const int LWA_ALPHA = 0x2;
        
        [StructLayout(LayoutKind.Sequential)]
        public struct RECT
        {
            public int Left;
            public int Top;
            public int Right;
            public int Bottom;
        }
        #endregion

        private readonly List<DimLayerForm> dimLayers = new List<DimLayerForm>();
        private bool isActive = true;
        private int opacity = 80;
        private System.Windows.Forms.Timer refreshTimer;
        private RECT lastActiveRect;

        public Form1()
        {
            InitializeComponent();
            InitializeTrayIcon();
            InitializeTimer();
            this.ShowInTaskbar = false;
            this.WindowState = FormWindowState.Minimized;
        }

        private class DimLayerForm : Form
        {
            public DimLayerForm()
            {
                FormBorderStyle = FormBorderStyle.None;
                ShowInTaskbar = false;
                TopMost = true;
                BackColor = Color.Black;
                StartPosition = FormStartPosition.Manual;
            }

            protected override CreateParams CreateParams
            {
                get
                {
                    CreateParams cp = base.CreateParams;
                    cp.ExStyle |= 0x80000; // WS_EX_LAYERED
                    cp.ExStyle |= 0x20;    // WS_EX_TRANSPARENT
                    return cp;
                }
            }
        }

        private void InitializeTrayIcon()
        {
            var trayIcon = new NotifyIcon
            {
                Icon = SystemIcons.Application,
                Text = "Focus Dimmer",
                Visible = true
            };

            var contextMenu = new ContextMenuStrip();
            
            var toggleItem = new ToolStripMenuItem("Toggle Dimming");
            toggleItem.Click += (s, e) => ToggleDimming();
            contextMenu.Items.Add(toggleItem);

            var trackBar = new TrackBar
            {
                Minimum = 20,
                Maximum = 200,
                Value = opacity,
                Width = 150,
                TickStyle = TickStyle.None
            };
            trackBar.ValueChanged += (s, e) => 
            {
                opacity = trackBar.Value;
                UpdateDimming();
            };
            contextMenu.Items.Add(new ToolStripControlHost(trackBar));

            contextMenu.Items.Add(new ToolStripSeparator());

            var exitItem = new ToolStripMenuItem("Exit");
            exitItem.Click += (s, e) => Application.Exit();
            contextMenu.Items.Add(exitItem);

            trayIcon.ContextMenuStrip = contextMenu;
        }

        private void InitializeTimer()
        {
            refreshTimer = new System.Windows.Forms.Timer { Interval = 100 };
            refreshTimer.Tick += (s, e) => UpdateDimming();
            refreshTimer.Start();
        }

        private void UpdateDimming()
        {
            if (!isActive) return;

            var foregroundHandle = GetForegroundWindow();
            GetWindowRect(foregroundHandle, out RECT currentActiveRect);

            if (currentActiveRect.Left == lastActiveRect.Left &&
                currentActiveRect.Top == lastActiveRect.Top &&
                currentActiveRect.Right == lastActiveRect.Right &&
                currentActiveRect.Bottom == lastActiveRect.Bottom)
            {
                return;
            }

            lastActiveRect = currentActiveRect;
            ClearDimLayers();
            CreateDimLayers();
        }

        private void CreateDimLayers()
        {
            foreach (var screen in Screen.AllScreens)
            {
                var screenRect = screen.Bounds;
                var screenRECT = new RECT
                {
                    Left = screenRect.Left,
                    Top = screenRect.Top,
                    Right = screenRect.Right,
                    Bottom = screenRect.Bottom
                };

                if (!IsRectIntersecting(screenRECT, lastActiveRect))
                {
                    AddDimLayer(screenRect);
                    continue;
                }

                // 上方区域
                if (lastActiveRect.Top > screenRect.Top)
                {
                    AddDimLayer(new Rectangle(
                        screenRect.Left,
                        screenRect.Top,
                        screenRect.Width,
                        lastActiveRect.Top - screenRect.Top
                    ));
                }

                // 下方区域
                if (lastActiveRect.Bottom < screenRect.Bottom)
                {
                    AddDimLayer(new Rectangle(
                        screenRect.Left,
                        lastActiveRect.Bottom,
                        screenRect.Width,
                        screenRect.Bottom - lastActiveRect.Bottom
                    ));
                }

                // 左侧区域
                if (lastActiveRect.Left > screenRect.Left)
                {
                    AddDimLayer(new Rectangle(
                        screenRect.Left,
                        screenRect.Top,
                        lastActiveRect.Left - screenRect.Left,
                        screenRect.Height
                    ));
                }

                // 右侧区域
                if (lastActiveRect.Right < screenRect.Right)
                {
                    AddDimLayer(new Rectangle(
                        lastActiveRect.Right,
                        screenRect.Top,
                        screenRect.Right - lastActiveRect.Right,
                        screenRect.Height
                    ));
                }
            }
        }

        private bool IsRectIntersecting(RECT a, RECT b)
        {
            return !(a.Right <= b.Left || 
                     a.Left >= b.Right || 
                     a.Bottom <= b.Top || 
                     a.Top >= b.Bottom);
        }

        private void AddDimLayer(Rectangle bounds)
        {
            if (bounds.Width <= 0 || bounds.Height <= 0) return;

            var layer = new DimLayerForm { Bounds = bounds };
            SetLayeredWindowAttributes(layer.Handle, 0, (byte)opacity, LWA_ALPHA);
            layer.Show();
            dimLayers.Add(layer);
        }

        private void ClearDimLayers()
        {
            foreach (var layer in dimLayers)
            {
                layer.Close();
                layer.Dispose();
            }
            dimLayers.Clear();
        }

        private void ToggleDimming()
        {
            isActive = !isActive;
            if (isActive) UpdateDimming();
            else ClearDimLayers();
        }
    }
}
```

end