python将ico图片 转成png  生成新文件

D:\project\nextron-main\nextron-main\examples\_template\js\resources\icon.ico
step：

```python
from PIL import Image
import os

# 原始ICO文件路径
ico_path = r'D:\project\nextron-main\nextron-main\examples\_template\js\resources\icon.ico'
# 生成PNG的路径（与原文件同目录，扩展名改为.png）
png_path = os.path.splitext(ico_path)[0] + '.png'

try:
    with Image.open(ico_path) as img:
        # 检查是否为多帧ICO（包含多个尺寸）
        if hasattr(img, 'n_frames'):
            max_size = (0, 0)
            max_index = 0
            # 遍历所有帧，找到最大尺寸的图片
            for i in range(img.n_frames):
                img.seek(i)
                current_size = img.size
                if current_size[0] * current_size[1] > max_size[0] * max_size[1]:
                    max_size = current_size
                    max_index = i
            img.seek(max_index)  # 定位到最大尺寸的帧
        # 保存为PNG
        img.save(png_path, 'PNG')
        print(f'成功生成PNG文件：{png_path}')
except Exception as e:
    print(f'转换失败：{e}')
```

end