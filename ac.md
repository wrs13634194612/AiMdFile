我希望对指定网页的，博客列表，获取url，然后保存成本地文件，用python实现
step1:
```python
 
 
import requests
from bs4 import BeautifulSoup
import json


def get_blog_links(url):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36'
    }

    try:
        response = requests.get(url, headers=headers, timeout=10)
        response.raise_for_status()

        soup = BeautifulSoup(response.text, 'html.parser')
        article_links = []

        # 查找所有包含文章链接的<a>标签
        for a_tag in soup.find_all('a', href=True):
            href = a_tag['href']
            if '/article/details/' in href:
                # 处理可能的相对路径
                if href.startswith('http'):
                    article_links.append(href)
                else:
                    article_links.append(f'https://blog.csdn.net{href}')

        # 去重并保持顺序
        seen = set()
        unique_links = [x for x in article_links if not (x in seen or seen.add(x))]

        return unique_links

    except Exception as e:
        print(f'抓取过程中出现错误: {str(e)}')
        return []


def save_to_json(data, filename):
    with open(filename, 'w', encoding='utf-8') as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f'已成功保存{len(data)}条链接到 {filename}')


if __name__ == '__main__':
    target_url = 'https://blog.csdn.net/cf8833?type=blog'
    output_file = 'csdn_blog_links.json'

    links = get_blog_links(target_url)

    if links:
        save_to_json(links, output_file)
    else:
        print('未找到有效文章链接')
```

end