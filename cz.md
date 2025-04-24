说明：
我希望用c++，写一个录音系统
1.点击按钮，开始录制音频
2.录制过程中，可以暂停和停止录制 有时长显示
3.点击停止录制 可以保存音频，保存在本地
4.找到刚刚保存的音频路径，可以点击播放 ，需要显示音频总时长

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/258f4f89305244d5ac4c27411f564545.png#pic_center)

step1:C:\Users\wangrusheng\CLionProjects\untitled8\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.20)
project(untitled8)

# 设置编译标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置可执行文件
add_executable(untitled8
        main.cpp
)

# Windows专用配置
if(WIN32)
    # 添加Unicode支持
    add_compile_definitions(UNICODE _UNICODE)

    # 链接库
    target_link_libraries(untitled8 PRIVATE
            winmm
            comctl32
    )

    # 链接选项
    target_link_options(untitled8 PRIVATE
            -municode
            -Wl,-subsystem,windows
    )
endif()
```

step2:C:\Users\wangrusheng\CLionProjects\untitled8\main.cpp

```cpp
 #include <Windows.h>
#include <mmsystem.h>
#include <commctrl.h>
#include <string>
#include <fstream>
#include <locale>
#include <codecvt>

#pragma comment(lib, "winmm.lib")
#pragma comment(lib, "comctl32.lib")

// 控件ID定义
#define ID_START      101
#define ID_PAUSE      102
#define ID_STOP       103
#define ID_PLAY       104
#define ID_TIMER      105

// 全局变量
HWAVEIN hWaveIn = nullptr;
WAVEFORMATEX wfx;
HWND hTimeDisplay, hPlayButton;
std::string savePath = "recorded_audio.wav";
DWORD recordStartTime = 0;
BOOL isRecording = FALSE;
BOOL isPaused = FALSE;
std::ofstream audioFile;

// 动态资源管理
WAVEHDR* waveHdr = nullptr;
char* audioBuffer = nullptr;
HWND hwndMain = nullptr;  // 主窗口句柄

// WAV文件头结构
#pragma pack(push, 1)
struct WAVHeader {
    char riff[4] = { 'R','I','F','F' };
    DWORD chunkSize;
    char wave[4] = { 'W','A','V','E' };
    char fmt[4] = { 'f','m','t',' ' };
    DWORD subchunk1Size = 16;
    WORD audioFormat = 1;
    WORD numChannels;
    DWORD sampleRate;
    DWORD byteRate;
    WORD blockAlign;
    WORD bitsPerSample;
    char data[4] = { 'd','a','t','a' };
    DWORD subchunk2Size;
};
#pragma pack(pop)

// 函数声明
void StartRecording(HWND hwnd);
void StopRecording();
void SaveAudioData(LPSTR data, DWORD length);
void UpdateTimeDisplay();
void PlayAudioFile();
void CleanupResources();

// 路径转换函数
std::wstring GetWidePath() {
    std::wstring_convert<std::codecvt_utf8_utf16<wchar_t>> converter;
    return converter.from_bytes(savePath);
}

LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
    switch (uMsg) {
    case WM_CREATE: {
        CreateWindowW(L"BUTTON", L"开始录音", WS_VISIBLE | WS_CHILD,
            50, 100, 100, 30, hwnd, (HMENU)ID_START, NULL, NULL);
        CreateWindowW(L"BUTTON", L"暂停", WS_VISIBLE | WS_CHILD,
            160, 100, 100, 30, hwnd, (HMENU)ID_PAUSE, NULL, NULL);
        CreateWindowW(L"BUTTON", L"停止", WS_VISIBLE | WS_CHILD,
            270, 100, 100, 30, hwnd, (HMENU)ID_STOP, NULL, NULL);
        hPlayButton = CreateWindowW(L"BUTTON", L"播放", WS_VISIBLE | WS_CHILD | WS_DISABLED,
            380, 100, 100, 30, hwnd, (HMENU)ID_PLAY, NULL, NULL);
        hTimeDisplay = CreateWindowW(L"STATIC", L"00:00", WS_VISIBLE | WS_CHILD,
            50, 150, 200, 30, hwnd, NULL, NULL, NULL);
        return 0;
    }
    case WM_COMMAND: {
        int wmId = LOWORD(wParam);
        switch (wmId) {
        case ID_START:
            if (!isRecording) StartRecording(hwnd);
            break;
        case ID_PAUSE:
            if (isRecording) {
                isPaused = !isPaused;
                if (isPaused) {
                    waveInStop(hWaveIn);
                } else {
                    waveInStart(hWaveIn);
                }
                SetWindowTextW((HWND)lParam, isPaused ? L"继续" : L"暂停");
            }
            break;
        case ID_STOP:
            if (isRecording) StopRecording();
            break;
        case ID_PLAY:
            PlayAudioFile();
            break;
        }
        return 0;
    }
    case WM_TIMER:
        if (wParam == ID_TIMER && isRecording && !isPaused) {
            UpdateTimeDisplay();
        }
        return 0;
    case MM_WIM_DATA: {
        LPWAVEHDR pWaveHdr = (LPWAVEHDR)lParam;
        if (isRecording && !isPaused) {
            SaveAudioData(pWaveHdr->lpData, pWaveHdr->dwBytesRecorded);
        }
        waveInAddBuffer(hWaveIn, pWaveHdr, sizeof(WAVEHDR));
        return 0;
    }
    case WM_DESTROY:
        StopRecording();  // 确保退出前释放资源
        PostQuitMessage(0);
        return 0;
    default:
        return DefWindowProcW(hwnd, uMsg, wParam, lParam);
    }
}

void StartRecording(HWND hwnd) {
    // 清理之前的录音
    if (hWaveIn != nullptr) {
        StopRecording();
    }

    // 初始化音频格式
    wfx.wFormatTag = WAVE_FORMAT_PCM;
    wfx.nChannels = 2;
    wfx.nSamplesPerSec = 44100;
    wfx.wBitsPerSample = 16;
    wfx.nBlockAlign = wfx.nChannels * wfx.wBitsPerSample / 8;
    wfx.nAvgBytesPerSec = wfx.nSamplesPerSec * wfx.nBlockAlign;

    // 打开录音设备
    if (waveInOpen(&hWaveIn, WAVE_MAPPER, &wfx, (DWORD_PTR)hwnd, 0, CALLBACK_WINDOW) != MMSYSERR_NOERROR) {
        MessageBoxW(hwnd, L"无法打开录音设备", L"错误", MB_OK);
        return;
    }

    // 初始化WAV文件头
    WAVHeader header;
    header.numChannels = wfx.nChannels;
    header.sampleRate = wfx.nSamplesPerSec;
    header.bitsPerSample = wfx.wBitsPerSample;
    header.byteRate = wfx.nAvgBytesPerSec;
    header.blockAlign = wfx.nBlockAlign;

    audioFile.open(savePath, std::ios::binary | std::ios::trunc);
    if (!audioFile.is_open()) {
        MessageBoxW(hwnd, L"无法创建音频文件", L"错误", MB_OK);
        CleanupResources();
        return;
    }
    audioFile.write(reinterpret_cast<char*>(&header), sizeof(header));

    // 分配缓冲区
    const DWORD bufferSize = 44100 * wfx.nBlockAlign;  // 1秒缓冲区
    audioBuffer = new char[bufferSize];
    waveHdr = new WAVEHDR();
    ZeroMemory(waveHdr, sizeof(WAVEHDR));
    waveHdr->lpData = audioBuffer;
    waveHdr->dwBufferLength = bufferSize;

    // 准备缓冲区
    if (waveInPrepareHeader(hWaveIn, waveHdr, sizeof(WAVEHDR)) != MMSYSERR_NOERROR) {
        MessageBoxW(hwnd, L"准备缓冲区失败", L"错误", MB_OK);
        CleanupResources();
        return;
    }

    if (waveInAddBuffer(hWaveIn, waveHdr, sizeof(WAVEHDR)) != MMSYSERR_NOERROR) {
        MessageBoxW(hwnd, L"添加缓冲区失败", L"错误", MB_OK);
        waveInUnprepareHeader(hWaveIn, waveHdr, sizeof(WAVEHDR));
        CleanupResources();
        return;
    }

    // 开始录音
    if (waveInStart(hWaveIn) != MMSYSERR_NOERROR) {
        MessageBoxW(hwnd, L"启动录音失败", L"错误", MB_OK);
        waveInUnprepareHeader(hWaveIn, waveHdr, sizeof(WAVEHDR));
        CleanupResources();
        return;
    }

    isRecording = TRUE;
    isPaused = FALSE;
    recordStartTime = GetTickCount();
    SetTimer(hwnd, ID_TIMER, 1000, NULL);
    EnableWindow(hPlayButton, FALSE);
}

void StopRecording() {
    if (!isRecording) return;

    KillTimer(hwndMain, ID_TIMER);
    waveInStop(hWaveIn);
    waveInReset(hWaveIn);

    if (waveHdr != nullptr) {
        waveInUnprepareHeader(hWaveIn, waveHdr, sizeof(WAVEHDR));
    }
    waveInClose(hWaveIn);
    hWaveIn = nullptr;

    // 更新WAV文件头
    audioFile.close();
    DWORD fileSize = 0;

    std::fstream file(savePath, std::ios::in | std::ios::out | std::ios::binary);
    if (file) {
        file.seekg(0, std::ios::end);
        fileSize = static_cast<DWORD>(file.tellg());
        file.seekg(0, std::ios::beg);

        WAVHeader header;
        file.read(reinterpret_cast<char*>(&header), sizeof(header));

        header.chunkSize = fileSize - 8;
        header.subchunk2Size = fileSize - sizeof(WAVHeader);

        file.seekp(0, std::ios::beg);
        file.write(reinterpret_cast<const char*>(&header), sizeof(header));
        file.close();
    }

    CleanupResources();
    EnableWindow(hPlayButton, TRUE);
    isRecording = FALSE;
    isPaused = FALSE;
}

void CleanupResources() {
    if (waveHdr != nullptr) {
        delete waveHdr;
        waveHdr = nullptr;
    }
    if (audioBuffer != nullptr) {
        delete[] audioBuffer;
        audioBuffer = nullptr;
    }
}

void SaveAudioData(LPSTR data, DWORD length) {
    if (audioFile.is_open()) {
        audioFile.write(data, length);
    }
}

void UpdateTimeDisplay() {
    DWORD elapsed = (GetTickCount() - recordStartTime) / 1000;
    std::wstring timeStr = std::to_wstring(elapsed / 60) + L":" +
                          (elapsed % 60 < 10 ? L"0" : L"") +
                          std::to_wstring(elapsed % 60);
    SetWindowTextW(hTimeDisplay, timeStr.c_str());
}

void PlayAudioFile() {
    std::wstring widePath = GetWidePath();
    PlaySoundW(widePath.c_str(), NULL, SND_FILENAME | SND_ASYNC);
}

int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, PWSTR pCmdLine, int nCmdShow) {
    const wchar_t CLASS_NAME[] = L"AudioRecorderClass";

    WNDCLASSEXW wc = { sizeof(WNDCLASSEXW) };
    wc.style = CS_HREDRAW | CS_VREDRAW;
    wc.lpfnWndProc = WindowProc;
    wc.hInstance = hInstance;
    wc.hCursor = LoadCursor(nullptr, IDC_ARROW);
    wc.hbrBackground = (HBRUSH)(COLOR_WINDOW + 1);
    wc.lpszClassName = CLASS_NAME;
    RegisterClassExW(&wc);

    hwndMain = CreateWindowExW(0, CLASS_NAME, L"音频录制系统",
        WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, 0, 600, 300,
        nullptr, nullptr, hInstance, nullptr);

    ShowWindow(hwndMain, nCmdShow);
    UpdateWindow(hwndMain);

    MSG msg;
    while (GetMessageW(&msg, nullptr, 0, 0)) {
        TranslateMessage(&msg);
        DispatchMessageW(&msg);
    }

    return 0;
}
```

step3: 生成的录音文件 本地路径

```bash
C:\Users\wangrusheng\CLionProjects\untitled8\cmake-build-debug\recorded_audio.wav
```

end