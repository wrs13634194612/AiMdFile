说明：

c++使用gstreamer完成录制电脑桌面的功能
我希望用gstreamer录屏，默认10秒，自动保存录屏文件到本地
这里是不带声音的版本，仅录屏，
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
    // 设置插件路径（Windows 必需）
    g_setenv("GST_PLUGIN_PATH", R"(D:\Program Files\gstreamer\1.0\msvc_x86_64\lib\gstreamer-1.0)", TRUE);

    // 初始化 GStreamer
    gst_init(&argc, &argv);
    g_setenv("GST_DEBUG", "4", TRUE);  // 设置日志级别

    // 构建屏幕录制管道
    // 修改管道描述为：
    const gchar* pipeline_desc =
        "dx9screencapsrc ! "                  // Windows 屏幕捕获源
        "videoconvert ! "                     // 视频格式转换
        "videorate ! "                        // 帧率控制
        "video/x-raw,framerate=30/1 ! "       // 设置输出帧率
        "x264enc tune=zerolatency ! "         // H.264 编码
        "mp4mux ! "                           // MP4 封装
        "filesink location=\"C:/Users/wangrusheng/Desktop/screen_recording.mp4\""; // 输出路径

    GError* error = nullptr;
    GstElement* pipeline = gst_parse_launch(pipeline_desc, &error);

    if (error) {
        std::cerr << "管道创建失败: " << error->message << std::endl;
        g_clear_error(&error);
        return -1;
    }

    if (!pipeline) {
        std::cerr << "无法创建录制管道" << std::endl;
        return -1;
    }

    // 启动管道
    GstStateChangeReturn ret = gst_element_set_state(pipeline, GST_STATE_PLAYING);
    if (ret == GST_STATE_CHANGE_FAILURE) {
        std::cerr << "无法开始录制" << std::endl;
        gst_object_unref(pipeline);
        return -1;
    }

    std::cout << "开始屏幕录制（10秒）..." << std::endl;

    // 等待10秒（单位：微秒）
    g_usleep(10 * 1000000);

    // 发送EOS信号结束录制
    gst_element_send_event(pipeline, gst_event_new_eos());
    std::cout << "正在保存录屏文件..." << std::endl;

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
            std::cout << "录屏已保存至桌面 screen_recording.mp4" << std::endl;
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
////我是分割线
下面是带声音的录屏

step101:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeLists.txt

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

step102:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include <gst/gst.h>
#include <iostream>

int main(int argc, char* argv[]) {
    // 设置 GStreamer 插件路径（Windows 必需）
    g_setenv("GST_PLUGIN_PATH", R"(D:\Program Files\gstreamer\1.0\msvc_x86_64\lib\gstreamer-1.0)", TRUE);

    // 初始化 GStreamer
    gst_init(&argc, &argv);
    g_setenv("GST_DEBUG", "4", TRUE);  // 设置详细日志级别

    // 构建音视频录制管道
    const gchar* pipeline_desc =
        // 视频采集分支
        "dx9screencapsrc ! "                   // Windows 屏幕捕获源
        "videoconvert ! "                      // 视频格式转换
        "videorate ! "                         // 帧率控制
        "video/x-raw,framerate=30/1 ! "        // 设置输出帧率为30fps
        "x264enc tune=zerolatency ! "          // H.264视频编码
        "queue ! "                             // 视频缓冲队列
        "mux.video_0 "                         // 视频流入口到混合器

        // 音频采集分支
        "wasapisrc ! "                         // Windows音频输入
        "audioconvert ! "                      // 音频格式转换
        "audioresample ! "                     // 音频重采样
        "audio/x-raw,channels=2,rate=44100 ! " // 设置音频参数（立体声，44.1kHz）
        "voaacenc bitrate=128000 ! "          // AAC音频编码（128kbps）
        "queue ! "                             // 音频缓冲队列
        "mux.audio_0 "                         // 音频流入口到混合器

        // 混合输出
        "mp4mux name=mux ! "                   // 音视频混合器
        "filesink location=\"C:/Users/wangrusheng/Desktop/screen_recording.mp4\"";  // 输出路径

    // 创建管道
    GError* error = nullptr;
    GstElement* pipeline = gst_parse_launch(pipeline_desc, &error);

    if (error) {
        std::cerr << "管道创建失败: " << error->message << std::endl;
        g_clear_error(&error);
        return -1;
    }

    if (!pipeline) {
        std::cerr << "无法创建录制管道" << std::endl;
        return -1;
    }

    // 启动管道
    GstStateChangeReturn ret = gst_element_set_state(pipeline, GST_STATE_PLAYING);
    if (ret == GST_STATE_CHANGE_FAILURE) {
        std::cerr << "无法开始录制" << std::endl;
        gst_object_unref(pipeline);
        return -1;
    }

    std::cout << "开始同步录制屏幕和音频（10秒）..." << std::endl;

    // 等待录制时间（10秒）
    g_usleep(10 * 1000000);

    // 发送结束信号
    gst_element_send_event(pipeline, gst_event_new_eos());
    std::cout << "正在保存文件..." << std::endl;

    // 监听总线事件
    GstBus* bus = gst_element_get_bus(pipeline);
    GstMessage* msg = gst_bus_timed_pop_filtered(
        bus,
        GST_CLOCK_TIME_NONE,
        static_cast<GstMessageType>(GST_MESSAGE_ERROR | GST_MESSAGE_EOS)
    );

    // 处理事件
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
            std::cout << "文件已保存至：C:/Users/wangrusheng/Desktop/screen_recording.mp4" << std::endl;
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