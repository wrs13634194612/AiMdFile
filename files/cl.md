说明：
forms分类列表，左边分类列表，右边小说列表
效果图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/23696fe48b054081b69cf111970cd5ad.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp2\WinFormsApp2\Form1.cs

```csharp
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Windows.Forms;

namespace WinFormsApp2;

public partial class Form1 : Form
{
    
    // 控件引用
    private SplitContainer mainSplitContainer;
    private FlowLayoutPanel categoryContainer;
    private FlowLayoutPanel novelContainer;

    // 数据相关
    private List<Category> categories = new List<Category>();
    private int currentCategoryId = -1;

    // UI组件
    private readonly Panel loadingPanel = new Panel();

    // 数据模型
    public class Chapter
    {
        public int chapter_id { get; set; }
        public int chapter_no { get; set; }
        public string title { get; set; } = string.Empty;
        public int word_count { get; set; }
        public DateTime created_at { get; set; }
    }

    public class Author
    {
        public int user_id { get; set; }
        public string username { get; set; } = string.Empty;
    }

    public class Novel
    {
        public int novel_id { get; set; }
        public string title { get; set; } = string.Empty;
        public string cover { get; set; } = string.Empty;
        public Author author { get; set; } = new Author();
        public string description { get; set; } = string.Empty;
        public string status { get; set; } = "serial";
        public int word_count { get; set; }
        public bool is_vip { get; set; }
        public DateTime created_at { get; set; }
        public DateTime? last_updated { get; set; }
        public List<Chapter> chapters { get; set; } = new List<Chapter>();
        public List<string> tags { get; set; } = new List<string>();
        public int bookshelf_count { get; set; }
    }

    public class Category
    {
        public int category_id { get; set; }
        public string name { get; set; } = string.Empty;
        public string description { get; set; } = string.Empty;
        public List<Novel> novels { get; set; } = new List<Novel>();
    }
    public Form1()
    {
        InitializeComponent();
        InitializeUI();
        LoadData();
    }
    
    private void InitializeUI()
        {
            // 窗体设置
            this.Size = new Size(1200, 800);
            this.Text = "小说阅读器";
            this.BackColor = Color.White;
            this.Font = new Font("微软雅黑", 9);
            this.Resize += (s, e) => UpdateLayout();

            // 初始化主布局
            CreateMainSplitContainer();
            CreateLoadingPanel();

            // 确保加载面板在最上层
            loadingPanel.BringToFront();
        }

        private void CreateMainSplitContainer()
        {
            mainSplitContainer = new SplitContainer
            {
                Dock = DockStyle.Fill,
                SplitterWidth = 1,
                Panel1 = { BackColor = Color.FromArgb(245, 245, 245) },
                Panel2 = { BackColor = Color.White }
            };

            // 初始化分类列表
            categoryContainer = new FlowLayoutPanel
            {
                Dock = DockStyle.Fill,
                FlowDirection = FlowDirection.TopDown,
                AutoScroll = true,
                Padding = new Padding(15),
                WrapContents = false
            };

            // 初始化小说列表
            novelContainer = new FlowLayoutPanel
            {
                Dock = DockStyle.Fill,
                FlowDirection = FlowDirection.TopDown,
                AutoScroll = true,
                Padding = new Padding(20),
                WrapContents = false
            };

            mainSplitContainer.Panel1.Controls.Add(categoryContainer);
            mainSplitContainer.Panel2.Controls.Add(novelContainer);

            this.Controls.Add(mainSplitContainer);
            UpdateLayout(); // 初始布局
        }

        private void UpdateLayout()
        {
            // 计算1:6的比例
            int splitterDistance = (int)(this.ClientSize.Width / 7.0);
            mainSplitContainer.SplitterDistance = splitterDistance;

            // 更新分类项宽度
            foreach (Control item in categoryContainer.Controls)
            {
                item.Width = categoryContainer.ClientSize.Width - categoryContainer.Padding.Horizontal;
            }

            // 更新小说卡片宽度
            foreach (Control card in novelContainer.Controls)
            {
                card.Width = novelContainer.ClientSize.Width - novelContainer.Padding.Horizontal;
                var picBox = card.Controls[0] as PictureBox;
                var contentPanel = card.Controls[1] as Panel;
                
                if (picBox != null && contentPanel != null)
                {
                    contentPanel.Width = card.Width - picBox.Width - 40;
                }
            }
        }

        private void CreateLoadingPanel()
        {
            loadingPanel.Dock = DockStyle.Fill;
            loadingPanel.BackColor = Color.White;
            loadingPanel.Visible = false;

            var loadingLabel = new Label
            {
                Text = "加载中...",
                Font = new Font("微软雅黑", 14),
                AutoSize = true,
                Location = new Point(20, 20)
            };

            loadingPanel.Controls.Add(loadingLabel);
            this.Controls.Add(loadingPanel);
        }

        private void LoadData()
        {
            loadingPanel.Visible = true;

            try
            {
                  // 固定JSON数据
                string json = @"[
                {
                    ""category_id"": 1,
                    ""name"": ""玄幻"",
                    ""description"": ""东方玄幻、异世大陆等幻想类作品"",
                    ""novels"": [
                        {
                            ""novel_id"": 1,
                            ""title"": ""斗破苍穹"",
                            ""cover"": ""https://randomuser.me/api/portraits/men/1.jpg"",
                            ""author"": {
                                ""user_id"": 2,
                                ""username"": ""author1""
                            },
                            ""description"": ""这里是属于斗气的世界，没有花俏艳丽的魔法..."",
                            ""status"": ""finished"",
                            ""word_count"": 5000000,
                            ""is_vip"": true,
                            ""created_at"": ""2025-03-29T15:27:59"",
                            ""last_updated"": null,
                            ""chapters"": [
                                {
                                    ""chapter_id"": 1,
                                    ""chapter_no"": 1,
                                    ""title"": ""陨落的天才"",
                                    ""word_count"": 3245,
                                    ""created_at"": ""2025-03-29T15:28:03""
                                },
                                {
                                    ""chapter_id"": 2,
                                    ""chapter_no"": 2,
                                    ""title"": ""斗气阁"",
                                    ""word_count"": 2987,
                                    ""created_at"": ""2025-03-29T15:28:03""
                                }
                            ],
                            ""tags"": [
                                ""废柴流"",
                                ""热血""
                            ],
                            ""bookshelf_count"": 1
                        },
                        {
                            ""novel_id"": 3,
                            ""title"": ""测试小说"",
                            ""cover"": ""https://randomuser.me/api/portraits/men/6.jpg"",
                            ""author"": {
                                ""user_id"": 2,
                                ""username"": ""author1""
                            },
                            ""description"": ""这是一个测试小说"",
                            ""status"": ""serial"",
                            ""word_count"": 5000,
                            ""is_vip"": true,
                            ""created_at"": ""2025-03-29T15:44:41"",
                            ""last_updated"": null,
                            ""chapters"": [],
                            ""tags"": [
                                ""热血"",
                                ""神医""
                            ],
                            ""bookshelf_count"": 0
                        }
                    ]
                },
                {
                    ""category_id"": 2,
                    ""name"": ""都市"",
                    ""description"": ""现代都市背景的言情、社会类作品"",
                    ""novels"": [
                        {
                            ""novel_id"": 2,
                            ""title"": ""都市神医"",
                            ""cover"": ""https://randomuser.me/api/portraits/men/5.jpg"",
                            ""author"": {
                                ""user_id"": 2,
                                ""username"": ""author1""
                            },
                            ""description"": ""落魄青年获得神秘传承，从此医武无双..."",
                            ""status"": ""serial"",
                            ""word_count"": 200000,
                            ""is_vip"": false,
                            ""created_at"": ""2025-03-29T15:27:59"",
                            ""last_updated"": null,
                            ""chapters"": [
                                {
                                    ""chapter_id"": 3,
                                    ""chapter_no"": 1,
                                    ""title"": ""医院奇遇"",
                                    ""word_count"": 4123,
                                    ""created_at"": ""2025-03-29T15:28:03""
                                },
                                {
                                    ""chapter_id"": 4,
                                    ""chapter_no"": 2,
                                    ""title"": ""神秘古玉"",
                                    ""word_count"": 3789,
                                    ""created_at"": ""2025-03-29T15:28:03""
                                }
                            ],
                            ""tags"": [
                                ""神医""
                            ],
                            ""bookshelf_count"": 1
                        }
                    ]
                }
            ]";

                categories = JsonConvert.DeserializeObject<List<Category>>(json) ?? new List<Category>();

                if (categories.Count > 0)
                {
                    currentCategoryId = categories[0].category_id;
                    UpdateCategoryList();
                    UpdateNovelList(categories[0].novels);
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show($"数据加载失败：{ex.Message}", "错误",
                    MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
            finally
            {
                loadingPanel.Visible = false;
            }
        }

        private void UpdateCategoryList()
        {
            categoryContainer.SuspendLayout();
            categoryContainer.Controls.Clear();

            foreach (var category in categories)
            {
                var isActive = category.category_id == currentCategoryId;
                var categoryItem = CreateCategoryItem(category, isActive);
                categoryContainer.Controls.Add(categoryItem);
            }

            categoryContainer.ResumeLayout();
        }

        private Panel CreateCategoryItem(Category category, bool isActive)
        {
            var panel = new Panel
            {
                Width = categoryContainer.ClientSize.Width - categoryContainer.Padding.Horizontal,
                Height = 80,
                Margin = new Padding(0, 0, 0, 15),
                BackColor = isActive ? Color.FromArgb(0, 123, 255) : Color.White,
                Cursor = Cursors.Hand,
                Tag = category.category_id
            };

            // 标题
            var lblTitle = new Label
            {
                Text = category.name,
                Font = new Font("微软雅黑", 11, FontStyle.Bold),
                ForeColor = isActive ? Color.White : Color.Black,
                Location = new Point(10, 10),
                AutoSize = true
            };

            // 描述
            var lblDesc = new Label
            {
                Text = category.description,
                Font = new Font("微软雅黑", 9),
                ForeColor = isActive ? Color.LightGray : Color.Gray,
                Location = new Point(10, 35),
                MaximumSize = new Size(panel.Width - 20, 40),
                AutoSize = true
            };

            // 事件处理
            panel.MouseEnter += (s, e) => UpdateCategoryItemHover(panel, isActive);
            panel.MouseLeave += (s, e) => UpdateCategoryItemLeave(panel, isActive);
            panel.Click += (s, e) => HandleCategoryClick(panel);

            panel.Controls.Add(lblTitle);
            panel.Controls.Add(lblDesc);
            return panel;
        }

        private void UpdateCategoryItemHover(Panel panel, bool isActive)
        {
            if (!isActive) panel.BackColor = Color.FromArgb(240, 240, 240);
        }

        private void UpdateCategoryItemLeave(Panel panel, bool isActive)
        {
            if (!isActive) panel.BackColor = Color.White;
        }

        private void HandleCategoryClick(Panel panel)
        {
            currentCategoryId = (int)panel.Tag;
            UpdateCategoryList();
            var selectedCategory = categories.Find(c => c.category_id == currentCategoryId);
            UpdateNovelList(selectedCategory?.novels ?? new List<Novel>());
        }

        private void UpdateNovelList(List<Novel> novels)
        {
            novelContainer.SuspendLayout();
            novelContainer.Controls.Clear();

            foreach (var novel in novels)
            {
                var novelCard = CreateNovelCard(novel);
                novelContainer.Controls.Add(novelCard);
            }

            novelContainer.ResumeLayout();
        }

        private Panel CreateNovelCard(Novel novel)
        {
            var card = new Panel
            {
                Width = novelContainer.ClientSize.Width - novelContainer.Padding.Horizontal,
                Height = 180,
                Margin = new Padding(0, 0, 0, 20),
                BackColor = Color.White,
                BorderStyle = BorderStyle.FixedSingle
            };

            // 封面图片
            var picCover = new PictureBox
            {
                Width = (int)(card.Height * 0.7),
                Height = (int)(card.Height * 0.8),
                Location = new Point(20, 20),
                SizeMode = PictureBoxSizeMode.Zoom,
                Image = LoadCoverImage(novel.cover)
            };

            // 内容区域
            var contentPanel = new Panel
            {
                Location = new Point(picCover.Right + 20, 20),
                Size = new Size(card.Width - picCover.Width - 40, card.Height - 40)
            };

            // 标题
            var lblTitle = new Label
            {
                Text = novel.title,
                Font = new Font("微软雅黑", 14, FontStyle.Bold),
                AutoSize = true,
                Location = new Point(0, 0)
            };

            // 元信息
            var metaPanel = CreateMetaPanel(novel);
            var descLabel = CreateDescriptionLabel(novel);
            var tagPanel = CreateTagPanel(novel);

            contentPanel.Controls.Add(lblTitle);
            contentPanel.Controls.Add(metaPanel);
            contentPanel.Controls.Add(descLabel);
            contentPanel.Controls.Add(tagPanel);

            card.Controls.Add(picCover);
            card.Controls.Add(contentPanel);
            return card;
        }

        private FlowLayoutPanel CreateMetaPanel(Novel novel)
        {
            var panel = new FlowLayoutPanel
            {
                FlowDirection = FlowDirection.LeftToRight,
                AutoSize = true,
                Location = new Point(0, 30)
            };

            panel.Controls.AddRange(new Control[] {
                CreateMetaLabel($"作者：{novel.author.username}"),
                CreateMetaLabel($"状态：{(novel.status == "finished" ? "完结" : "连载")}"),
                CreateMetaLabel($"字数：{(novel.word_count / 10000.0):F1}万字")
            });

            return panel;
        }

        private Label CreateDescriptionLabel(Novel novel)
        {
            return new Label
            {
                Text = novel.description,
                Font = new Font("微软雅黑", 10),
                ForeColor = Color.Gray,
                MaximumSize = new Size(600, 40),
                AutoSize = true,
                Location = new Point(0, 60)
            };
        }

        private FlowLayoutPanel CreateTagPanel(Novel novel)
        {
            var panel = new FlowLayoutPanel
            {
                FlowDirection = FlowDirection.LeftToRight,
                AutoSize = true,
                Location = new Point(0, 100)
            };

            foreach (var tag in novel.tags)
            {
                panel.Controls.Add(CreateTagLabel(tag));
            }
            return panel;
        }

        private Image LoadCoverImage(string path)
        {
            try
            {
                if (!string.IsNullOrEmpty(path) && File.Exists(path))
                {
                    return Image.FromFile(path);
                }
            }
            catch { }

            return GenerateDefaultCover();
        }

        private Bitmap GenerateDefaultCover()
        {
            var bmp = new Bitmap(100, 140);
            using (var g = Graphics.FromImage(bmp))
            {
                g.Clear(Color.LightGray);
                using (var font = new Font("微软雅黑", 12))
                {
                    var format = new StringFormat
                    {
                        Alignment = StringAlignment.Center,
                        LineAlignment = StringAlignment.Center
                    };
                    g.DrawString("封面", font, Brushes.White, new RectangleF(0, 0, 100, 140), format);
                }
            }
            return bmp;
        }

        private Label CreateMetaLabel(string text)
        {
            return new Label
            {
                Text = text,
                Font = new Font("微软雅黑", 10),
                ForeColor = Color.DimGray,
                Margin = new Padding(0, 0, 20, 0),
                AutoSize = true
            };
        }

        private Label CreateTagLabel(string text)
        {
            return new Label
            {
                Text = text,
                Font = new Font("微软雅黑", 9),
                ForeColor = Color.Gray,
                BackColor = Color.FromArgb(240, 240, 240),
                Padding = new Padding(5, 2, 5, 2),
                Margin = new Padding(0, 0, 5, 0),
                AutoSize = true
            };
        }
}
    
```

end