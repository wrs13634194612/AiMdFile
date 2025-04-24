说明：
我希望将pdf文件转换成jpg文件
请去下载并安装 Ghostscript，gs10050w64.exe
配置环境变量：D:\Program Files\gs\gs10.05.0\bin
本地pdf路径：C:\Users\wangrusheng\Documents\name.pdf
输出文件目录：C:\Users\wangrusheng\Documents\PdfToJpgOutput
效果图:
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e573b4735e8e46e2b6afc4482abeb863.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp18\WinFormsApp18\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WinFormsApp18
{
    public partial class Form1 : Form
    {
        // 固定PDF路径配置 C:\Users\wangrusheng\Documents
        public const string FixedPdfPath = @"C:\Users\wangrusheng\Documents\age.pdf";
        private string _currentTempDir;

        // UI控件
        private TextBox txtSaveDir;
        private NumericUpDown numStartPage;
        private NumericUpDown numEndPage;
        private ComboBox cmbQuality;
        private CheckBox chkMerge;
        private ComboBox cmbOrientation;

        public Form1()
        {
            InitializeComponent();
            InitializeComponents();
            LoadSettings();
        }

        private void InitializeComponents()
        {
            // 固定PDF路径显示
            var lblFile = new Label
            {
                Text = "PDF文件路径:",
                Location = new Point(20, 20),
                AutoSize = true
            };

            var txtFixedPath = new TextBox
            {
                Text = FixedPdfPath,
                Location = new Point(120, 17),
                Width = 400,
                ReadOnly = true,
                BackColor = SystemColors.Window
            };

            // 保存目录区域
            var lblSaveDir = new Label
            {
                Text = "保存目录:",
                Location = new Point(20, 60),
                AutoSize = true
            };

            txtSaveDir = new TextBox
            {
                Text = Path.Combine(
                    Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments),
                    "PdfToJpgOutput"),
                Location = new Point(120, 57),
                Width = 400,
                ReadOnly = true
            };

            var btnSelectDir = new Button
            {
                Text = "浏览...",
                Location = new Point(530, 55),
                AutoSize = true
            };
            btnSelectDir.Click += BtnSelectDir_Click;

            // 页码选择
            var lblPages = new Label
            {
                Text = "页码范围:",
                Location = new Point(20, 100),
                AutoSize = true
            };

            numStartPage = new NumericUpDown
            {
                Location = new Point(120, 97),
                Minimum = 1,
                Maximum = 10000
            };

            numEndPage = new NumericUpDown
            {
                Location = new Point(220, 97),
                Minimum = 0,
                Maximum = 10000
            };

            // 质量选择
            var lblQuality = new Label
            {
                Text = "输出质量:",
                Location = new Point(20, 140),
                AutoSize = true
            };

            cmbQuality = new ComboBox
            {
                Location = new Point(120, 137),
                Items = { "300", "400", "500", "600" },
                SelectedIndex = 2
            };

            // 合并选项
            chkMerge = new CheckBox
            {
                Text = "合并为单文件",
                Location = new Point(20, 180),
                AutoSize = true
            };

            cmbOrientation = new ComboBox
            {
                Location = new Point(120, 177),
                Items = { "垂直拼接", "水平拼接" },
                SelectedIndex = 0
            };

            // 操作按钮
            var btnRun = new Button
            {
                Text = "开始转换",
                Location = new Point(20, 220),
                AutoSize = true
            };
            btnRun.Click += BtnRun_Click;

            // 添加所有控件
            Controls.AddRange(new Control[]
            {
                lblFile, txtFixedPath,
                lblSaveDir, txtSaveDir, btnSelectDir,
                lblPages, numStartPage, numEndPage,
                lblQuality, cmbQuality,
                chkMerge, cmbOrientation,
                btnRun
            });
        }

        private void BtnSelectDir_Click(object sender, EventArgs e)
        {
            using var dialog = new FolderBrowserDialog
            {
                SelectedPath = txtSaveDir.Text,
                Description = "选择保存目录"
            };

            if (dialog.ShowDialog() == DialogResult.OK)
            {
                if (FileHelper.ValidatePath(dialog.SelectedPath))
                {
                    txtSaveDir.Text = dialog.SelectedPath;
                }
                else
                {
                    MessageBox.Show("选择的目录路径无效");
                }
            }
        }

        private async void BtnRun_Click(object sender, EventArgs e)
        {
            try
            {
                if (!ValidateInputs()) return;

                _currentTempDir = FileHelper.GetTempWorkspace();

                var result = await ConvertPdfToImagesAsync();

                if (result.Success)
                {
                    if (chkMerge.Checked)
                    {
                        await MergeImagesAsync();
                    }

                    MoveFinalFiles();
                    MessageBox.Show("转换成功完成");
                }
                else
                {
                    throw new Exception(result.ErrorMessage);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"操作失败: {ex.Message}");
            }
            finally
            {
                FileHelper.CleanTempFiles(_currentTempDir);
            }
        }

        private bool ValidateInputs()
        {
            if (!File.Exists(FixedPdfPath))
            {
                MessageBox.Show($"自动读取的PDF文件不存在：{FixedPdfPath}");
                return false;
            }

            if (numStartPage.Value > numEndPage.Value && numEndPage.Value != 0)
            {
                MessageBox.Show("起始页码不能大于结束页码");
                return false;
            }

            return true;
        }

        private async Task<(bool Success, string ErrorMessage)> ConvertPdfToImagesAsync()
        {
            var args = new List<string>
            {
                "-dNOSAFER",
                $"-r{cmbQuality.SelectedItem}",
                "-sDEVICE=jpeg",
                "-dBATCH",
                "-dNOPAUSE",
                "-dEPSCrop",
                $"-dFirstPage={numStartPage.Value}",
                numEndPage.Value > 0 ? $"-dLastPage={numEndPage.Value}" : "",
                $"-sOutputFile={Path.Combine(_currentTempDir, "page_%d.jpg")}",
                $"\"{FixedPdfPath}\""
            };

            var (success, output) = FileHelper.ExecuteCommand("gswin64c", string.Join(" ", args), _currentTempDir);
            return (success, output);
        }

        private async Task MergeImagesAsync()
        {
            var args = $"{Path.Combine(_currentTempDir, "page_*.jpg")} " +
                       $"{(cmbOrientation.SelectedIndex == 0 ? "-append" : "+append")} " +
                       $"{Path.Combine(_currentTempDir, "merged.jpg")}";

            var (success, output) = FileHelper.ExecuteCommand("magick", args, _currentTempDir);
            if (!success) throw new Exception(output);
        }

        private void MoveFinalFiles()
        {
            var destDir = txtSaveDir.Text;
            Directory.CreateDirectory(destDir);

            var filesToMove = chkMerge.Checked
                ? new[] { "merged.jpg" }
                : Directory.GetFiles(_currentTempDir, "page_*.jpg");

            foreach (var file in filesToMove)
            {
                var destPath = Path.Combine(destDir, Path.GetFileName(file));
                File.Move(file, destPath, true);
            }
        }

        private void LoadSettings()
        {
            // 可添加其他配置加载逻辑
            numStartPage.Value = 1;
            numEndPage.Value = 0;
        }

        private void SaveSettings()
        {
            // 可添加配置保存逻辑
        }

        protected override void OnFormClosing(FormClosingEventArgs e)
        {
            SaveSettings();
            base.OnFormClosing(e);
        }
    }
}
```

step2:C:\Users\wangrusheng\RiderProjects\WinFormsApp18\WinFormsApp18\FileHelper.cs

```csharp
using System;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices;

namespace WinFormsApp18
{
    public static class FileHelper
    {
        private const string GhostscriptExeName = "gswin64c.exe";
        private static readonly string[] GhostscriptSearchPaths = 
        {
            @"D:\Program Files\gs"
        };

        public static string GetTempWorkspace()
        {
            var tempDir = Path.Combine(
                Path.GetTempPath(),
                "PdfToJpg",
                DateTime.Now.ToString("yyyyMMdd_HHmmss")
            );
            
            Directory.CreateDirectory(tempDir);
            return tempDir;
        }

        public static bool ValidatePath(string path)
        {
            try
            {
                var fullPath = Path.GetFullPath(path);
                
                if (path.IndexOfAny(Path.GetInvalidPathChars()) >= 0)
                    return false;

                if (RuntimeInformation.IsOSPlatform(OSPlatform.Windows))
                {
                    if (!fullPath.StartsWith(ApplicationInfo.AppRootPath, StringComparison.OrdinalIgnoreCase))
                        return false;
                }
                else
                {
                    if (!fullPath.StartsWith(ApplicationInfo.AppRootPath))
                        return false;
                }

                return true;
            }
            catch
            {
                return false;
            }
        }

        public static (bool Success, string Output) ExecuteCommand(string command, string args, string workingDir)
        {
            try
            {
                var fullCommandPath = command.Equals("gswin64c", StringComparison.OrdinalIgnoreCase)
                    ? FindGhostscriptPath()
                    : command;

                if (fullCommandPath == null)
                    throw new FileNotFoundException("Ghostscript not found. Please install from https://www.ghostscript.com/");

                var startInfo = new ProcessStartInfo
                {
                    FileName = fullCommandPath,
                    Arguments = args,
                    WorkingDirectory = workingDir,
                    UseShellExecute = false,
                    RedirectStandardOutput = true,
                    RedirectStandardError = true,
                    CreateNoWindow = true
                };

                using var process = Process.Start(startInfo);
                var output = process.StandardOutput.ReadToEnd();
                var error = process.StandardError.ReadToEnd();
                
                if (!process.WaitForExit(30000))
                    throw new TimeoutException("Process execution timed out");

                return (
                    process.ExitCode == 0, 
                    $"Exit Code: {process.ExitCode}\nOutput:\n{output}\nErrors:\n{error}"
                );
            }
            catch (Exception ex)
            {
                return (false, ex.Message);
            }
        }

        private static string FindGhostscriptPath()
        {
            // Check if ghostscript is in PATH
            var pathEnv = Environment.GetEnvironmentVariable("PATH") ?? "";
            foreach (var path in pathEnv.Split(Path.PathSeparator))
            {
                var fullPath = Path.Combine(path, GhostscriptExeName);
                if (File.Exists(fullPath))
                    return fullPath;
            }

            // Search common installation directories
            foreach (var basePath in GhostscriptSearchPaths)
            {
                if (!Directory.Exists(basePath)) continue;

                var versions = Directory.GetDirectories(basePath)
                    .OrderByDescending(d => d)
                    .ToList();

                foreach (var versionDir in versions)
                {
                    var exePath = Path.Combine(versionDir, "bin", GhostscriptExeName);
                    if (File.Exists(exePath))
                        return exePath;
                }
            }

            throw new FileNotFoundException($"Ghostscript executable ({GhostscriptExeName}) not found. " +
                "Please install from https://www.ghostscript.com/");
        }

        public static void CleanTempFiles(string tempDir)
        {
            try
            {
                if (Directory.Exists(tempDir))
                {
                    Directory.Delete(tempDir, true);
                }
            }
            catch (Exception ex)
            {
                Debug.WriteLine($"Error cleaning temp files: {ex.Message}");
            }
        }
    }

    public static class ApplicationInfo
    {
        public static string AppRootPath => AppDomain.CurrentDomain.BaseDirectory;
    }
}
```

end