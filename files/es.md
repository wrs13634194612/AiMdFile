说明:
我计划写一个程序，识别图片中的文字，然后将识别的文字，在控制台打印出来，默认识别英文，中文的话，需要在安装tesseract，勾选并下载中文包

效果图：
1.英文识别
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b2ccde6da2ec4c53966df2fbafc86d11.png#pic_center)

2.中文识别
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bc13bbc43756433cbc7f7f0d828a1745.png#pic_center)

step1:

```bash
pip install pillow pytesseract

```

step2:C:\Users\Administrator\PycharmProjects\FastAPIProject\image.py

```python
from PIL import Image
import pytesseract
# 如果是Windows系统，且Tesseract未添加到环境变量，需手动指定路径（取消下面注释）
# pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'
def ocr_image(image_path):
    try:
        # 打开图片文件
        img = Image.open(image_path)
        # 使用Tesseract进行OCR识别
        text = pytesseract.image_to_string(img, lang='chi_sim+eng')  # 中英文混合识别
        return text.strip()
    except Exception as e:
        return f"识别失败：{str(e)}"


# 启动应用
if __name__ == "__main__":
    # 图片路径（注意使用原始字符串或双反斜杠）
    image_path = r"C:\Users\Administrator\Desktop\pdfs.png"
    result = ocr_image(image_path)
    print("识别结果：")
    print(result)

```

end