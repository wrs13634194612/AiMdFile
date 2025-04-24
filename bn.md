说明：
我希望用gstreamer录音，默认10秒，自动保存录音文件到本地
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
    g_setenv("GST_DEBUG", "4", TRUE);  // 4=WARNING 及以上

    // 构建录音管道
    const gchar* pipeline_desc =
        "autoaudiosrc "                                   // 自动选择音频输入源
        "! audioconvert "                                 // 音频格式转换
        "! audioresample "                                // 重采样
        "! audio/x-raw,format=S16LE,channels=2,rate=44100"// 设置RAW格式
        "! wavenc "                                      // WAV编码
        "! filesink location=\"C:/Users/wangrusheng/Desktop/recording.wav\""; // 输出路径

    GError* error = nullptr;
    GstElement* pipeline = gst_parse_launch(pipeline_desc, &error);

    if (error) {
        std::cerr << "管道创建失败: " << error->message << std::endl;
        g_clear_error(&error);
        return -1;
    }

    if (!pipeline) {
        std::cerr << "无法创建录音管道" << std::endl;
        return -1;
    }

    // 启动管道
    GstStateChangeReturn ret = gst_element_set_state(pipeline, GST_STATE_PLAYING);
    if (ret == GST_STATE_CHANGE_FAILURE) {
        std::cerr << "无法启动录音" << std::endl;
        gst_object_unref(pipeline);
        return -1;
    }

    std::cout << "开始录音（10秒）..." << std::endl;

    // 等待10秒（单位：微秒）
    g_usleep(10 * 1000000);

    // 发送EOS信号结束录音
    gst_element_send_event(pipeline, gst_event_new_eos());
    std::cout << "正在保存录音文件..." << std::endl;

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
            std::cout << "录音已保存至桌面 recording.wav" << std::endl;
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