说明：
我希望用form实现笔记本新建保存加密功能
笔记管理应用，具备创建、保存、删除笔记的功能，并且有简单的加密保护。


```bash

1.笔记管理：
	1.1新建笔记：清除标题和内容，取消列表选择。
	1.2保存笔记：验证标题，处理加密，保存到指定目录。
	1.3删除笔记：需要密码验证（如果是私密笔记），删除文件后刷新列表。

2.加密功能：
	2.1使用凯撒密码（位移3）对内容和密码进行加密。
	2.2私密笔记需要输入密码，保存时加密密码，并在访问时验证。

3.文件操作：
	3.1自动创建notes目录，位于应用程序启动路径下。
	3.2加载笔记时，遍历目录下的.txt文件，显示在列表框中。

4.用户界面：
	4.1使用SplitContainer分割左右面板，左侧显示笔记列表，右侧编辑区和操作按钮。
	4.2选择笔记时，自动加载内容，处理加密验证。

5.文件存储路径：
	5.1{应用程序启动目录}\notes\
	     示例：C:\YourApp\bin\Debug\notes\My_Note.txt
```

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/ae9d57ef63f2451fbc97ded358bfded1.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp21\WinFormsApp21\Form1.cs

```csharp
using Microsoft.VisualBasic;
namespace WinFormsApp21;

public partial class Form1 : Form
{
    
    private readonly string notesDirectory = Path.Combine(Application.StartupPath, "notes");
    private ListBox lstNotes;
    private TextBox txtTitle;
    private TextBox txtContent;
    private Button btnNew;
    private Button btnSave;
    private Button btnDelete;
    private SplitContainer splitContainer1;
    public Form1()
    {
        InitializeComponent();
        this.splitContainer1 = new SplitContainer();
        this.lstNotes = new ListBox();
        this.txtTitle = new TextBox();
        this.txtContent = new TextBox();
        this.btnNew = new Button();
        this.btnSave = new Button();
        this.btnDelete = new Button();
        
        // SplitContainer配置
        splitContainer1.Dock = DockStyle.Fill;
        splitContainer1.SplitterDistance = 200;
        
        // 左侧面板（列表）
        splitContainer1.Panel1.Controls.Add(lstNotes);
        
        // 右侧面板（编辑器）
        splitContainer1.Panel2.Controls.Add(btnDelete);
        splitContainer1.Panel2.Controls.Add(btnSave);
        splitContainer1.Panel2.Controls.Add(btnNew);
        splitContainer1.Panel2.Controls.Add(txtContent);
        splitContainer1.Panel2.Controls.Add(txtTitle);
        
        // 列表配置
        lstNotes.Dock = DockStyle.Fill;
        lstNotes.SelectedIndexChanged += lstNotes_SelectedIndexChanged;
        
        // 文本框配置
        txtTitle.Location = new System.Drawing.Point(20, 20);
        txtTitle.Size = new System.Drawing.Size(300, 23);
        
        txtContent.Multiline = true;
        txtContent.Location = new System.Drawing.Point(20, 60);
        txtContent.Size = new System.Drawing.Size(300, 300);
        
        // 按钮配置
        ConfigureButton(btnNew, 20, "New", btnNew_Click);
        ConfigureButton(btnSave, 100, "Save", btnSave_Click);
        ConfigureButton(btnDelete, 180, "Delete", btnDelete_Click);
        
        // 窗体配置
        ClientSize = new System.Drawing.Size(800, 450);
        Controls.Add(splitContainer1);
        Text = "Notes Application";
        Directory.CreateDirectory(notesDirectory);
        LoadNotes();
    }
    
    
    private void ConfigureButton(Button btn, int x, string text, EventHandler handler)
    {
        btn.Location = new System.Drawing.Point(x, 370);
        btn.Text = text;
        btn.Click += handler;
        btn.Size = new System.Drawing.Size(75, 23);
    }

    
    
        private void LoadNotes()
        {
            lstNotes.Items.Clear();
            foreach (var file in Directory.GetFiles(notesDirectory, "*.txt"))
            {
                lstNotes.Items.Add(Path.GetFileName(file));
            }
        }

        private string SanitizeFileName(string name)
        {
            return name.Replace(" ", "_");
        }

        private void lstNotes_SelectedIndexChanged(object sender, EventArgs e)
        {
            if (lstNotes.SelectedItem == null) return;

            var fileName = lstNotes.SelectedItem.ToString();
            var filePath = Path.Combine(notesDirectory, fileName);

            try
            {
                var lines = File.ReadAllLines(filePath);
                if (lines.Length < 1)
                {
                    MessageBox.Show("Invalid note format");
                    return;
                }

                var header = lines[0];
                var encryptedContent = string.Join(Environment.NewLine, lines.Skip(1));

                if (header.StartsWith("PRIVATE:"))
                {
                    var encryptedPassword = header.Substring(8);
                    string password = Interaction.InputBox("This note is private. Enter password:", "Password Required");
                    
                    if (string.IsNullOrEmpty(password)) return;
                    
                    if (EncryptText(password) != encryptedPassword)
                    {
                        MessageBox.Show("Incorrect password");
                        return;
                    }
                }

                txtTitle.Text = Path.GetFileNameWithoutExtension(fileName).Replace("_", " ");
                txtContent.Text = DecryptText(encryptedContent);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error reading note: {ex.Message}");
            }
        }

        private void btnNew_Click(object sender, EventArgs e)
        {
            txtTitle.Clear();
            txtContent.Clear();
            lstNotes.ClearSelected();
        }

        private void btnSave_Click(object sender, EventArgs e)
        {
            if (string.IsNullOrWhiteSpace(txtTitle.Text))
            {
                MessageBox.Show("Please enter a title");
                return;
            }

            try
            {
                var sanitized = SanitizeFileName(txtTitle.Text) + ".txt";
                var filePath = Path.Combine(notesDirectory, sanitized);

                var isPrivate = MessageBox.Show("Make this note private?", "Security", 
                    MessageBoxButtons.YesNo) == DialogResult.Yes;

                string password = null;
                if (isPrivate)
                {
                    password = Interaction.InputBox("Enter password for this note:", "Set Password");
                    if (string.IsNullOrEmpty(password))
                    {
                        MessageBox.Show("Password required for private notes");
                        return;
                    }
                }

                var header = isPrivate ? $"PRIVATE:{EncryptText(password)}" : "PUBLIC";
                var content = header + Environment.NewLine + EncryptText(txtContent.Text);

                File.WriteAllText(filePath, content);
                LoadNotes();
                MessageBox.Show("Note saved successfully");
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error saving note: {ex.Message}");
            }
        }

        private void btnDelete_Click(object sender, EventArgs e)
        {
            if (lstNotes.SelectedItem == null) return;

            var fileName = lstNotes.SelectedItem.ToString();
            var filePath = Path.Combine(notesDirectory, fileName);

            try
            {
                var lines = File.ReadAllLines(filePath);
                if (lines.Length > 0 && lines[0].StartsWith("PRIVATE:"))
                {
                    var encryptedPassword = lines[0].Substring(8);
                    string password = Interaction.InputBox("Enter password to delete:", "Password Verification");
                    
                    if (EncryptText(password) != encryptedPassword)
                    {
                        MessageBox.Show("Incorrect password");
                        return;
                    }
                }

                File.Delete(filePath);
                LoadNotes();
                txtTitle.Clear();
                txtContent.Clear();
                MessageBox.Show("Note deleted successfully");
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Error deleting note: {ex.Message}");
            }
        }

        public static string EncryptText(string text, int shift = 3)
        {
            char[] result = text.ToCharArray();
            for (int i = 0; i < result.Length; i++)
            {
                char c = result[i];
                if (char.IsLetter(c))
                {
                    char baseChar = char.IsLower(c) ? 'a' : 'A';
                    c = (char)(((c - baseChar + shift) % 26 + baseChar));
                }
                result[i] = c;
            }
            return new string(result);
        }

        public static string DecryptText(string text, int shift = 3)
        {
            return EncryptText(text, 26 - shift);
        }

    
}
```

end