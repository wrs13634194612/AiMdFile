说明：
forms+ffmpeg实现屏幕录制
1.开始按钮，点击开始录屏
2.设置要保存视频的本地存储路径
3.录屏中...
4.停止按钮，结束录屏
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/dc52fb9b373c434ca27a2c7da048835d.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp1\WinFormsApp1\Form1.cs

```csharp
using System;
using System.Diagnostics;
using System.Windows.Forms;
using System.Drawing;

namespace WinFormsApp1;

public partial class Form1 : Form
{
    // FFmpeg配置
    private readonly string ffmpegPath = @"C:\Users\wangrusheng\AppData\Local\Microsoft\WinGet\Links\ffmpeg.exe";
    
    // 进程控制
    private Process ffmpegProcess;
    private Button recordButton;
    private Button stopButton;
    private string outputPath;

    public Form1()
    {
        InitializeComponent();
        InitializeUI();
    }

    private void InitializeUI()
    {
        // 主标签
        var titleLabel = new Label
        {
            Text = "屏幕录制工具",
            Location = new Point(20, 20),
            AutoSize = true
        };
        Controls.Add(titleLabel);

        // 开始按钮
        recordButton = new Button
        {
            Text = "开始录制",
            Location = new Point(20, 50),
            Size = new Size(80, 30)
        };
        recordButton.Click += StartRecording_Click;
        Controls.Add(recordButton);

        // 停止按钮
        stopButton = new Button
        {
            Text = "停止录制",
            Location = new Point(120, 50),
            Size = new Size(80, 30),
            Enabled = false
        };
        stopButton.Click += StopRecording_Click;
        Controls.Add(stopButton);
    }

    private void StartRecording_Click(object sender, EventArgs e)
    {
        using (var saveDialog = new SaveFileDialog())
        {
            saveDialog.Filter = "MP4视频|*.mp4";
            if (saveDialog.ShowDialog() == DialogResult.OK)
            {
                outputPath = saveDialog.FileName;
                StartFFmpegProcess();
                ToggleControls(true);
            }
        }
    }

    private void StopRecording_Click(object sender, EventArgs e)
    {
        StopFFmpegProcess();
        ToggleControls(false);
    }

    private void StartFFmpegProcess()
    {
        try
        {
            ffmpegProcess = new Process
            {
                StartInfo = new ProcessStartInfo
                {
                    FileName = ffmpegPath,
                    Arguments = $"-y -f gdigrab -framerate 30 -i desktop -c:v libx264 -preset ultrafast \"{outputPath}\"",
                    UseShellExecute = false,
                    CreateNoWindow = true,
                    RedirectStandardInput = true,
                    RedirectStandardError = true
                }
            };

            ffmpegProcess.ErrorDataReceived += (s, args) => Debug.WriteLine($"[FFmpeg] {args.Data}");
            ffmpegProcess.Start();
            ffmpegProcess.BeginErrorReadLine();
        }
        catch (Exception ex)
        {
            MessageBox.Show($"启动失败：{ex.Message}", "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    }

    private void StopFFmpegProcess()
    {
        try
        {
            if (ffmpegProcess != null && !ffmpegProcess.HasExited)
            {
                // 发送退出命令
                ffmpegProcess.StandardInput.WriteLine("q");
                ffmpegProcess.WaitForExit(3000);
                
                if (!ffmpegProcess.HasExited)
                {
                    ffmpegProcess.Kill();
                    MessageBox.Show("视频已强制保存", "警告", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                }
                else
                {
                    MessageBox.Show($"保存成功：{outputPath}", "完成", MessageBoxButtons.OK, MessageBoxIcon.Information);
                }
            }
        }
        finally
        {
            ffmpegProcess?.Dispose();
            ffmpegProcess = null;
        }
    }

    private void ToggleControls(bool recording)
    {
        recordButton.Enabled = !recording;
        stopButton.Enabled = recording;
    }
}

```

step2:10秒版录屏

```csharp
 修改：

1.新增停止录制按钮，
2.新增录制时间文本
3.点击停止录制，将录制视频保存在本地

using System;
using System.Diagnostics;
using System.Windows.Forms;

namespace WinFormsApp1;

public partial class Form1 : Form
{
    // 配置FFmpeg路径（需修改为实际路径）
    private readonly string ffmpegPath = @"C:\Users\wangrusheng\AppData\Local\Microsoft\WinGet\Links\ffmpeg.exe";     
        
    public Form1()
    {
        InitializeComponent();

        // 添加基础标签
        Label label = new Label();
        label.Text = "屏幕录制工具";
        label.AutoSize = true;
        label.Location = new Point(20, 20);
        this.Controls.Add(label);

        // 添加录屏按钮
        Button recordButton = new Button();
        recordButton.Text = "开始录屏";
        recordButton.Location = new Point(20, 50);
        recordButton.Click += RecordButton_Click;
        this.Controls.Add(recordButton);
    }

    private void RecordButton_Click(object sender, EventArgs e)
    {
        using (SaveFileDialog saveDialog = new SaveFileDialog())
        {
            saveDialog.Filter = "MP4视频|*.mp4";
            saveDialog.Title = "保存录屏文件";
            
            if (saveDialog.ShowDialog() == DialogResult.OK)
            {
                // 构建FFmpeg命令参数
                string arguments = $"-y -f gdigrab -framerate 30 -i desktop " +
                                   $"-t 10 -c:v libx264 -preset ultrafast \"{saveDialog.FileName}\"";

                // 配置进程参数
                ProcessStartInfo startInfo = new ProcessStartInfo
                {
                    FileName = ffmpegPath,
                    Arguments = arguments,
                    UseShellExecute = false,
                    CreateNoWindow = true,
                    RedirectStandardError = true
                };

                try
                {
                    using (Process process = new Process())
                    {
                        process.StartInfo = startInfo;
                        
                        // 接收错误输出
                        process.ErrorDataReceived += (s, args) => 
                            Debug.WriteLine($"FFmpeg输出: {args.Data}");
                        
                        process.Start();
                        process.BeginErrorReadLine();
                        
                        // 显示录制提示
                        MessageBox.Show("开始录制屏幕，10秒后自动保存...", "录制中", 
                            MessageBoxButtons.OK, MessageBoxIcon.Information);
                        
                        // 等待录制完成
                        process.WaitForExit(11000); // 多给1秒缓冲时间

                        if (process.HasExited && process.ExitCode == 0)
                        {
                            MessageBox.Show($"视频保存成功：{saveDialog.FileName}", 
                                "完成", MessageBoxButtons.OK, MessageBoxIcon.Information);
                        }
                        else
                        {
                            MessageBox.Show("视频生成失败，请检查FFmpeg配置", 
                                "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
                        }
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"程序异常：{ex.Message}", 
                        "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
                }
            }
        }
    }
}

```

end