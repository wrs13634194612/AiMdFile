c++清理内存
1.内存状态监控	实时显示物理内存/备用内存使用情况
2.单进程内存清理	清理当前进程工作集内存
3.系统级内存清理	清理备用列表、已修改页、组合列表
4.全局进程优化	强制清理所有进程的工作集
5.权限管理	启用调试权限以执行敏感操作
6.用户交互	控制台菜单操作与实时监控模式
ps：清理内存需要管理员权限
去这个目录C:\Users\wangrusheng\source\repos\CMakeProject1\out\build\x64-debug\CMakeProject1，找到exe，用管理员打开。
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/eb2ef61b37b24e2eb5e0f9225898e38b.png#pic_center)

step1:cmake
C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.12)
project(CMakeProject1)

set(CMAKE_CXX_STANDARD 20)

find_library(Ntdll_Library NAMES ntdll)

add_executable(CMakeProject1
    CMakeProject1.cpp
    memory_cleaner.cpp
    memory_cleaner.h
)

target_link_libraries(CMakeProject1 
    Psapi
    Advapi32
    ${Ntdll_Library}
)

if(MSVC)
    target_compile_options(CMakeProject1 PRIVATE /W4 /WX)
else()
    target_compile_options(CMakeProject1 PRIVATE -Wall -Wextra -Werror)
endif()
```

step2: main.cpp
C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include "memory_cleaner.h"
#include <iostream>
#include <chrono>
#include <thread>
#include <conio.h>

using namespace std;
using namespace MemoryUtils;

void PrintMenu() {
    system("cls");
    cout << "==== 内存清理工具 ====\n";
    cout << "1. 显示当前内存状态\n";
    cout << "2. 执行内存清理\n";
    cout << "3. 实时监控内存（每秒刷新）\n";
    cout << "4. 退出\n";
    cout << "请选择操作: ";
}

void CleanMemory() {
    MEMORYSTATUSEX memBefore{ sizeof(MEMORYSTATUSEX) };
    GlobalMemoryStatusEx(&memBefore);

    if (!CleanAllProcessesWorkingSet()) {
        cerr << "清理工作集失败！\n";
        return;
    }
    CleanStandbyList();
    CleanModifiedPageList();

    MEMORYSTATUSEX memAfter{ sizeof(MEMORYSTATUSEX) };
    GlobalMemoryStatusEx(&memAfter);

    cout << "已清理内存: "
        << (memAfter.ullAvailPhys - memBefore.ullAvailPhys) / (1024 * 1024)
        << " MB\n";
}

void MonitorMemory() {
    while (!_kbhit()) {
        DisplayMemoryStatus();
        this_thread::sleep_for(1s);
    }
    _getch(); // 清除键盘缓冲区
}

int main() {
    if (!EnableDebugPrivilege()) {
        cerr << "警告: 无法获取调试权限，部分清理功能可能受限！\n";
    }

    while (true) {
        PrintMenu();

        switch (_getch()) {
        case '1':
            DisplayMemoryStatus();
            system("pause");
            break;
        case '2':
            CleanMemory();
            system("pause");
            break;
        case '3':
            MonitorMemory();
            break;
        case '4':
            return 0;
        default:
            cerr << "无效选项！\n";
            this_thread::sleep_for(1s);
        }
    }
}
```

step3: clear.h
C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\memory_cleaner.h

```cpp
#ifndef MEMORY_CLEANER_H
#define MEMORY_CLEANER_H

#include <windows.h>

// 定义 NTSTATUS 类型
#ifndef NTSTATUS
typedef LONG NTSTATUS;
#endif

// 定义 NT_SUCCESS 宏
#ifndef NT_SUCCESS
#define NT_SUCCESS(Status) (((NTSTATUS)(Status)) >= 0)
#endif

typedef enum _SYSTEM_MEMORY_LIST_COMMAND {
    MemoryPurgeStandbyList = 1,
    MemoryPurgeLowPriorityStandbyList,
    MemoryPurgeModifiedPageList,
    MemoryPurgeCombinedPageList
} SYSTEM_MEMORY_LIST_COMMAND;

#define SystemMemoryListInformation 80

#ifdef __cplusplus
extern "C" {
#endif

    NTSTATUS NTAPI NtSetSystemInformation(
        _In_ INT SystemInformationClass,
        _Inout_ PVOID SystemInformation,
        _In_ ULONG SystemInformationLength
    );

#ifdef __cplusplus
}
#endif

namespace MemoryUtils {
    void DisplayMemoryStatus();
    bool CleanWorkingSet();
    bool CleanStandbyList(bool clearLowPriority = false);
    bool CleanModifiedPageList();
    bool CleanCombinedPageList();
    bool CleanAllProcessesWorkingSet();
    bool EnableDebugPrivilege();
}

#endif // MEMORY_CLEANER_H
```

step4:clear.cpp
C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\memory_cleaner.cpp

```cpp
#include "memory_cleaner.h"
#include <iostream>
#include <psapi.h>
#include <tlhelp32.h>

using namespace std;

// 声明函数指针类型
typedef NTSTATUS(NTAPI* PNtSetSystemInformation)(
    _In_ INT SystemInformationClass,
    _Inout_ PVOID SystemInformation,
    _In_ ULONG SystemInformationLength
    );

// 全局函数指针
static PNtSetSystemInformation NtSetSystemInformationPtr = nullptr;

// 初始化函数指针
static bool InitializeNtFunction() {
    HMODULE hNtdll = GetModuleHandleW(L"ntdll.dll");
    if (hNtdll) {
        NtSetSystemInformationPtr = (PNtSetSystemInformation)GetProcAddress(hNtdll, "NtSetSystemInformation");
        return (NtSetSystemInformationPtr != nullptr);
    }
    return false;
}

bool MemoryUtils::EnableDebugPrivilege() {
    HANDLE hToken;
    if (!OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken))
        return false;

    TOKEN_PRIVILEGES tp;
    LUID luid;

    if (!LookupPrivilegeValue(nullptr, SE_DEBUG_NAME, &luid)) {
        CloseHandle(hToken);
        return false;
    }

    tp.PrivilegeCount = 1;
    tp.Privileges[0].Luid = luid;
    tp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;

    if (!AdjustTokenPrivileges(hToken, FALSE, &tp, sizeof(TOKEN_PRIVILEGES), nullptr, nullptr)) {
        CloseHandle(hToken);
        return false;
    }

    CloseHandle(hToken);
    return true;
}

void MemoryUtils::DisplayMemoryStatus() {
    MEMORYSTATUSEX memStatus;
    memStatus.dwLength = sizeof(MEMORYSTATUSEX);
    GlobalMemoryStatusEx(&memStatus);

    cout << "物理内存使用: "
        << (memStatus.ullTotalPhys - memStatus.ullAvailPhys) / (1024ULL * 1024 * 1024) << " GB / "
        << memStatus.ullTotalPhys / (1024ULL * 1024 * 1024) << " GB ("
        << memStatus.dwMemoryLoad << "%)\n";

    cout << "备用内存列表: "
        << memStatus.ullAvailExtendedVirtual / (1024ULL * 1024) << " MB\n";
}

bool MemoryUtils::CleanWorkingSet() {
    return EmptyWorkingSet(GetCurrentProcess());
}

bool MemoryUtils::CleanStandbyList(bool clearLowPriority) {
    if (!NtSetSystemInformationPtr && !InitializeNtFunction()) {
        return false;
    }

    SYSTEM_MEMORY_LIST_COMMAND command = clearLowPriority ?
        MemoryPurgeLowPriorityStandbyList : MemoryPurgeStandbyList;

    NTSTATUS status = NtSetSystemInformationPtr(
        SystemMemoryListInformation,
        &command,
        sizeof(SYSTEM_MEMORY_LIST_COMMAND)
    );
    return NT_SUCCESS(status);
}

bool MemoryUtils::CleanModifiedPageList() {
    if (!NtSetSystemInformationPtr && !InitializeNtFunction()) {
        return false;
    }

    SYSTEM_MEMORY_LIST_COMMAND command = MemoryPurgeModifiedPageList;
    NTSTATUS status = NtSetSystemInformationPtr(
        SystemMemoryListInformation,
        &command,
        sizeof(SYSTEM_MEMORY_LIST_COMMAND)
    );
    return NT_SUCCESS(status);
}

bool MemoryUtils::CleanCombinedPageList() {
    if (!NtSetSystemInformationPtr && !InitializeNtFunction()) {
        return false;
    }

    SYSTEM_MEMORY_LIST_COMMAND command = MemoryPurgeCombinedPageList;
    NTSTATUS status = NtSetSystemInformationPtr(
        SystemMemoryListInformation,
        &command,
        sizeof(SYSTEM_MEMORY_LIST_COMMAND)
    );
    return NT_SUCCESS(status);
}

bool MemoryUtils::CleanAllProcessesWorkingSet() {
    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hProcessSnap == INVALID_HANDLE_VALUE) return false;

    PROCESSENTRY32 pe32;
    pe32.dwSize = sizeof(PROCESSENTRY32);

    if (!Process32First(hProcessSnap, &pe32)) {
        CloseHandle(hProcessSnap);
        return false;
    }

    do {
        if (pe32.th32ProcessID == 0) continue;

        HANDLE hProcess = OpenProcess(
            PROCESS_QUERY_INFORMATION | PROCESS_SET_QUOTA | PROCESS_VM_OPERATION | PROCESS_VM_READ,
            FALSE,
            pe32.th32ProcessID);

        if (hProcess) {
            EmptyWorkingSet(hProcess);
            CloseHandle(hProcess);
        }
    } while (Process32Next(hProcessSnap, &pe32));

    CloseHandle(hProcessSnap);
    return true;
}
```
step5:选择全部重新生成
step6,直接运行
step7:关掉控制台，C:\Users\wangrusheng\source\repos\CMakeProject1\out\build\x64-debug\CMakeProject1
在这个目录，找到exe文件，管理员权限打开，输入1显示内存，输入2，清理内存
end