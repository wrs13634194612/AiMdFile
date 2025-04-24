说明：
我希望用python，将name.mp3这段录音文件，添加背景音乐，bg.mp3，然后生成新的文件
step1: 添加依赖

```bash
pip install pydub
```

step2:下载ffmpeg

```bash
1.打开windows powershell  ，管理员运行
2.winget install ffmpeg
3.大约160M，等待下载成功
4.关闭windows powershell 
5.重新打开windows powershell 
6.验证
7.where ffmpeg
8.where ffprobe
9.ffmpeg --version
10.ffprobe --version
如果没报错，说明安装成功


```

step3:在这里创建文件C:\Users\wangrusheng> python hello.py

```python
from pydub import AudioSegment
import os

AudioSegment.converter = r"C:\Users\wangrusheng\AppData\Local\Microsoft\WinGet\Links\ffmpeg.exe"

# 输入文件路径
name_path = r"C:\Users\wangrusheng\Documents\name.mp3"
bg_path = r"C:\Users\wangrusheng\Documents\bg.mp3"
output_path = r"C:\Users\wangrusheng\Documents\combineds.mp3"

# 加载音频文件
name_audio = AudioSegment.from_mp3(name_path)
bg_audio = AudioSegment.from_mp3(bg_path)

# 计算主音频长度
duration = len(name_audio)

# 延长背景音乐至与主音频同长（循环）
bg_repeated = bg_audio
while len(bg_repeated) < duration:
    bg_repeated += bg_audio  # 循环叠加
bg_repeated = bg_repeated[:duration]  # 截取相同长度

# 降低背景音量（可选，示例降低7分贝）
bg_repeated = bg_repeated - 7

# 混合音频（保留主音频，叠加背景）
combined = name_audio.overlay(bg_repeated)

# 导出文件
combined.export(output_path, format="mp3")

print(f"合并完成，文件已保存至：{output_path}")
```

step4:重新打开windows powershell  ，管理员运行

```bash
PS C:\WINDOWS\system32> pwd

Path
----
C:\WINDOWS\system32


PS C:\WINDOWS\system32> cd ..
PS C:\WINDOWS> cd ..
PS C:\> cd C:\Users\wangrusheng
PS C:\Users\wangrusheng> python --version
Python 3.12.9
PS C:\Users\wangrusheng> python hello.py
合并完成，文件已保存至：C:\Users\wangrusheng\Documents\combined.mp3
PS C:\Users\wangrusheng>
```
step5:去这个目录，C:\Users\wangrusheng\Documents\combined.mp3，打开文件，播放音频


///分割线

step1:fastapi

```python
from fastapi import FastAPI
from pydub import AudioSegment
import os

app = FastAPI()

# 固定路径配置
AudioSegment.converter = r"C:\Users\wangrusheng\AppData\Local\Microsoft\WinGet\Links\ffmpeg.exe"
NAME_PATH = r"C:\Users\wangrusheng\Documents\name.mp3"
BG_PATH = r"C:\Users\wangrusheng\Documents\bg.mp3"
OUTPUT_PATH = r"C:\Users\wangrusheng\Documents\combineds.mp3"

@app.get("/combine-audio")
def combine_audio():
    try:
        # 加载音频文件
        name_audio = AudioSegment.from_mp3(NAME_PATH)
        bg_audio = AudioSegment.from_mp3(BG_PATH)

        # 计算并循环背景音乐
        duration = len(name_audio)
        bg_repeated = bg_audio
        while len(bg_repeated) < duration:
            bg_repeated += bg_audio
        bg_repeated = bg_repeated[:duration]

        # 调整音量并混合
        bg_repeated = bg_repeated - 7
        combined = name_audio.overlay(bg_repeated)

        # 导出文件
        combined.export(OUTPUT_PATH, format="mp3")

        return {"status": "success", "output_path": OUTPUT_PATH}

    except Exception as e:
        return {"status": "error", "message": str(e)}

@app.get("/")
def health_check():
    return {"status": "ready"}
```
step2:在windows power shell里面 ，安装fastapi

```bash
PS C:\Users\wangrusheng> pip install fastapi uvicorn
```
step3:运行

```bash
PS C:\Users\wangrusheng> uvicorn hello:app --reload
[32mINFO[0m:     Will watch for changes in these directories: ['C:\\Users\\wangrusheng']
[32mINFO[0m:     Uvicorn running on [1mhttp://127.0.0.1:8000[0m (Press CTRL+C to quit)
[32mINFO[0m:     Started reloader process [[36m[1m41936[0m] using [36m[1mStatReload[0m
[32mINFO[0m:     Started server process [[36m39540[0m]
[32mINFO[0m:     Waiting for application startup.
[32mINFO[0m:     Application startup complete.
[32mINFO[0m:     127.0.0.1:55553 - "[1mGET /combine-audio HTTP/1.1[0m" [32m200 OK[0m
```

step4:在postman里面

```bash
get : http://localhost:8000/combine-audio


{
    "status": "success",
    "output_path": "C:\\Users\\wangrusheng\\Documents\\combineds.mp3"
}
```
step5:去本地路径打开，新生成的mp3文件

 end