博客爬虫算法

我希望从某些网站，把博客文章保存成本地的md文件，用python实现
不管你怎么想，反正我是成功了

step1:C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py

```python
import requests
from bs4 import BeautifulSoup
import html2text  # 新增HTML转Markdown库

# 目标文章URL
url = 'https://blog.csdn.net/cf8833/article/details/147413479'

# 设置请求头模拟浏览器
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
}

try:
    # 发送HTTP请求
    response = requests.get(url, headers=headers)
    response.raise_for_status()

    # 解析HTML内容
    soup = BeautifulSoup(response.text, 'html.parser')

    # 查找文章主体
    article_content = soup.find('div', {'id': 'content_views'})

    if article_content:
        # 创建HTML转Markdown转换器
        h = html2text.HTML2Text()
        h.body_width = 0  # 禁用自动换行
        h.emphasis_mark = '*'  # 设置斜体符号
        h.strong_mark = '**'  # 设置加粗符号
        h.ul_item_mark = '-'  # 设置无序列表符号

        # 转换HTML内容为Markdown
        md_content = h.handle(str(article_content))

        # 保存到文件
        with open('csdn_article2.md', 'w', encoding='utf-8') as f:
            f.write(md_content)
        print('文章已成功保存为Markdown格式！')
    else:
        print('未找到文章内容，请检查页面结构。')

except requests.exceptions.RequestException as e:
    print(f'请求出错：{e}')
except Exception as e:
    print(f'发生错误：{e}')
```
end