说明：
c++实现计算器
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/6ee8c6987b93411ca24875e4cf0c1659.png#pic_center)

step1:


```cpp
#include <Windows.h>
#include <string>

// 控件ID定义
#define IDC_0        100
#define IDC_1        101
#define IDC_2        102
#define IDC_3        103
#define IDC_4        104
#define IDC_5        105
#define IDC_6        106
#define IDC_7        107
#define IDC_8        108
#define IDC_9        109
#define IDC_ADD      110
#define IDC_SUBTRACT 111
#define IDC_MULTIPLY 112
#define IDC_DIVIDE   113
#define IDC_EQUALS   114
#define IDC_CLEAR    115
#define IDC_DECIMAL  116
#define IDC_DISPLAY  117

// 计算器状态
struct CalculatorState {
    double result = 0.0;
    double operand = 0.0;
    wchar_t operation = L'\0';
    bool newNumber = true;
    std::wstring display;
} state;

// 创建按钮控件
void CreateButton(HWND hwnd, const wchar_t* text, int x, int y, int width, int height, int id) {
    CreateWindowW(
        L"BUTTON", text,
        WS_VISIBLE | WS_CHILD | BS_PUSHBUTTON,
        x, y, width, height,
        hwnd, (HMENU)id, NULL, NULL
    );
}

// 更新显示
void UpdateDisplay(HWND hwnd) {
    SetWindowTextW(GetDlgItem(hwnd, IDC_DISPLAY), state.display.c_str());
}

// 执行计算
void PerformOperation(HWND hwnd) {
    switch (state.operation) {
        case L'+': state.result += state.operand; break;
        case L'-': state.result -= state.operand; break;
        case L'*': state.result *= state.operand; break;
        case L'/':
            if (state.operand != 0) {
                state.result /= state.operand;
            } else {
                state.display = L"Error";
                UpdateDisplay(hwnd);
                return;
            }
            break;
        default: state.result = state.operand;
    }

    state.display = std::to_wstring(state.result);
    state.operand = state.result;
    UpdateDisplay(hwnd);
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
        case WM_CREATE: {
            // 创建显示区域
            CreateWindowW(
                L"STATIC", L"0",
                WS_VISIBLE | WS_CHILD | SS_RIGHT,
                10, 10, 360, 40,
                hwnd, (HMENU)IDC_DISPLAY, NULL, NULL
            );

            // 创建按钮
            int btnWidth = 80, btnHeight = 60;
            CreateButton(hwnd, L"7", 10,  60, btnWidth, btnHeight, IDC_7);
            CreateButton(hwnd, L"8", 100, 60, btnWidth, btnHeight, IDC_8);
            CreateButton(hwnd, L"9", 190, 60, btnWidth, btnHeight, IDC_9);
            CreateButton(hwnd, L"/", 280, 60, btnWidth, btnHeight, IDC_DIVIDE);

            CreateButton(hwnd, L"4", 10,  130, btnWidth, btnHeight, IDC_4);
            CreateButton(hwnd, L"5", 100, 130, btnWidth, btnHeight, IDC_5);
            CreateButton(hwnd, L"6", 190, 130, btnWidth, btnHeight, IDC_6);
            CreateButton(hwnd, L"*", 280, 130, btnWidth, btnHeight, IDC_MULTIPLY);

            CreateButton(hwnd, L"1", 10,  200, btnWidth, btnHeight, IDC_1);
            CreateButton(hwnd, L"2", 100, 200, btnWidth, btnHeight, IDC_2);
            CreateButton(hwnd, L"3", 190, 200, btnWidth, btnHeight, IDC_3);
            CreateButton(hwnd, L"-", 280, 200, btnWidth, btnHeight, IDC_SUBTRACT);

            CreateButton(hwnd, L"0", 10,  270, btnWidth, btnHeight, IDC_0);
            CreateButton(hwnd, L".", 100, 270, btnWidth, btnHeight, IDC_DECIMAL);
            CreateButton(hwnd, L"=", 190, 270, btnWidth, btnHeight, IDC_EQUALS);
            CreateButton(hwnd, L"+", 280, 270, btnWidth, btnHeight, IDC_ADD);

            CreateButton(hwnd, L"C", 10, 340, 350, btnHeight, IDC_CLEAR);
            return 0;
        }

        case WM_COMMAND: {
            int controlID = LOWORD(wParam);

            // 处理数字按钮
            if (controlID >= IDC_0 && controlID <= IDC_9) {
                if (state.newNumber) {
                    state.display.clear();
                    state.newNumber = false;
                }
                wchar_t num = L'0' + (controlID - IDC_0);
                state.display += num;
                UpdateDisplay(hwnd);
            }
            // 处理小数点
            else if (controlID == IDC_DECIMAL) {
                if (state.display.find(L'.') == std::wstring::npos) {
                    state.display += L'.';
                    UpdateDisplay(hwnd);
                }
            }
            // 处理运算符
            else if (controlID >= IDC_ADD && controlID <= IDC_DIVIDE) {
                state.operand = std::wcstod(state.display.c_str(), nullptr);
                state.operation =
                    controlID == IDC_ADD ? L'+' :
                    controlID == IDC_SUBTRACT ? L'-' :
                    controlID == IDC_MULTIPLY ? L'*' : L'/';
                state.result = state.operand;
                state.newNumber = true;
            }
            // 处理等号
            else if (controlID == IDC_EQUALS) {
                state.operand = std::wcstod(state.display.c_str(), nullptr);
                PerformOperation(hwnd);
                state.newNumber = true;
            }
            // 清除
            else if (controlID == IDC_CLEAR) {
                state = CalculatorState();
                state.display = L"0";
                UpdateDisplay(hwnd);
            }
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
    const wchar_t CLASS_NAME[] = L"CalculatorClass";

    WNDCLASSEXW wc = {};
    wc.cbSize = sizeof(WNDCLASSEXW);
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.lpszClassName = CLASS_NAME;
    wc.hbrBackground = (HBRUSH)(COLOR_BTNFACE + 1);
    RegisterClassExW(&wc);

    HWND hwnd = CreateWindowExW(
        0, CLASS_NAME, L"Calculator",
        WS_OVERLAPPEDWINDOW & ~WS_THICKFRAME,
        CW_USEDEFAULT, CW_USEDEFAULT, 400, 480,
        NULL, NULL, hInstance, NULL
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