说明：
forms实现录音降噪fft频谱
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/593d12ce97114e33926d89c32ebd461a.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp10\WinFormsApp10\WinFormsApp10.csproj

```bash
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>net9.0-windows</TargetFramework>
        <Nullable>enable</Nullable>
        <UseWindowsForms>true</UseWindowsForms>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <ItemGroup>
      <PackageReference Include="MathNet.Numerics" Version="6.0.0-beta2" />
      <PackageReference Include="MathNet.Numerics.MKL.Win-x64" Version="3.0.0" />
      <PackageReference Include="NAudio" Version="2.2.1" />
    </ItemGroup>

</Project>
```

step2:C:\Users\wangrusheng\RiderProjects\WinFormsApp10\WinFormsApp10\Form1.cs

```csharp
namespace WinFormsApp10;

public partial class Form1 : Form
{
    
    private AudioManager audioManager;
    private readonly WaveformPainter waveformPainter = new();
    private Button btnRecord;
    private Button btnStop;
    private TrackBar trbThreshold;
    private Label lblThreshold;
    
    
    public Form1()
    {
        InitializeComponent();
    
        this.btnRecord = new Button();
        this.btnStop = new Button();
        this.trbThreshold = new TrackBar();
        this.lblThreshold = new Label();
        ((System.ComponentModel.ISupportInitialize)(this.trbThreshold)).BeginInit();
        
        // btnRecord
        this.btnRecord.Location = new System.Drawing.Point(20, 20);
        this.btnRecord.Name = "btnRecord";
        this.btnRecord.Size = new System.Drawing.Size(80, 30);
        this.btnRecord.Text = "开始录音";
        this.btnRecord.Click += new EventHandler(this.btnRecord_Click);
        
        // btnStop
        this.btnStop.Location = new System.Drawing.Point(120, 20);
        this.btnStop.Name = "btnStop";
        this.btnStop.Size = new System.Drawing.Size(80, 30);
        this.btnStop.Text = "停止录音";
        this.btnStop.Enabled = false;
        this.btnStop.Click += new EventHandler(this.btnStop_Click);
        
        // trbThreshold
        this.trbThreshold.Location = new System.Drawing.Point(20, 60);
        this.trbThreshold.Width = 200;
        this.trbThreshold.Minimum = -60;
        this.trbThreshold.Maximum = 0;
        this.trbThreshold.Value = -30;
        this.trbThreshold.Scroll += new EventHandler(this.trbThreshold_Scroll);
        
        // lblThreshold
        this.lblThreshold.Location = new System.Drawing.Point(240, 60);
        this.lblThreshold.AutoSize = true;
        this.lblThreshold.Text = "Threshold: -30 dB";
        
        // MainForm
        this.AutoScaleDimensions = new System.Drawing.SizeF(7F, 17F);
        this.AutoScaleMode = AutoScaleMode.Font;
        this.ClientSize = new System.Drawing.Size(400, 300);
        this.Controls.Add(this.btnRecord);
        this.Controls.Add(this.btnStop);
        this.Controls.Add(this.trbThreshold);
        this.Controls.Add(this.lblThreshold);
        this.Name = "MainForm";
        this.Text = "Audio Processor";
        ((System.ComponentModel.ISupportInitialize)(this.trbThreshold)).EndInit();
        
        InitializeAudio();
        InitializeWaveformPainter();
        
    }
    
    
    private void InitializeWaveformPainter()
    {
        waveformPainter.Dock = DockStyle.Bottom;
        waveformPainter.Height = 200;
        waveformPainter.BackColor = Color.Black;
        Controls.Add(waveformPainter);
    }

    private void InitializeAudio()
    {
        audioManager = new AudioManager();
        audioManager.DataAvailable += (_, buffer) => 
        {
            waveformPainter.AddSamples(buffer);
        };
    }

    private void btnRecord_Click(object sender, EventArgs e)
    {
        audioManager.StartRecording();
        btnRecord.Enabled = false;
        btnStop.Enabled = true;
    }

    private void btnStop_Click(object sender, EventArgs e)
    {
        audioManager.StopRecording();
        btnRecord.Enabled = true;
        btnStop.Enabled = false;
    }

    private void trbThreshold_Scroll(object sender, EventArgs e)
    {
        audioManager.Processor.ThresholdDb = trbThreshold.Value;
        lblThreshold.Text = $"Threshold: {trbThreshold.Value} dB";
    }

    protected override void OnFormClosing(FormClosingEventArgs e)
    {
        audioManager.Dispose();
        base.OnFormClosing(e);
    }



}
```

step3:C:\Users\wangrusheng\RiderProjects\WinFormsApp10\WinFormsApp10\AudioManager.cs

```csharp
using NAudio.Wave;
using System;
using System.IO;

namespace WinFormsApp10
{
    public class AudioManager : IDisposable
    {
        private const int BufferSize = 4096;
        
        public AudioProcessor Processor { get; } = new AudioProcessor();
        private WaveInEvent waveIn;
        private float[] reusableBuffer = new float[BufferSize];
        private WaveFileWriter writer;
        private string outputFilePath;

        public event EventHandler<float[]> DataAvailable;

        public AudioManager()
        {
            InitializeDevices();
        }

        private void InitializeDevices()
        {
            waveIn = new WaveInEvent 
            {
                DeviceNumber = 0,
                WaveFormat = new WaveFormat(44100, 16, 1),
                BufferMilliseconds = 50,
                NumberOfBuffers = 4
            };

            waveIn.DataAvailable += (s, e) => 
            {
                int sampleCount = e.BytesRecorded / 2;
                EnsureBufferCapacity(ref reusableBuffer, sampleCount);
                
                // 将原始字节数据转换为float数组进行处理
                for (int i = 0; i < sampleCount; i++)
                {
                    reusableBuffer[i] = BitConverter.ToInt16(e.Buffer, i * 2) / 32768f;
                }
                
                // 应用音频处理
                Processor.ProcessAudio(reusableBuffer, waveIn.WaveFormat.SampleRate);
                
                // 触发数据可用事件（用于波形绘制）
                DataAvailable?.Invoke(this, reusableBuffer);
                
                // 将处理后的float数据转换为字节并写入文件
                if (writer != null)
                {
                    byte[] processedBytes = new byte[sampleCount * 2];
                    for (int i = 0; i < sampleCount; i++)
                    {
                        short sample = (short)(reusableBuffer[i] * 32767f);
                        processedBytes[i * 2] = (byte)(sample & 0xFF);
                        processedBytes[i * 2 + 1] = (byte)(sample >> 8);
                    }
                    writer.Write(processedBytes, 0, processedBytes.Length);
                }
            };
        }

        private void EnsureBufferCapacity(ref float[] buffer, int requiredSize)
        {
            if (buffer.Length < requiredSize)
            {
                Array.Resize(ref buffer, requiredSize);
            }
        }

        public void StartRecording()
        {
            // 确保录音目录存在
            string documentsPath = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
            string recordingsDir = Path.Combine(documentsPath, "Recordings");
            Directory.CreateDirectory(recordingsDir);
            
            // 生成唯一文件名
            outputFilePath = Path.Combine(recordingsDir, 
                $"Recording_{DateTime.Now:yyyyMMddHHmmss}.wav");
            
            writer = new WaveFileWriter(outputFilePath, waveIn.WaveFormat);
            waveIn.StartRecording();
        }

        public void StopRecording()
        {
            waveIn.StopRecording();
            writer?.Dispose();
            writer = null;
        }

        public void Dispose()
        {
            writer?.Dispose();
            waveIn?.Dispose();
            GC.SuppressFinalize(this);
        }
    }
}
```

step4:C:\Users\wangrusheng\RiderProjects\WinFormsApp10\WinFormsApp10\AudioProcessor.cs

```csharp
// C:\Users\wangrusheng\RiderProjects\WinFormsApp10\WinFormsApp10\AudioProcessor.cs
using MathNet.Numerics;
using MathNet.Numerics.IntegralTransforms;
using System;
using System.Collections.Generic;
using System.Linq;
using Complex = MathNet.Numerics.Complex32;

namespace WinFormsApp10
{
    public class AudioProcessor
    {
        private const int FftSize = 4096;
        private const int HopSize = FftSize / 2;
        
        private readonly float[] hannWindow;
        private readonly Complex[] fftBuffer;
        private readonly Queue<float> lookaheadBuffer = new Queue<float>(480); // 10ms@48kHz
        private float hpLastIn, hpLastOut;
        private float currentGain = 1f;

        public float ThresholdDb { get; set; } = -30f;
        public float LowpassFreq { get; set; } = 20000f;
        public float HighpassFreq { get; set; } = 75f;
        public float LimiterThreshold { get; set; } = 0.95f;

        public AudioProcessor()
        {
            hannWindow = Window.Hann(FftSize).Select(x => (float)x).ToArray();
            fftBuffer = new Complex[FftSize];
        }

        public void ProcessAudio(float[] buffer, int sampleRate)
        {
            ApplyFilters(buffer, sampleRate);
            ApplySpectralGate(buffer, sampleRate);
            ApplyLookaheadLimiter(buffer);
        }

        private void ApplyFilters(float[] buffer, int sampleRate)
        {
            float dt = 1f / sampleRate;
            float rc = 1f / (2 * (float)Math.PI * HighpassFreq);
            float alpha = rc / (rc + dt);

            for (int i = 0; i < buffer.Length; i++)
            {
                float inVal = buffer[i];
                float outVal = alpha * (hpLastOut + inVal - hpLastIn);
                buffer[i] = outVal;
                hpLastIn = inVal;
                hpLastOut = outVal;
            }
        }

        private void ApplySpectralGate(float[] buffer, int sampleRate)
        {
            float threshold = (float)Math.Pow(10, ThresholdDb / 20);

            for (int i = 0; i < buffer.Length; i += HopSize)
            {
                int length = Math.Min(FftSize, buffer.Length - i);
                
                Array.Clear(fftBuffer, 0, FftSize);
                for (int j = 0; j < length; j++)
                {
                    fftBuffer[j] = new Complex(buffer[i + j] * hannWindow[j], 0f);
                }

                Fourier.Forward(fftBuffer, FourierOptions.Matlab);
                
                for (int j = 0; j < fftBuffer.Length; j++)
                {
                    float freq = j * sampleRate / (float)FftSize;
                    if (freq < HighpassFreq || freq > LowpassFreq)
                    {
                        fftBuffer[j] = Complex.Zero;
                        continue;
                    }

                    if (fftBuffer[j].Magnitude < threshold)
                        fftBuffer[j] = Complex.Zero;
                }

                Fourier.Inverse(fftBuffer, FourierOptions.Matlab);
                
                int end = Math.Min(i + FftSize, buffer.Length);
                for (int j = 0; j < end - i; j++)
                {
                    buffer[i + j] += (float)(fftBuffer[j].Real * hannWindow[j] / FftSize);
                }
            }
        }

        private void ApplyLookaheadLimiter(float[] buffer)
        {
            const float attack = 0.999f; 
            const float release = 0.999f;

            foreach (var sample in buffer)
            {
                lookaheadBuffer.Enqueue(sample);
            }

            for (int i = 0; i < buffer.Length; i++)
            {
                float peak = lookaheadBuffer.Take(480).Max(x => Math.Abs(x));
                float targetGain = peak > LimiterThreshold ? LimiterThreshold / peak : 1f;

                if (targetGain < currentGain)
                    currentGain = targetGain;
                else
                    currentGain = currentGain * release + targetGain * (1 - release);

                buffer[i] = lookaheadBuffer.Dequeue() * currentGain;
            }
        }
    }
}
```

step5:C:\Users\wangrusheng\RiderProjects\WinFormsApp10\WinFormsApp10\WaveformPainter.cs

```csharp
using System.Drawing;
using System.Windows.Forms;

namespace WinFormsApp10;

public class WaveformPainter : Control
{
    private readonly System.Collections.Concurrent.ConcurrentQueue<float> samplesQueue 
        = new();
    private readonly Pen waveformPen = new Pen(Color.Cyan, 1f);
    private readonly float[] displayBuffer = new float[4096];

    public void AddSamples(float[] newSamples)
    {
        foreach (var sample in newSamples)
        {
            samplesQueue.Enqueue(sample);
        }
        
        while (samplesQueue.Count > 4096)
        {
            samplesQueue.TryDequeue(out _);
        }
        
        if (InvokeRequired)
        {
            BeginInvoke(new Action(Invalidate));
        }
        else
        {
            Invalidate();
        }
    }

    protected override void OnPaint(PaintEventArgs e)
    {
        base.OnPaint(e);
        
        int count = samplesQueue.Count;
        if (count < 2) return;

        samplesQueue.CopyTo(displayBuffer, 0);
        var g = e.Graphics;
        float width = ClientSize.Width;
        float height = ClientSize.Height;
        float yCenter = height / 2;

        g.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;
        
        for (int i = 1; i < count; i++)
        {
            float x1 = (i - 1) * width / count;
            float x2 = i * width / count;
            float y1 = yCenter - displayBuffer[i - 1] * yCenter;
            float y2 = yCenter - displayBuffer[i] * yCenter;
            
            g.DrawLine(waveformPen, x1, y1, x2, y2);
        }
    }
}
```

step6:保存录音文件路径C:\Users\wangrusheng\Documents\Recordings


end