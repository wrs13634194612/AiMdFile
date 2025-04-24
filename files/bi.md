 说明：
forms实现任务文档功能
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/89b76e95c5584188afde53bd18d44c6f.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp26\WinFormsApp26\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Text.RegularExpressions;
using System.Windows.Forms;

namespace WinFormsApp26
{
    /*本地文件存储路径  C:\Users\wangrusheng\AppData\Roaming\TodoListApp\tasks.dat*/
    public partial class Form1 : Form
    {
        // 控件声明
        private TextBox txtTitle = new TextBox();
        private TextBox txtDescription = new TextBox();
        private ComboBox cmbPriority = new ComboBox();
        private DateTimePicker dtpDueDate = new DateTimePicker();
        private ListBox lstTasks = new ListBox();
        private Button btnNew = new Button();
        private Button btnSave = new Button();
        private Button btnDelete = new Button();

        // 数据相关
        private BindingList<Task> tasks = new BindingList<Task>();
        private Task currentTask = null;
        private string dataFilePath;

        public Form1()
        {
            InitializeDataPath();
            InitializeComponents();
            LoadTasks();
            SetupDataBinding();
        }

        private void InitializeDataPath()
        {
            var appDataDir = Path.Combine(
                Environment.GetFolderPath(Environment.SpecialFolder.ApplicationData),
                "TodoListApp");
            
            Directory.CreateDirectory(appDataDir);
            dataFilePath = Path.Combine(appDataDir, "tasks.dat");
        }

        private void InitializeComponents()
        {
            // 窗体设置
            this.Text = "Todo List Manager";
            this.Size = new Size(1000, 600);
            this.StartPosition = FormStartPosition.CenterScreen;

            // 布局参数
            int splitPosition = 350;
            int leftMargin = 20;
            int rightMargin = splitPosition + 20;
            int topMargin = 20;
            int controlWidth = 300;

            // ============== 左侧任务列表 ==============
            lstTasks.Anchor = AnchorStyles.Top | AnchorStyles.Bottom | AnchorStyles.Left | AnchorStyles.Right;
            lstTasks.Location = new Point(leftMargin, topMargin);
            lstTasks.Size = new Size(splitPosition - 40, this.ClientSize.Height - 60);
            lstTasks.DoubleClick += LstTasks_DoubleClick;
            this.Controls.Add(lstTasks);

            // ============== 右侧编辑区域 ==============
            // 标题
            var lblTitle = new Label
            {
                Text = "Title:",
                Location = new Point(rightMargin, topMargin)
            };
            this.Controls.Add(lblTitle);

            txtTitle.Location = new Point(rightMargin + 120, topMargin);
            txtTitle.Size = new Size(controlWidth, 25);
            this.Controls.Add(txtTitle);

            // 描述
            var lblDesc = new Label
            {
                Text = "Description:",
                Location = new Point(rightMargin, topMargin + 40)
            };
            this.Controls.Add(lblDesc);

            txtDescription.Multiline = true;
            txtDescription.ScrollBars = ScrollBars.Vertical;
            txtDescription.Location = new Point(rightMargin + 120, topMargin + 40);
            txtDescription.Size = new Size(controlWidth, 80);
            this.Controls.Add(txtDescription);

            // 优先级
            var lblPriority = new Label
            {
                Text = "Priority:",
                Location = new Point(rightMargin, topMargin + 130)
            };
            this.Controls.Add(lblPriority);

            cmbPriority.DropDownStyle = ComboBoxStyle.DropDownList;
            cmbPriority.DataSource = Enum.GetValues(typeof(PriorityLevel));
            cmbPriority.Location = new Point(rightMargin + 120, topMargin + 130);
            cmbPriority.Size = new Size(controlWidth, 25);
            this.Controls.Add(cmbPriority);

            // 截止日期
            var lblDueDate = new Label
            {
                Text = "Due Date:",
                Location = new Point(rightMargin, topMargin + 170)
            };
            this.Controls.Add(lblDueDate);

            dtpDueDate.Format = DateTimePickerFormat.Custom;
            dtpDueDate.CustomFormat = "dd/MM/yyyy";
            dtpDueDate.Location = new Point(rightMargin + 120, topMargin + 170);
            dtpDueDate.Size = new Size(controlWidth, 25);
            this.Controls.Add(dtpDueDate);

            // ============== 按钮区域 ==============
            int buttonY = topMargin + 220;
            btnNew.Text = "New";
            btnNew.Location = new Point(rightMargin, buttonY);
            btnNew.Size = new Size(80, 30);
            btnNew.Click += btnNew_Click;
            this.Controls.Add(btnNew);

            btnSave.Text = "Save";
            btnSave.Location = new Point(rightMargin + 100, buttonY);
            btnSave.Size = new Size(80, 30);
            btnSave.Click += btnSave_Click;
            this.Controls.Add(btnSave);

            btnDelete.Text = "Delete";
            btnDelete.Location = new Point(rightMargin + 200, buttonY);
            btnDelete.Size = new Size(80, 30);
            btnDelete.Click += btnDelete_Click;
            this.Controls.Add(btnDelete);
        }

        private void SetupDataBinding()
        {
            lstTasks.DataSource = tasks;
            lstTasks.DisplayMember = "DisplayText";
        }

        #region 数据操作
        private void LoadTasks()
        {
            try
            {
                if (File.Exists(dataFilePath))
                {
                    var loaded = Task.LoadFromFile(dataFilePath);
                    tasks.Clear();
                    foreach (var task in loaded.OrderBy(t => t.Priority))
                    {
                        tasks.Add(task);
                    }
                    Task.ResetIdCounter(tasks);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"加载任务失败: {ex.Message}");
            }
        }

        private void SaveTasks()
        {
            try
            {
                Task.SaveToFile(dataFilePath, tasks.ToList());
            }
            catch (Exception ex)
            {
                MessageBox.Show($"保存任务失败: {ex.Message}");
            }
        }
        #endregion

        #region 事件处理
        private void btnNew_Click(object sender, EventArgs e)
        {
            currentTask = null;
            txtTitle.Clear();
            txtDescription.Clear();
            cmbPriority.SelectedIndex = 0;
            dtpDueDate.Value = DateTime.Today;
            txtTitle.Focus();
        }

        private void btnSave_Click(object sender, EventArgs e)
        {
            try
            {
                if (string.IsNullOrWhiteSpace(txtTitle.Text))
                    throw new ArgumentException("标题不能为空");

                var dueDate = new TaskDate(
                    dtpDueDate.Value.Day,
                    dtpDueDate.Value.Month,
                    dtpDueDate.Value.Year
                );

                if (currentTask == null)
                {
                    var newTask = new Task
                    {
                        Title = txtTitle.Text.Trim(),
                        Description = txtDescription.Text.Trim(),
                        Priority = (PriorityLevel)cmbPriority.SelectedItem,
                        DueDate = dueDate,
                        Status = TaskStatus.Pending
                    };
                    tasks.Add(newTask);
                }
                else
                {
                    currentTask.Title = txtTitle.Text.Trim();
                    currentTask.Description = txtDescription.Text.Trim();
                    currentTask.Priority = (PriorityLevel)cmbPriority.SelectedItem;
                    currentTask.DueDate = dueDate;
                }

                SaveTasks();
                lstTasks.SelectedIndex = -1;
                currentTask = null;
                btnNew_Click(null, EventArgs.Empty);
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message, "输入错误", 
                    MessageBoxButtons.OK, MessageBoxIcon.Warning);
            }
        }

        private void btnDelete_Click(object sender, EventArgs e)
        {
            if (lstTasks.SelectedItem is Task selectedTask)
            {
                if (MessageBox.Show("确认删除该任务吗？", "删除确认", 
                    MessageBoxButtons.YesNo) == DialogResult.Yes)
                {
                    tasks.Remove(selectedTask);
                    SaveTasks();
                }
            }
        }

        private void LstTasks_DoubleClick(object sender, EventArgs e)
        {
            if (lstTasks.SelectedItem is Task task)
            {
                currentTask = task;
                txtTitle.Text = task.Title;
                txtDescription.Text = task.Description;
                cmbPriority.SelectedItem = task.Priority;
                dtpDueDate.Value = new DateTime(
                    task.DueDate.Year,
                    task.DueDate.Month,
                    task.DueDate.Day
                );
            }
        }
        #endregion
    }

    #region 数据模型
    public class Task
    {
        private static int _idCounter = 1;

        public int Id { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
        public PriorityLevel Priority { get; set; }
        public TaskDate DueDate { get; set; }
        public TaskStatus Status { get; set; }

        public string DisplayText => $"{Title} ({DueDate}) - {Priority}";

        public static List<Task> LoadFromFile(string path)
        {
            var tasks = new List<Task>();
            try
            {
                if (File.Exists(path))
                {
                    var lines = File.ReadAllLines(path);
                    foreach (var line in lines)
                    {
                        if (string.IsNullOrWhiteSpace(line)) continue;

                        var parts = line.Split('|');
                        if (parts.Length != 6) continue;

                        tasks.Add(new Task
                        {
                            Id = int.Parse(parts[0]),
                            Title = parts[1],
                            Description = parts[2],
                            Priority = (PriorityLevel)Enum.Parse(typeof(PriorityLevel), parts[3]),
                            DueDate = TaskDate.Parse(parts[4]),
                            Status = (TaskStatus)Enum.Parse(typeof(TaskStatus), parts[5])
                        });
                    }
                }
            }
            catch { /* 错误处理 */ }
            return tasks;
        }

        public static void SaveToFile(string path, List<Task> tasks)
        {
            var lines = tasks.Select(t => 
                $"{t.Id}|{t.Title}|{t.Description}|{t.Priority}|{t.DueDate}|{t.Status}");
            File.WriteAllLines(path, lines);
        }

        public static void ResetIdCounter(IEnumerable<Task> tasks)
        {
            _idCounter = tasks.Any() ? tasks.Max(t => t.Id) + 1 : 1;
        }
    }

    public struct TaskDate
    {
        public int Day { get; }
        public int Month { get; }
        public int Year { get; }

        public TaskDate(int day, int month, int year)
        {
            if (!IsValidDate(day, month, year))
                throw new ArgumentException("无效的日期");

            Day = day;
            Month = month;
            Year = year;
        }

        public static TaskDate Parse(string input)
        {
            var match = Regex.Match(input, @"(\d{2})/(\d{2})/(\d{4})");
            if (!match.Success)
                throw new FormatException("日期格式应为 dd/MM/yyyy");

            int day = int.Parse(match.Groups[1].Value);
            int month = int.Parse(match.Groups[2].Value);
            int year = int.Parse(match.Groups[3].Value);

            return new TaskDate(day, month, year);
        }

        private static bool IsValidDate(int day, int month, int year)
        {
            try
            {
                new DateTime(year, month, day);
                return true;
            }
            catch
            {
                return false;
            }
        }

        public override string ToString() => $"{Day:00}/{Month:00}/{Year}";
    }

    public enum PriorityLevel { Low, Medium, High }
    public enum TaskStatus { Pending, Completed, Expired }
    #endregion
}
```

end