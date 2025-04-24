说明：
opus+ffmpeg+c++实现录音
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a6e2042b2be8467a85e61ecff06fc6b9.png#pic_center)

step1:C:\Users\wangrusheng\source\repos\WindowsProject1\WindowsProject1\WindowsProject1.cpp

```cpp
// WindowsProject1.cpp : 定义应用程序的入口点。
//

#include "framework.h"
#include "WindowsProject1.h"
#include <commctrl.h>
#include <shlwapi.h>
#include <process.h>
#include <string>

#pragma comment(lib, "shlwapi.lib")

#define MAX_LOADSTRING 100
#define IDC_BTN_START   1001
#define IDC_BTN_STOP    1002
#define IDC_STATUS      1003
#define IDC_PATH_LABEL  1004

// 全局变量:
HINSTANCE hInst;                                // 当前实例
WCHAR szTitle[MAX_LOADSTRING];                  // 标题栏文本
WCHAR szWindowClass[MAX_LOADSTRING];            // 主窗口类名

// FFmpeg配置         private readonly string ffmpegPath = @"C:\Users\wangrusheng\AppData\Local\Microsoft\WinGet\Links\ffmpeg.exe";
const wchar_t* ffmpegPath = L"C:\\Users\\wangrusheng\\AppData\\Local\\Microsoft\\WinGet\\Links\\ffmpeg.exe";
const wchar_t* audioDevice = L"audio=@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\\wave_{2D7B50BB-AD6C-4823-8F7F-5552B9D873B9}";
const wchar_t* outputFile = L"recording.opus";
// 存储的录音本地文件路径 C:\Users\wangrusheng\source\repos\WindowsProject1\WindowsProject1\recording.opus
// 进程和管道句柄
HANDLE hFFmpegProcess = NULL;
HANDLE hInputWrite = NULL;

// 函数声明
ATOM                MyRegisterClass(HINSTANCE hInstance);
BOOL                InitInstance(HINSTANCE, int);
LRESULT CALLBACK    WndProc(HWND, UINT, WPARAM, LPARAM);
INT_PTR CALLBACK    About(HWND, UINT, WPARAM, LPARAM);
void                StartRecording(HWND hWnd);
void                StopRecording();
void                UpdateStatus(HWND hWnd, const wchar_t* text);
void                EnableControls(HWND hWnd, BOOL start, BOOL stop);
DWORD WINAPI        ReadErrorThread(LPVOID lpParam);

int APIENTRY wWinMain(_In_ HINSTANCE hInstance,
    _In_opt_ HINSTANCE hPrevInstance,
    _In_ LPWSTR    lpCmdLine,
    _In_ int       nCmdShow)
{
    UNREFERENCED_PARAMETER(hPrevInstance);
    UNREFERENCED_PARAMETER(lpCmdLine);

    // 初始化全局字符串
    LoadStringW(hInstance, IDS_APP_TITLE, szTitle, MAX_LOADSTRING);
    LoadStringW(hInstance, IDC_WINDOWSPROJECT1, szWindowClass, MAX_LOADSTRING);
    MyRegisterClass(hInstance);

    // 执行应用程序初始化
    if (!InitInstance(hInstance, nCmdShow))
    {
        return FALSE;
    }

    HACCEL hAccelTable = LoadAccelerators(hInstance, MAKEINTRESOURCE(IDC_WINDOWSPROJECT1));

    MSG msg;

    // 主消息循环
    while (GetMessage(&msg, nullptr, 0, 0))
    {
        if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
        {
            TranslateMessage(&msg);
            DispatchMessage(&msg);
        }
    }

    return (int)msg.wParam;
}

ATOM MyRegisterClass(HINSTANCE hInstance)
{
    WNDCLASSEXW wcex;

    wcex.cbSize = sizeof(WNDCLASSEX);

    wcex.style = CS_HREDRAW | CS_VREDRAW;
    wcex.lpfnWndProc = WndProc;
    wcex.cbClsExtra = 0;
    wcex.cbWndExtra = 0;
    wcex.hInstance = hInstance;
    wcex.hIcon = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_WINDOWSPROJECT1));
    wcex.hCursor = LoadCursor(nullptr, IDC_ARROW);
    wcex.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);
    wcex.lpszMenuName = MAKEINTRESOURCEW(IDC_WINDOWSPROJECT1);
    wcex.lpszClassName = szWindowClass;
    wcex.hIconSm = LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));

    return RegisterClassExW(&wcex);
}

BOOL InitInstance(HINSTANCE hInstance, int nCmdShow)
{
    hInst = hInstance;

    HWND hWnd = CreateWindowW(szWindowClass, szTitle, WS_OVERLAPPEDWINDOW,
        CW_USEDEFAULT, 0, 640, 240, nullptr, nullptr, hInstance, nullptr);

    if (!hWnd)
    {
        return FALSE;
    }

    ShowWindow(hWnd, nCmdShow);
    UpdateWindow(hWnd);

    return TRUE;
}

LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    switch (message)
    {
    case WM_CREATE:
    {
        // 创建界面控件
        CreateWindowW(L"STATIC", L"准备就绪",
            WS_CHILD | WS_VISIBLE,
            20, 20, 260, 20, hWnd,
            (HMENU)IDC_STATUS, hInst, NULL);

        CreateWindowW(L"BUTTON", L"开始录音",
            WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON,
            20, 60, 100, 40, hWnd,
            (HMENU)IDC_BTN_START, hInst, NULL);

        CreateWindowW(L"BUTTON", L"停止录音",
            WS_CHILD | WS_VISIBLE | WS_DISABLED | BS_PUSHBUTTON,
            140, 60, 100, 40, hWnd,
            (HMENU)IDC_BTN_STOP, hInst, NULL);

        // 显示保存路径
        wchar_t pathText[MAX_PATH];
        GetCurrentDirectoryW(MAX_PATH, pathText);
        PathCombineW(pathText, pathText, outputFile);

        CreateWindowW(L"STATIC", pathText,
            WS_CHILD | WS_VISIBLE,
            20,120, 600, 20, hWnd,
            (HMENU)IDC_PATH_LABEL, hInst, NULL);

        // 检查FFmpeg是否存在
        if (!PathFileExistsW(ffmpegPath))
        {
            MessageBoxW(hWnd,
                L"FFmpeg未找到，请检查路径配置",
                L"错误", MB_OK | MB_ICONERROR);
            EnableWindow(GetDlgItem(hWnd, IDC_BTN_START), FALSE);
        }
    }
    break;
    case WM_COMMAND:
    {
        int wmId = LOWORD(wParam);
        switch (wmId)
        {
        case IDC_BTN_START:
            StartRecording(hWnd);
            break;
        case IDC_BTN_STOP:
            StopRecording();
            UpdateStatus(hWnd, L"录音已停止");
            EnableControls(hWnd, TRUE, FALSE);
            break;
        case IDM_ABOUT:
            DialogBox(hInst, MAKEINTRESOURCE(IDD_ABOUTBOX), hWnd, About);
            break;
        case IDM_EXIT:
            DestroyWindow(hWnd);
            break;
        default:
            return DefWindowProc(hWnd, message, wParam, lParam);
        }
    }
    break;
    case WM_DESTROY:
        if (hFFmpegProcess != NULL)
        {
            TerminateProcess(hFFmpegProcess, 0);
            CloseHandle(hFFmpegProcess);
        }
        PostQuitMessage(0);
        break;
    default:
        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}

void StartRecording(HWND hWnd)
{
    if (hFFmpegProcess != NULL) return;

    SECURITY_ATTRIBUTES saAttr;
    saAttr.nLength = sizeof(SECURITY_ATTRIBUTES);
    saAttr.bInheritHandle = TRUE;
    saAttr.lpSecurityDescriptor = NULL;

    // 创建输入管道
    HANDLE hInputRead = NULL;
    if (!CreatePipe(&hInputRead, &hInputWrite, &saAttr, 0))
    {
        MessageBoxW(hWnd, L"创建管道失败", L"错误", MB_OK | MB_ICONERROR);
        return;
    }

    // 配置进程参数
    STARTUPINFOW si = { sizeof(STARTUPINFOW) };
    PROCESS_INFORMATION pi = { 0 };
    si.dwFlags = STARTF_USESTDHANDLES;
    si.hStdInput = hInputRead;
    si.hStdOutput = GetStdHandle(STD_OUTPUT_HANDLE);
    si.hStdError = GetStdHandle(STD_ERROR_HANDLE);

    // 构建命令行
    wchar_t cmdLine[1024];
    swprintf_s(cmdLine,
        L"\"%s\" -f dshow -i \"%s\" -c:a libopus -b:a 64k -y \"%s\"",
        ffmpegPath, audioDevice, outputFile);

    // 启动FFmpeg进程
    if (CreateProcessW(
        NULL,                   // 应用程序名称
        cmdLine,                // 命令行
        NULL,                   // 进程安全属性
        NULL,                   // 线程安全属性
        TRUE,                   // 继承句柄
        CREATE_NO_WINDOW,       // 创建标志
        NULL,                   // 环境变量
        NULL,                   // 当前目录
        &si,                   // 启动信息
        &pi))                   // 进程信息
    {
        CloseHandle(pi.hThread);
        hFFmpegProcess = pi.hProcess;
        CloseHandle(hInputRead);

        // 启动错误读取线程
        HANDLE hThread = CreateThread(
            NULL,
            0,
            ReadErrorThread,
            NULL,
            0,
            NULL);
        CloseHandle(hThread);

        UpdateStatus(hWnd, L"录音进行中...");
        EnableControls(hWnd, FALSE, TRUE);
    }
    else
    {
        DWORD err = GetLastError();
        wchar_t errMsg[256];
        swprintf_s(errMsg, L"启动失败 (错误码: 0x%08X)", err);
        MessageBoxW(hWnd, errMsg, L"错误", MB_OK | MB_ICONERROR);

        CloseHandle(hInputRead);
        CloseHandle(hInputWrite);
        hInputWrite = NULL;
    }
}

void StopRecording()
{
    if (hFFmpegProcess != NULL && hInputWrite != NULL)
    {
        // 发送停止命令
        DWORD bytesWritten;
        WriteFile(hInputWrite, "q\n", 2, &bytesWritten, NULL);

        // 等待进程退出
        if (WaitForSingleObject(hFFmpegProcess, 1500) == WAIT_TIMEOUT)
        {
            TerminateProcess(hFFmpegProcess, 0);
        }

        // 清理资源
        CloseHandle(hFFmpegProcess);
        CloseHandle(hInputWrite);
        hFFmpegProcess = NULL;
        hInputWrite = NULL;
    }
}

DWORD WINAPI ReadErrorThread(LPVOID lpParam)
{
    HANDLE hStdError = GetStdHandle(STD_ERROR_HANDLE);
    const DWORD BUFF_SIZE = 4096;
    CHAR buffer[BUFF_SIZE];
    DWORD bytesRead;

    while (true)
    {
        if (!ReadFile(hStdError, buffer, BUFF_SIZE - 1, &bytesRead, NULL) || bytesRead == 0)
            break;

        buffer[bytesRead] = '\0';
        OutputDebugStringA(buffer);
    }

    return 0;
}

void UpdateStatus(HWND hWnd, const wchar_t* text)
{
    SetWindowTextW(GetDlgItem(hWnd, IDC_STATUS), text);
}

void EnableControls(HWND hWnd, BOOL start, BOOL stop)
{
    EnableWindow(GetDlgItem(hWnd, IDC_BTN_START), start);
    EnableWindow(GetDlgItem(hWnd, IDC_BTN_STOP), stop);
}

// 关于对话框
INT_PTR CALLBACK About(HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)
{
    UNREFERENCED_PARAMETER(lParam);
    switch (message)
    {
    case WM_INITDIALOG:
        return (INT_PTR)TRUE;

    case WM_COMMAND:
        if (LOWORD(wParam) == IDOK || LOWORD(wParam) == IDCANCEL)
        {
            EndDialog(hDlg, LOWORD(wParam));
            return (INT_PTR)TRUE;
        }
        break;
    }
    return (INT_PTR)FALSE;
}
```

end