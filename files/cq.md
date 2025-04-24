说明:
C++实现屏幕截图
效果图：存储图片的本地路径

```bash
C:\Users\wangrusheng\CLionProjects\untitled23\cmake-build-debug\screenshot.bmp
```

step1:C:\Users\wangrusheng\CLionProjects\untitled23\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.30)
project(untitled23)

set(CMAKE_CXX_STANDARD 20)

add_executable(untitled23 main.cpp)

# 添加Windows API链接库
target_link_libraries(untitled23 gdi32 user32)
```

step2:C:\Users\wangrusheng\CLionProjects\untitled23\main.cpp

```cpp
#include <iostream>
#include <windows.h>
#include <fstream>

bool SaveBitmapToFile(HBITMAP hBitmap, const char* filename) {
    BITMAP bmp;
    GetObject(hBitmap, sizeof(BITMAP), &bmp);

    BITMAPFILEHEADER bmfHeader;
    BITMAPINFOHEADER bi;

    bi.biSize = sizeof(BITMAPINFOHEADER);
    bi.biWidth = bmp.bmWidth;
    bi.biHeight = bmp.bmHeight;
    bi.biPlanes = 1;
    bi.biBitCount = 24;  // 24位位图
    bi.biCompression = BI_RGB;
    bi.biSizeImage = 0;
    bi.biXPelsPerMeter = 0;
    bi.biYPelsPerMeter = 0;
    bi.biClrUsed = 0;
    bi.biClrImportant = 0;

    DWORD dwBmpSize = ((bmp.bmWidth * bi.biBitCount + 31) / 32) * 4 * bmp.bmHeight;
    HANDLE hDIB = GlobalAlloc(GHND, dwBmpSize);
    char* lpbitmap = (char*)GlobalLock(hDIB);

    HDC hDC = GetDC(NULL);
    GetDIBits(hDC, hBitmap, 0, (UINT)bmp.bmHeight, lpbitmap, (BITMAPINFO*)&bi, DIB_RGB_COLORS);
    ReleaseDC(NULL, hDC);

    std::ofstream file(filename, std::ios::binary);
    if (!file) {
        GlobalUnlock(hDIB);
        GlobalFree(hDIB);
        return false;
    }

    // 设置位图文件头
    bmfHeader.bfType = 0x4D42;  // "BM"
    bmfHeader.bfSize = sizeof(BITMAPFILEHEADER) + sizeof(BITMAPINFOHEADER) + dwBmpSize;
    bmfHeader.bfReserved1 = 0;
    bmfHeader.bfReserved2 = 0;
    bmfHeader.bfOffBits = sizeof(BITMAPFILEHEADER) + sizeof(BITMAPINFOHEADER);

    file.write((char*)&bmfHeader, sizeof(BITMAPFILEHEADER));
    file.write((char*)&bi, sizeof(BITMAPINFOHEADER));
    file.write(lpbitmap, dwBmpSize);

    GlobalUnlock(hDIB);
    GlobalFree(hDIB);
    file.close();
    return true;
}

void CaptureScreen(const char* filename) {
    HDC hScreenDC = CreateDC("DISPLAY", NULL, NULL, NULL);
    HDC hMemoryDC = CreateCompatibleDC(hScreenDC);

    int width = GetDeviceCaps(hScreenDC, HORZRES);
    int height = GetDeviceCaps(hScreenDC, VERTRES);

    HBITMAP hBitmap = CreateCompatibleBitmap(hScreenDC, width, height);
    HBITMAP hOldBitmap = (HBITMAP)SelectObject(hMemoryDC, hBitmap);

    BitBlt(hMemoryDC, 0, 0, width, height, hScreenDC, 0, 0, SRCCOPY);
    hBitmap = (HBITMAP)SelectObject(hMemoryDC, hOldBitmap);

    SaveBitmapToFile(hBitmap, filename);

    DeleteDC(hMemoryDC);
    DeleteDC(hScreenDC);
    DeleteObject(hBitmap);
}

int main() {
    CaptureScreen("screenshot.bmp");
    std::cout << "Screenshot saved to screenshot.bmp" << std::endl;
    return 0;
}
```

end