说明：
c++实现GUI窗口显示helloworld
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e3db810f923c455ea1bc8758dd422113.png#pic_center)

step1:使用clion创建c++项目

```bash
1.new
2.project
3.select c++ executable
4.language standard:C++20
5.勾选 add onboarding tips
6.点击create
```

step2:C:\Users\wangrusheng\CLionProjects\untitled7\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.30)
project(untitled7)

set(CMAKE_CXX_STANDARD 20)

add_executable(untitled7 main.cpp)

# 添加以下链接选项
target_link_options(untitled7 PRIVATE -municode -Wl,-subsystem,windows)
```

step3:C:\Users\wangrusheng\CLionProjects\untitled7\main.cpp

```cpp
#include <Windows.h>

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
        case WM_PAINT: {
            // 处理窗口绘制消息
            PAINTSTRUCT ps;
            HDC hdc = BeginPaint(hwnd, &ps);

            // 设置白色背景
            FillRect(hdc, &ps.rcPaint, (HBRUSH)(COLOR_WINDOW + 1));

            // 输出文本
            TextOutW(hdc, 50, 50, L"Hello World!", 12);
            EndPaint(hwnd, &ps);
            return 0;
        }
        case WM_DESTROY:
            PostQuitMessage(0);
        return 0;
        default:
            return DefWindowProcW(hwnd, uMsg, wParam, lParam);
    }
}

int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PWSTR pCmdLine, int nCmdShow) {
    const wchar_t CLASS_NAME[] = L"MyWindowClass";

    // 注册窗口类时设置背景色
    WNDCLASSEXW wc = {};
    wc.cbSize = sizeof(WNDCLASSEXW);
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);  // 设置窗口背景色
    RegisterClassExW(&wc);

    // 创建窗口
    HWND hwnd = CreateWindowExW(
        0,
        CLASS_NAME,
        L"Hello World Window",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, 800, 600,
        NULL,
        NULL,
        hInstance,
        NULL
    );

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);  // 强制立即重绘窗口

    // 消息循环
    MSG msg;
    while (GetMessageW(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessageW(&msg);
    }

    return 0;
}
```

end