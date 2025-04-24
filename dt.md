说明：
python统计本地文件字符数目和文件大小
step1:

```python
def count_characters(file_path):
    try:
        with open(file_path, 'r', encoding='utf-8') as file:
            content = file.read()
            return len(content)
    except FileNotFoundError:
        print(f"错误：文件 {file_path} 未找到")
        return -1
    except UnicodeDecodeError:
        try:
            with open(file_path, 'r', encoding='gbk') as file:
                content = file.read()
                return len(content)
        except Exception as e:
            print(f"解码失败: {str(e)}")
            return -1


if __name__ == "__main__":
    file_path = r"C:\Users\wangrusheng\Documents\old.txt"
    count = count_characters(file_path)

    if count >= 0:
        print(f"文件字符总数: {count}")
        print("\n统计详情：")
        print(f"• 总字符数（含空格）: {count}")

        # 统计不含空格的数量
        with open(file_path, 'r', encoding='utf-8') as file:
            content = file.read()
            no_spaces = len(content.replace(' ', '').replace('\t', ''))
            print(f"• 不含空格和制表符: {no_spaces}")

            # 统计换行符数量
            newlines = content.count('\n')
            print(f"• 换行符数量: {newlines}")

            # 统计中文字符（近似统计）
            chinese_chars = sum(1 for c in content if '\u4e00' <= c <= '\u9fff')
            print(f"• 中文字符数量: {chinese_chars}")

        # 文件大小统计
        import os

        file_size = os.path.getsize(file_path)
        print(f"\n文件大小统计:")
        print(f"• 字节大小: {file_size} bytes")
        print(f"• 约为: {file_size / 1024:.2f} KB")
    else:
        print("统计失败，请检查文件路径和编码")
```

step2:

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject> python hello.py
文件字符总数: 3668

统计详情：
• 总字符数（含空格）: 3668
• 不含空格和制表符: 3597
• 换行符数量: 67
• 中文字符数量: 0

文件大小统计:
• 字节大小: 3779 bytes
• 约为: 3.69 KB
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject> python hello.py
文件字符总数: 34739

统计详情：
• 总字符数（含空格）: 34739
• 不含空格和制表符: 26693
• 换行符数量: 1229
• 中文字符数量: 2147

文件大小统计:
• 字节大小: 40524 bytes
• 约为: 39.57 KB
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject> 

```

end