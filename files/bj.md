说明：
我希望在visual studil 2022项目中集成SFML,创建一个示例程序

step1:visual_studio_2022 创建新项目

```bash
文件 
	新建
		 项目
			c++
				空项目	
					创建 
```

step2:创建cpp

```bash
选中 project5
	右键 
		添加
			类
				输入main
					点击确定
 
```

step3:下载SFML

```bash
下载SFML https://www.sfml-dev.org/download/
	选择3.0.0
		选择64-bit
			Visual C++ 17 (2022) - Download | 37 MB

下载完成，解压后，放在D:\Program Files\SFML-3.0.0\bin

```

step4:配置

```bash
配置1：
project5
	右键
		属性
			选择顶部 所有配置
				c/c++
					语言
						c++语言标准  选择 ISO C++ 17 标准(/std:c++17)

完成

///我是分割线

配置2：
project5
	右键
		属性
			选择顶部 所有配置
				c/c++
					常规
						附加包含目录 
							编辑
								D:\Program Files\SFML-3.0.0\include
								点击确定

完成

///我是分割线
配置3：
project5
	右键
		属性
			选择顶部 所有配置
				链接器
					常规
						附加库目录
							编辑
								D:\Program Files\SFML-3.0.0\lib
								确定

完成

///我是分割线

配置4：
project5
	右键
		属性
			选择顶部 所有配置
				链接器
					输入
						附加依赖项
							编辑
								sfml-graphics-d.lib
								sfml-window-d.lib
								sfml-system-d.lib
								sfml-audio-d.lib
								sfml-network-d.lib
									确定
完成
```

step5:C:\Users\wangrusheng\source\repos\Project5\Project5\main.h

```cpp
#include <SFML/Graphics.hpp>

int main()
{
    sf::RenderWindow window(sf::VideoMode({ 200, 200 }), "SFML works!");
    sf::CircleShape shape(100.f);
    shape.setFillColor(sf::Color::Green);

    while (window.isOpen())
    {
        while (const std::optional event = window.pollEvent())
        {
            if (event->is<sf::Event::Closed>())
                window.close();
        }

        window.clear();
        window.draw(shape);
        window.display();
    }
}
```

step6:生成，生成解决方案

```bash
生成开始于 20:24...
1>------ 已启动生成: 项目: Project5, 配置: Debug x64 ------
1>main.cpp
1>D:\Program Files\SFML-3.0.0\include\SFML\System\Exception.hpp(41,47): warning C4275: 非 dll 接口 class“std::runtime_error”用作 dll 接口 class“sf::Exception”的基
1>(编译源文件“main.cpp”)
1>    D:\Program Files\Microsoft Visual Studio\2022\Professional\VC\Tools\MSVC\14.43.34808\include\stdexcept(100,19):
1>    参见“std::runtime_error”的声明
1>    D:\Program Files\SFML-3.0.0\include\SFML\System\Exception.hpp(41,23):
1>    参见“sf::Exception”的声明
1>Project5.vcxproj -> C:\Users\wangrusheng\source\repos\Project5\x64\Debug\Project5.exe
1>已完成生成项目“Project5.vcxproj”的操作。
========== 生成: 1 成功，0 失败，0 最新，0 已跳过 ==========
========== 生成 于 20:24 完成，耗时 07.521 秒 ==========
```

step7:累计10个dll文件

```bash
将这个目录D:\Program Files\SFML-3.0.0\bin中的所有dll文件，
复制到C:\Users\wangrusheng\source\repos\Project5\x64\Debug
```

step8:点击运行，看到弹窗，展示绿色圆球，表示成功

end