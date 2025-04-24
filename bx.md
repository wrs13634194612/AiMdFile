说明
form+openGL绘制三角形和立方体
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3ef47fd94b484afb85c73332c11a102e.png#pic_center)

立方体
step1：
C:\Users\wangrusheng\RiderProjects\WinFormsApp7\WinFormsApp7\Form1.cs

```csharp
using Silk.NET.GLFW;
using Silk.NET.OpenGL;
using System;
using System.Runtime.InteropServices;
using System.Windows.Forms;
using System.Drawing;
namespace WinFormsApp7;

public unsafe partial class Form1 : Form
{
    private GL gl;
    private Glfw glfw;
    private WindowHandle* glfwWindow;
    private System.Windows.Forms.Panel glPanel;
    private uint shaderProgram;
    private uint vao;
    private uint vbo;
      private DateTime _startTime = DateTime.Now;
    [DllImport("user32.dll")]
    private static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);

    [DllImport("user32.dll")]
    private static extern bool SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter, 
        int X, int Y, int cx, int cy, SetWindowPosFlags uFlags);
    [DllImport("glfw3.dll", CallingConvention = CallingConvention.Cdecl)]
    private static extern IntPtr glfwGetWin32Window(WindowHandle* window);

    [Flags]
    private enum SetWindowPosFlags : uint
    {
        FrameChanged = 0x0020
    }

    // 其他代码保持不变...
    


    public Form1()
    {
        InitializeComponent();
        InitializeOpenGLPanel();
        this.Load += Form1_Load;
    }
    private void Form1_Load(object? sender, EventArgs e)
    {
        SetupGLFWWindow();
        InitializeOpenGL();
        SetupCube();
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
            glfw.ShowWindow(glfwWindow); // 显式显示窗口

            nint hwnd = glfwGetWin32Window(glfwWindow);
            SetParent(hwnd, glPanel.Handle);
            SetWindowPos(hwnd, IntPtr.Zero, 0, 0, 
                glPanel.Width, glPanel.Height, SetWindowPosFlags.FrameChanged);

            glfw.MakeContextCurrent(glfwWindow);
            glfw.SwapInterval(1); // 启用垂直同步
        }
    }



    
    private void InitializeOpenGL()
    {
        gl = GL.GetApi(glfw.GetProcAddress);
        gl.Viewport(0, 0, (uint)glPanel.Width, (uint)glPanel.Height);
       gl.Enable(EnableCap.DepthTest);
        // 创建着色器程序
        const string vertexShaderSource = @"#version 330 core
            layout (location = 0) in vec3 aPos;
            layout (location = 1) in vec3 aColor;
            out vec3 ourColor;
			   uniform float time;
            void main()
            {
           
                mat4 rotateX = mat4(
                    1.0, 0.0, 0.0, 0.0,
                    0.0, cos(time), -sin(time), 0.0,
                    0.0, sin(time), cos(time), 0.0,
                    0.0, 0.0, 0.0, 1.0
                );
                mat4 rotateY = mat4(
                    cos(time), 0.0, sin(time), 0.0,
                    0.0, 1.0, 0.0, 0.0,
                    -sin(time), 0.0, cos(time), 0.0,
                    0.0, 0.0, 0.0, 1.0
                );
                gl_Position = rotateY * rotateX * vec4(aPos, 1.0);
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

        // 设置渲染定时器
        var timer = new System.Windows.Forms.Timer { Interval = 16 };
        timer.Tick += (s, e) => Render();
        timer.Start();
    }
    // 其他代码保持不变...
    
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

        gl.ClearColor(0.3f, 0.3f, 0.3f, 1.0f);
       gl.Clear(ClearBufferMask.ColorBufferBit | ClearBufferMask.DepthBufferBit);

        gl.UseProgram(shaderProgram);
		
		     float time = (float)(DateTime.Now - _startTime).TotalSeconds;
        gl.Uniform1(gl.GetUniformLocation(shaderProgram, "time"), time);
		
		
        gl.BindVertexArray(vao);
        gl.DrawArrays(PrimitiveType.Triangles, 0, 36);

        unsafe
        {
            glfw.SwapBuffers(glfwWindow);
        }
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
    
    protected override void OnFormClosing(FormClosingEventArgs e)
    {
        base.OnFormClosing(e);
        
        // 清理资源
        if (gl != null)
        {
            gl.DeleteBuffer(vbo);
            gl.DeleteVertexArray(vao);
            gl.DeleteProgram(shaderProgram);
        }
        glfw?.Terminate();
    }



}
```

step2:
C:\Users\wangrusheng\RiderProjects\WinFormsApp7\WinFormsApp7\WinFormsApp7.csproj 

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
      <PackageReference Include="Silk.NET.Core" Version="2.22.0" />
      <PackageReference Include="Silk.NET.GLFW" Version="2.22.0" />
      <PackageReference Include="Silk.NET.OpenGL" Version="2.22.0" />
    </ItemGroup>

</Project>
```

end

///我是分割线
三角形：
step1:

```csharp


using Silk.NET.GLFW;
using Silk.NET.OpenGL;
using System;
using System.Runtime.InteropServices;
using System.Windows.Forms;
using System.Drawing;
namespace WinFormsApp7;

public unsafe partial class Form1 : Form
{
    private GL gl;
    private Glfw glfw;
    private WindowHandle* glfwWindow;
    private System.Windows.Forms.Panel glPanel;
    private uint shaderProgram;
    private uint vao;
    private uint vbo;
    
    [DllImport("user32.dll")]
    private static extern IntPtr SetParent(IntPtr hWndChild, IntPtr hWndNewParent);

    [DllImport("user32.dll")]
    private static extern bool SetWindowPos(IntPtr hWnd, IntPtr hWndInsertAfter, 
        int X, int Y, int cx, int cy, SetWindowPosFlags uFlags);
    [DllImport("glfw3.dll", CallingConvention = CallingConvention.Cdecl)]
    private static extern IntPtr glfwGetWin32Window(WindowHandle* window);

    [Flags]
    private enum SetWindowPosFlags : uint
    {
        FrameChanged = 0x0020
    }

    // 其他代码保持不变...
    


    public Form1()
    {
        InitializeComponent();
        InitializeOpenGLPanel();
        this.Load += Form1_Load;
    }
    private void Form1_Load(object? sender, EventArgs e)
    {
        SetupGLFWWindow();
        InitializeOpenGL();
        SetupTriangle();
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
            glfw.ShowWindow(glfwWindow); // 显式显示窗口

            nint hwnd = glfwGetWin32Window(glfwWindow);
            SetParent(hwnd, glPanel.Handle);
            SetWindowPos(hwnd, IntPtr.Zero, 0, 0, 
                glPanel.Width, glPanel.Height, SetWindowPosFlags.FrameChanged);

            glfw.MakeContextCurrent(glfwWindow);
            glfw.SwapInterval(1); // 启用垂直同步
        }
    }



    
    private void InitializeOpenGL()
    {
        gl = GL.GetApi(glfw.GetProcAddress);
        gl.Viewport(0, 0, (uint)glPanel.Width, (uint)glPanel.Height);

        // 创建着色器程序
        const string vertexShaderSource = @"#version 330 core
            layout (location = 0) in vec3 aPos;
            layout (location = 1) in vec3 aColor;
            out vec3 ourColor;
            void main()
            {
                gl_Position = vec4(aPos, 1.0);
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

        // 设置渲染定时器
        var timer = new System.Windows.Forms.Timer { Interval = 16 };
        timer.Tick += (s, e) => Render();
        timer.Start();
    }
    // 其他代码保持不变...
    
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
    private void Render()
    {
        if (gl == null) return;

        gl.ClearColor(0.3f, 0.3f, 0.3f, 1.0f);
        gl.Clear(ClearBufferMask.ColorBufferBit);

        gl.UseProgram(shaderProgram);
        gl.BindVertexArray(vao);
        gl.DrawArrays(PrimitiveType.Triangles, 0, 3);

        unsafe
        {
            glfw.SwapBuffers(glfwWindow);
        }
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
    
    protected override void OnFormClosing(FormClosingEventArgs e)
    {
        base.OnFormClosing(e);
        
        // 清理资源
        if (gl != null)
        {
            gl.DeleteBuffer(vbo);
            gl.DeleteVertexArray(vao);
            gl.DeleteProgram(shaderProgram);
        }
        glfw?.Terminate();
    }



}
```

end