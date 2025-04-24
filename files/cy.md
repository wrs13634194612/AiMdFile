说明：
用c++，将name.mp3这段录音文件，添加背景音乐，bg.mp3，然后生成新的文件

1.使用ffmpeg框架
2.背景音乐的音量不要超过主音频
step1:C:\Users\wangrusheng\CLionProjects\untitled9\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.30)
project(untitled9)

set(CMAKE_CXX_STANDARD 20)

add_executable(untitled9 main.cpp)

```

step2:C:\Users\wangrusheng\CLionProjects\untitled9\main.cpp

```cpp
#include <iostream>
#include <cstdio>
#include <string>
#include <cstdlib>
#include <sstream>

using namespace std;

string escapePath(const string& path) {
    return "\"" + path + "\"";
}

double getDuration(const string& filePath) {
    string cmd = "ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 " + escapePath(filePath);

    FILE* pipe = popen(cmd.c_str(), "r");
    if (!pipe) {
        cerr << "Failed to execute ffprobe command" << endl;
        return -1;
    }

    char buffer[128];
    string result;
    while (!feof(pipe)) {
        if (fgets(buffer, 128, pipe) != nullptr)
            result += buffer;
    }
    pclose(pipe);

    try {
        return stod(result);
    } catch (...) {
        cerr << "Failed to parse duration" << endl;
        return -1;
    }
}

int main() {
    // File path configuration
    const string name_path = "C:\\Users\\wangrusheng\\Documents\\name.mp3";
    const string bg_path = "C:\\Users\\wangrusheng\\Documents\\bgs.mp3";
    const string output_path = "C:\\Users\\wangrusheng\\Documents\\combineds.mp3";

    // Get main audio duration
    double duration = getDuration(name_path);
    if (duration <= 0) {
        cerr << "Failed to get main audio duration" << endl;
        return 1;
    }

    // Construct FFmpeg command
    ostringstream cmd;
    cmd << "ffmpeg -y "
        << "-i " << escapePath(name_path) << " "
        << "-stream_loop -1 -i " << escapePath(bg_path) << " "
        << "-filter_complex \""
        << "[1:a]volume=-7dB[bg];"
        << "[bg]atrim=0:" << duration << "[bg_trim];"
        << "[0:a][bg_trim]amix=inputs=2:duration=first:normalize=0\" "
        << "-c:a libmp3lame -b:a 256k " // Added audio codec and bitrate settings
        << escapePath(output_path);

    cout << "Executing command:\n" << cmd.str() << endl;

    // Execute command
    int ret = system(cmd.str().c_str());
    if (ret != 0) {
        cerr << "FFmpeg execution failed, error code: " << ret << endl;
        return 1;
    }

    cout << "Merge completed. Output file: " << output_path << endl;
    return 0;
}
```
step3:去这个路径，C:\\Users\\wangrusheng\\Documents\\combineds.mp3，用播放器打开此音频，发现成功
end