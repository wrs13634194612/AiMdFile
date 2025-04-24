说明：我希望用nodejs 写一个小工具，去除本地txt文件中的html字符，去除字符后生成新的文件，同样保存在桌面 文件的具体位置是C:\Users\Administrator\Desktop\file.txt
step1:C:\Users\Administrator\WebstormProjects\untitled4\file.js

```javascript
const fs = require('fs');
const path = require('path');
// 定义文件路径
const desktopPath = 'C:\\Users\\Administrator\\Desktop';
const inputFile = path.join(desktopPath, 'file.txt');
const outputFile = path.join(desktopPath, 'clean_file.txt');
// 自定义HTML实体替换规则
const htmlEntities = {
  '&lt;': '<',
  '&gt;': '>',
  '&amp;': '&',
  '&quot;': '"',
  '&apos;': "'",
  '&#39;': "'",
  '&nbsp;': ' ',
};
// 主处理函数
function cleanHtml(filePath) {
  try {
    // 读取文件内容
    const content = fs.readFileSync(filePath, 'utf8');

    // 分步骤清理内容
    let cleaned = content
      // 移除HTML标签
      .replace(/<[^>]+>/g, '')
      // 替换HTML实体
      .replace(/&(?:[a-z]+|#\d+);/gi, (match) =>
        htmlEntities[match.toLowerCase()] || match
      )
      // 清理多余换行和空格
      .replace(/\n{3,}/g, '\n\n')
      .replace(/ {2,}/g, ' ');
    // 写入新文件
    fs.writeFileSync(outputFile, cleaned, 'utf8');

    console.log(`文件清理完成，已保存至：${outputFile}`);
    console.log(`清理前字符数：${content.length}`);
    console.log(`清理后字符数：${cleaned.length}`);
    console.log(`移除字符数：${content.length - cleaned.length}`);
  } catch (error) {
    console.error('处理文件时发生错误：');
    console.error(error.message);
    process.exit(1);
  }
}
// 执行清理
if (require.main === module) {
  // 检查源文件是否存在
  if (!fs.existsSync(inputFile)) {
    console.error(`错误：源文件 ${inputFile} 不存在`);
    process.exit(1);
  }
  console.log('正在清理HTML字符...');
  cleanHtml(inputFile);
}

```

step2: 运行

```bash
PS C:\Users\Administrator\WebstormProjects\untitled4> node file.js
正在清理HTML字符...
文件清理完成，已保存至：C:\Users\Administrator\Desktop\clean_file.txt
清理前字符数：2235
清理后字符数：1697
移除字符数：538

```
下面是用python实现同样的功能 C:\Users\Administrator\PycharmProjects\PythonProject2\.venv\Scripts\activate_this.py

```python
from pathlib import Path
import re
import sys

# 定义文件路径
desktop_path = Path(r'C:\Users\Administrator\Desktop')
input_file = desktop_path / 'file.txt'
output_file = desktop_path / 'cleans_file.txt'

# 自定义HTML实体替换规则
html_entities = {
    '&lt;': '<',
    '&gt;': '>',
    '&amp;': '&',
    '&quot;': '"',
    '&apos;': "'",
    '&#39;': "'",
    '&nbsp;': ' ',
}

def clean_html(file_path):
    try:
        # 读取文件内容
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()

        # 分步骤清理内容
        cleaned = content
        # 移除HTML标签
        cleaned = re.sub(r'<[^>]+>', '', cleaned)
        # 替换HTML实体
        def replace_entity(match):
            entity = match.text.lower()
            return html_entities.get(entity, match.text)
        cleaned = re.sub(
            r'&(?:[a-z]+|#\d+);',
            replace_entity,
            cleaned,
            flags=re.IGNORECASE
        )
        # 清理多余换行和空格
        cleaned = re.sub(r'\n{3,}', '\n\n', cleaned)
        cleaned = re.sub(r' {2,}', ' ', cleaned)

        # 写入新文件
        with open(output_file, 'w', encoding='utf-8') as f:
            f.write(cleaned)

        print(f'文件清理完成，已保存至：{output_file}')
        print(f'清理前字符数：{len(content)}')
        print(f'清理后字符数：{len(cleaned)}')
        print(f'移除字符数：{len(content) - len(cleaned)}')

    except Exception as e:
        print('处理文件时发生错误：')
        print(f'{e}')
        sys.exit(1)

if __name__ == "__main__":
    # 检查源文件是否存在
    if not input_file.exists():
        print(f'错误：源文件 {input_file} 不存在')
        sys.exit(1)
    
    print('正在清理HTML字符...')
    clean_html(input_file)
```

end