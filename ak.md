forms实现快读阅读器

主要功能包括：
1.文本自动分块显示：按设定的速度逐词显示文本内容。
2.阅读控制：开始/停止按钮以及回车键控制。
3.界面自定义：包括字体、颜色（前景色和背景色）的设置。
4.暗黑模式：支持暗黑主题，并有相应的渲染器。
5.全屏模式：通过F11或按钮切换全屏。
6.速度调节：通过WPM（每分钟字数）数值调节器控制显示速度。
7.循环阅读：复选框控制是否循环阅读。
8.设置持久化：保存用户的设置（如字体、颜色、速度等）到配置文件。
9.文件自动加载：从固定路径加载文本文件。
10.错误处理：文件不存在或内容为空时的提示。
说明：
C:\Users\wangrusheng\AppData\Roaming\WinFormsApp38\settings.json
这个目录下，有一个json配置文件  主要配置字体，字体颜色，背景颜色，阅读速度等，第一次运行会自动生成
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7347042acb6142cda33b84ca3b19094e.png#pic_center)

step1：
C:\Users\wangrusheng\RiderProjects\WinFormsApp38\WinFormsApp38\Form1.cs

```csharp


using System;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Windows.Forms;
using Newtonsoft.Json;

namespace WinFormsApp38
{
    public partial class Form1 : Form
    {
        private const string FixedFilePath = @"C:\Users\wangrusheng\Downloads\ages.txt";
        private Settings settings;
        private System.Windows.Forms.Timer timer;
        private List<string> tokens;
        private int currentIndex;
        private FormWindowState lastWindowState;
        private Color currentForeground;
        private Color currentBackground;

        // 控件声明
        private Label lblText;
        private ToolStrip toolStrip;
        private ToolStripButton btnFullScreen;
        private ToolStripButton btnColors;
        private ToolStripButton btnFont;
        private ToolStripButton btnStart;
        private NumericUpDown numWPM;
        private CheckBox chkRepeat;

        public Form1()
        {
            InitializeComponent();
            InitializeDynamicComponents();
            settings = Settings.Load();
            LoadSettings();
            SetupEventHandlers();
            ApplyDarkTheme();
        }

        private void InitializeDynamicComponents()
        {
            // 主标签初始化
            lblText = new Label
            {
                Dock = DockStyle.Fill,
                TextAlign = ContentAlignment.MiddleCenter,
                Font = new Font("微软雅黑", 24)
            };

            // 工具栏初始化
            toolStrip = new ToolStrip
            {
                Renderer = new DarkToolStripRenderer(),
                GripStyle = ToolStripGripStyle.Hidden
            };

            // 工具栏按钮
            btnFullScreen = new ToolStripButton("全屏 (F11)");
            btnColors = new ToolStripButton("颜色");
            btnFont = new ToolStripButton("字体");
            btnStart = new ToolStripButton("开始 (Enter)");

            // 速度控件
            numWPM = new NumericUpDown
            {
                Minimum = 50,
                Maximum = 1000,
                Value = 150,
                Width = 80,
                DecimalPlaces = 0,
                ForeColor = Color.White,
                BackColor = Color.FromArgb(45, 45, 45)
            };

            // 循环复选框
            chkRepeat = new CheckBox
            {
                Text = "循环阅读",
                AutoSize = true,
                ForeColor = Color.White,
                BackColor = Color.FromArgb(45, 45, 45),
                FlatStyle = FlatStyle.Flat
            };

            // 添加工具栏项
            toolStrip.Items.AddRange(new ToolStripItem[]
            {
                btnFullScreen,
                btnColors,
                btnFont,
                new ToolStripSeparator(),
                new ToolStripLabel("速度:") { ForeColor = Color.White },
                new ToolStripControlHost(numWPM),
                new ToolStripControlHost(chkRepeat),
                new ToolStripSeparator(),
                btnStart
            });

            // 窗体设置
            Controls.Add(lblText);
            Controls.Add(toolStrip);
            Text = "快速阅读器";
            Size = new Size(1200, 600);
            KeyPreview = true;
           // WindowState = FormWindowState.Maximized; 自动全屏功能
            FormBorderStyle = FormWindowState.Maximized == WindowState ? FormBorderStyle.None : FormBorderStyle.Sizable;
            DoubleBuffered = true;

            // 定时器
            timer = new System.Windows.Forms.Timer { Interval = 60000 / 150 };
        }

        private void LoadSettings()
        {
            currentForeground = settings.ForegroundColor;
            currentBackground = settings.BackgroundColor;
            lblText.Font = settings.TextFont;
            numWPM.Value = settings.WPM;
            chkRepeat.Checked = settings.Repeat;
            UpdateLabelAppearance();
        }

        private void ApplyDarkTheme()
        {
            // 强制使用暗黑主题色
            currentForeground = Color.White;
            currentBackground = Color.FromArgb(30, 30, 30);
    
            // 主窗体颜色
            BackColor = currentBackground;
            ForeColor = currentForeground;
            
         

            // 工具栏样式
            toolStrip.BackColor = Color.FromArgb(45, 45, 45);
            toolStrip.ForeColor = Color.White;

            // 工具栏按钮样式
            foreach (ToolStripItem item in toolStrip.Items)
            {
                if (item is ToolStripButton btn)
                {
                    btn.BackColor = Color.FromArgb(45, 45, 45);
                    btn.ForeColor = Color.White;
                }
            }

            // 数值控件
            numWPM.BackColor = Color.FromArgb(60, 60, 60);
            numWPM.ForeColor = Color.White;
            numWPM.BorderStyle = BorderStyle.FixedSingle;

            // 复选框
            chkRepeat.BackColor = Color.FromArgb(60, 60, 60);
            chkRepeat.ForeColor = Color.White;
            chkRepeat.FlatAppearance.CheckedBackColor = Color.FromArgb(30, 30, 30);
            UpdateLabelAppearance(); // 强制更新标签颜色
        }

        private void SetupEventHandlers()
        {
            btnStart.Click += BtnStart_Click;
            btnFont.Click += BtnFont_Click;
            btnColors.Click += BtnColors_Click;
            btnFullScreen.Click += BtnFullScreen_Click;
            numWPM.ValueChanged += NumWPM_ValueChanged;
            timer.Tick += Timer_Tick;
            KeyDown += Form1_KeyDown;

            // 自定义绘制事件
            numWPM.Paint += (s, e) => ControlPaint.DrawBorder(e.Graphics, numWPM.ClientRectangle, Color.Gray, ButtonBorderStyle.Solid);
            chkRepeat.Paint += (s, e) => 
            {
                e.Graphics.FillRectangle(new SolidBrush(Color.FromArgb(60, 60, 60)), e.ClipRectangle);
                TextRenderer.DrawText(e.Graphics, chkRepeat.Text, chkRepeat.Font, 
                    new Point(20, 0), chkRepeat.Checked ? Color.Lime : Color.White);
            };
        }

        private void UpdateLabelAppearance()
        {
            lblText.BackColor = currentBackground;
            lblText.ForeColor = currentForeground;
        }

        // [事件处理方法保持不变，与原始代码相同]
        // ... 包括 BtnStart_Click, BtnFont_Click 等其他事件处理方法 ...





        private void BtnStart_Click(object sender, EventArgs e)
        {
            if (timer.Enabled)
            {
                timer.Stop();
                btnStart.Text = "开始 (Enter)";
            }
            else
            {
                try
                {
                    string text = File.ReadAllText(FixedFilePath);
                    tokens = new List<string>(text.Split(new[] { ' ', '\t', '\n', '\r' }, 
                        StringSplitOptions.RemoveEmptyEntries));
                    currentIndex = 0;
                    timer.Interval = 60000 / (int)numWPM.Value;
                    timer.Start();
                    btnStart.Text = "停止 (Enter)";
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"读取文件失败: {ex.Message}");
                }
            }
        }



        private void BtnFont_Click(object sender, EventArgs e)
        {
            using (FontDialog fontDialog = new FontDialog())
            {
                fontDialog.Font = lblText.Font;
                if (fontDialog.ShowDialog() == DialogResult.OK)
                {
                    lblText.Font = fontDialog.Font;
                    settings.TextFont = fontDialog.Font;
                }
            }
        }

        private void BtnColors_Click(object sender, EventArgs e)
        {
            // 前景色
            using (ColorDialog colorDialog = new ColorDialog())
            {
                colorDialog.Color = currentForeground;
                if (colorDialog.ShowDialog() == DialogResult.OK)
                {
                    currentForeground = colorDialog.Color;
                    lblText.ForeColor = currentForeground;
                    settings.ForegroundColor = currentForeground;
                }
            }

            // 背景色
            using (ColorDialog colorDialog = new ColorDialog())
            {
                colorDialog.Color = currentBackground;
                if (colorDialog.ShowDialog() == DialogResult.OK)
                {
                    currentBackground = colorDialog.Color;
                    lblText.BackColor = currentBackground;
                    settings.BackgroundColor = currentBackground;
                }
            }
        }

        private void NumWPM_ValueChanged(object sender, EventArgs e)
        {
            settings.WPM = (int)numWPM.Value;
            timer.Interval = 60000 / (int)numWPM.Value;
        }

        private void Timer_Tick(object sender, EventArgs e)
        {
            if (currentIndex < tokens.Count)
            {
                lblText.Text = tokens[currentIndex];
                currentIndex++;
            }
            else if (chkRepeat.Checked)
            {
                currentIndex = 0;
            }
            else
            {
                timer.Stop();
                btnStart.Text = "开始";
            }
        }




        private void ToggleFullscreen()
        {
            if (WindowState == FormWindowState.Maximized)
            {
                WindowState = lastWindowState;
                FormBorderStyle = FormBorderStyle.Sizable;
            }
            else
            {
                lastWindowState = WindowState;
                WindowState = FormWindowState.Maximized;
                FormBorderStyle = FormBorderStyle.None;
            }
        }


        private void BtnFullScreen_Click(object sender, EventArgs e)
        {
            ToggleFullscreen();
        }

        private void Form1_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.F11)
            {
                ToggleFullscreen();
            }
            else if (e.KeyCode == Keys.Enter)
            {
                BtnStart_Click(sender, e);
            }
        }

          // 修改保存触发点
        protected override void OnFormClosing(FormClosingEventArgs e)
        {
            settings.ForegroundColor = currentForeground;
            settings.BackgroundColor = currentBackground;
            settings.Save();
            base.OnFormClosing(e);
        }



        // 暗黑主题渲染器
        private class DarkToolStripRenderer : ToolStripProfessionalRenderer
        {
            public DarkToolStripRenderer() : base(new DarkColorTable()) 
            {
                RoundedEdges = false;
            }

            protected override void OnRenderButtonBackground(ToolStripItemRenderEventArgs e)
            {
                if (e.Item.Selected)
                {
                    e.Graphics.FillRectangle(new SolidBrush(Color.FromArgb(60, 60, 60)), e.Item.ContentRectangle);
                }
                else
                {
                    base.OnRenderButtonBackground(e);
                }
            }
        }

        private class DarkColorTable : ProfessionalColorTable
        {
            public override Color ToolStripDropDownBackground => Color.FromArgb(45, 45, 45);
            public override Color MenuItemSelected => Color.FromArgb(60, 60, 60);
            public override Color MenuItemBorder => Color.FromArgb(30, 30, 30);
            public override Color MenuBorder => Color.FromArgb(45, 45, 45);
            public override Color ToolStripBorder => Color.FromArgb(45, 45, 45);
            public override Color ImageMarginGradientBegin => Color.FromArgb(45, 45, 45);
            public override Color ImageMarginGradientMiddle => Color.FromArgb(45, 45, 45);
            public override Color ImageMarginGradientEnd => Color.FromArgb(45, 45, 45);
            public override Color MenuItemPressedGradientBegin => Color.FromArgb(45, 45, 45);
            public override Color MenuItemPressedGradientMiddle => Color.FromArgb(45, 45, 45);
            public override Color MenuItemPressedGradientEnd => Color.FromArgb(45, 45, 45);
            public override Color ButtonSelectedBorder => Color.FromArgb(60, 60, 60);
            public override Color ButtonSelectedGradientBegin => Color.FromArgb(60, 60, 60);
            public override Color ButtonSelectedGradientMiddle => Color.FromArgb(60, 60, 60);
            public override Color ButtonSelectedGradientEnd => Color.FromArgb(60, 60, 60);
            public override Color CheckBackground => Color.FromArgb(30, 30, 30);
            public override Color CheckPressedBackground => Color.FromArgb(60, 60, 60);
            public override Color CheckSelectedBackground => Color.FromArgb(60, 60, 60);
        }

        
    }

    // [保留Settings类代码，与原始代码相同]
    

        public class Settings
        {
            // 序列化用的中间结构
            public class SerializableFont
            {
                public string FontFamily { get; set; }
                public float FontSize { get; set; }
                public FontStyle FontStyle { get; set; }
            }

            // 将Color转换为字符串表示
            public string ForegroundColorHex { get; set; }
            public string BackgroundColorHex { get; set; }
            
            // 字体使用可序列化的中间类型
            public SerializableFont TextFontData { get; set; }
            
            public int WPM { get; set; } = 66;
            public bool Repeat { get; set; }

            [JsonIgnore] // 忽略实际使用的Font类型
            public Font TextFont
            {
                get => new Font(
                    TextFontData.FontFamily,
                    TextFontData.FontSize,
                    TextFontData.FontStyle);
                set => TextFontData = new SerializableFont
                {
                    FontFamily = value.FontFamily.Name,
                    FontSize = value.Size,
                    FontStyle = value.Style
                };
            }

            [JsonIgnore]
            public Color ForegroundColor
            {
                get => ColorTranslator.FromHtml(ForegroundColorHex);
                set => ForegroundColorHex = ColorTranslator.ToHtml(value);
            }

            [JsonIgnore]
            public Color BackgroundColor
            {
                get => ColorTranslator.FromHtml(BackgroundColorHex);
                set => BackgroundColorHex = ColorTranslator.ToHtml(value);
            }

            private static string ConfigPath => 
                Path.Combine(
                    Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
                    "WinFormsApp38",
                    "settings.json");

            public void Save()
            {
                try
                {
                    var dir = Path.GetDirectoryName(ConfigPath);
                    if (!Directory.Exists(dir))
                        Directory.CreateDirectory(dir);
                    
                    File.WriteAllText(ConfigPath, JsonConvert.SerializeObject(this));
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"保存设置失败: {ex.Message}");
                }
            }

            public static Settings Load()
            {
                try
                {
                    if (File.Exists(ConfigPath))
                    {
                        var content = File.ReadAllText(ConfigPath);
                        return JsonConvert.DeserializeObject<Settings>(content);
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"加载设置失败: {ex.Message}");
                }
                
                // 返回默认设置
                var defaults = new Settings
                {
                    ForegroundColor = Color.White,
                    BackgroundColor = Color.FromArgb(30, 30, 30), // 改为暗黑背景色
                    TextFont = new Font("微软雅黑", 24)
                };
                defaults.Save(); // 创建默认配置文件
                return defaults;
            }
        }

        
}

```

end