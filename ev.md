说明
我计划用python对图片压缩
step1:

```bash
(.venv) PS C:\Users\Administrator\PycharmProjects\PythonProject2> pip install pillow
Collecting pillow
  Downloading pillow-11.1.0-cp311-cp311-win_amd64.whl.metadata (9.3 kB)
Downloading pillow-11.1.0-cp311-cp311-win_amd64.whl (2.6 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.6/2.6 MB 127.7 kB/s eta 0:00:00
Installing collected packages: pillow
Successfully installed pillow-11.1.0

```

step2:C:\Users\Administrator\PycharmProjects\PythonProject2\.venv\Scripts\activate_this.py

```python
import os
from PIL import Image
from pathlib import Path


def compress_image(input_path, output_folder, quality=85, optimize=True):
    """
    智能图片压缩函数

    参数:
    input_path (str): 输入文件路径
    output_folder (str): 输出目录
    quality (int): 压缩质量 (1-100)
    optimize (bool): 是否启用优化模式
    """
    try:
        # 验证输入文件
        if not os.path.exists(input_path):
            raise FileNotFoundError(f"输入文件不存在: {input_path}")

        # 创建输出目录
        Path(output_folder).mkdir(parents=True, exist_ok=True)

        # 生成输出文件名
        filename = Path(input_path).stem
        ext = Path(input_path).suffix.lower()
        output_path = os.path.join(output_folder, f"{filename}_compressed{ext}")

        # 打开并处理图片
        with Image.open(input_path) as img:
            # 保持原始格式
            file_format = img.format

            # 自动旋转校正（针对手机照片）
            if hasattr(img, '_getexif'):
                exif = img._getexif()
                if exif:
                    orientation = exif.get(0x0112)
                    if orientation in [3, 6, 8]:
                        angle = {
                            3: 180,
                            6: 270,
                            8: 90
                        }[orientation]
                        img = img.rotate(angle, expand=True)

            # 保存优化后的图片
            save_kwargs = {
                'format': file_format,
                'quality': quality,
                'optimize': optimize
            }

            # 格式特定参数
            if file_format == 'JPEG':
                save_kwargs['progressive'] = True
                save_kwargs['subsampling'] = 'keep'

            img.save(output_path, **save_kwargs)

        # 显示压缩结果
        orig_size = os.path.getsize(input_path)
        new_size = os.path.getsize(output_path)
        ratio = (1 - new_size / orig_size) * 100

        print(f"压缩完成: {Path(input_path).name}")
        print(f"原始大小: {orig_size / 1024:.1f} KB")
        print(f"压缩大小: {new_size / 1024:.1f} KB")
        print(f"压缩率: {ratio:.1f}%")

        return output_path

    except Exception as e:
        print(f"错误: {str(e)}")
        return None


if __name__ == "__main__":
    # 配置路径
    input_image = r"C:\Users\Administrator\Desktop\file.jpg"
    output_dir = r"C:\Users\Administrator\Desktop"

    # 执行压缩（质量可调整，建议80-90）
    compress_image(input_image, output_dir, quality=10)
```

step3:

```bash
C:\Users\Administrator\PycharmProjects\PythonProject2\.venv\Scripts\python.exe C:\Users\Administrator\PycharmProjects\PythonProject2\.venv\Scripts\activate_this.py 
压缩完成: file.jpg
原始大小: 3623.9 KB
压缩大小: 276.1 KB
压缩率: 92.4%

Process finished with exit code 0
```

end