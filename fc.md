说明：
我计划用一个Android程序，打印出平板屏幕的尺寸，大小，dpi等参数信息
效果图：

```bash


分辨率: 1280x752
 DPI: 213
 物理尺寸(英寸): 对角线 9.4
```

step1:

```java
package com.example.myapplication;

import android.os.Bundle;
import android.util.DisplayMetrics;
import android.util.Log;
import androidx.appcompat.app.AppCompatActivity;

public class TestActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test);
        printScreenInfo();

    }

    private void printScreenInfo() {
        DisplayMetrics metrics = new DisplayMetrics();
        getWindowManager().getDefaultDisplay().getMetrics(metrics);

        // 分辨率
        int widthPixels = metrics.widthPixels;
        int heightPixels = metrics.heightPixels;

        // DPI（密度分级）
        int densityDpi = metrics.densityDpi;

        // 屏幕物理尺寸计算（英寸）
        float widthInches = (float) widthPixels / metrics.xdpi;
        float heightInches = (float) heightPixels / metrics.ydpi;
        float diagonalInches = (float) Math.sqrt(
                Math.pow(widthInches, 2) + Math.pow(heightInches, 2)
        );

        Log.e("ScreenInfo", "分辨率: " + widthPixels + "x" + heightPixels);
        Log.e("ScreenInfo", "DPI: " + densityDpi);
        Log.e("ScreenInfo", "物理尺寸(英寸): 对角线 " + String.format("%.1f", diagonalInches));
    }
}
```