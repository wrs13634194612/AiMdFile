功能分别是：

1.解压ZIP到指定目录的子文件夹。
2.批量重命名文件夹为双字母格式。
3.复制并重命名特定目录的TS/TSX文件为TXT。
4.批量处理双字母子目录中的TS文件，复制并改名。
5.合并双字母文件夹中的TS文件内容到总文件。

step1：C盘脚本（解压模块）
功能：自动解压Downloads目录下所有ZIP文件
特点：
解压到"Unzipped_Files"目录下的同名子文件夹（如test.zip→Unzipped_Files/test）
自动过滤非ZIP文件
支持错误捕获和友好提示

```python
import os
import zipfile


def decompress_zip(zip_path, extract_root):
    """
    解压单个ZIP文件到指定根目录下的同名子文件夹
    :param zip_path: ZIP文件的完整路径
    :param extract_root: 解压的根目录路径
    """
    if not zipfile.is_zipfile(zip_path):
        print(f"警告：跳过无效ZIP文件 {zip_path}")
        return

    # 创建解压路径
    zip_name = os.path.basename(zip_path)
    folder_name = os.path.splitext(zip_name)[0]  # 去除扩展名
    extract_path = os.path.join(extract_root, folder_name)

    # 确保目标目录存在
    os.makedirs(extract_path, exist_ok=True)

    try:
        with zipfile.ZipFile(zip_path, 'r') as zip_ref:
            zip_ref.extractall(extract_path)
        print(f"成功解压：{zip_name} → {extract_path}")
    except Exception as e:
        print(f"解压失败：{zip_name} | 错误：{str(e)}")


def main():
    # 配置路径
    downloads_dir = r"C:\Users\wangrusheng\Downloads"
    extract_base = os.path.join(downloads_dir, "Unzipped_Files")

    # 创建解压根目录
    os.makedirs(extract_base, exist_ok=True)

    # 遍历并解压所有ZIP文件
    for filename in os.listdir(downloads_dir):
        file_path = os.path.join(downloads_dir, filename)

        # 过滤ZIP文件且排除可能的目录
        if filename.lower().endswith(".zip") and os.path.isfile(file_path):
            decompress_zip(file_path, extract_base)


if __name__ == "__main__":
    main()
```

step2:D盘脚本（重命名模块）
功能：批量重命名文件夹为双字母格式
特点：
支持从aa到zz的676种组合
分两步重命名避免冲突（先temp_*临时名）
自动排序原始文件夹名称
包含最大数量保护机制

```python
import os


def generate_folder_names():
    """生成aa, ab,..., zz形式的文件夹名称序列"""
    names = []
    for first in 'abcdefghijklmnopqrstuvwxyz':
        for second in 'abcdefghijklmnopqrstuvwxyz':
            names.append(f"{first}{second}")
    return names


def rename_folders():
    path = r'C:\Users\wangrusheng\Downloads\Unzipped_Files'

    # 获取所有文件夹并排序（按名称升序）
    folders = [f for f in os.listdir(path)
               if os.path.isdir(os.path.join(path, f))]
    sorted_folders = sorted(folders)

    # 生成名称列表
    new_names = generate_folder_names()

    if len(sorted_folders) > len(new_names):
        print("错误：文件夹数量超过命名规则支持的最大数量（676）")
        return

    # 分两步重命名避免冲突
    # 第一步：重命名为临时名称
    temp_names = []
    for i, folder in enumerate(sorted_folders):
        temp_name = f"temp_{i}"
        src = os.path.join(path, folder)
        dst = os.path.join(path, temp_name)
        os.rename(src, dst)
        temp_names.append(temp_name)

    # 第二步：重命名为最终名称
    for temp_name, new_name in zip(temp_names, new_names):
        src = os.path.join(path, temp_name)
        dst = os.path.join(path, new_name)
        os.rename(src, dst)
        print(f"已重命名 {temp_name} -> {new_name}")


if __name__ == "__main__":
    # 运行前请注意备份数据！
    rename_folders()
```

step3:E盘脚本（文件转换模块-单目录版）
功能：复制指定目录的TS/TSX文件并转换扩展名
特点：
直接处理指定单一目录（如re\aa→ce\aa）
保持目录结构不变
自动转换.ts/.tsx为.txt
支持递归遍历子目录

```python
import shutil
from pathlib import Path


def copy_ts_files(source_dir, dest_dir):
    """
    复制源目录中的所有.ts和.tsx文件到目标目录，并将其扩展名改为.txt
    直接处理‌指定的单一目录‌（re\aa → ce\aa）
    不自动过滤子目录名称，完全依赖用户输入的路径
    参数:
        source_dir (str): 源目录路径
        dest_dir (str): 目标目录路径
    """
    source_path = Path(source_dir)
    dest_path = Path(dest_dir)

    try:
        # 创建目标目录（如果不存在）
        dest_path.mkdir(parents=True, exist_ok=True)

        # 递归遍历源目录
        for file_path in source_path.rglob('*'):
            try:
                # 过滤普通文件且扩展名为.ts或.tsx
                if file_path.is_file() and file_path.suffix.lower() in ('.ts', '.tsx'):
                    # 生成新文件名
                    new_name = f"{file_path.stem}.txt"
                    dest_file = dest_path / new_name

                    # 复制并重命名文件
                    shutil.copy2(file_path, dest_file)
                    print(f"已复制: {file_path.name} => {new_name}")

            except Exception as file_error:
                print(f"处理文件 {file_path} 时出错: {str(file_error)}")

    except Exception as dir_error:
        print(f"目录操作出错: {str(dir_error)}")


if __name__ == "__main__":
    # 使用示例路径（替换为实际路径）
    copy_ts_files(
        r"C:\Users\wangrusheng\Downloads\re\aa",
        r"C:\Users\wangrusheng\Downloads\ce\aa"
    )
```

step4:F盘脚本（文件转换模块-批量版）
功能：批量处理双字母目录的TS文件
特点：
自动遍历re目录下所有两位小写字母命名的子目录（如aa, zb）
在xe目录创建对应子目录
支持多级目录处理
包含目录名格式过滤机制

```python
import shutil
from pathlib import Path

def copy_ts_files(source_dir, dest_dir):
    """
    复制源目录中的所有.ts和.tsx文件到目标目录，并将其扩展名改为.txt
    自动遍历源目录（C:\Users\wangrusheng\Downloads\re）下的‌所有子目录‌
    仅处理‌两位小写字母命名‌的子目录（如 aa, bb）
    参数:
        source_dir (str): 源目录路径
        dest_dir (str): 目标目录路径
    """
    source_path = Path(source_dir)
    dest_path = Path(dest_dir)

    try:
        # 创建目标目录（如果不存在）
        dest_path.mkdir(parents=True, exist_ok=True)

        # 递归遍历源目录
        for file_path in source_path.rglob('*'):
            try:
                # 过滤普通文件且扩展名为.ts或.tsx
                if file_path.is_file() and file_path.suffix.lower() in ('.ts', '.tsx'):
                    # 生成新文件名
                    new_name = f"{file_path.stem}.txt"
                    dest_file = dest_path / new_name

                    # 复制并重命名文件
                    shutil.copy2(file_path, dest_file)
                    print(f"已复制: {file_path.name} => {new_name}")

            except Exception as file_error:
                print(f"处理文件 {file_path} 时出错: {str(file_error)}")

    except Exception as dir_error:
        print(f"目录操作出错: {str(dir_error)}")

if __name__ == "__main__":
    # 基础路径
    base_source = Path(r"C:\Users\wangrusheng\Downloads\re")
    base_dest = Path(r"C:\Users\wangrusheng\Downloads\xe")

    # 遍历源目录下的所有子目录
    for source_subdir in base_source.iterdir():
        if source_subdir.is_dir():
            # 过滤只处理两位小写字母命名的目录（可选）
            dir_name = source_subdir.name
            if len(dir_name) == 2 and dir_name.isalpha() and dir_name.islower():
                dest_subdir = base_dest / dir_name
                copy_ts_files(str(source_subdir), str(dest_subdir))
                print(f"处理完成目录: {dir_name}")
```

sep5:G盘脚本（文件合并模块）
功能：聚合双字母目录中的TS文件内容
特点：
为每个双字母目录生成汇总文件（aaaaaaaaaaaaaaaaava_files_content.txt）
自动跳过不存在的目录
保留原始文件名作为内容分隔符
支持从aa到zz的全组合遍历

```python
import os
import itertools


def process_folder(folder_path):
    output_file = os.path.join(folder_path, "aaaaaaaaaaaaaaaaava_files_content.txt")

    try:
        with open(output_file, 'w', encoding='utf-8') as outfile:
            # 遍历目录中的所有文件
            for entry in os.scandir(folder_path):
                if entry.is_file() and entry.name.lower().endswith('.ts'):
                    # 跳过输出文件自身
                    if entry.name == os.path.basename(output_file):
                        continue

                    try:
                        with open(entry.path, 'r', encoding='utf-8') as infile:
                            # 写入文件头
                            outfile.write(f"=== File: {entry.name} ===\n")
                            # 写入文件内容
                            outfile.write(infile.read())
                            outfile.write("\n\n")
                    except Exception as e:
                        print(f"警告：无法读取文件 {entry.path} - {str(e)}")
        print(f"成功处理文件夹：{os.path.basename(folder_path)}")
        return True
    except Exception as e:
        print(f"处理文件夹 {folder_path} 时出错 - {str(e)}")
        return False


def main():
    base_dir = r"C:\Users\wangrusheng\Downloads\ye"

    # 生成所有可能的双字母组合（aa, ab, ..., zz）
    for chars in itertools.product('abcdefghijklmnopqrstuvwxyz', repeat=2):
        folder_name = ''.join(chars)
        folder_path = os.path.join(base_dir, folder_name)

        # 检查文件夹是否存在
        if os.path.isdir(folder_path):
            process_folder(folder_path)
        else:
            print(f"跳过不存在的文件夹：{folder_name}")


if __name__ == "__main__":
    main()
```

end

补充版本：

```python
import os
import itertools


def process_folder(folder_path):
    output_file = os.path.join(folder_path, "zzzzzxxaaaaaaaaaaaaaaaaava_files_content.txt")

    try:
        with open(output_file, 'w', encoding='utf-8') as outfile:
            for entry in os.scandir(folder_path):
                if entry.is_file() and entry.name.lower().endswith('.cs'):
                    if entry.name == os.path.basename(output_file):
                        continue

                    try:
                        # 修改点：添加 errors='replace' 参数
                        with open(entry.path, 'r', encoding='utf-8', errors='replace') as infile:
                            outfile.write(f"=== File: {entry.name} ===\n")
                            outfile.write(infile.read())
                            outfile.write("\n\n")
                    except Exception as e:
                        print(f"警告：无法读取文件 {entry.path} - {str(e)}")
        print(f"成功处理文件夹：{os.path.basename(folder_path)}")
        return True
    except Exception as e:
        print(f"处理文件夹 {folder_path} 时出错 - {str(e)}")
        return False


def main():
    base_dir = r"C:\Users\wangrusheng\Downloads\fe"

    for chars in itertools.product('abcdefghijklmnopqrstuvwxyz', repeat=2):
        folder_name = ''.join(chars)
        folder_path = os.path.join(base_dir, folder_name)

        if os.path.isdir(folder_path):
            process_folder(folder_path)
        else:
            print(f"跳过不存在的文件夹：{folder_name}")


if __name__ == "__main__":
    main()

```