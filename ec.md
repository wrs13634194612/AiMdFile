说明：
我计划用python写一个程序  

1.将下面文件中的汉字转拼音，C:\Users\wangrusheng\Desktop\new.txt
2.并生成新文件 C:\Users\wangrusheng\Desktop\new_pinyin.txt
3.用对比工具代码 ，查看新旧文件的差异
4.仔细核对，发现中文转拼音成功
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/564baaa5ec884fafb86902eeaad035ab.png#pic_center)

代码实现：

```python
import re
from pypinyin import lazy_pinyin, Style

def convert_to_pinyin(text):
    """将文本中的汉字转换为拼音（不带声调），非汉字保留原样"""
    parts = re.findall(r'[\u4e00-\u9fff]+|[^\u4e00-\u9fff]+', text)
    result = []
    for part in parts:
        if re.fullmatch(r'[\u4e00-\u9fff]+', part):
            pinyin = ''.join(lazy_pinyin(part, style=Style.NORMAL))
            result.append(pinyin)
        else:
            result.append(part)
    return ''.join(result)

# 文件路径配置
input_file = r'C:\Users\wangrusheng\Desktop\new.txt'
output_file = r'C:\Users\wangrusheng\Desktop\new_pinyin.txt'

# 读取文件（自动尝试UTF-8和GBK编码）
try:
    with open(input_file, 'r', encoding='utf-8') as f:
        content = f.read()
except UnicodeDecodeError:
    try:
        with open(input_file, 'r', encoding='gbk') as f:
            content = f.read()
    except Exception as e:
        print(f"错误：文件解码失败 ({e})")
        exit(1)

# 转换拼音
pinyin_content = convert_to_pinyin(content)

# 写入新文件
try:
    with open(output_file, 'w', encoding='utf-8') as f:
        f.write(pinyin_content)
    print(f"转换完成！生成文件：{output_file}")
except Exception as e:
    print(f"错误：文件写入失败 ({e})")
    exit(1)
```

end