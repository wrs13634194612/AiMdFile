
python对文件和文件夹压缩和解压zip
1. 创建测试环境，生成示例文件和目录结构。
2. 使用zipfile库压缩单个文件到指定位置。
3. 压缩整个文件夹为ZIP格式，保留目录结构。
4. 解压ZIP文件到指定目录。
5. 使用tarfile库压缩文件夹为tar.gz格式。
6. 解压tar.gz文件。
7. 使用shutil库进行快捷的压缩和解压操作。
8. 通用的单个文件压缩功能，可指定输出路径。
9. 通用的文件夹压缩功能，同样指定输出。
10. 通用的解压功能，处理ZIP文件到指定目录。

 step1:C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py


```python
import os
import zipfile
import tarfile
import shutil
import gzip


# ---------- 初始化：创建测试文件和文件夹 ----------
def create_test_env():
    # 创建测试目录结构
    os.makedirs("data/my_folder/subdir", exist_ok=True)
    os.makedirs("output", exist_ok=True)
    os.makedirs("extracted", exist_ok=True)

    # 创建测试文件
    with open("data/file.txt", "w") as f:
        f.write("This is a sample text file.")

    # 在文件夹中创建测试文件
    with open("data/my_folder/subdir/note.txt", "w") as f:
        f.write("This is a file inside subdirectory.")


# ---------- 函数定义 ----------
def zip_single_file():
    """压缩单个文件到固定路径"""
    src_file = "data/file.txt"
    dst_zip = "output/single_file.zip"

    with zipfile.ZipFile(dst_zip, "w", zipfile.ZIP_DEFLATED) as zipf:
        zipf.write(src_file, arcname=os.path.basename(src_file))
    print(f"ZIP压缩单个文件完成：{dst_zip}")


def zip_entire_folder():
    """压缩整个文件夹到固定路径"""
    src_dir = "data/my_folder"
    dst_zip = "output/folder.zip"

    with zipfile.ZipFile(dst_zip, "w", zipfile.ZIP_DEFLATED) as zipf:
        for root, dirs, files in os.walk(src_dir):
            for file in files:
                file_path = os.path.join(root, file)
                arcname = os.path.relpath(file_path, start=src_dir)
                zipf.write(file_path, arcname=arcname)
    print(f"ZIP压缩文件夹完成：{dst_zip}")


def unzip_file():
    """解压ZIP文件到固定路径"""
    zip_path = "output/folder.zip"
    extract_dir = "extracted/zip_contents"

    with zipfile.ZipFile(zip_path, "r") as zipf:
        zipf.extractall(extract_dir)
    print(f"ZIP解压完成到：{extract_dir}")


def tar_gz_folder():
    """压缩文件夹为tar.gz到固定路径"""
    src_dir = "data/my_folder"
    dst_tar = "output/folder.tar.gz"

    with tarfile.open(dst_tar, "w:gz") as tar:
        tar.add(src_dir, arcname=os.path.basename(src_dir))
    print(f"TAR.GZ压缩完成：{dst_tar}")


def untar_gz_file():
    """解压tar.gz文件到固定路径"""
    tar_path = "output/folder.tar.gz"
    extract_dir = "extracted/tar_contents"

    with tarfile.open(tar_path, "r:*") as tar:
        tar.extractall(extract_dir)
    print(f"TAR.GZ解压完成到：{extract_dir}")


def shutil_demo():
    """使用shutil快捷操作"""
    # 压缩
    shutil.make_archive("output/shutil_archive", "zip", "data/my_folder")
    # 解压
    shutil.unpack_archive("output/shutil_archive.zip", "extracted/shutil_contents")
    print("shutil压缩/解压操作完成")


def compress_file(src_path):
    """
    压缩单个文件到指定输出目录
    :param src_path: 要压缩的文件路径（如：C:\\Users\\wangrusheng\\Downloads\\black.txt）
    """
    output_dir = r"C:\Users\wangrusheng\Pictures"

    # 验证输入路径
    if not os.path.isfile(src_path):
        raise ValueError(f"路径不是文件：{src_path}")

    # 创建输出目录
    os.makedirs(output_dir, exist_ok=True)

    # 生成输出文件名
    base_name = os.path.splitext(os.path.basename(src_path))[0]
    dst_zip = os.path.join(output_dir, f"{base_name}.zip")

    # 执行压缩
    with zipfile.ZipFile(dst_zip, "w", zipfile.ZIP_DEFLATED) as zipf:
        zipf.write(src_path, arcname=os.path.basename(src_path))

    print(f"文件压缩完成：{dst_zip}")
    return dst_zip


def compress_folder(src_path):
    """
    压缩整个文件夹到指定输出目录
    :param src_path: 要压缩的文件夹路径（如：C:\\Users\\wangrusheng\\Downloads\\red）
    """
    output_dir = r"C:\Users\wangrusheng\Pictures"

    # 验证输入路径
    src_path = src_path.rstrip(os.sep)  # 去除末尾分隔符
    if not os.path.isdir(src_path):
        raise ValueError(f"路径不是文件夹：{src_path}")

    # 创建输出目录
    os.makedirs(output_dir, exist_ok=True)

    # 生成输出文件名
    folder_name = os.path.basename(src_path)
    dst_zip = os.path.join(output_dir, f"{folder_name}.zip")

    # 执行压缩
    with zipfile.ZipFile(dst_zip, "w", zipfile.ZIP_DEFLATED) as zipf:
        for root, dirs, files in os.walk(src_path):
            for file in files:
                file_path = os.path.join(root, file)
                arcname = os.path.relpath(file_path, start=src_path)
                zipf.write(file_path, arcname=arcname)

    print(f"文件夹压缩完成：{dst_zip}")
    return dst_zip


def decompress(zip_path):
    """
    解压文件到指定输出目录
    :param zip_path: 要解压的zip文件路径（如：C:\\Users\\wangrusheng\\Downloads\\yellow.zip）
    """
    output_dir = r"C:\Users\wangrusheng\Pictures"

    # 验证输入文件
    if not zipfile.is_zipfile(zip_path):
        raise ValueError(f"不是有效的ZIP文件：{zip_path}")

    # 创建解压目录
    extract_folder = os.path.splitext(os.path.basename(zip_path))[0]
    extract_path = os.path.join(output_dir, extract_folder)
    os.makedirs(extract_path, exist_ok=True)

    # 执行解压
    with zipfile.ZipFile(zip_path, "r") as zipf:
        zipf.extractall(extract_path)

    print(f"解压完成到：{extract_path}")
    return extract_path


# ---------- 执行示例 ----------
if __name__ == "__main__":
    #我是分割线开始
    # 初始化测试环境
    #create_test_env()

    # 执行所有操作
    #zip_single_file()  # ZIP压缩单个文件
    #zip_entire_folder()  # ZIP压缩文件夹
    #unzip_file()  # ZIP解压

    #tar_gz_folder()  # TAR.GZ压缩
    #untar_gz_file()  # TAR.GZ解压

    #shutil_demo()  # 使用shutil工具
    #我是分割线结尾
    # 压缩单个文件
    compress_file(r"C:\Users\wangrusheng\Downloads\black.txt")

    # 压缩文件夹（注意路径不要以反斜杠结尾）
    compress_folder(r"C:\Users\wangrusheng\Downloads\red")

    # 解压文件
    decompress(r"C:\Users\wangrusheng\Downloads\yellow.zip")

    print("\n操作全部完成！检查 output/ 和 extracted/ 目录查看结果")
```


step2:运行

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py 
ZIP压缩单个文件完成：output/single_file.zip
ZIP压缩文件夹完成：output/folder.zip
ZIP解压完成到：extracted/zip_contents
TAR.GZ压缩完成：output/folder.tar.gz
C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py:75: DeprecationWarning: Python 3.14 will, by default, filter extracted tar archives and reject files or modify their metadata. Use the filter argument to control this behavior.
  tar.extractall(extract_dir)
TAR.GZ解压完成到：extracted/tar_contents
shutil压缩/解压操作完成

操作全部完成！检查 output/ 和 extracted/ 目录查看结果
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py 
文件压缩完成：C:\Users\wangrusheng\Pictures\black.zip
文件夹压缩完成：C:\Users\wangrusheng\Pictures\red.zip
解压完成到：C:\Users\wangrusheng\Pictures\yellow

操作全部完成！检查 output/ 和 extracted/ 目录查看结果
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> 

```

end