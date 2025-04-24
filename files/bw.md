说明：
python+form+opengl显示动态图形数据
我希望做一款动态opengl图形数据
1.用python脚本，输入指定参数
2.生成一组数据，
3.将数据保持成本地文件
4.在c#中调用此文件，解析
5.将数据用opengl展示


效果图:
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5fea36a3b4a143bd8c1f987990c5c8dc.png#pic_center)

step1:添加依赖  
C:\Users\wangrusheng\RiderProjects\WinFormsApp8\WinFormsApp8\WinFormsApp8.csproj

```bash
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>net9.0-windows</TargetFramework>
        <Nullable>enable</Nullable>
        <UseWindowsForms>true</UseWindowsForms>
        <ImplicitUsings>enable</ImplicitUsings>
        <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    </PropertyGroup>

    <ItemGroup>
      <PackageReference Include="ncalc" Version="1.3.8" />
      <PackageReference Include="Silk.NET.Core" Version="2.22.0" />
      <PackageReference Include="Silk.NET.GLFW" Version="2.22.0" />
      <PackageReference Include="Silk.NET.OpenGL" Version="2.22.0" />
    </ItemGroup>

</Project>
```

step2:forms
C:\Users\wangrusheng\RiderProjects\WinFormsApp8\WinFormsApp8\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Numerics;
using System.Windows.Forms;
using System.Drawing;
using NCalc;
using Silk.NET.GLFW;
using Silk.NET.OpenGL;
using System.Runtime.InteropServices;

namespace WinFormsApp8
{

    public unsafe partial class Form1 : Form
    {
        // OpenGL相关成员
        private GL gl;
        private Glfw glfw;
        private WindowHandle* glfwWindow;
        private System.Windows.Forms.Panel glPanel;
        private uint shaderProgram;
        private uint vao;
        private uint vbo;

        // 数据相关成员
        private readonly Dictionary<string, double> variables = new Dictionary<string, double>();
        private readonly List<float> verticesData = new List<float>();
        private DateTime _startTime = DateTime.Now;
		
        // 摄像机控制
        private Vector2 _cameraAngle = new Vector2(0, (float)(Math.PI / 6)); // 30度的弧度值 ≈ 0.5236
        private Point _lastMousePos;
        private bool _mouseDown;
		

        // Win32 API
        [DllImport("user32.dll")]
        private static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);
        [DllImport("user32.dll")]
        private static extern bool SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter,
            int X, int Y, int cx, int cy, SetWindowPosFlags uFlags);
        [DllImport("glfw3.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern IntPtr glfwGetWin32Window(WindowHandle* window);

        // 使用逐字字符串(@)避免转义反斜杠
        private string absolutePath = @"C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\input.txt";
        
        [Flags]
        private enum SetWindowPosFlags : uint
        {
            FrameChanged = 0x0020
        }

        public Form1()
        {
            InitializeComponent();
            InitializeOpenGLPanel();
            this.Load += Form1_Load;
            this.Resize += Form1_Resize;
			 this.MouseDown += Form1_MouseDown;
            this.MouseMove += Form1_MouseMove;
            this.MouseUp += Form1_MouseUp;
        }
   
        private void Form1_Load(object? sender, EventArgs e)
        {
            try
            {
                SetupGLFWWindow();
                InitializeOpenGL();
                ProcessInputFile(absolutePath);
                SetupGeometry();
                StartRenderTimer();
            }
            catch (Exception ex)
            {
                MessageBox.Show($"初始化失败: {ex.Message}");
                Close();
            }
        }


        private void InitializeOpenGLPanel()
        {
            glPanel = new Panel
            {
                Dock = DockStyle.Fill,
                BackColor = Color.Black
            };
            Controls.Add(glPanel);
        }


        private void SetupGLFWWindow()
        {
            glfw = Glfw.GetApi();
            glfw.Init();

            glfw.WindowHint(WindowHintInt.ContextVersionMajor, 3);
            glfw.WindowHint(WindowHintInt.ContextVersionMinor, 3);
            glfw.WindowHint(WindowHintOpenGlProfile.OpenGlProfile, OpenGlProfile.Core);
            glfw.WindowHint(WindowHintBool.Decorated, false);

            unsafe
            {
                glfwWindow = glfw.CreateWindow(glPanel.Width, glPanel.Height, "GLFW", null, null);
                glfw.ShowWindow(glfwWindow);

                var hwnd = glfwGetWin32Window(glfwWindow);
                SetParent(hwnd, glPanel.Handle);
                SetWindowPos(hwnd, IntPtr.Zero, 0, 0,
                    glPanel.Width, glPanel.Height, SetWindowPosFlags.FrameChanged);

                glfw.MakeContextCurrent(glfwWindow);
                glfw.SwapInterval(1); 
            }
        }


       private void InitializeOpenGL()
        {
            gl = GL.GetApi(glfw.GetProcAddress);
            gl.Viewport(0, 0, (uint)glPanel.Width, (uint)glPanel.Height);
            gl.Enable(EnableCap.DepthTest);

            // 着色器程序
            const string vertexShaderSource = @"#version 330 core
                layout (location = 0) in vec3 aPos;
                layout (location = 1) in vec3 aColor;
                out vec3 ourColor;
              
			    uniform mat4 projection;
                uniform mat4 view;
                uniform mat4 model;

                void main()
                {
                    gl_Position = projection * view * model * vec4(aPos, 1.0);
                    ourColor = aColor;
                }";

            const string fragmentShaderSource = @"#version 330 core
                in vec3 ourColor;
                out vec4 FragColor;
                void main()
                {
                    FragColor = vec4(ourColor, 1.0);
                }";

            uint vertexShader = gl.CreateShader(ShaderType.VertexShader);
            gl.ShaderSource(vertexShader, vertexShaderSource);
            gl.CompileShader(vertexShader);

            uint fragmentShader = gl.CreateShader(ShaderType.FragmentShader);
            gl.ShaderSource(fragmentShader, fragmentShaderSource);
            gl.CompileShader(fragmentShader);

            shaderProgram = gl.CreateProgram();
            gl.AttachShader(shaderProgram, vertexShader);
            gl.AttachShader(shaderProgram, fragmentShader);
            gl.LinkProgram(shaderProgram);

            gl.DeleteShader(vertexShader);
            gl.DeleteShader(fragmentShader);
        }

        private void ProcessInputFile(string path)
        {
            if (!File.Exists(path))
                throw new FileNotFoundException("找不到输入文件", path);

            variables.Clear();
            verticesData.Clear();

            foreach (var line in File.ReadLines(path))
            {
                var trimmed = line.Trim();
                if (string.IsNullOrEmpty(trimmed) || trimmed.StartsWith("#"))
                    continue;

                ProcessLine(trimmed);
            }

            if (verticesData.Count % 6 != 0)
                throw new InvalidDataException("顶点数据格式不正确，应为每行6个浮点数");
        }
        
        
        private void ProcessLine(string line)
        {
            try
            {
                if (line.Contains("=")) // 变量赋值
                {
                    var parts = line.Split(new[] { '=' }, 2);
                    var varName = parts[0].Trim();
                    var expr = parts[1].Trim();
                    variables[varName] = EvaluateExpression(expr);
                }
                else // 顶点数据
                {
                    var values = line.Split(new[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
                    foreach (var value in values)
                    {
                        var result = EvaluateExpression(value.Trim());
                        verticesData.Add((float)result);
                    }
                }
            }
            catch (Exception ex)
            {
                throw new InvalidDataException($"处理行时出错: {line}\n错误信息: {ex.Message}");
            }
        }
        
        private double EvaluateExpression(string expr)
        {
            var e = new Expression(expr, EvaluateOptions.IgnoreCase);
            
            
            var paramDict = variables.ToDictionary(
                kvp => kvp.Key, 
                kvp => (object)kvp.Value 
            );
            e.Parameters = paramDict;
            

            e.EvaluateFunction += (name, args) =>
            {
            
                if (name.Equals("sin", StringComparison.OrdinalIgnoreCase))
                    args.Result = Math.Sin(Convert.ToDouble(args.Parameters[0].Evaluate()));
                else if (name.Equals("cos", StringComparison.OrdinalIgnoreCase))
                    args.Result = Math.Cos(Convert.ToDouble(args.Parameters[0].Evaluate()));
            };

             return Convert.ToDouble(e.Evaluate());
        }
        
        
        private void SetupGeometry()
        {
            vao = gl.GenVertexArray();
            vbo = gl.GenBuffer();

            gl.BindVertexArray(vao);
            gl.BindBuffer(BufferTargetARB.ArrayBuffer, vbo);

            var vertices = verticesData.ToArray();
            unsafe
            {
                fixed (float* ptr = vertices)
                {
                    gl.BufferData(BufferTargetARB.ArrayBuffer,
                        (nuint)(vertices.Length * sizeof(float)),
                        ptr, BufferUsageARB.StaticDraw);
                }
            }

            gl.VertexAttribPointer(0, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)0);
            gl.EnableVertexAttribArray(0);
            gl.VertexAttribPointer(1, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)(3 * sizeof(float)));
            gl.EnableVertexAttribArray(1);
        }

        private void StartRenderTimer()
        {
            var timer = new System.Windows.Forms.Timer { Interval = 16 };
            timer.Tick += (s, e) => Render();
            timer.Start();
        }
        
		
 
        private void Render()
        {
            if (gl == null) return;

            gl.ClearColor(0.1f, 0.1f, 0.1f, 1.0f);
            gl.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

            // 计算矩阵
            float aspect = (float)glPanel.Width / glPanel.Height;
            float time = (float)(DateTime.Now - _startTime).TotalSeconds;

            var model = Matrix4x4.CreateRotationY(time);
            var view = CreateViewMatrix();
            var projection = Matrix4x4.CreatePerspectiveFieldOfView(
                MathHelper.PiOver4, aspect, 0.1f, 100.0f);

            // 传递矩阵到着色器
            gl.UseProgram(shaderProgram);
            unsafe
            {
                gl.UniformMatrix4(gl.GetUniformLocation(shaderProgram, "model"), 1, false, (float*)&model);
                gl.UniformMatrix4(gl.GetUniformLocation(shaderProgram, "view"), 1, false, (float*)&view);
                gl.UniformMatrix4(gl.GetUniformLocation(shaderProgram, "projection"), 1, false, (float*)&projection);
            }

            // 绘制
            gl.BindVertexArray(vao);
            gl.DrawArrays(PrimitiveType.Triangles, 0, (uint)(verticesData.Count / 6));

            unsafe { glfw.SwapBuffers(glfwWindow); }
            glfw.PollEvents();
        }

    
		
		  private Matrix4x4 CreateViewMatrix()
        {
            var eyePos = new Vector3(
                (float)(Math.Sin(_cameraAngle.X) * Math.Cos(_cameraAngle.Y)),
                (float)Math.Sin(_cameraAngle.Y),
                (float)(Math.Cos(_cameraAngle.X) * Math.Cos(_cameraAngle.Y))
            ) * 3.0f;

            return Matrix4x4.CreateLookAt(
                eyePos,
                Vector3.Zero,
                Vector3.UnitY
            );
        }

        private void Form1_Resize(object? sender, EventArgs e)
        {
            if (glfwWindow == null || glPanel == null) return;

            glfw.SetWindowSize(glfwWindow, glPanel.Width, glPanel.Height);
            var hwnd = glfwGetWin32Window(glfwWindow);
            SetWindowPos(hwnd, IntPtr.Zero, 0, 0,
                glPanel.Width, glPanel.Height, SetWindowPosFlags.FrameChanged);
            gl?.Viewport(0, 0, (uint)glPanel.Width, (uint)glPanel.Height);
        }
		
		private void Form1_MouseDown(object sender, MouseEventArgs e)
        {
            _mouseDown = true;
            _lastMousePos = e.Location;
        }

        private void Form1_MouseMove(object sender, MouseEventArgs e)
        {
            if (_mouseDown)
            {
                var delta = new Vector2(e.X - _lastMousePos.X, e.Y - _lastMousePos.Y);
                _cameraAngle += delta * 0.01f;
                _cameraAngle.Y = Math.Clamp(_cameraAngle.Y, -MathHelper.PiOver2 + 0.1f, MathHelper.PiOver2 - 0.1f);
                _lastMousePos = e.Location;
            }
        }

        private void Form1_MouseUp(object sender, MouseEventArgs e)
        {
            _mouseDown = false;
        }
		
        protected override void OnFormClosing(FormClosingEventArgs e)
        {
            base.OnFormClosing(e);
 
            if (gl != null)
            {
                gl.DeleteBuffer(vbo);
                gl.DeleteVertexArray(vao);
                gl.DeleteProgram(shaderProgram);
            }
            glfw?.Terminate();
        }

    }
	 public static class MathHelper
    {
        public const float Pi = (float)Math.PI;
        public const float PiOver2 = Pi / 2.0f;
        public const float PiOver4 = Pi / 4.0f;
    }

}
```


step3:到这里opengl写完了  运行 验证没问题，至少渲染部分解决了，接下来处理python脚本 
C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\generate_shape.py

```python
import math
import argparse


def generate_triangle(size=0.5, color=(1.0, 0.0, 0.0)):
    """生成三角形顶点数据"""
    return [
        (-size, -size, 0.0, *color),
        (size, -size, 0.0, *color),
        (0.0, size, 0.0, *color)
    ]


def generate_cube(size=0.5):
    """生成立方体顶点数据（每个面不同颜色）"""
    vertices = []
    # 前面（红色）
    vertices.extend(_cube_face(size, "front", (1.0, 0.0, 0.0)))
    # 后面（绿色）
    vertices.extend(_cube_face(size, "back", (0.0, 1.0, 0.0)))
    # 左面（蓝色）
    vertices.extend(_cube_face(size, "left", (0.0, 0.0, 1.0)))
    # 右面（黄色）
    vertices.extend(_cube_face(size, "right", (1.0, 1.0, 0.0)))
    # 顶面（品红）
    vertices.extend(_cube_face(size, "top", (1.0, 0.0, 1.0)))
    # 底面（青色）
    vertices.extend(_cube_face(size, "bottom", (0.0, 1.0, 1.0)))
    return vertices


def _cube_face(size, face, color):
    """生成单个立方体面的两个三角形"""
    s = size
    if face == "front":
        return [
            (-s, -s, s, *color), (s, -s, s, *color), (s, s, s, *color),
            (s, s, s, *color), (-s, s, s, *color), (-s, -s, s, *color)
        ]
    elif face == "back":
        return [
            (-s, -s, -s, *color), (s, -s, -s, *color), (s, s, -s, *color),
            (s, s, -s, *color), (-s, s, -s, *color), (-s, -s, -s, *color)
        ]
    elif face == "left":
        return [
            (-s, -s, -s, *color), (-s, -s, s, *color), (-s, s, s, *color),
            (-s, s, s, *color), (-s, s, -s, *color), (-s, -s, -s, *color)
        ]
    elif face == "right":
        return [
            (s, -s, -s, *color), (s, -s, s, *color), (s, s, s, *color),
            (s, s, s, *color), (s, s, -s, *color), (s, -s, -s, *color)
        ]
    elif face == "top":
        return [
            (-s, s, -s, *color), (s, s, -s, *color), (s, s, s, *color),
            (s, s, s, *color), (-s, s, s, *color), (-s, s, -s, *color)
        ]
    elif face == "bottom":
        return [
            (-s, -s, -s, *color), (s, -s, -s, *color), (s, -s, s, *color),
            (s, -s, s, *color), (-s, -s, s, *color), (-s, -s, -s, *color)
        ]


import math


def generate_sphere(
        radius=0.5,
        rings=16,
        sectors=32,
        color_top=(1.0, 0.0, 0.0),
        color_bottom=(0.0, 0.0, 1.0)
):
    vertices = []
    # 生成顶点数据（带颜色插值）
    for i in range(rings + 1):
        phi = math.pi * i / rings  # 纬度角 0~π
        for j in range(sectors + 1):
            theta = 2 * math.pi * j / sectors  # 经度角 0~2π
            x = radius * math.sin(phi) * math.cos(theta)
            y = radius * math.cos(phi)
            z = radius * math.sin(phi) * math.sin(theta)
            # 颜色插值
            t = i / rings
            red = color_top[0] * (1 - t) + color_bottom[0] * t
            green = color_top[1] * (1 - t) + color_bottom[1] * t
            blue = color_top[2] * (1 - t) + color_bottom[2] * t
            vertices.append((x, y, z, red, green, blue))

    # 生成三角形索引
    indices = []
    for i in range(rings):
        for j in range(sectors):
            a = i * (sectors + 1) + j
            b = a + 1
            c = (i + 1) * (sectors + 1) + j
            d = c + 1
            # 三角形1: a-c-b
            indices.extend([a, c, b])
            # 三角形2: b-c-d
            indices.extend([b, c, d])

    # 展开索引为顶点数据
    expanded_vertices = [vertices[i] for i in indices]
    return expanded_vertices


def save_to_file(filename, vertices, variables=None):
    """保存到文件"""
    with open(filename, 'w') as f:
        # 写入变量定义
        if variables:
            for name, value in variables.items():
                f.write(f"{name} = {value}\n")

        # 写入顶点数据
        for v in vertices:
            line = ", ".join(f"{x:.4f}" if isinstance(x, float) else str(x) for x in v)
            f.write(line + "\n")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='生成3D形状的顶点数据文件')
    parser.add_argument('shape', choices=['triangle', 'cube', 'sphere'],
                        help='要生成的形状')
    parser.add_argument('-o', '--output', default='input.txt',
                        help='输出文件名')
    parser.add_argument('--size', type=float, default=0.5,
                        help='形状尺寸（立方体边长/球体半径）')
    parser.add_argument('--color', nargs=3, type=float, default=None,
                        metavar=('R', 'G', 'B'),
                        help='统一颜色（0.0-1.0）')
    parser.add_argument('--rings', type=int, default=16,
                        help='球体的纬度分段数（默认：16）')
    parser.add_argument('--sectors', type=int, default=32,
                        help='球体的经度分段数（默认：32）')
    parser.add_argument('--color-top', nargs=3, type=float, default=[1.0, 0.0, 0.0],
                        metavar=('R', 'G', 'B'),
                        help='球体顶部颜色（默认：红色）')
    parser.add_argument('--color-bottom', nargs=3, type=float, default=[0.0, 0.0, 1.0],
                        metavar=('R', 'G', 'B'),
                        help='球体底部颜色（默认：蓝色）')

    args = parser.parse_args()

    # 生成顶点数据
    color = args.color if args.color else None
    variables = {"object_size": args.size}

    if args.shape == 'triangle':
        vertices = generate_triangle(args.size, color or (1.0, 0.0, 0.0))
    elif args.shape == 'cube':
        vertices = generate_cube(args.size)
    elif args.shape == 'sphere':
        vertices = generate_sphere(
            radius=args.size,
            rings=args.rings,
            sectors=args.sectors,
            color_top=args.color_top,
            color_bottom=args.color_bottom
        )
    # 保存文件
    save_to_file(args.output, vertices, variables)
    print(f"成功生成 {args.shape} 数据到 {args.output}，包含 {len(vertices)} 个顶点")
```

step4:在终端里，输入指令，生成数据文件

```bash

(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python generate_shape.py sphere --rings 20 --sectors 40 --color-top 1 1 0 --color-bottom 0 1 0 -o input.txt
成功生成 sphere 数据到 input.txt，包含 4800 个顶点
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python generate_shape.py sphere -o input.txt                                                               
成功生成 sphere 数据到 input.txt，包含 3072 个顶点
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python generate_shape.py cube --size 0.8 -o input.txt                                                      
成功生成 cube 数据到 input.txt，包含 36 个顶点
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python generate_shape.py triangle -o input.txt       
成功生成 triangle 数据到 input.txt，包含 3 个顶点
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1>
```

step5:生成的文件路径
C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\input.txt

```bash
object_size = 0.5
-0.5000, -0.5000, 0.0000, 1.0000, 0.0000, 0.0000
0.5000, -0.5000, 0.0000, 1.0000, 0.0000, 0.0000
0.0000, 0.5000, 0.0000, 1.0000, 0.0000, 0.0000

```

///我是分割线 旧的实现方式 
step101:

```bash

至少旧样式可以了
参考下面的代码格式，给我生成一个立方体的参数 ，在input.txt文件中


 
  


4. 示例输入文件格式 (input.txt)


# 变量定义
size = 0.8
color_red = 1.0

# 顶点数据（前3个为位置，后3个为颜色）
-size, -size, 0, color_red, 0, 0
size, -size, 0, 0, 1, 0
0, size, 0, 0, 0, 1



5. 示例输入文件格式 (input.txt)

# 定义变量
size = 0.5
red = 1.0
green = 0.5
blue = sin(3.14159/2)  # 支持数学函数

# 顶点数据（位置xyz + 颜色rgb）
-size, -size, 0, red, 0, 0
size, -size, 0, 0, green, 0
0, size, 0, 0, 0, blue

# 第二个三角形
0.2, 0.2, 0, 1, 1, 0
0.7, 0.2, 0, 1, 0, 1
0.45, 0.7, 0, 0, 1, 1



6. 立方体 示例输入文件格式 (input.txt)
# 定义变量
size = 0.5
red = 1.0
green = 1.0
blue = 1.0

# 顶点数据（位置xyz + 颜色rgb）

# 前面（红色）
-size, -size, size, red, 0, 0
size, -size, size, red, 0, 0
size, size, size, red, 0, 0
size, size, size, red, 0, 0
-size, size, size, red, 0, 0
-size, -size, size, red, 0, 0

# 后面（绿色）
-size, -size, -size, 0, green, 0
size, -size, -size, 0, green, 0
size, size, -size, 0, green, 0
size, size, -size, 0, green, 0
-size, size, -size, 0, green, 0
-size, -size, -size, 0, green, 0

# 左面（蓝色）
-size, -size, size, 0, 0, blue
-size, size, size, 0, 0, blue
-size, size, -size, 0, 0, blue
-size, size, -size, 0, 0, blue
-size, -size, -size, 0, 0, blue
-size, -size, size, 0, 0, blue

# 右面（黄色：红+绿）
size, -size, size, red, green, 0
size, size, size, red, green, 0
size, size, -size, red, green, 0
size, size, -size, red, green, 0
size, -size, -size, red, green, 0
size, -size, size, red, green, 0

# 顶面（品红：红+蓝）
-size, size, size, red, 0, blue
size, size, size, red, 0, blue
size, size, -size, red, 0, blue
size, size, -size, red, 0, blue
-size, size, -size, red, 0, blue
-size, size, size, red, 0, blue

# 底面（青色：绿+蓝）
-size, -size, size, 0, green, blue
size, -size, size, 0, green, blue
size, -size, -size, 0, green, blue
size, -size, -size, 0, green, blue
-size, -size, -size, 0, green, blue
-size, -size, size, 0, green, blue


7. 钻石形 示例输入文件格式 (input.txt)

# 定义变量
radius = 0.8
red = 1.0
green = 0.5
blue = sin(3.14159/2)

# 顶点数据（位置xyz + 颜色rgb）
# -------------------------------
# 顶部顶点 (北极点，红色渐变)
0, 0, radius, red, 0, 0

# 赤道顶点 (动态颜色混合)
radius, 0, 0, 0, green, 0
0, radius, 0, red, green, 0
-radius, 0, 0, 0, green, blue
0, -radius, 0, red, 0, blue

# 底部顶点 (南极点，蓝色渐变)
0, 0, -radius, 0, 0, blue

# -------------------------------
# 三角形面数据（每个面3个顶点）
# 上半球
0, 0, radius, radius, 0, 0, 0, radius, 0, 0, green, 0
0, 0, radius, 0, radius, 0, -radius, 0, 0, 0, green, blue
0, 0, radius, -radius, 0, 0, 0, -radius, 0, red, 0, blue
0, 0, radius, 0, -radius, 0, radius, 0, 0, 0, green, 0

# 下半球
0, 0, -radius, radius, 0, 0, 0, radius, 0, 0, green, 0
0, 0, -radius, 0, radius, 0, -radius, 0, 0, 0, green, blue
0, 0, -radius, -radius, 0, 0, 0, -radius, 0, red, 0, blue
0, 0, -radius, 0, -radius, 0, radius, 0, 0, 0, green, 0



```
step102:

```csharp
 

using System;
using System.Collections.Generic;
using System.IO;
using System.Windows.Forms;
using System.Drawing;
using NCalc;
using Silk.NET.GLFW;
using Silk.NET.OpenGL;
using System.Runtime.InteropServices;

namespace WinFormsApp8
{

    public unsafe partial class Form1 : Form
    {
        // OpenGL相关成员
        private GL gl;
        private Glfw glfw;
        private WindowHandle* glfwWindow;
        private System.Windows.Forms.Panel glPanel;
        private uint shaderProgram;
        private uint vao;
        private uint vbo;

        // 数据相关成员
        private readonly Dictionary<string, double> variables = new Dictionary<string, double>();
        private readonly List<float> verticesData = new List<float>();
        private DateTime _startTime = DateTime.Now;

        // Win32 API
        [DllImport("user32.dll")]
        private static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);
        [DllImport("user32.dll")]
        private static extern bool SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter,
            int X, int Y, int cx, int cy, SetWindowPosFlags uFlags);
        [DllImport("glfw3.dll", CallingConvention = CallingConvention.Cdecl)]
        private static extern IntPtr glfwGetWin32Window(WindowHandle* window);

        // 使用逐字字符串(@)避免转义反斜杠
        private string absolutePath = @"C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\input.txt";
        
        [Flags]
        private enum SetWindowPosFlags : uint
        {
            FrameChanged = 0x0020
        }

        public Form1()
        {
            InitializeComponent();
            InitializeOpenGLPanel();
            this.Load += Form1_Load;
            this.Resize += Form1_Resize;
        }
   
        private void Form1_Load(object? sender, EventArgs e)
        {
            try
            {
                SetupGLFWWindow();
                InitializeOpenGL();
                ProcessInputFile(absolutePath);
                SetupGeometry();
                StartRenderTimer();
            }
            catch (Exception ex)
            {
                MessageBox.Show($"初始化失败: {ex.Message}");
                Close();
            }
        }


        private void InitializeOpenGLPanel()
        {
            glPanel = new Panel
            {
                Dock = DockStyle.Fill,
                BackColor = Color.Black
            };
            Controls.Add(glPanel);
        }


        private void SetupGLFWWindow()
        {
            glfw = Glfw.GetApi();
            glfw.Init();

            glfw.WindowHint(WindowHintInt.ContextVersionMajor, 3);
            glfw.WindowHint(WindowHintInt.ContextVersionMinor, 3);
            glfw.WindowHint(WindowHintOpenGlProfile.OpenGlProfile, OpenGlProfile.Core);
            glfw.WindowHint(WindowHintBool.Decorated, false);

            unsafe
            {
                glfwWindow = glfw.CreateWindow(glPanel.Width, glPanel.Height, "GLFW", null, null);
                glfw.ShowWindow(glfwWindow);

                var hwnd = glfwGetWin32Window(glfwWindow);
                SetParent(hwnd, glPanel.Handle);
                SetWindowPos(hwnd, IntPtr.Zero, 0, 0,
                    glPanel.Width, glPanel.Height, SetWindowPosFlags.FrameChanged);

                glfw.MakeContextCurrent(glfwWindow);
                glfw.SwapInterval(1); 
            }
        }


       private void InitializeOpenGL()
        {
            gl = GL.GetApi(glfw.GetProcAddress);
            gl.Viewport(0, 0, (uint)glPanel.Width, (uint)glPanel.Height);
            gl.Enable(EnableCap.DepthTest);

            // 着色器程序
            const string vertexShaderSource = @"#version 330 core
                layout (location = 0) in vec3 aPos;
                layout (location = 1) in vec3 aColor;
                out vec3 ourColor;
                uniform float time;

                void main()
                {
                    mat4 rotate = mat4(
                        cos(time), 0.0, sin(time), 0.0,
                        0.0, 1.0, 0.0, 0.0,
                        -sin(time), 0.0, cos(time), 0.0,
                        0.0, 0.0, 0.0, 1.0
                    );
                    gl_Position = rotate * vec4(aPos, 1.0);
                    ourColor = aColor;
                }";

            const string fragmentShaderSource = @"#version 330 core
                in vec3 ourColor;
                out vec4 FragColor;
                void main()
                {
                    FragColor = vec4(ourColor, 1.0);
                }";

            uint vertexShader = gl.CreateShader(ShaderType.VertexShader);
            gl.ShaderSource(vertexShader, vertexShaderSource);
            gl.CompileShader(vertexShader);

            uint fragmentShader = gl.CreateShader(ShaderType.FragmentShader);
            gl.ShaderSource(fragmentShader, fragmentShaderSource);
            gl.CompileShader(fragmentShader);

            shaderProgram = gl.CreateProgram();
            gl.AttachShader(shaderProgram, vertexShader);
            gl.AttachShader(shaderProgram, fragmentShader);
            gl.LinkProgram(shaderProgram);

            gl.DeleteShader(vertexShader);
            gl.DeleteShader(fragmentShader);
        }

        private void ProcessInputFile(string path)
        {
            if (!File.Exists(path))
                throw new FileNotFoundException("找不到输入文件", path);

            variables.Clear();
            verticesData.Clear();

            foreach (var line in File.ReadLines(path))
            {
                var trimmed = line.Trim();
                if (string.IsNullOrEmpty(trimmed) || trimmed.StartsWith("#"))
                    continue;

                ProcessLine(trimmed);
            }

            if (verticesData.Count % 6 != 0)
                throw new InvalidDataException("顶点数据格式不正确，应为每行6个浮点数");
        }
        
        
        private void ProcessLine(string line)
        {
            try
            {
                if (line.Contains("=")) // 变量赋值
                {
                    var parts = line.Split(new[] { '=' }, 2);
                    var varName = parts[0].Trim();
                    var expr = parts[1].Trim();
                    variables[varName] = EvaluateExpression(expr);
                }
                else // 顶点数据
                {
                    var values = line.Split(new[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
                    foreach (var value in values)
                    {
                        var result = EvaluateExpression(value.Trim());
                        verticesData.Add((float)result);
                    }
                }
            }
            catch (Exception ex)
            {
                throw new InvalidDataException($"处理行时出错: {line}\n错误信息: {ex.Message}");
            }
        }
        
        private double EvaluateExpression(string expr)
        {
            var e = new Expression(expr, EvaluateOptions.IgnoreCase);
            
            // 转换字典类型：Dictionary<string, double> → Dictionary<string, object>
            var paramDict = variables.ToDictionary(
                kvp => kvp.Key, 
                kvp => (object)kvp.Value // 添加显式类型转换
            );
            e.Parameters = paramDict;
            

            e.EvaluateFunction += (name, args) =>
            {
                // 示例：支持sin函数
                if (name.Equals("sin", StringComparison.OrdinalIgnoreCase))
                {
                    args.Result = Math.Sin(Convert.ToDouble(args.Parameters[0].Evaluate()));
                }
            };

            var result = e.Evaluate();
            return Convert.ToDouble(result);
        }
        
        
        private void SetupGeometry()
        {
            vao = gl.GenVertexArray();
            vbo = gl.GenBuffer();

            gl.BindVertexArray(vao);
            gl.BindBuffer(BufferTargetARB.ArrayBuffer, vbo);

            var vertices = verticesData.ToArray();
            unsafe
            {
                fixed (float* ptr = vertices)
                {
                    gl.BufferData(BufferTargetARB.ArrayBuffer,
                        (nuint)(vertices.Length * sizeof(float)),
                        ptr, BufferUsageARB.StaticDraw);
                }
            }

            gl.VertexAttribPointer(0, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)0);
            gl.EnableVertexAttribArray(0);
            gl.VertexAttribPointer(1, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)(3 * sizeof(float)));
            gl.EnableVertexAttribArray(1);
        }

        private void StartRenderTimer()
        {
            var timer = new System.Windows.Forms.Timer { Interval = 16 };
            timer.Tick += (s, e) => Render();
            timer.Start();
        }
        
        
        private void SetupTriangle()
        {
            float[] vertices = {
            -0.5f, -0.5f, 0.0f, 1.0f, 0.0f, 0.0f, // 左下 - 红色
            0.5f, -0.5f, 0.0f, 0.0f, 1.0f, 0.0f, // 右下 - 绿色
            0.0f,  0.5f, 0.0f, 0.0f, 0.0f, 1.0f  // 顶部 - 蓝色
        };

            // 创建VAO/VBO
            vao = gl.GenVertexArray();
            vbo = gl.GenBuffer();

            gl.BindVertexArray(vao);
            gl.BindBuffer(BufferTargetARB.ArrayBuffer, vbo);

            fixed (float* ptr = vertices)
            {
                gl.BufferData(BufferTargetARB.ArrayBuffer,
                    (nuint)(vertices.Length * sizeof(float)),
                    ptr, BufferUsageARB.StaticDraw);
            }

            // 位置属性
            gl.VertexAttribPointer(0, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)0);
            gl.EnableVertexAttribArray(0);

            // 颜色属性
            gl.VertexAttribPointer(1, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)(3 * sizeof(float)));
            gl.EnableVertexAttribArray(1);
        }

        private void SetupCube()
        {
            float[] vertices = {
            // 前面（红色）
            -0.5f, -0.5f,  0.5f, 1.0f, 0.0f, 0.0f,
             0.5f, -0.5f,  0.5f, 1.0f, 0.0f, 0.0f,
             0.5f,  0.5f,  0.5f, 1.0f, 0.0f, 0.0f,
             0.5f,  0.5f,  0.5f, 1.0f, 0.0f, 0.0f,
            -0.5f,  0.5f,  0.5f, 1.0f, 0.0f, 0.0f,
            -0.5f, -0.5f,  0.5f, 1.0f, 0.0f, 0.0f,

            // 后面（绿色）
            -0.5f, -0.5f, -0.5f, 0.0f, 1.0f, 0.0f,
             0.5f, -0.5f, -0.5f, 0.0f, 1.0f, 0.0f,
             0.5f,  0.5f, -0.5f, 0.0f, 1.0f, 0.0f,
             0.5f,  0.5f, -0.5f, 0.0f, 1.0f, 0.0f,
            -0.5f,  0.5f, -0.5f, 0.0f, 1.0f, 0.0f,
            -0.5f, -0.5f, -0.5f, 0.0f, 1.0f, 0.0f,

            // 左面（蓝色）
            -0.5f,  0.5f,  0.5f, 0.0f, 0.0f, 1.0f,
            -0.5f,  0.5f, -0.5f, 0.0f, 0.0f, 1.0f,
            -0.5f, -0.5f, -0.5f, 0.0f, 0.0f, 1.0f,
            -0.5f, -0.5f, -0.5f, 0.0f, 0.0f, 1.0f,
            -0.5f, -0.5f,  0.5f, 0.0f, 0.0f, 1.0f,
            -0.5f,  0.5f,  0.5f, 0.0f, 0.0f, 1.0f,

            // 右面（黄色）
             0.5f,  0.5f,  0.5f, 1.0f, 1.0f, 0.0f,
             0.5f,  0.5f, -0.5f, 1.0f, 1.0f, 0.0f,
             0.5f, -0.5f, -0.5f, 1.0f, 1.0f, 0.0f,
             0.5f, -0.5f, -0.5f, 1.0f, 1.0f, 0.0f,
             0.5f, -0.5f,  0.5f, 1.0f, 1.0f, 0.0f,
             0.5f,  0.5f,  0.5f, 1.0f, 1.0f, 0.0f,

            // 顶面（品红）
            -0.5f,  0.5f, -0.5f, 1.0f, 0.0f, 1.0f,
             0.5f,  0.5f, -0.5f, 1.0f, 0.0f, 1.0f,
             0.5f,  0.5f,  0.5f, 1.0f, 0.0f, 1.0f,
             0.5f,  0.5f,  0.5f, 1.0f, 0.0f, 1.0f,
            -0.5f,  0.5f,  0.5f, 1.0f, 0.0f, 1.0f,
            -0.5f,  0.5f, -0.5f, 1.0f, 0.0f, 1.0f,

            // 底面（青色）
            -0.5f, -0.5f, -0.5f, 0.0f, 1.0f, 1.0f,
             0.5f, -0.5f, -0.5f, 0.0f, 1.0f, 1.0f,
             0.5f, -0.5f,  0.5f, 0.0f, 1.0f, 1.0f,
             0.5f, -0.5f,  0.5f, 0.0f, 1.0f, 1.0f,
            -0.5f, -0.5f,  0.5f, 0.0f, 1.0f, 1.0f,
            -0.5f, -0.5f, -0.5f, 0.0f, 1.0f, 1.0f
        };

            vao = gl.GenVertexArray();
            vbo = gl.GenBuffer();

            gl.BindVertexArray(vao);
            gl.BindBuffer(BufferTargetARB.ArrayBuffer, vbo);

            fixed (float* ptr = vertices)
            {
                gl.BufferData(BufferTargetARB.ArrayBuffer,
                    (nuint)(vertices.Length * sizeof(float)),
                    ptr, BufferUsageARB.StaticDraw);
            }

            gl.VertexAttribPointer(0, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)0);
            gl.EnableVertexAttribArray(0);

            gl.VertexAttribPointer(1, 3, VertexAttribPointerType.Float, false, 6 * sizeof(float), (void*)(3 * sizeof(float)));
            gl.EnableVertexAttribArray(1);
        }

        private void Render()
        {
            if (gl == null) return;

            gl.ClearColor(0.2f, 0.3f, 0.3f, 1.0f);
            gl.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

            gl.UseProgram(shaderProgram);

            float time = (float)(DateTime.Now - _startTime).TotalSeconds;
            gl.Uniform1(gl.GetUniformLocation(shaderProgram, "time"), time);


            gl.BindVertexArray(vao);
            gl.DrawArrays(PrimitiveType.Triangles, 0, (uint)(verticesData.Count / 6));

            unsafe{glfw.SwapBuffers(glfwWindow);}
            glfw.PollEvents();
        }


        protected override void OnResize(EventArgs e)
        {
            base.OnResize(e);
            if (glfwWindow != null && glPanel != null)
            {
                glfw.SetWindowSize(glfwWindow, glPanel.Width, glPanel.Height);
                nint hwnd = glfwGetWin32Window(glfwWindow);
                SetWindowPos(hwnd, IntPtr.Zero, 0, 0,
                    glPanel.Width, glPanel.Height, SetWindowPosFlags.FrameChanged);
                gl?.Viewport(0, 0, (uint)glPanel.Width, (uint)glPanel.Height);
            }
        }

       #region 窗口事件处理
        private void Form1_Resize(object? sender, EventArgs e)
        {
            if (glfwWindow == null || glPanel == null) return;

            glfw.SetWindowSize(glfwWindow, glPanel.Width, glPanel.Height);
            var hwnd = glfwGetWin32Window(glfwWindow);
            SetWindowPos(hwnd, IntPtr.Zero, 0, 0,
                glPanel.Width, glPanel.Height, SetWindowPosFlags.FrameChanged);
            gl?.Viewport(0, 0, (uint)glPanel.Width, (uint)glPanel.Height);
        }
		
        protected override void OnFormClosing(FormClosingEventArgs e)
        {
            base.OnFormClosing(e);
 
            if (gl != null)
            {
                gl.DeleteBuffer(vbo);
                gl.DeleteVertexArray(vao);
                gl.DeleteProgram(shaderProgram);
            }
            glfw?.Terminate();
        }
   #endregion

    }

}
```
////我是分割线
我希望用python生成 人体3d图
step201:

```python
import math
import argparse


def generate_triangle(size=0.5, color=(1.0, 0.0, 0.0)):
    """生成三角形顶点数据"""
    return [
        (-size, -size, 0.0, *color),
        (size, -size, 0.0, *color),
        (0.0, size, 0.0, *color)
    ]


def generate_cuboid(width, height, depth, color):
    """生成长方体顶点数据（单一颜色）"""
    vertices = []
    w, h, d = width / 2, height / 2, depth / 2

    # 前面 (z = d)
    vertices.extend([(-w, -h, d, *color), (w, -h, d, *color), (w, h, d, *color)])
    vertices.extend([(w, h, d, *color), (-w, h, d, *color), (-w, -h, d, *color)])

    # 后面 (z = -d)
    vertices.extend([(-w, -h, -d, *color), (w, -h, -d, *color), (w, h, -d, *color)])
    vertices.extend([(w, h, -d, *color), (-w, h, -d, *color), (-w, -h, -d, *color)])

    # 左面 (x = -w)
    vertices.extend([(-w, -h, -d, *color), (-w, -h, d, *color), (-w, h, d, *color)])
    vertices.extend([(-w, h, d, *color), (-w, h, -d, *color), (-w, -h, -d, *color)])

    # 右面 (x = w)
    vertices.extend([(w, -h, -d, *color), (w, -h, d, *color), (w, h, d, *color)])
    vertices.extend([(w, h, d, *color), (w, h, -d, *color), (w, -h, -d, *color)])

    # 顶面 (y = h)
    vertices.extend([(-w, h, -d, *color), (w, h, -d, *color), (w, h, d, *color)])
    vertices.extend([(w, h, d, *color), (-w, h, d, *color), (-w, h, -d, *color)])

    # 底面 (y = -h)
    vertices.extend([(-w, -h, -d, *color), (w, -h, -d, *color), (w, -h, d, *color)])
    vertices.extend([(w, -h, d, *color), (-w, -h, d, *color), (-w, -h, -d, *color)])

    return vertices


def generate_sphere(
        radius=0.5,
        rings=16,
        sectors=32,
        color_top=(1.0, 0.0, 0.0),
        color_bottom=(0.0, 0.0, 1.0)
):
    """生成球体顶点数据"""
    vertices = []
    for i in range(rings + 1):
        phi = math.pi * i / rings
        for j in range(sectors + 1):
            theta = 2 * math.pi * j / sectors
            x = radius * math.sin(phi) * math.cos(theta)
            y = radius * math.cos(phi)
            z = radius * math.sin(phi) * math.sin(theta)
            t = i / rings
            r = color_top[0] * (1 - t) + color_bottom[0] * t
            g = color_top[1] * (1 - t) + color_bottom[1] * t
            b = color_top[2] * (1 - t) + color_bottom[2] * t
            vertices.append((x, y, z, r, g, b))

    indices = []
    for i in range(rings):
        for j in range(sectors):
            a = i * (sectors + 1) + j
            b = a + 1
            c = (i + 1) * (sectors + 1) + j
            d = c + 1
            indices.extend([a, c, b, b, c, d])

    return [vertices[i] for i in indices]


def generate_human(
        torso_size=(0.3, 0.5, 0.2),
        head_radius=0.2,
        arm_length=0.4,
        arm_thickness=0.1,
        leg_length=0.6,
        leg_thickness=0.15,
        torso_color=(0.5, 0.5, 0.5),
        head_color=(1.0, 0.8, 0.6),
        arm_color=(0.0, 0.0, 1.0),
        leg_color=(0.0, 0.0, 1.0)
):
    """生成人体模型顶点数据"""
    vertices = []
    torso_w, torso_h, torso_d = torso_size

    # 躯干
    torso = generate_cuboid(torso_w, torso_h, torso_d, torso_color)
    vertices.extend(torso)

    # 头部
    head = generate_sphere(
        radius=head_radius,
        rings=10,
        sectors=20,
        color_top=head_color,
        color_bottom=head_color
    )
    head = [(x, y + torso_h / 2 + head_radius, z, *c) for x, y, z, *c in head]
    vertices.extend(head)

    # 手臂
    arm_w, arm_h, arm_d = arm_thickness, arm_length, arm_thickness
    # 左臂
    left_arm = generate_cuboid(arm_w, arm_h, arm_d, arm_color)
    left_arm = [(x - torso_w / 2 - arm_w / 2, y, z, *c) for x, y, z, *c in left_arm]
    vertices.extend(left_arm)
    # 右臂
    right_arm = generate_cuboid(arm_w, arm_h, arm_d, arm_color)
    right_arm = [(x + torso_w / 2 + arm_w / 2, y, z, *c) for x, y, z, *c in right_arm]
    vertices.extend(right_arm)

    # 腿部
    leg_w, leg_h, leg_d = leg_thickness, leg_length, leg_thickness
    # 左腿
    left_leg = generate_cuboid(leg_w, leg_h, leg_d, leg_color)
    left_leg = [(x - torso_w / 4, y - torso_h / 2 - leg_h / 2, z, *c) for x, y, z, *c in left_leg]
    vertices.extend(left_leg)
    # 右腿
    right_leg = generate_cuboid(leg_w, leg_h, leg_d, leg_color)
    right_leg = [(x + torso_w / 4, y - torso_h / 2 - leg_h / 2, z, *c) for x, y, z, *c in right_leg]
    vertices.extend(right_leg)

    return vertices


def save_to_file(filename, vertices):
    with open(filename, 'w') as f:
        for v in vertices:
            line = ", ".join(f"{x:.4f}" if isinstance(x, float) else str(x) for x in v)
            f.write(line + "\n")


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="生成3D形状顶点数据")
    parser.add_argument("shape", choices=["triangle", "cube", "sphere", "human"])
    parser.add_argument("-o", "--output", default="output.txt")

    # 通用参数
    parser.add_argument("--size", type=float, default=0.5)
    parser.add_argument("--color", nargs=3, type=float)

    # 球体参数
    parser.add_argument("--rings", type=int, default=16)
    parser.add_argument("--sectors", type=int, default=32)
    parser.add_argument("--color-top", nargs=3, type=float, default=[1, 0, 0])
    parser.add_argument("--color-bottom", nargs=3, type=float, default=[0, 0, 1])

    # 人体参数
    parser.add_argument("--torso-size", nargs=3, type=float, default=[0.3, 0.5, 0.2])
    parser.add_argument("--head-radius", type=float, default=0.2)
    parser.add_argument("--arm-length", type=float, default=0.4)
    parser.add_argument("--arm-thickness", type=float, default=0.1)
    parser.add_argument("--leg-length", type=float, default=0.6)
    parser.add_argument("--leg-thickness", type=float, default=0.15)
    parser.add_argument("--torso-color", nargs=3, type=float, default=[0.5, 0.5, 0.5])
    parser.add_argument("--head-color", nargs=3, type=float, default=[1.0, 0.8, 0.6])
    parser.add_argument("--arm-color", nargs=3, type=float, default=[0, 0, 1])
    parser.add_argument("--leg-color", nargs=3, type=float, default=[0, 0, 1])

    args = parser.parse_args()

    # 生成顶点数据
    if args.shape == "triangle":
        vertices = generate_triangle(args.size, args.color or (1, 0, 0))
    elif args.shape == "cube":
        vertices = generate_cuboid(args.size, args.size, args.size, args.color or (1, 0, 0))
    elif args.shape == "sphere":
        vertices = generate_sphere(
            radius=args.size,
            rings=args.rings,
            sectors=args.sectors,
            color_top=args.color_top,
            color_bottom=args.color_bottom
        )
    elif args.shape == "human":
        vertices = generate_human(
            torso_size=args.torso_size,
            head_radius=args.head_radius,
            arm_length=args.arm_length,
            arm_thickness=args.arm_thickness,
            leg_length=args.leg_length,
            leg_thickness=args.leg_thickness,
            torso_color=args.torso_color,
            head_color=args.head_color,
            arm_color=args.arm_color,
            leg_color=args.leg_color
        )

    save_to_file(args.output, vertices)
    print(f"已生成 {len(vertices)} 个顶点到 {args.output}")
```

step202:新建文件 C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\run_script.bat

```bash
@echo off
python generate_shape.py human --torso-size 0.25 0.6 0.15 --head-radius 0.18 --arm-length 0.35 --arm-thickness 0.08 --leg-length 0.7 --leg-thickness 0.12 --torso-color 0.7 0.7 0.7 --head-color 1.0 0.9 0.7 --arm-color 0.2 0.2 1.0 --leg-color 0.2 0.2 1.0 -o custom_human.txt
pause
```
或者使用默认数据也行

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python generate_shape.py human -o human.txt
已生成 1380 个顶点到 human.txt

```

end