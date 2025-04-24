c++将jpg转换为灰度图 

step1:添加依赖

```bash
下载这两个文件，放在cpp同一目录下，编译生成
https://github.com/nothings/stb/blob/master/stb_image_write.h
https://github.com/nothings/stb/blob/master/stb_image.h
```

step2:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include <vector>
#include <string>
#include <iostream>

// 必须在使用前定义这些宏
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"       // 图像加载库
#define STB_IMAGE_WRITE_IMPLEMENTATION
#include "stb_image_write.h" // 图像保存库

void convert_to_grayscale(const std::string& input_path,
    const std::string& output_path,
    int quality = 75)
{
    // 加载原始图像
    int width, height, channels;
    unsigned char* orig_data = stbi_load(input_path.c_str(), &width, &height, &channels, 0);

    if (!orig_data) {
        std::cerr << "错误：无法加载图像 " << input_path
            << "\n原因: " << stbi_failure_reason() << std::endl;
        return;
    }
    std::cout << "已加载图像: " << width << "x" << height
        << "，通道数: " << channels << std::endl;

    // 创建灰度数据缓冲区（单通道）
    std::vector<unsigned char> gray_data(width * height);

    // 灰度转换核心算法
    const float R_WEIGHT = 0.299f;
    const float G_WEIGHT = 0.587f;
    const float B_WEIGHT = 0.114f;

    for (int y = 0; y < height; ++y) {
        for (int x = 0; x < width; ++x) {
            // 计算当前像素的内存位置
            const int pixel_index = (y * width + x) * channels;

            // 获取RGB分量（自动兼容3/4通道图像）
            unsigned char r = orig_data[pixel_index];
            unsigned char g = orig_data[pixel_index + 1];
            unsigned char b = orig_data[pixel_index + 2];

            // 计算灰度值（加权平均法）
            gray_data[y * width + x] = static_cast<unsigned char>(
                R_WEIGHT * r +
                G_WEIGHT * g +
                B_WEIGHT * b
                );
        }
    }

    // 保存灰度图像（JPEG格式）
    if (!stbi_write_jpg(output_path.c_str(), width, height, 1, gray_data.data(), quality)) {
        std::cerr << "错误：无法保存图像到 " << output_path << std::endl;
    }

    // 释放原始图像内存
    stbi_image_free(orig_data);
}

int main() {
    // 输入输出路径（使用原始字符串避免转义）
    const std::string input_file = R"(C:\Users\wangrusheng\Downloads\red.jpg)";
    const std::string output_file = R"(C:\Users\wangrusheng\Downloads\new_gray.jpg)";

    // 执行转换并设置JPEG质量为85%
    convert_to_grayscale(input_file, output_file, 85);

    std::cout << "转换完成，请检查输出文件: " << output_file << std::endl;
    return 0;
}
```

end