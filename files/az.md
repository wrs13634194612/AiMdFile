说明：python实现哈希计算器

step1:规则说明：

```bash
(1)主要功能和作用
	1.计算哈希值
		1.1字符串哈希：支持对输入字符串计算哈希值（如 python script.py -md5 -s "hello"）
		1.2文件哈希：支持对文件内容计算哈希值（如 python script.py -sha256 -f data.txt）

	2.支持的算法
		2.1MD5
		2.2SHA-1
		2.3SHA-256

(2)应用场景
	1.数据完整性验证
	2.文件传输后，通过对比哈希值确认文件未被篡改。
	3.软件发布时提供安装包的哈希值供用户验证。
	4.敏感信息存储
	5.哈希化存储密码（需结合盐值提升安全性，但该工具本身不提供盐值功能）。
```

step2:C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py

```python
import argparse
import hashlib
import sys

def calculate_string_hash(data: str, algorithm: str) -> str:
    """计算字符串的哈希值 在线哈希验证工具 https://www.lddgo.net/encrypt/hash """
    try:
        hasher = hashlib.new(algorithm)
    except ValueError:
        raise ValueError(f"Unsupported hash algorithm: {algorithm}")
    hasher.update(data.encode('utf-8'))
    return hasher.hexdigest()

def calculate_file_hash(file_path: str, algorithm: str) -> str:
    """计算文件的哈希值"""
    try:
        hasher = hashlib.new(algorithm)
        with open(file_path, 'rb') as f:
            while chunk := f.read(4096):
                hasher.update(chunk)
        return hasher.hexdigest()
    except FileNotFoundError:
        raise FileNotFoundError(f"File not found: {file_path}")
    except IsADirectoryError:
        raise IsADirectoryError(f"Input is a directory, not a file: {file_path}")

def main():
    parser = argparse.ArgumentParser(description='Calculate hashes for strings or files.')
    # 算法选项组（互斥且必须选择一个）
    algorithm_group = parser.add_mutually_exclusive_group(required=True)
    algorithm_group.add_argument('-md5', action='store_true', help='Compute MD5 hash')
    algorithm_group.add_argument('-sha1', action='store_true', help='Compute SHA-1 hash')
    algorithm_group.add_argument('-sha256', action='store_true', help='Compute SHA-256 hash')

    # 输入类型组（互斥且必须选择一个）
    input_group = parser.add_mutually_exclusive_group(required=True)
    input_group.add_argument('-s', '--string', type=str, help='Input string')
    input_group.add_argument('-f', '--file', type=str, help='Input file path')

    args = parser.parse_args()

    # 确定选择的算法
    algorithm = 'md5' if args.md5 else 'sha1' if args.sha1 else 'sha256'

    try:
        if args.string is not None:
            hash_value = calculate_string_hash(args.string, algorithm)
        else:
            hash_value = calculate_file_hash(args.file, algorithm)
        print(hash_value)
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

step3:验证并运行，缺乏文件去对应目录，用记事本新建，文件内可以写内容

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py -md5 -s "hello"                      
5d41402abc4b2a76b9719d911017c592
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py  -sha1 -s "hello"  
aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py   -sha256 -s "hello"  
2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py   -md5 -f test.txt    
Error: File not found: test.txt
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py   -md5 -f test.txt 
d41d8cd98f00b204e9800998ecf8427e
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py   -sha256 -f test.txt
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py   -sha256 -f test.txt
ddd6a6df001aff00e1e48d168b7d4f938e330029a6cd968d67d3dc4f7fe12784
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py   -md5 -f test.txt   
60fbb436adbbd267e26b6a99f4f3191f
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> 

```
step4:用浏览器搜索在线哈希计算工具，然后用网页工具生成哈希，与我们本地生成的哈希做比对，如果一样，证明代码正确，生成的哈希值是对的
end