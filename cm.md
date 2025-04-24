说明：
forms+mysql增删改查，可以直接操作数据库 很夸张
  感觉这个东西 有点意思 ，
  到时候上线怎么处理 我还不知道 
   也许这种程序，是直接跑在本地的，安装mysql，
   然后建表，然后直接连接数据库，操作数据库就行，
   根本不需要上线 ，你觉得呢  
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7ec8e9f1c79e49bfa0a9aa2c75227062.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp2\WinFormsApp2\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Drawing;
using System.Windows.Forms;
using MySql.Data.MySqlClient;

namespace WinFormsApp2;

public partial class Form1 : Form
{

 
    // MySQL连接配置
    private const string ConnectionString = "server=localhost;user=root;password=123456;database=db_school;Charset=utf8mb4;";

    private DataGridView dataGridView;
    private Button btnLoad;
    private Button btnAdd;
    private Button btnEdit;
    private Button btnDelete;
    
    
    public Form1()
    {
        InitializeComponent();
        CreateControls();
        LoadData(); // 初始加载数据
    }
    
    
        private void CreateControls()
        {
            // 窗体设置
            this.Text = "用户数据管理";
            this.Size = new Size(800, 550);
            this.StartPosition = FormStartPosition.CenterScreen;

            // 标签
            Label label = new Label
            {
                Text = "用户数据列表",
                Font = new Font("微软雅黑", 12, FontStyle.Bold),
                AutoSize = true,
                Location = new Point(20, 20)
            };

            // 数据表格
            dataGridView = new DataGridView
            {
                Location = new Point(20, 60),
                Size = new Size(740, 350),
                Anchor = AnchorStyles.Top | AnchorStyles.Left | AnchorStyles.Right,
                BackgroundColor = Color.White,
                BorderStyle = BorderStyle.Fixed3D,
                AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
                ColumnHeadersHeight = 30,
                EnableHeadersVisualStyles = false,
                SelectionMode = DataGridViewSelectionMode.FullRowSelect
            };

            // 按钮设置
            btnLoad = CreateButton("加载数据", 20, 420, Color.SteelBlue);
            btnLoad.Click += BtnLoad_Click;

            btnAdd = CreateButton("新增用户", 130, 420, Color.Green);
            btnAdd.Click += BtnAdd_Click;

            btnEdit = CreateButton("编辑用户", 240, 420, Color.Orange);
            btnEdit.Click += BtnEdit_Click;

            btnDelete = CreateButton("删除用户", 350, 420, Color.Red);
            btnDelete.Click += BtnDelete_Click;

            // 添加控件
            this.Controls.AddRange(new Control[] { label, dataGridView, btnLoad, btnAdd, btnEdit, btnDelete });
        }

        private Button CreateButton(string text, int x, int y, Color backColor)
        {
            return new Button
            {
                Text = text,
                Font = new Font("微软雅黑", 10),
                Size = new Size(100, 35),
                Location = new Point(x, y),
                BackColor = backColor,
                ForeColor = Color.White
            };
        }

        // 加载数据
        private void LoadData()
        {
            try
            {
                using (var conn = new MySqlConnection(ConnectionString))
                {
                    conn.Open();
                    var adapter = new MySqlDataAdapter("SELECT id, name, email, age FROM users", conn);
                    var dt = new DataTable();
                    adapter.Fill(dt);
                    dataGridView.DataSource = dt;
                    FormatDataGridColumns();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"数据库操作失败: {ex.Message}");
            }
        }

        private void FormatDataGridColumns()
        {
            dataGridView.Columns["id"].HeaderText = "ID";
            dataGridView.Columns["name"].HeaderText = "姓名";
            dataGridView.Columns["email"].HeaderText = "邮箱";
            dataGridView.Columns["age"].HeaderText = "年龄";

            dataGridView.ColumnHeadersDefaultCellStyle.BackColor = Color.LightSteelBlue;
            dataGridView.ColumnHeadersDefaultCellStyle.Font = new Font("微软雅黑", 10, FontStyle.Bold);
            dataGridView.EnableHeadersVisualStyles = false;
        }

        // 新增用户
        private void BtnAdd_Click(object sender, EventArgs e)
        {
            using (var dialog = new Form())
            {
                dialog.Text = "新增用户";
                dialog.Size = new Size(300, 200);
                dialog.StartPosition = FormStartPosition.CenterParent;

                var lblName = new Label { Text = "姓名：", Location = new Point(20, 20), AutoSize = true };
                var txtName = new TextBox { Location = new Point(80, 20), Width = 180 };

                var lblEmail = new Label { Text = "邮箱：", Location = new Point(20, 60), AutoSize = true };
                var txtEmail = new TextBox { Location = new Point(80, 60), Width = 180 };

                var lblAge = new Label { Text = "年龄：", Location = new Point(20, 100), AutoSize = true };
                var txtAge = new TextBox { Location = new Point(80, 100), Width = 180 };

                var btnOk = new Button { Text = "确定", Location = new Point(80, 140), DialogResult = DialogResult.OK };
                var btnCancel = new Button { Text = "取消", Location = new Point(160, 140), DialogResult = DialogResult.Cancel };

                dialog.Controls.AddRange(new Control[] { lblName, txtName, lblEmail, txtEmail, lblAge, txtAge, btnOk, btnCancel });
                dialog.AcceptButton = btnOk;
                dialog.CancelButton = btnCancel;

                if (dialog.ShowDialog(this) == DialogResult.OK)
                {
                    if (string.IsNullOrWhiteSpace(txtName.Text) || 
                        string.IsNullOrWhiteSpace(txtEmail.Text) || 
                        !int.TryParse(txtAge.Text, out int age))
                    {
                        MessageBox.Show("请输入有效的姓名、邮箱和年龄！", "输入错误", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                        return;
                    }

                    try
                    {
                        const string sql = @"INSERT INTO users (name, email, age) 
                                           VALUES (@name, @email, @age);
                                           SELECT LAST_INSERT_ID();";

                        using (var conn = new MySqlConnection(ConnectionString))
                        {
                            conn.Open();
                            using (var cmd = new MySqlCommand(sql, conn))
                            {
                                cmd.Parameters.AddWithValue("@name", txtName.Text.Trim());
                                cmd.Parameters.AddWithValue("@email", txtEmail.Text.Trim());
                                cmd.Parameters.AddWithValue("@age", age);
                                
                                var newId = cmd.ExecuteScalar();
                                MessageBox.Show($"用户创建成功！ID: {newId}", "成功", 
                                    MessageBoxButtons.OK, MessageBoxIcon.Information);
                                LoadData();
                            }
                        }
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show($"数据库操作失败: {ex.Message}", "错误", 
                            MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
        }

        // 编辑用户
        private void BtnEdit_Click(object sender, EventArgs e)
        {
            if (dataGridView.SelectedRows.Count == 0)
            {
                MessageBox.Show("请先选择要编辑的用户！", "提示", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                return;
            }

            DataGridViewRow selectedRow = dataGridView.SelectedRows[0];
            using (var dialog = new Form())
            {
                dialog.Text = "编辑用户";
                dialog.Size = new Size(300, 250);
                dialog.StartPosition = FormStartPosition.CenterParent;

                var lblId = new Label { Text = "ID：", Location = new Point(20, 20), AutoSize = true };
                var txtId = new TextBox { 
                    Text = selectedRow.Cells["id"].Value.ToString(),
                    ReadOnly = true,
                    Location = new Point(80, 20), 
                    Width = 180 
                };

                var lblName = new Label { Text = "姓名：", Location = new Point(20, 60), AutoSize = true };
                var txtName = new TextBox { 
                    Text = selectedRow.Cells["name"].Value.ToString(),
                    Location = new Point(80, 60), 
                    Width = 180 
                };

                var lblEmail = new Label { Text = "邮箱：", Location = new Point(20, 100), AutoSize = true };
                var txtEmail = new TextBox { 
                    Text = selectedRow.Cells["email"].Value.ToString(),
                    Location = new Point(80, 100), 
                    Width = 180 
                };

                var lblAge = new Label { Text = "年龄：", Location = new Point(20, 140), AutoSize = true };
                var txtAge = new TextBox { 
                    Text = selectedRow.Cells["age"].Value.ToString(),
                    Location = new Point(80, 140), 
                    Width = 180 
                };

                var btnOk = new Button { Text = "确定", Location = new Point(80, 180), DialogResult = DialogResult.OK };
                var btnCancel = new Button { Text = "取消", Location = new Point(160, 180), DialogResult = DialogResult.Cancel };

                dialog.Controls.AddRange(new Control[] { lblId, txtId, lblName, txtName, lblEmail, txtEmail, lblAge, txtAge, btnOk, btnCancel });
                dialog.AcceptButton = btnOk;
                dialog.CancelButton = btnCancel;

                if (dialog.ShowDialog(this) == DialogResult.OK)
                {
                    if (!int.TryParse(txtId.Text, out int userId))
                    {
                        MessageBox.Show("无效的用户ID！", "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
                        return;
                    }

                    try
                    {
                        const string sql = @"UPDATE users 
                                           SET name = @name, 
                                               email = @email, 
                                               age = @age 
                                           WHERE id = @id";

                        using (var conn = new MySqlConnection(ConnectionString))
                        {
                            conn.Open();
                            using (var cmd = new MySqlCommand(sql, conn))
                            {
                                cmd.Parameters.AddWithValue("@id", userId);
                                cmd.Parameters.AddWithValue("@name", txtName.Text.Trim());
                                cmd.Parameters.AddWithValue("@email", txtEmail.Text.Trim());
                                cmd.Parameters.AddWithValue("@age", int.Parse(txtAge.Text));
                                
                                int affected = cmd.ExecuteNonQuery();
                                if (affected > 0)
                                {
                                    MessageBox.Show("用户更新成功！", "成功");
                                    LoadData();
                                }
                            }
                        }
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show($"数据库操作失败: {ex.Message}", "错误");
                    }
                }
            }
        }

        // 删除用户
        private void BtnDelete_Click(object sender, EventArgs e)
        {
            if (dataGridView.SelectedRows.Count == 0)
            {
                MessageBox.Show("请先选择要删除的用户！", "提示", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                return;
            }

            DataGridViewRow selectedRow = dataGridView.SelectedRows[0];
            int userId = Convert.ToInt32(selectedRow.Cells["id"].Value);

            var confirm = MessageBox.Show(
                $"确定要删除用户 [{selectedRow.Cells["name"].Value}] (ID: {userId}) 吗？",
                "确认删除",
                MessageBoxButtons.YesNo,
                MessageBoxIcon.Question
            );

            if (confirm == DialogResult.Yes)
            {
                try
                {
                    const string sql = "DELETE FROM users WHERE id = @id";
                    
                    using (var conn = new MySqlConnection(ConnectionString))
                    {
                        conn.Open();
                        using (var cmd = new MySqlCommand(sql, conn))
                        {
                            cmd.Parameters.AddWithValue("@id", userId);
                            
                            int affected = cmd.ExecuteNonQuery();
                            if (affected > 0)
                            {
                                MessageBox.Show("用户删除成功！", "成功");
                                LoadData();
                            }
                        }
                    }
                }
                catch (Exception ex)
                {
                    MessageBox.Show($"数据库操作失败: {ex.Message}", "错误");
                }
            }
        }

        private void BtnLoad_Click(object sender, EventArgs e)
        {
            LoadData();
        }
}
```

end