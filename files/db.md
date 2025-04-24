说明：
c++实现登录页
1.新建一个窗口
2.两个输入框  user and password
3.一个按钮，开始登录
4.弹窗显示输入的  user and password
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/84c67a1f0939499c88521af9c49358c1.png#pic_center)

step0:C:\Users\wangrusheng\CLionProjects\untitled7\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.30)
project(untitled7)

set(CMAKE_CXX_STANDARD 20)

add_executable(untitled7 main.cpp)

# 添加以下链接选项
target_link_options(untitled7 PRIVATE -municode -Wl,-subsystem,windows)
```

step1:C:\Users\wangrusheng\CLionProjects\untitled7\main.cpp

```c

#include <Windows.h>

// 定义控件ID
#define ID_USER_EDIT    1001
#define ID_PASSWORD_EDIT 1002
#define ID_LOGIN_BUTTON 1003

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
        case WM_CREATE: {
            // 创建用户输入框
            CreateWindowW(
                L"EDIT", L"",
                WS_VISIBLE | WS_CHILD | WS_BORDER | ES_AUTOHSCROLL,
                50, 50, 200, 25,
                hwnd,
                (HMENU)ID_USER_EDIT,
                NULL, NULL
            );

            // 创建密码输入框（带星号显示）
            CreateWindowW(
                L"EDIT", L"",
                WS_VISIBLE | WS_CHILD | WS_BORDER | ES_PASSWORD,
                50, 85, 200, 25,
                hwnd,
                (HMENU)ID_PASSWORD_EDIT,
                NULL, NULL
            );

            // 创建登录按钮
            CreateWindowW(
                L"BUTTON", L"登录",
                WS_VISIBLE | WS_CHILD | BS_DEFPUSHBUTTON,
                50, 120, 80, 30,
                hwnd,
                (HMENU)ID_LOGIN_BUTTON,
                NULL, NULL
            );
            break;
        }
        case WM_COMMAND: {
            // 处理按钮点击事件
            if (LOWORD(wParam) == ID_LOGIN_BUTTON && HIWORD(wParam) == BN_CLICKED) {
                // 获取输入框句柄
                HWND hUser = GetDlgItem(hwnd, ID_USER_EDIT);
                HWND hPass = GetDlgItem(hwnd, ID_PASSWORD_EDIT);

                // 获取用户名
                int userLen = GetWindowTextLengthW(hUser) + 1;
                wchar_t* user = new wchar_t[userLen];
                GetWindowTextW(hUser, user, userLen);

                // 获取密码
                int passLen = GetWindowTextLengthW(hPass) + 1;
                wchar_t* password = new wchar_t[passLen];
                GetWindowTextW(hPass, password, passLen);

                // 输出到调试控制台
                OutputDebugStringW(L"用户名: ");
                OutputDebugStringW(user);
                OutputDebugStringW(L"\n密码: ");
                OutputDebugStringW(password);
                OutputDebugStringW(L"\n");

                // 弹出对话框显示输入内容
                wchar_t msg[256];
                wsprintfW(msg, L"用户名: %s\n密码: %s", user, password);
                MessageBoxW(hwnd, msg, L"登录信息", MB_OK);

                // 释放内存
                delete[] user;
                delete[] password;
            }
            break;
        }
        case WM_DESTROY: {
            PostQuitMessage(0);
            return 0;
        }
        default:
            return DefWindowProcW(hwnd, uMsg, wParam, lParam);
    }
    return 0;
}

int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PWSTR pCmdLine, int nCmdShow) {
    const wchar_t CLASS_NAME[] = L"LoginWindowClass";

    WNDCLASSEXW wc = {};
    wc.cbSize = sizeof(WNDCLASSEXW);
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);
    RegisterClassExW(&wc);

    HWND hwnd = CreateWindowExW(
        0,
        CLASS_NAME,
        L"登录界面",
        WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, CW_USEDEFAULT, 300, 250,
        NULL,
        NULL,
        hInstance,
        NULL
    );

    ShowWindow(hwnd, nCmdShow);
    UpdateWindow(hwnd);

    MSG msg;
    while (GetMessageW(&msg, NULL, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessageW(&msg);
    }

    return 0;
}
```

end