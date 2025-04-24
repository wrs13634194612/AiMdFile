说明：
我计划用python，把pdf文件转换成word文件
step1:把python环境安装好，然后把helloworld跑起来

step2:安装依赖： 首先需要安装必要的Python库，在终端中运行，会开始下载依赖包，等待下载完成

```bash
C:\Users\Administrator>pip --version
pip 25.0.1 from C:\Users\Administrator\AppData\Local\Programs\Python\Python312\Lib\site-packages\pip (python 3.12)

C:\Users\Administrator>pip install pdf2docx python-docx

Collecting pdf2docx
  Using cached pdf2docx-0.5.8-py3-none-any.whl.metadata (3.2 kB)
  Downloading fonttools-4.56.0-cp311-cp311-win_amd64.whl.metadata (103 kB)
Collecting numpy>=1.17.2 (from pdf2docx)
  Downloading numpy-2.2.3-cp311-cp311-win_amd64.whl.metadata (60 kB)
Collecting opencv-python-headless>=4.5 (from pdf2docx)
  Using cached opencv_python_headless-4.11.0.86-cp37-abi3-win_amd64.whl.metadata (20 kB)
Collecting fire>=0.3.0 (from pdf2docx)
  Using cached fire-0.7.0.tar.gz (87 kB)
  Preparing metadata (setup.py) ... done
Collecting lxml>=3.1.0 (from python-docx)
  Downloading lxml-5.3.1-cp311-cp311-win_amd64.whl.metadata (3.8 kB)
Collecting typing-extensions>=4.9.0 (from python-docx)
  Using cached typing_extensions-4.12.2-py3-none-any.whl.metadata (3.0 kB)
Collecting termcolor (from fire>=0.3.0->pdf2docx)
  Using cached termcolor-2.5.0-py3-none-any.whl.metadata (6.1 kB)
Using cached pdf2docx-0.5.8-py3-none-any.whl (132 kB)
Using cached python_docx-1.1.2-py3-none-any.whl (244 kB)
Downloading fonttools-4.56.0-cp311-cp311-win_amd64.whl (2.2 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.2/2.2 MB 104.8 kB/s eta 0:00:00
Downloading lxml-5.3.1-cp311-cp311-win_amd64.whl (3.8 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.8/3.8 MB 91.1 kB/s eta 0:00:00
Downloading numpy-2.2.3-cp311-cp311-win_amd64.whl (12.9 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 12.9/12.9 MB 95.3 kB/s eta 0:00:00
Downloading opencv_python_headless-4.11.0.86-cp37-abi3-win_amd64.whl (39.4 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 39.4/39.4 MB 94.9 kB/s eta 0:00:00
Downloading pymupdf-1.25.3-cp39-abi3-win_amd64.whl (16.5 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 16.5/16.5 MB 91.1 kB/s eta 0:00:00
Downloading typing_extensions-4.12.2-py3-none-any.whl (37 kB)
Downloading termcolor-2.5.0-py3-none-any.whl (7.8 kB)
Building wheels for collected packages: fire
  Building wheel for fire (setup.py) ... done
  Created wheel for fire: filename=fire-0.7.0-py3-none-any.whl size=114261 sha256=d635b5056afde4d2d8f1b04a79ad6c18207aef0d16c499c683ef7eac73194231
  Stored in directory: c:\users\administrator\appdata\local\pip\cache\wheels\46\54\24\1624fd5b8674eb1188623f7e8e17cdf7c0f6c24b609dfb8a89
Successfully built fire
Installing collected packages: typing-extensions, termcolor, PyMuPDF, numpy, lxml, fonttools, python-docx, opencv-python-headless, fire, pdf2docx
Successfully installed PyMuPDF-1.25.3 fire-0.7.0 fonttools-4.56.0 lxml-5.3.1 numpy-2.2.3 opencv-python-headless-4.11.0.86 pdf2docx-0.5.8 python-docx-1.1.2 termcolor-2.5.0 typing-extensions-4.12.2


```

step3:C:\Users\Administrator\PycharmProjects\PythonProject2\.venv\Scripts\activate_this.py

```python
# pdf_to_word.py
import os
from pdf2docx import Converter
from datetime import datetime


def convert_pdf_to_word(input_path, output_folder):
    """
    将PDF转换为Word文档

    参数:
    input_path (str): 输入的PDF文件路径
    output_folder (str): 输出文件夹路径

    返回:
    str: 生成的Word文件路径
    """
    try:
        # 生成输出文件名（带时间戳）
        filename = os.path.basename(input_path)
        output_name = f"{os.path.splitext(filename)[0]}_converted_{datetime.now().strftime('%Y%m%d%H%M%S')}.docx"
        output_path = os.path.join(output_folder, output_name)
        # 初始化转换器
        cv = Converter(input_path)

        # 设置进度回调函数
        def progress_callback(percentage):
            print(f"\r转换进度: {percentage:.0f}%", end='', flush=True)

        # 执行转换（多线程优化）
        cv.convert(output_path,
                   start=0,
                   end=None,
                   progress_callback=progress_callback)

        cv.close()
        print(f"\n转换成功！文件已保存至：{output_path}")
        return output_path
    except Exception as e:
        print(f"\n转换失败：{str(e)}")
        return None


if __name__ == "__main__":
    # 输入输出路径配置
    desktop = os.path.join(os.path.expanduser("~"), "Desktop")
    input_pdf = r"C:\Users\Administrator\Desktop\qe.pdf"
    output_dir = desktop
    # 检查输入文件
    if not os.path.exists(input_pdf):
        print(f"错误：输入的PDF文件不存在 - {input_pdf}")
        exit(1)
    # 创建输出目录（如果不存在）
    os.makedirs(output_dir, exist_ok=True)
    # 执行转换
    print("开始PDF转换...")
    result = convert_pdf_to_word(input_pdf, output_dir)

    if result:
        print("操作完成！")
    else:
        print("转换过程中出现问题。")
```

step4:点击run运行

```bash
C:\Users\Administrator\PycharmProjects\PythonProject2\.venv\Scripts\python.exe C:\Users\Administrator\PycharmProjects\PythonProject2\.venv\Scripts\activate_this.py 
[INFO] Start to convert C:\Users\Administrator\Desktop\files.pdf
[INFO] [1/4] Opening document...
[INFO] [2/4] Analyzing document...
开始PDF转换...
[INFO] [3/4] Parsing pages...
[INFO] (1/1) Page 1
[INFO] [4/4] Creating pages...
[INFO] (1/1) Page 1
[INFO] Terminated in 0.10s.

转换成功！文件已保存至：C:\Users\Administrator\Desktop\files_converted_20250306142813.docx
操作完成！

Process finished with exit code 0
```

end