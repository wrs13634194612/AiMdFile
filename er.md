说明：
我计划用python写一个程序，将file.webp文件转换成png
step1:安装依赖

```bash
pip install Pillow
```

step2:

```python
from PIL import Image
import os

def convert_webp_to_png(input_path):
    try:
        # 打开WebP文件
        img = Image.open(input_path)

        # 生成输出路径（替换后缀为.png）
        output_path = os.path.splitext(input_path)[0] + ".png"

        # 转换为PNG并保存（保留透明通道）
        img.save(output_path, "PNG", optimize=True)
        return f"转换成功！保存路径：{output_path}"
    except Exception as e:
        return f"转换失败：{str(e)}"

# 启动应用
if __name__ == "__main__":
    # 原始文件路径（注意Windows路径需使用双反斜杠或原始字符串）
    input_path = r"C:\Users\Administrator\Desktop\file.webp"  # 推荐原始字符串写法
    # 执行转换
    result = convert_webp_to_png(input_path)
    print(result)
```

step3:

```bash
C:\Users\Administrator\PycharmProjects\FastAPIProject\replace_img.py 
转换成功！保存路径：C:\Users\Administrator\Desktop\file.png
```

end