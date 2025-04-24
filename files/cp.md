说明：
**forms+fastapi实现mysql表增删改查**
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9f67f4ca3a6a489db5117850800e93ce.png#pic_center)

step1: formsC:\Users\wangrusheng\RiderProjects\WinFormsApp1\WinFormsApp1\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Net.Http;
using System.Threading.Tasks;
using System.Windows.Forms;
using Newtonsoft.Json;

namespace WinFormsApp1
{
    public partial class Form1 : Form
    {
        private DataGridView dataGridView;
        private Button btnLoad;
        private Button btnAdd;
        private Button btnEdit;
        private Button btnDelete;

        public Form1()
        {
            InitializeComponent();
            CreateControls();
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
        private async void BtnLoad_Click(object sender, EventArgs e)
        {
            btnLoad.Enabled = false;
            btnLoad.Text = "加载中...";
            await LoadDataAsync();
            btnLoad.Text = "加载数据";
            btnLoad.Enabled = true;
        }

        private async Task LoadDataAsync()
        {
            try
            {
                using (var client = new HttpClient())
                {
                    client.Timeout = TimeSpan.FromSeconds(10);
                    var response = await client.GetAsync("http://localhost:8000/query?table_name=users");

                    if (!response.IsSuccessStatusCode)
                    {
                        MessageBox.Show($"请求失败: {response.StatusCode}", "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
                        return;
                    }

                    var json = await response.Content.ReadAsStringAsync();
                    var result = JsonConvert.DeserializeObject<ApiResponse>(json);

                    if (result?.Status != "success")
                    {
                        MessageBox.Show("接口返回错误状态", "警告", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                        return;
                    }

                    dataGridView.DataSource = result.Data;
                    FormatDataGridColumns();
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"发生错误: {ex.Message}", "错误", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        private void FormatDataGridColumns()
        {
            dataGridView.Columns["Id"].HeaderText = "ID";
            dataGridView.Columns["Name"].HeaderText = "姓名";
            dataGridView.Columns["Email"].HeaderText = "邮箱";
            dataGridView.Columns["Age"].HeaderText = "年龄";

            dataGridView.ColumnHeadersDefaultCellStyle.BackColor = Color.LightSteelBlue;
            dataGridView.ColumnHeadersDefaultCellStyle.Font = new Font("微软雅黑", 10, FontStyle.Bold);
            dataGridView.EnableHeadersVisualStyles = false;
        }

        // 新增用户
        private async void BtnAdd_Click(object sender, EventArgs e)
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

                    var newUser = new User
                    {
                        Name = txtName.Text.Trim(),
                        Email = txtEmail.Text.Trim(),
                        Age = age
                    };

                    await CreateUserAsync(newUser);
                    await LoadDataAsync();
                }
            }
        }

        private async Task CreateUserAsync(User user)
        {
            try
            {
                using (var client = new HttpClient())
                {
                    var content = new StringContent(
                        JsonConvert.SerializeObject(new { 
                            name = user.Name, 
                            email = user.Email, 
                            age = user.Age 
                        }),
                        System.Text.Encoding.UTF8, 
                        "application/json"
                    );

                    var response = await client.PostAsync("http://localhost:8000/users", content);
                    var responseJson = await response.Content.ReadAsStringAsync();
                    var result = JsonConvert.DeserializeObject<ApiResponse>(responseJson);

                    if (result?.Status == "success")
                    {
                        MessageBox.Show($"用户创建成功！ID: {result.Data}", "成功", 
                            MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    else
                    {
                        MessageBox.Show($"创建失败: {result?.Message ?? "未知错误"}", "错误", 
                            MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"请求异常: {ex.Message}", "错误", 
                    MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        // 编辑用户（修改后）
        private async void BtnEdit_Click(object sender, EventArgs e)
        {
            using (var dialog = new Form())
            {
                dialog.Text = "编辑用户";
                dialog.Size = new Size(300, 250);
                dialog.StartPosition = FormStartPosition.CenterParent;

                var lblId = new Label { Text = "ID：", Location = new Point(20, 20), AutoSize = true };
                var txtId = new TextBox { Location = new Point(80, 20), Width = 180 };

                var lblName = new Label { Text = "姓名：", Location = new Point(20, 60), AutoSize = true };
                var txtName = new TextBox { Location = new Point(80, 60), Width = 180 };

                var lblEmail = new Label { Text = "邮箱：", Location = new Point(20, 100), AutoSize = true };
                var txtEmail = new TextBox { Location = new Point(80, 100), Width = 180 };

                var lblAge = new Label { Text = "年龄：", Location = new Point(20, 140), AutoSize = true };
                var txtAge = new TextBox { Location = new Point(80, 140), Width = 180 };

                var btnOk = new Button { Text = "确定", Location = new Point(80, 180), DialogResult = DialogResult.OK };
                var btnCancel = new Button { Text = "取消", Location = new Point(160, 180), DialogResult = DialogResult.Cancel };

                dialog.Controls.AddRange(new Control[] { lblId, txtId, lblName, txtName, lblEmail, txtEmail, lblAge, txtAge, btnOk, btnCancel });
                dialog.AcceptButton = btnOk;
                dialog.CancelButton = btnCancel;

                if (dialog.ShowDialog(this) == DialogResult.OK)
                {
                    if (!int.TryParse(txtId.Text, out int userId))
                    {
                        MessageBox.Show("请输入有效的用户ID！", "输入错误", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                        return;
                    }

                    if (string.IsNullOrWhiteSpace(txtName.Text) || 
                        string.IsNullOrWhiteSpace(txtEmail.Text) || 
                        !int.TryParse(txtAge.Text, out int age))
                    {
                        MessageBox.Show("请输入有效的姓名、邮箱和年龄！", "输入错误", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                        return;
                    }

                    var updateData = new Dictionary<string, object>
                    {
                        { "name", txtName.Text.Trim() },
                        { "email", txtEmail.Text.Trim() },
                        { "age", age }
                    };

                    await UpdateUserAsync(userId, updateData);
                    await LoadDataAsync();
                }
            }
        }

        private async Task UpdateUserAsync(int userId, Dictionary<string, object> updateData)
        {
            try
            {
                using (var client = new HttpClient())
                {
                    var content = new StringContent(
                        JsonConvert.SerializeObject(updateData),
                        System.Text.Encoding.UTF8,
                        "application/json"
                    );

                    var response = await client.PutAsync($"http://localhost:8000/users/{userId}", content);
                    var responseJson = await response.Content.ReadAsStringAsync();
                    var result = JsonConvert.DeserializeObject<ApiResponse>(responseJson);

                    if (result?.Status == "success")
                    {
                        MessageBox.Show("用户更新成功！", "成功", 
                            MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    else
                    {
                        MessageBox.Show($"更新失败: {result?.Message ?? "未知错误"}", "错误", 
                            MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"请求异常: {ex.Message}", "错误", 
                    MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        // 删除用户（修改后）
        private async void BtnDelete_Click(object sender, EventArgs e)
        {
            using (var dialog = new Form())
            {
                dialog.Text = "删除用户";
                dialog.Size = new Size(300, 120);
                dialog.StartPosition = FormStartPosition.CenterParent;

                var lblId = new Label { Text = "ID：", Location = new Point(20, 20), AutoSize = true };
                var txtId = new TextBox { Location = new Point(80, 20), Width = 180 };

                var btnOk = new Button { Text = "确定", Location = new Point(80, 60), DialogResult = DialogResult.OK };
                var btnCancel = new Button { Text = "取消", Location = new Point(160, 60), DialogResult = DialogResult.Cancel };

                dialog.Controls.AddRange(new Control[] { lblId, txtId, btnOk, btnCancel });
                dialog.AcceptButton = btnOk;
                dialog.CancelButton = btnCancel;

                if (dialog.ShowDialog(this) == DialogResult.OK)
                {
                    if (!int.TryParse(txtId.Text, out int userId))
                    {
                        MessageBox.Show("请输入有效的用户ID！", "输入错误", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                        return;
                    }

                    var confirm = MessageBox.Show($"确定要删除用户ID为{userId}吗？", "确认删除", 
                        MessageBoxButtons.YesNo, MessageBoxIcon.Question);

                    if (confirm == DialogResult.Yes)
                    {
                        await DeleteUserAsync(userId);
                        await LoadDataAsync();
                    }
                }
            }
        }

        private async Task DeleteUserAsync(int userId)
        {
            try
            {
                using (var client = new HttpClient())
                {
                    var response = await client.DeleteAsync($"http://localhost:8000/users/{userId}");
                    var responseJson = await response.Content.ReadAsStringAsync();
                    var result = JsonConvert.DeserializeObject<ApiResponse>(responseJson);

                    if (result?.Status == "success")
                    {
                        MessageBox.Show("用户删除成功！", "成功", 
                            MessageBoxButtons.OK, MessageBoxIcon.Information);
                    }
                    else
                    {
                        MessageBox.Show($"删除失败: {result?.Message ?? "未知错误"}", "错误", 
                            MessageBoxButtons.OK, MessageBoxIcon.Error);
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"请求异常: {ex.Message}", "错误", 
                    MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }

    public class ApiResponse
    {
        public string Status { get; set; }
        public dynamic Data { get; set; }
        public string Message { get; set; }
    }

    public class User
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Email { get; set; }
        public int Age { get; set; }
    }
}
```

step2:sql

```sql
CREATE TABLE users (
    id INT NOT NULL AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    age INT NOT NULL,
    PRIMARY KEY (id),
    UNIQUE KEY email_unique (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

step3:fastapi C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\main.py

```python
from fastapi import FastAPI, HTTPException, Body, Path
import pymysql.cursors
from typing import Optional, Dict
app = FastAPI()
# 数据库连接配置
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}
# 查询数据库的函数
def query_database(query: str, params=None):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
# 查询表数据的 API 端点
@app.get("/query")
async def query_table(table_name: str):
    query = f"SELECT * FROM {table_name}"
    try:
        data = query_database(query)
        return {"status": "success", "data": data}
    except HTTPException as e:
        return {"status": "error", "message": e.detail}

# 新增用户 (POST)
@app.post("/users")
async def create_user(
    user_data: Dict = Body(..., example={
        "name": "诸葛亮",
        "email": "zhugeliang@shu.com",
        "age": 54
    })
):
    try:
        # 检查邮箱唯一性
        check_query = "SELECT id FROM users WHERE email = %s"
        exist = query_database(check_query, (user_data["email"],))
        if exist:
            raise HTTPException(409, "邮箱已存在")

        # 执行插入操作
        insert_query = """
        INSERT INTO users (name, email, age)
        VALUES (%s, %s, %s)
        """
        with pymysql.connect(**DB_CONFIG) as conn:
            with conn.cursor() as cursor:
                cursor.execute(insert_query, (
                    user_data["name"],
                    user_data["email"],
                    user_data["age"]
                ))
                new_id = cursor.lastrowid
                conn.commit()
        return {"status": "success", "id": new_id}
    except KeyError as e:
        raise HTTPException(400, f"缺失必要字段: {e}")


# 更新用户 (PUT)
@app.put("/users/{user_id}")
async def update_user(
        user_id: int = Path(..., gt=0),
        update_data: Dict = Body(..., example={
            "name": "更新名称",
            "age": 99
        })
):
    try:
        # 检查用户是否存在
        exist = query_database("SELECT id FROM users WHERE id = %s", (user_id,))
        if not exist:
            raise HTTPException(404, "用户不存在")

        # 动态生成更新语句
        set_clause = ", ".join([f"{k}=%s" for k in update_data.keys()])
        update_query = f"UPDATE users SET {set_clause} WHERE id = %s"

        with pymysql.connect(**DB_CONFIG) as conn:
            with conn.cursor() as cursor:
                cursor.execute(update_query, (
                    *update_data.values(),
                    user_id
                ))
                conn.commit()
        return {"status": "success", "affected_rows": cursor.rowcount}
    except pymysql.err.IntegrityError:
        raise HTTPException(409, "邮箱冲突或数据约束失败")
# 删除用户 (DELETE)
@app.delete("/users/{user_id}")
async def delete_user(user_id: int = Path(..., gt=0)):
    try:
        with pymysql.connect(**DB_CONFIG) as conn:
            with conn.cursor() as cursor:
                cursor.execute(
                    "DELETE FROM users WHERE id = %s",
                    (user_id,)
                )
                conn.commit()
                if cursor.rowcount == 0:
                    raise HTTPException(404, "用户不存在")
        return {"status": "success", "deleted_id": user_id}
    except pymysql.err.Error as e:
        raise HTTPException(500, f"数据库错误: {e}")
# 启动应用
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)

```

end