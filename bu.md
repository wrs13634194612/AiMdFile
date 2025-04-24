说明：
form+ffmpeg+opus录音压缩音频
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bbdc9f9ad30a40f6b79dffa854cc1934.png#pic_center)

step1:opus格式录音
C:\Users\wangrusheng\RiderProjects\WinFormsApp11\WinFormsApp11\Form1.cs

```csharp
using System;
using System.Diagnostics;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp11
{
    public partial class Form1 : Form
    {
        // FFmpeg配置
        private readonly string ffmpegPath = @"C:\Users\wangrusheng\AppData\Local\Microsoft\WinGet\Links\ffmpeg.exe";
        /*
         *输入cmd指令，获取麦克风地址，动态获取，把你的麦克风地址 替换成下面的地址
         C:\Users\wangrusheng>ffmpeg -list_devices true -f dshow -i dummy
            [dshow @ 00000205b9227200] "阵列麦克风 (AMD Audio Device)" (audio)
            [dshow @ 00000205b9227200]   Alternative name "@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{2D7B50BB-AD6C-4823-8F7F-5552B9D873B9}"
         */
        private const string AudioDevice = @"audio=@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{2D7B50BB-AD6C-4823-8F7F-5552B9D873B9}";
        private const string OutputFile = "recording.opus";
        
        private Process ffmpegProcess;
        private Label statusLabel;
        private Button btnStart;
        private Button btnStop;

        public Form1()
        {
            InitializeComponent();
            InitializeUI();
            CheckPrerequisites();
        }

        private void InitializeUI()
        {
            // 窗体设置
            this.Text = "音频录音机";
            this.Size = new System.Drawing.Size(300, 200);

            // 状态标签
            statusLabel = new Label
            {
                Text = "准备就绪",
                Location = new System.Drawing.Point(20, 20),
                AutoSize = true
            };
            this.Controls.Add(statusLabel);

            // 开始按钮
            btnStart = new Button
            {
                Text = "开始录音",
                Location = new System.Drawing.Point(20, 60),
                Size = new System.Drawing.Size(100, 40)
            };
            btnStart.Click += BtnStart_Click;
            this.Controls.Add(btnStart);

            // 停止按钮
            btnStop = new Button
            {
                Text = "停止录音",
                Location = new System.Drawing.Point(140, 60),
                Size = new System.Drawing.Size(100, 40),
                Enabled = false
            };
            btnStop.Click += BtnStop_Click;
            this.Controls.Add(btnStop);

            // 文件路径显示
            var pathLabel = new Label
            {
                Text = $"保存路径：{Path.Combine(Application.StartupPath, OutputFile)}",
                Location = new System.Drawing.Point(20, 120),
                AutoSize = true
            };
            this.Controls.Add(pathLabel);
        }

        private void CheckPrerequisites()
        {
            // 检查FFmpeg是否存在
            if (!File.Exists(ffmpegPath))
            {
                MessageBox.Show($"FFmpeg未找到：{ffmpegPath}");
                btnStart.Enabled = false;
            }
        }

        private void BtnStart_Click(object sender, EventArgs e)
        {
            if (ffmpegProcess != null && !ffmpegProcess.HasExited)
            {
                MessageBox.Show("请先停止当前录音");
                return;
            }

            try
            {
                var startInfo = new ProcessStartInfo
                {
                    FileName = ffmpegPath,
                    // 修改2: 添加opus编码参数
                    Arguments = $"-f dshow -i \"{AudioDevice}\" -c:a libopus -b:a 64k -y \"{OutputFile}\"",
                    UseShellExecute = false,
                    CreateNoWindow = true,
                    RedirectStandardInput = true,
                    RedirectStandardError = true
                };

                ffmpegProcess = new Process { StartInfo = startInfo };
                ffmpegProcess.ErrorDataReceived += (s, args) => 
                    Debug.WriteLine($"[FFmpeg] {args.Data}");
                
                ffmpegProcess.Start();
                ffmpegProcess.BeginErrorReadLine();
                
                btnStart.Enabled = false;
                btnStop.Enabled = true;
                statusLabel.Text = "录音进行中...";
            }
            catch (Exception ex)
            {
                MessageBox.Show($"启动失败：{ex.Message}");
                ResetControls();
            }
        }

        private void BtnStop_Click(object sender, EventArgs e)
        {
            if (ffmpegProcess == null || ffmpegProcess.HasExited) return;

            try
            {
                ffmpegProcess.StandardInput.WriteLine("q");
                if (!ffmpegProcess.WaitForExit(1500))
                {
                    ffmpegProcess.Kill();
                }
                
                statusLabel.Text = $"录音已保存到：{OutputFile}";
            }
            catch (Exception ex)
            {
                MessageBox.Show($"停止失败：{ex.Message}");
            }
            finally
            {
                ResetControls();
                ffmpegProcess?.Dispose();
                ffmpegProcess = null;
            }
        }

        private void ResetControls()
        {
            btnStart.Enabled = true;
            btnStop.Enabled = false;
        }

        protected override void OnFormClosing(FormClosingEventArgs e)
        {
            if (ffmpegProcess != null && !ffmpegProcess.HasExited)
            {
                ffmpegProcess.Kill();
                ffmpegProcess.Dispose();
            }
            base.OnFormClosing(e);
        }
    }
}
```

step2:wav格式录音

```csharp
 至少普通录音可以了
 
 
 using System;
using System.Diagnostics;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp11
{
    public partial class Form1 : Form
    {
        // FFmpeg配置
        private readonly string ffmpegPath = @"C:\Users\wangrusheng\AppData\Local\Microsoft\WinGet\Links\ffmpeg.exe";
        /*
         *输入cmd指令，获取麦克风地址，动态获取，把你的麦克风地址 替换成下面的地址
         C:\Users\wangrusheng>ffmpeg -list_devices true -f dshow -i dummy
            [dshow @ 00000205b9227200] "阵列麦克风 (AMD Audio Device)" (audio)
            [dshow @ 00000205b9227200]   Alternative name "@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{2D7B50BB-AD6C-4823-8F7F-5552B9D873B9}"
         */
        private const string AudioDevice = @"audio=@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{2D7B50BB-AD6C-4823-8F7F-5552B9D873B9}";
        private const string OutputFile = "recording.wav";
        
        private Process ffmpegProcess;
        private Label statusLabel;
        private Button btnStart;
        private Button btnStop;

        public Form1()
        {
            InitializeComponent();
            InitializeUI();
            CheckPrerequisites();
        }

        private void InitializeUI()
        {
            // 窗体设置
            this.Text = "音频录音机";
            this.Size = new System.Drawing.Size(300, 200);

            // 状态标签
            statusLabel = new Label
            {
                Text = "准备就绪",
                Location = new System.Drawing.Point(20, 20),
                AutoSize = true
            };
            this.Controls.Add(statusLabel);

            // 开始按钮
            btnStart = new Button
            {
                Text = "开始录音",
                Location = new System.Drawing.Point(20, 60),
                Size = new System.Drawing.Size(100, 40)
            };
            btnStart.Click += BtnStart_Click;
            this.Controls.Add(btnStart);

            // 停止按钮
            btnStop = new Button
            {
                Text = "停止录音",
                Location = new System.Drawing.Point(140, 60),
                Size = new System.Drawing.Size(100, 40),
                Enabled = false
            };
            btnStop.Click += BtnStop_Click;
            this.Controls.Add(btnStop);

            // 文件路径显示
            var pathLabel = new Label
            {
                Text = $"保存路径：{Path.Combine(Application.StartupPath, OutputFile)}",
                Location = new System.Drawing.Point(20, 120),
                AutoSize = true
            };
            this.Controls.Add(pathLabel);
        }

        private void CheckPrerequisites()
        {
            // 检查FFmpeg是否存在
            if (!File.Exists(ffmpegPath))
            {
                MessageBox.Show($"FFmpeg未找到：{ffmpegPath}");
                btnStart.Enabled = false;
            }
        }

        private void BtnStart_Click(object sender, EventArgs e)
        {
            if (ffmpegProcess != null && !ffmpegProcess.HasExited)
            {
                MessageBox.Show("请先停止当前录音");
                return;
            }

            try
            {
                var startInfo = new ProcessStartInfo
                {
                    FileName = ffmpegPath,
                    Arguments = $"-f dshow -i \"{AudioDevice}\" -y \"{OutputFile}\"",
                    UseShellExecute = false,
                    CreateNoWindow = true,
                    RedirectStandardInput = true,
                    RedirectStandardError = true
                };

                ffmpegProcess = new Process { StartInfo = startInfo };
                ffmpegProcess.ErrorDataReceived += (s, args) => 
                    Debug.WriteLine($"[FFmpeg] {args.Data}");
                
                ffmpegProcess.Start();
                ffmpegProcess.BeginErrorReadLine();
                
                btnStart.Enabled = false;
                btnStop.Enabled = true;
                statusLabel.Text = "录音进行中...";
            }
            catch (Exception ex)
            {
                MessageBox.Show($"启动失败：{ex.Message}");
                ResetControls();
            }
        }

        private void BtnStop_Click(object sender, EventArgs e)
        {
            if (ffmpegProcess == null || ffmpegProcess.HasExited) return;

            try
            {
                ffmpegProcess.StandardInput.WriteLine("q");
                if (!ffmpegProcess.WaitForExit(1500))
                {
                    ffmpegProcess.Kill();
                }
                
                statusLabel.Text = $"录音已保存到：{OutputFile}";
            }
            catch (Exception ex)
            {
                MessageBox.Show($"停止失败：{ex.Message}");
            }
            finally
            {
                ResetControls();
                ffmpegProcess?.Dispose();
                ffmpegProcess = null;
            }
        }

        private void ResetControls()
        {
            btnStart.Enabled = true;
            btnStop.Enabled = false;
        }

        protected override void OnFormClosing(FormClosingEventArgs e)
        {
            if (ffmpegProcess != null && !ffmpegProcess.HasExited)
            {
                ffmpegProcess.Kill();
                ffmpegProcess.Dispose();
            }
            base.OnFormClosing(e);
        }
    }
}



```

steo3: 输出文件
同样是11秒的录音文件，保存后的大小，相差20倍

recording.opus    98kb
recording.wav	1895kb
end