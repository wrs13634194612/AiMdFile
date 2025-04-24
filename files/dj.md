说明：Electron+nextjs+Electron_Forge跨端平台方案

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3c5f1f983bad4cb4aa2e32146f0aba51.png#pic_center)

step1:下载依赖

```bash
下载依赖
 npm install --save electron electron-squirrel-startup
 npm install --save-dev @electron-forge/cli      
 npm install --save-dev concurrently           
 npm install --save-dev @electron-forge/maker-zip     
 npm install --save-dev @electron-forge/maker-squirrel
 npm install --save-dev copyfiles wait-on 

打包前的预加载
npx electron-forge import          

打包
npm run build    
npm run package    

运行
npm run electron:start      
```

step2:配置启动项
C:\Users\wangrusheng\WebstormProjects\untitled10\package.json

```bash
{
  "name": "untitled10",
  "version": "0.1.0",
  "private": true,
  "main": "electron/main.js",
  "scripts": {
    "dev": "next dev --port 3000",
    "build": "next build && npm run postbuild",
    "postbuild": "copyfiles -u 1 src/app/*.html out/app/",
    "start": "electron-forge start",
    "lint": "next lint",
    "electron:start": "concurrently \"npm run dev\" \"electron .\"",
    "electron:build": "npm run build && electron-forge package",
    "package": "electron-forge package",
    "make": "electron-forge make"
  },
  "dependencies": {
    "electron-squirrel-startup": "^1.0.1",
    "next": "15.2.4",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@electron-forge/cli": "^7.8.0",
    "@electron-forge/maker-deb": "^7.8.0",
    "@electron-forge/maker-rpm": "^7.8.0",
    "@electron-forge/maker-squirrel": "^7.8.0",
    "@electron-forge/maker-zip": "^7.8.0",
    "@electron-forge/plugin-auto-unpack-natives": "^7.8.0",
    "@electron-forge/plugin-fuses": "^7.8.0",
    "@electron/fuses": "^1.8.0",
    "@eslint/eslintrc": "^3",
    "@tailwindcss/postcss": "^4",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "concurrently": "^9.1.2",
    "copyfiles": "^2.4.1",
    "electron": "^35.1.1",
    "eslint": "^9",
    "eslint-config-next": "15.2.4",
    "tailwindcss": "^4",
    "typescript": "^5",
    "wait-on": "^8.0.3"
  },
  "config": {
    "forge": {
      "packagerConfig": {},
      "makers": [
        {
          "name": "@electron-forge/maker-squirrel",
          "config": {
            "name": "untitled10"
          }
        },
        {
          "name": "@electron-forge/maker-zip",
          "platforms": [
            "darwin"
          ]
        }
      ]
    }
  }
}

```

step3:主进程，渲染ui，C:\Users\wangrusheng\WebstormProjects\untitled10\electron\main.js

```typescript
const { app, BrowserWindow } = require('electron')

let mainWindow

function createWindow() {
    mainWindow = new BrowserWindow({
        width: 1200,
        height: 800,
        webPreferences: {
            nodeIntegration: false,
            contextIsolation: true
        }
    })

    mainWindow.loadURL('http://localhost:4200')
    // mainWindow.webContents.openDevTools()


    // 开发环境加载 Next.js 开发服务器
    // if (process.env.NODE_ENV === 'development') {
    //     // 打开开发者工具
    // }
 /*   else {
        // 生产环境加载构建后的文件（需要先构建 Next.js）
        mainWindow.loadFile('next-app/out/index.html')
    }*/

    mainWindow.on('closed', () => {
        mainWindow = null
    })
}

app.whenReady().then(createWindow)

app.on('window-all-closed', () => {
    if (process.platform !== 'darwin') {
        app.quit()
    }
})

app.on('activate', () => {
    if (mainWindow === null) {
        createWindow()
    }
})
```

step4:C:\Users\wangrusheng\WebstormProjects\untitled10\electron\index.html，当你需要加载本地html的时候用这个，与之相配套的main.js代码

```typescript
const { app, BrowserWindow } = require('electron');
const path = require('path');

let mainWindow = null;

function createWindow() {
    mainWindow = new BrowserWindow({
        width: 800,
        height: 600,
        webPreferences: {
            nodeIntegration: true,
            contextIsolation: false,
            preload: path.join(__dirname, 'preload.js')
        }
    });

    // 直接加载本地 HTML 文件
    mainWindow.loadFile(path.join(__dirname, 'index.html')); // 确保此路径指向你的 HTML 文件
    
    // 开发工具开关（可选）
    // mainWindow.webContents.openDevTools();

    mainWindow.on('closed', () => {
        mainWindow = null;
    });
}

app.whenReady().then(createWindow);

app.on('window-all-closed', () => {
    if (process.platform !== 'darwin') {
        app.quit();
    }
});

app.on('activate', () => {
    if (mainWindow === null) {
        createWindow();
    }
});
<!DOCTYPE html>
<html>
<head>
    <title>Hello World</title>
</head>
<body>
<h1>Hello World!</h1>
</body>
</html>
```


step5:C:\Users\wangrusheng\WebstormProjects\untitled10\electron\preload.js

```typescript
window.addEventListener('DOMContentLoaded', () => {
    const replaceText = (selector, text) => {
        const element = document.getElementById(selector)
        if (element) element.innerText = text
    }

    for (const type of ['chrome', 'node', 'electron']) {
        replaceText(`${type}-version`, process.versions[type])
    }
})
```

step6:C:\Users\wangrusheng\WebstormProjects\untitled10\next.config.ts

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
    /* config options here */
    output: "export",  // 添加静态导出配置
    distDir: "out",    // 指定输出目录
    trailingSlash: true,
    images: {
        unoptimized: true
    }
};

export default nextConfig;

```

step7：运行，并打包

```bash
开发模式运行：

npm run electron:start

生产打包：


npm run build
npm run package

生成的可执行文件会在 /out/untitled10-win32-x64/ 目录下


详细路径 ：C:\Users\wangrusheng\WebstormProjects\untitled10\out\untitled10-win32-x64
```

end