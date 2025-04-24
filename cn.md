说明：
c++图片压缩
step1:下载依赖 ， stb_image.h和stb_image_write.h

```bash
# 在项目目录打开PowerShell执行：
curl -Uri "https://raw.githubusercontent.com/nothings/stb/master/stb_image.h" -OutFile "stb_image.h"
curl -Uri "https://raw.githubusercontent.com/nothings/stb/master/stb_image_write.h" -OutFile "stb_image_write.h"
```

step2:下载后请将这两个文件与您的.cpp文件放在同一目录，并确保在代码中包含正确的宏定义：



step3:C:\Users\wangrusheng\CLionProjects\untitled27\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.30)
project(untitled27)

set(CMAKE_CXX_STANDARD 20)

# 添加当前目录到头文件路径
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_executable(untitled27
        main.cpp
        stb_image.h
        stb_image_write.h)
```

step4:C:\Users\wangrusheng\CLionProjects\untitled27\main.cpp

```cpp
#include <iostream>
#include <cstdio>
#include <string>

#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
#define STB_IMAGE_WRITE_IMPLEMENTATION
#include "stb_image_write.h"

bool compress_image(const std::string& input_path, const std::string& output_path, int quality = 80) {
    // 加载原始图片
    int width, height, channels;
    unsigned char* data = stbi_load(input_path.c_str(), &width, &height, &channels, 0);

    if (!data) {
        std::cerr << "Error loading image: " << stbi_failure_reason() << std::endl;
        return false;
    }

    // 如果原始图片有Alpha通道，转换为RGB
    if (channels == 4) {
        unsigned char* rgb_data = new unsigned char[width * height * 3];
        for (int i = 0; i < width * height; ++i) {
            rgb_data[i*3]   = data[i*4];
            rgb_data[i*3+1] = data[i*4+1];
            rgb_data[i*3+2] = data[i*4+2];
        }
        stbi_image_free(data);
        data = rgb_data;
        channels = 3;
    }

    // 保存为JPEG（压缩）
    int success = stbi_write_jpg(output_path.c_str(), width, height, channels, data, quality);

    // 清理内存
    if (channels == 4) {
        delete[] data;
    } else {
        stbi_image_free(data);
    }

    if (!success) {
        std::cerr << "Error writing compressed image" << std::endl;
        return false;
    }

    return true;
}


int main() {
    std::string input_path = R"(C:\Users\wangrusheng\Downloads\map.jpg)";
    std::string output_path = R"(C:\Users\wangrusheng\Desktop\compressed.jpg)";

    if (compress_image(input_path, output_path, 30)) {
        std::cout << "Image compressed successfully!" << std::endl;
        std::cout << "Output file: " << output_path << std::endl;
    } else {
        std::cerr << "Failed to compress image" << std::endl;
    }

    return 0;
}
```

end