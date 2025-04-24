说明：
我计划用python，实现一个对比工具的功能，具体做法是：
1.桌面两个文件，old和new，
2.然后让程序解析这两个文件，
3.对比两个文件的差异，
4.然后生成新的html，file:///C:/Users/wangrusheng/Desktop/diff.html
5.打开html，可以清晰的看到新旧文件的差异和改动点 
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f45e8f8770704cc1884bbf81c3275d4f.png#pic_center)

代码实现：

```python
import difflib

# 配置文件路径
old_file = r'C:\Users\wangrusheng\Desktop\old.txt'
new_file = r'C:\Users\wangrusheng\Desktop\new.txt'
output_html = r'C:\Users\wangrusheng\Desktop\diff.html'

# 读取文件内容（处理可能的编码问题）
try:
    with open(old_file, 'r', encoding='utf-8') as f:
        old_lines = f.readlines()
    with open(new_file, 'r', encoding='utf-8') as f:
        new_lines = f.readlines()
except UnicodeDecodeError:
    # 如果utf-8解码失败，尝试使用系统默认编码
    with open(old_file, 'r') as f:
        old_lines = f.readlines()
    with open(new_file, 'r') as f:
        new_lines = f.readlines()

# 生成HTML差异对比
differ = difflib.HtmlDiff(tabsize=4, wrapcolumn=80)
html_content = differ.make_file(
    fromlines=old_lines,
    tolines=new_lines,
    fromdesc='旧文件 (old.txt)',
    todesc='新文件 (new.txt)',
    context=True,
    numlines=3
)

# 自定义CSS样式增强显示效果
custom_css = """
<style>
    table.diff {
        width: 100%%;
        border-collapse: collapse;
        font-family: Consolas, 'Courier New', monospace;
    }
    .diff_header {
        background-color: #e0e0e0;
        padding: 2px 5px;
    }
    td {
        vertical-align: top;
        padding: 2px 5px;
    }
    .diff_add, .diff_sub, .diff_chg {
        background-color: #ffcccc !important;
    }
    .diff_add { background-color: #ccffcc !important; }  /* 新增行保持绿色 */
    tr:hover td { background-color: #f0f0f0; }
    .diff_next {
        background-color: #c0c0c0;
    }
</style>
"""

# 插入自定义CSS并添加响应式设计
html_content = html_content.replace('</head>', custom_css + '</head>')
html_content = html_content.replace(
    '<body>',
    '<body>\n<div style="max-width: 1200px; margin: 20px auto; padding: 0 20px;">'
)
html_content = html_content.replace(
    '</body>',
    '</div>\n</body>'
)

# 写入HTML文件
with open(output_html, 'w', encoding='utf-8') as f:
    f.write(html_content)

print(f"差异报告已生成：{output_html}")
```

end