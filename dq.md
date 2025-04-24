说明：
我希望用fastapi，下载在线图片url到本地
step1:下载依赖

```bash
(.venv) PS C:\Users\FastAPIProject1> pip install requests   
```

step2:本地版

```python
import requests
import os
from concurrent.futures import ThreadPoolExecutor

# 配置保存路径（自动创建目录）
save_dir = "./downloaded_images"
os.makedirs(save_dir, exist_ok=True)

# 需要下载的图片数据
image_data = [
    (1, 'https://randomuser.me/api/portraits/men/1.jpg', 1),
    (2, 'https://randomuser.me/api/portraits/men/2.jpg', 1),
    (3, 'https://randomuser.me/api/portraits/men/3.jpg', 1),
    (4, 'https://randomuser.me/api/portraits/men/4.jpg', 1),
    (5, 'https://randomuser.me/api/portraits/men/5.jpg', 1),
    (6, 'https://randomuser.me/api/portraits/men/6.jpg', 1)
]


def download_image(item):
    """下载单个图片"""
    img_id, url, category_id = item

    try:
        # 发送HTTP请求
        response = requests.get(url, timeout=10)
        response.raise_for_status()  # 自动处理4xx/5xx错误

        # 生成文件名（可自定义命名规则）
        filename = f"category{category_id}_image{img_id}.jpg"
        save_path = os.path.join(save_dir, filename)

        # 保存文件
        with open(save_path, 'wb') as f:
            f.write(response.content)

        print(f"成功下载：{filename}")
        return True

    except requests.exceptions.RequestException as e:
        print(f"下载失败[{url}]：{str(e)}")
        return False


def download_all():
    """多线程下载所有图片"""
    with ThreadPoolExecutor(max_workers=4) as executor:
        results = executor.map(download_image, image_data)

    success_count = sum(results)
    print(f"\n下载完成，成功{success_count}/共{len(image_data)}张")


if __name__ == "__main__":
    download_image(image_data[2])
```

step3:在线版

```python
from fastapi import FastAPI
import requests
import os

app = FastAPI()

# 固定配置
SAVE_DIR = r"C:\downloaded_images"  # 本地绝对路径
os.makedirs(SAVE_DIR, exist_ok=True)  # 自动创建目录

IMAGE_URLS = [
    'https://randomuser.me/api/portraits/men/1.jpg',
    'https://randomuser.me/api/portraits/men/2.jpg',
    'https://randomuser.me/api/portraits/men/3.jpg',
    'https://randomuser.me/api/portraits/men/4.jpg',
    'https://randomuser.me/api/portraits/men/5.jpg',
    'https://randomuser.me/api/portraits/men/6.jpg'
]


@app.get("/download-images")
def download_images():
    """固定路径下载接口"""
    results = []

    for index, url in enumerate(IMAGE_URLS, 1):
        try:
            # 下载图片
            response = requests.get(url)
            response.raise_for_status()

            # 保存文件（固定命名规则）
            filepath = os.path.join(SAVE_DIR, f"image_{index}.jpg")
            with open(filepath, "wb") as f:
                f.write(response.content)

            results.append(f"图片{index}下载成功")

        except Exception as e:
            results.append(f"图片{index}下载失败: {str(e)}")

    return {"status": "完成", "results": results}


@app.get("/")
def health_check():
    return {"usage": "访问 /download-images 下载图片"}
```

step4:postman验证

```bash
http://localhost:8000/download-images

{
    "status": "完成",
    "results": [
        "图片1下载成功",
        "图片2下载成功",
        "图片3下载成功",
        "图片4下载成功",
        "图片5下载成功",
        "图片6下载成功"
    ]
}
```

end