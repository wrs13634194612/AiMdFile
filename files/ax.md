c++和python复制java文件到指定目录
1.递归遍历目录
2.扁平化复制到目标目录
快速收集分散在多个子目录中的 .java 文件到一个统一目录。
备份或简单迁移Java项目文件（无需保留目录结构时）。
该代码是一个实用的Java文件收集工具，适合需要快速提取所有Java文件但无需考虑目录结构的场景，使用者需注意可能存在的文件覆盖问题。

step1:

```bash
输入路径：C:\Users\wangrusheng\Downloads\Dousheng-master
输出路径：C:\JavaFilesCopyz
```

step2:c++ 
C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include <filesystem>
#include <iostream>
#include <string>

namespace fs = std::filesystem;

void copy_java_files(const std::string& source_dir, const std::string& dest_dir) {
    try {
        fs::path source_path(source_dir);
        fs::path dest_path(dest_dir);

        // 创建目标根目录
        fs::create_directories(dest_path);

        // 递归遍历源目录
        for (const auto& entry : fs::recursive_directory_iterator(source_path)) {
            if (entry.is_regular_file() && entry.path().extension() == ".java") {
                // 直接使用文件名作为目标路径
                fs::path full_dest_path = dest_path / entry.path().filename();

                // 复制文件（覆盖已存在的）
                fs::copy_file(entry.path(),
                    full_dest_path,
                    fs::copy_options::overwrite_existing);

                std::cout << "已复制: " << entry.path().filename() << std::endl;
            }
        }
    }
    catch (const fs::filesystem_error& e) {
        std::cerr << "文件操作错误: " << e.what() << std::endl;
    }
    catch (const std::exception& e) {
        std::cerr << "标准异常: " << e.what() << std::endl;
    }
    catch (...) {
        std::cerr << "未知错误发生" << std::endl;
    }
}

int main() {
    copy_java_files(
        R"(C:\Users\wangrusheng\Downloads\Dousheng-master)",
        R"(C:\JavaFilesCopys)"
    );

    return 0;
}
```

step3:python
C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py

```python
import os
import shutil
from pathlib import Path


def copy_java_files(source_dir: str, dest_dir: str) -> None:
    try:
        # 创建目标目录
        Path(dest_dir).mkdir(parents=True, exist_ok=True)

        # 递归遍历源目录
        for root, _, files in os.walk(source_dir):
            for file in files:
                if file.endswith('.java'):
                    src_path = os.path.join(root, file)
                    dest_path = os.path.join(dest_dir, file)

                    # 复制文件（自动覆盖）
                    shutil.copy2(src_path, dest_path)
                    print(f"已复制: {file}")

    except PermissionError as e:
        print(f"权限错误: {str(e)}")
    except OSError as e:
        print(f"操作系统错误: {str(e)}")
    except Exception as e:
        print(f"未知错误: {str(e)}")


if __name__ == "__main__":
    source = r"C:\Users\wangrusheng\Downloads\Dousheng-master"
    destination = r"C:\JavaFilesCopyz"

    copy_java_files(source, destination)

```
step4:复制文件内容

```cpp
#include <iostream>
#include <filesystem>
#include <fstream>
#include <string>
#include <algorithm>

namespace fs = std::filesystem;

int main() {
    // 定义目标目录和输出文件路径
    fs::path target_dir = "C:/Users/wangrusheng/Downloads/UnoGame-main/JavaFilesCopyz";
    fs::path output_file = target_dir / "java_files_content.txt";

    // 创建并打开输出文件
    std::ofstream outfile(output_file);
    if (!outfile.is_open()) {
        std::cerr << "错误：无法创建输出文件！" << std::endl;
        return 1;
    }

    try {
        // 遍历目标目录
        for (const auto& entry : fs::directory_iterator(target_dir)) {
            if (entry.is_regular_file()) {
                // 获取文件扩展名并转换为小写
                std::string ext = entry.path().extension().string();
                std::transform(ext.begin(), ext.end(), ext.begin(), ::tolower);

                // 判断是否为Java文件
                if (ext == ".java") {
                    std::ifstream infile(entry.path());
                    if (infile) {
                        // 添加文件分隔标识
                        outfile << "=== File: " << entry.path().filename().string()
                            << " ===\n";
                        // 写入文件内容
                        outfile << infile.rdbuf() << "\n\n";
                        infile.close();
                    }
                    else {
                        std::cerr << "警告：无法读取文件 "
                            << entry.path() << std::endl;
                    }
                }
            }
        }
    }
    catch (const fs::filesystem_error& e) {
        std::cerr << "文件系统错误: " << e.what() << std::endl;
        return 1;
    }

    std::cout << "成功合并所有Java文件内容到：" << output_file << std::endl;
    return 0;
}
```
step5:功能整合，完成版

```cpp
#include <filesystem>
#include <iostream>
#include <fstream>
#include <string>
#include <algorithm>

namespace fs = std::filesystem;

// 复制所有Java文件到目标目录
void copy_java_files(const std::string& source_dir, const std::string& dest_dir) {
    try {
        fs::path source_path(source_dir);
        fs::path dest_path(dest_dir);

        fs::create_directories(dest_path); // 确保目标目录存在

        for (const auto& entry : fs::recursive_directory_iterator(source_path)) {
            if (entry.is_regular_file() && entry.path().extension() == ".java") {
                fs::path full_dest_path = dest_path / entry.path().filename();
                fs::copy_file(entry.path(), full_dest_path, fs::copy_options::overwrite_existing);
                std::cout << "已复制: " << entry.path().filename() << std::endl;
            }
        }
    }
    catch (const fs::filesystem_error& e) {
        std::cerr << "文件操作错误: " << e.what() << std::endl;
    }
    catch (const std::exception& e) {
        std::cerr << "标准异常: " << e.what() << std::endl;
    }
    catch (...) {
        std::cerr << "未知错误发生" << std::endl;
    }
}

// 合并Java文件内容到文本文件
void merge_java_files(const std::string& target_dir, const std::string& output_filename) {
    try {
        fs::path output_path = fs::path(target_dir) / output_filename;
        std::ofstream outfile(output_path);

        if (!outfile.is_open()) {
            std::cerr << "错误：无法创建输出文件！" << std::endl;
            return;
        }

        for (const auto& entry : fs::directory_iterator(target_dir)) {
            if (entry.is_regular_file()) {
                std::string ext = entry.path().extension().string();
                std::transform(ext.begin(), ext.end(), ext.begin(), ::tolower);

                if (ext == ".java") {
                    std::ifstream infile(entry.path());
                    if (infile) {
                        outfile << "=== File: " << entry.path().filename().string() << " ===\n";
                        outfile << infile.rdbuf() << "\n\n";
                        infile.close();
                    }
                    else {
                        std::cerr << "警告：无法读取文件 " << entry.path() << std::endl;
                    }
                }
            }
        }
        std::cout << "成功合并所有Java文件内容到：" << output_path << std::endl;
    }
    catch (const fs::filesystem_error& e) {
        std::cerr << "文件系统错误: " << e.what() << std::endl;
    }
    catch (const std::exception& e) {
        std::cerr << "标准异常: " << e.what() << std::endl;
    }
    catch (...) {
        std::cerr << "未知错误发生" << std::endl;
    }
}

int main() {
    const std::string source_dir = R"(C:\Users\wangrusheng\Downloads\Dousheng-master)";
    const std::string dest_dir = R"(C:\JavaFilesCopyx)";
    const std::string output_filename = "java_files_content.txt";

    // 第一步：复制Java文件
    copy_java_files(source_dir, dest_dir);

    // 第二步：合并文件内容
    merge_java_files(dest_dir, output_filename);

    return 0;
}
```
step6:python版本

```python
import os
import shutil
from pathlib import Path


def copy_java_files(source_dir: str, dest_dir: str) -> None:
    try:
        source_path = Path(source_dir)
        dest_path = Path(dest_dir)

        # 确保目标目录存在
        os.makedirs(dest_path, exist_ok=True)

        # 递归查找所有Java文件
        for java_file in source_path.rglob('*.java'):
            dest_file = dest_path / java_file.name
            # 复制并覆盖已存在的文件
            shutil.copy2(java_file, dest_file)
            print(f"已复制: {java_file.name}")

    except OSError as e:
        print(f"文件操作错误: {e}")
    except Exception as e:
        print(f"标准异常: {e}")
    except:
        print("未知错误发生")


def merge_java_files(target_dir: str, output_filename: str) -> None:
    try:
        target_path = Path(target_dir)
        output_path = target_path / output_filename

        with open(output_path, 'w', encoding='utf-8') as outfile:
            # 遍历目标目录下的文件
            for entry in target_path.iterdir():
                if entry.is_file() and entry.suffix.lower() == '.java':
                    try:
                        with open(entry, 'r', encoding='utf-8') as infile:
                            content = infile.read()
                            outfile.write(f"=== File: {entry.name} ===\n")
                            outfile.write(content + "\n\n")
                    except OSError as e:
                        print(f"警告：无法读取文件 {entry.name}: {e}")
                    except Exception as e:
                        print(f"处理文件 {entry.name} 时发生异常: {e}")

            print(f"成功合并所有Java文件内容到：{output_path}")

    except OSError as e:
        print(f"文件系统错误: {e}")
    except Exception as e:
        print(f"标准异常: {e}")
    except:
        print("未知错误发生")


def main():
    source_dir = r"C:\Users\wangrusheng\Downloads\Dousheng-master"
    dest_dir = r"C:\JavaFilesCopyw"
    output_filename = "java_files_content.txt"

    copy_java_files(source_dir, dest_dir)
    merge_java_files(dest_dir, output_filename)


if __name__ == "__main__":
    main()
```

end