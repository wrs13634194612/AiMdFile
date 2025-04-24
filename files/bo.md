说明：
我希望用gstreamer将本地的mp3格式文件转换成wav格式

step0:去官网下载gstreamer开发时和运行时两个msi文件，安装选complete，然后添加环境变量 

```bash
环境变量path：
D:\Program Files\gstreamer\1.0\msvc_x86_64\lib
D:\Program Files\gstreamer\1.0\msvc_x86_64\bin
验证版本号：
C:\Users\wangrusheng>gst-launch-1.0.exe --version
gst-launch-1.0 version 1.26.0
GStreamer 1.26.0
Unknown package origin
查看安装位置：
C:\Users\wangrusheng>where gst-launch-1.0.exe
D:\Program Files\gstreamer\1.0\msvc_x86_64\bin\gst-launch-1.0.exe
===分割线
"filesrc location=\"C:/Users/wangrusheng/Desktop/name.mp3\" "  // 输入文件
"filesink location=\"C:/Users/wangrusheng/Desktop/output.wav\""; // 输出文件
```

step1:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.20)
project(CMakeProject1)

# 查找 GStreamer
find_package(PkgConfig REQUIRED)
pkg_check_modules(GSTREAMER REQUIRED gstreamer-1.0 gstreamer-app-1.0)

# 包含目录
include_directories(
    ${GSTREAMER_INCLUDE_DIRS}
)

# 链接目录
link_directories(
    ${GSTREAMER_LIBRARY_DIRS}
)

add_executable(CMakeProject1 CMakeProject1.cpp)

# 链接库
target_link_libraries(CMakeProject1
    ${GSTREAMER_LIBRARIES}
)

# C++ 标准设置
set_target_properties(CMakeProject1 PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
)
```

step2:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include <gst/gst.h>
#include <iostream>

int main(int argc, char* argv[]) {
    // 强制设置插件路径（仅 Windows 需要）
    g_setenv("GST_PLUGIN_PATH", R"(D:\Program Files\gstreamer\1.0\msvc_x86_64\lib\gstreamer-1.0)", TRUE);

    // 初始化 GStreamer
    gst_init(&argc, &argv);
    // 设置 GStreamer 调试级别（0=无日志，9=最详细）
    g_setenv("GST_DEBUG", "4", TRUE);  // 4=WARNING 及以上

    // 构建 GStreamer 管道描述字符串
    const gchar* pipeline_desc =
        "filesrc location=\"C:/Users/wangrusheng/Desktop/name.mp3\" "  // 输入文件
        "! mpegaudioparse "                                          // MP3解析
        "! avdec_mp3 "                                               // MP3解码
        "! audioconvert "                                            // 格式转换
        "! audioresample "                                           // 重采样
        "! audio/x-raw,format=S16LE,channels=2,rate=44100 "          // 设置RAW格式
        "! wavenc "                                                 // WAV编码
        "! filesink location=\"C:/Users/wangrusheng/Desktop/output.wav\""; // 输出文件

    GError* error = nullptr;

    // 创建管道
    GstElement* pipeline = gst_parse_launch(pipeline_desc, &error);

    if (error) {
        std::cerr << "管道创建失败: " << error->message << std::endl;
        g_clear_error(&error);
        return -1;
    }

    if (!pipeline) {
        std::cerr << "无法创建 GStreamer 管道" << std::endl;
        return -1;
    }

    // 启动管道
    GstStateChangeReturn ret = gst_element_set_state(pipeline, GST_STATE_PLAYING);
    if (ret == GST_STATE_CHANGE_FAILURE) {
        std::cerr << "无法启动管道" << std::endl;
        gst_object_unref(pipeline);
        return -1;
    }

    // 监听总线事件
    GstBus* bus = gst_element_get_bus(pipeline);
    GstMessage* msg = gst_bus_timed_pop_filtered(
        bus,
        GST_CLOCK_TIME_NONE,
        static_cast<GstMessageType>(GST_MESSAGE_ERROR | GST_MESSAGE_EOS)
    );

    // 处理消息
    if (msg != nullptr) {
        switch (GST_MESSAGE_TYPE(msg)) {
        case GST_MESSAGE_ERROR: {
            GError* err = nullptr;
            gchar* debug = nullptr;
            gst_message_parse_error(msg, &err, &debug);
            std::cerr << "错误: " << err->message << std::endl;
            if (debug) std::cerr << "调试信息: " << debug << std::endl;
            g_clear_error(&err);
            g_free(debug);
            break;
        }
        case GST_MESSAGE_EOS:
            std::cout << "转换完成" << std::endl;
            break;
        default:
            break;
        }
        gst_message_unref(msg);
    }

    // 清理资源
    gst_object_unref(bus);
    gst_element_set_state(pipeline, GST_STATE_NULL);
    gst_object_unref(pipeline);

    return 0;
}
```

end