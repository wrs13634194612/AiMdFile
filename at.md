c#和form实现WebSocket在线聊天室
功能点

```bash
 
后端程序 (Program.cs)

1.WebSocket 聊天服务器核心功能
	a.管理客户端连接（ConnectionManager 类）
	b.支持公聊消息广播（所有用户可见）
	c.支持私聊消息（通过 @用户ID 格式指定接收方）
	d.自动广播用户加入/离开通知
	e.实时同步在线用户列表
	f.消息时间戳记录（精确到秒）

2.HTTP 服务增强功能
	a.提供 /hello 测试端点（经典 "Hello World"）
	b.完整的 WebSocket 握手协议支持
	c.详细的 HTTP 请求日志记录（包含请求头/响应体）
	d.性能监控中间件（记录请求处理耗时）

3.技术特性
	a.线程安全的并发连接管理（ConcurrentDictionary）
	b.JSON 序列化优化（驼峰式命名策略）
	c.WebSocket 消息完整接收保障（分帧消息合并）
	d.强类型消息模型（使用 C# 9 record 类型）


客户端程序 (Form1.cs)

1.用户界面功能
	a.双栏布局：左侧在线用户列表，右侧聊天区
	b.彩色消息区分（系统消息灰色、公聊蓝色、私聊紫色）
	c.自动滚动保持最新消息可见
	d.支持回车键快速发送（Shift+Enter 换行）
	e.用户列表动态更新（双击可查看用户 ID）

2.聊天功能实现
	a.自动生成唯一用户标识（GUID 简化形式）
	b.智能消息解析（自动识别私聊语法 @用户ID 内容）
	c.断线重连处理（需手动重开客户端）
	d.消息本地持久化（窗体关闭前保留聊天记录）

3.技术亮点
	a.异步 WebSocket 通信（非阻塞 UI）
	b.跨线程 UI 更新安全机制（Control.Invoke）
	c.JSON 消息高效解析（JsonDocument 流式处理）
	d.自适应窗体布局（百分比尺寸分配）

 
```
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/57319e497f2546ddaee405f6a5eda3af.png#pic_center)

step1: service socket
C:\Users\wangrusheng\RiderProjects\WebApplication4\WebApplication4\Program.cs

```csharp
using System.Collections.Concurrent;
using System.Net.WebSockets;
using System.Text;
using System.Text.Json;
using Microsoft.AspNetCore.HttpLogging;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSingleton<ConnectionManager>();
// 添加控制台日志和调试日志
builder.Logging.AddConsole()
    .AddDebug()
    .SetMinimumLevel(LogLevel.Debug);

// 配置HTTP日志
builder.Services.AddHttpLogging(logging =>
{
    logging.LoggingFields = HttpLoggingFields.All;
    logging.RequestHeaders.Add("sec-ch-ua");
    logging.ResponseHeaders.Add("MyResponseHeader");
    logging.MediaTypeOptions.AddText("application/json");
    logging.RequestBodyLogLimit = 4096;
    logging.ResponseBodyLogLimit = 4096;
});
var app = builder.Build();

// 使用请求日志中间件（必须放在其他中间件之前）
app.UseHttpLogging();

// 自定义请求日志中间件
app.Use(async (context, next) => {
    var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();
    
    logger.LogInformation($"Request Start: {context.Request.Method} {context.Request.Path}");
    var startTime = DateTime.UtcNow;
    
    await next();
    
    var elapsed = (DateTime.UtcNow - startTime).TotalMilliseconds;
    logger.LogInformation($"Request End: {context.Response.StatusCode} - {elapsed}ms");
});

// 启用WebSocket支持
app.UseWebSockets();

// 保留原有Hello World端点
app.MapGet("/hello", () => "Hello World!");

// WebSocket聊天室端点
app.Map("/ws", async context =>
{
    if (context.WebSockets.IsWebSocketRequest)
    {
        var webSocket = await context.WebSockets.AcceptWebSocketAsync();
        var manager = context.RequestServices.GetRequiredService<ConnectionManager>();
        await manager.HandleConnectionAsync(webSocket);
    }
    else
    {
        context.Response.StatusCode = StatusCodes.Status400BadRequest;
    }
});

await app.RunAsync();

public class ConnectionManager
{
    private readonly ConcurrentDictionary<string, ClientInfo> _clients = new();
    private readonly JsonSerializerOptions _jsonOptions = new() { PropertyNamingPolicy = JsonNamingPolicy.CamelCase };

    public async Task HandleConnectionAsync(WebSocket webSocket)
    {
        var clientId = Guid.NewGuid().ToString()[..8];
        var initMessage = await ReceiveFullMessage(webSocket);
        var initData = JsonSerializer.Deserialize<Dictionary<string, string>>(initMessage, _jsonOptions);
        var username = initData!["username"];

        var clientInfo = new ClientInfo(webSocket, username, clientId);
        _clients.TryAdd(clientId, clientInfo);

        await BroadcastSystemMessage($"{username} 进入聊天室");
        await BroadcastUserList();

        try
        {
            while (webSocket.State == WebSocketState.Open)
            {
                var message = await ReceiveFullMessage(webSocket);
                await ProcessMessage(clientInfo, message);
            }
        }
        finally
        {
            _clients.TryRemove(clientId, out _);
            await BroadcastSystemMessage($"{username} 已离开");
            await BroadcastUserList();
        }
    }

    private async Task ProcessMessage(ClientInfo sender, string rawMessage)
    {
        var message = JsonSerializer.Deserialize<Dictionary<string, object>>(rawMessage, _jsonOptions)!;
        var type = message["type"].ToString()!;

        switch (type)
        {
            case "public":
                await BroadcastPublicMessage(sender, message["content"].ToString()!);
                break;
            case "private":
                await SendPrivateMessage(
                    sender,
                    message["to"].ToString()!,
                    message["content"].ToString()!
                );
                break;
        }
    }

    private async Task BroadcastPublicMessage(ClientInfo sender, string content)
    {
        var message = new PublicMessage(
            "public",
            sender.ClientId,
            sender.Username,
            content,
            DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
        );

        await BroadcastMessage(JsonSerializer.Serialize(message, _jsonOptions));
    }

    private async Task SendPrivateMessage(ClientInfo sender, string receiverId, string content)
    {
        if (!_clients.TryGetValue(receiverId, out var receiver)) return;

        var message = new PrivateMessage(
            "private",
            sender.ClientId,
            receiverId,
            sender.Username,
            content,
            DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
        );

        var json = JsonSerializer.Serialize(message, _jsonOptions);
        await SendMessageAsync(sender.WebSocket, json);
        await SendMessageAsync(receiver.WebSocket, json);
    }

    private async Task BroadcastSystemMessage(string content)
    {
        var message = new SystemMessage(
            "system",
            content,
            DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")
        );

        await BroadcastMessage(JsonSerializer.Serialize(message, _jsonOptions));
    }

    private async Task BroadcastUserList()
    {
        var users = _clients.Values.Select(c => new UserInfo(c.ClientId, c.Username)).ToList();
        var message = new UserListMessage("user_list", users);
        await BroadcastMessage(JsonSerializer.Serialize(message, _jsonOptions));
    }

    private async Task BroadcastMessage(string message)
    {
        var buffer = Encoding.UTF8.GetBytes(message);
        foreach (var client in _clients.Values.Where(c => c.WebSocket.State == WebSocketState.Open))
        {
            await client.WebSocket.SendAsync(
                new ArraySegment<byte>(buffer),
                WebSocketMessageType.Text,
                true,
                CancellationToken.None
            );
        }
    }

    private static async Task<string> ReceiveFullMessage(WebSocket webSocket)
    {
        var buffer = new byte[1024 * 4];
        var message = new List<byte>();
        WebSocketReceiveResult result;

        do
        {
            result = await webSocket.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);
            message.AddRange(new ArraySegment<byte>(buffer, 0, result.Count));
        } while (!result.EndOfMessage);

        return Encoding.UTF8.GetString(message.ToArray());
    }

    private static async Task SendMessageAsync(WebSocket webSocket, string message)
    {
        var bytes = Encoding.UTF8.GetBytes(message);
        await webSocket.SendAsync(
            new ArraySegment<byte>(bytes),
            WebSocketMessageType.Text,
            true,
            CancellationToken.None
        );
    }

    private record ClientInfo(WebSocket WebSocket, string Username, string ClientId);
    private record UserInfo(string ClientId, string Username);
    private record PublicMessage(string Type, string From, string SenderName, string Content, string Timestamp);
    private record PrivateMessage(string Type, string From, string To, string SenderName, string Content, string Timestamp);
    private record SystemMessage(string Type, string Content, string Timestamp);
    private record UserListMessage(string Type, List<UserInfo> Users);
}
```

step1:client console
C:\Users\wangrusheng\RiderProjects\ConsoleApp4\ConsoleApp4\Program.cs

```csharp
using System.Net.WebSockets;
using System.Text;
using System.Text.Json;

class Program
{
    static async Task Main(string[] args)
    {
        Console.Write("请输入用户名: ");
        var username = Console.ReadLine()!;

        using var ws = new ClientWebSocket();
        await ws.ConnectAsync(new Uri("ws://localhost:5079/ws"), CancellationToken.None);

        // 发送初始化信息
        var initMessage = JsonSerializer.Serialize(new { username });
        await Send(ws, initMessage);

        // 启动接收任务
        var receiveTask = ReceiveMessages(ws);
        var sendTask = SendMessages(ws);

        await Task.WhenAll(receiveTask, sendTask);
    }

    static async Task SendMessages(ClientWebSocket ws)
    {
        var jsonOptions = new JsonSerializerOptions { PropertyNamingPolicy = JsonNamingPolicy.CamelCase };

        while (true)
        {
            Console.WriteLine("\n输入消息格式:\n1. 群发消息 → 直接输入\n2. 私聊消息 → @用户ID 内容\n3. 退出 → exit");
            var input = Console.ReadLine();

            if (string.IsNullOrEmpty(input)) continue;

            if (input.Equals("exit", StringComparison.OrdinalIgnoreCase))
            {
                await ws.CloseAsync(WebSocketCloseStatus.NormalClosure, null, CancellationToken.None);
                return;
            }

            if (input.StartsWith("@"))
            {
                var parts = input.Split(new[] { ' ' }, 2, StringSplitOptions.RemoveEmptyEntries);
                if (parts.Length == 2)
                {
                    var message = new
                    {
                        type = "private",
                        to = parts[0][1..],
                        content = parts[1]
                    };
                    await Send(ws, JsonSerializer.Serialize(message, jsonOptions));
                }
            }
            else
            {
                var message = new { type = "public", content = input };
                await Send(ws, JsonSerializer.Serialize(message, jsonOptions));
            }
        }
    }

    static async Task ReceiveMessages(ClientWebSocket ws)
    {
        var buffer = new byte[4096];
        var jsonOptions = new JsonSerializerOptions { PropertyNamingPolicy = JsonNamingPolicy.CamelCase };

        try
        {
            while (ws.State == WebSocketState.Open)
            {
                var result = await ws.ReceiveAsync(new ArraySegment<byte>(buffer), CancellationToken.None);
                if (result.MessageType == WebSocketMessageType.Text)
                {
                    var message = Encoding.UTF8.GetString(buffer, 0, result.Count);
                    HandleMessage(message, jsonOptions);
                }
            }
        }
        catch (WebSocketException)
        {
            Console.WriteLine("连接已断开");
        }
    }

    static void HandleMessage(string jsonMessage, JsonSerializerOptions options)
    {
        using var doc = JsonDocument.Parse(jsonMessage);
        var root = doc.RootElement;
        var type = root.GetProperty("type").GetString()!;

        switch (type)
        {
            case "user_list":
                PrintUserList(root);
                break;
            case "private":
                PrintPrivateMessage(root);
                break;
            case "public":
                PrintPublicMessage(root);
                break;
            case "system":
                PrintSystemMessage(root);
                break;
        }
    }

    static void PrintUserList(JsonElement element)
    {
        Console.WriteLine("\n" + new string('=', 50));
        Console.WriteLine("在线用户列表:");
        foreach (var user in element.GetProperty("users").EnumerateArray())
        {
            Console.WriteLine($"  {user.GetProperty("clientId").GetString()} | {user.GetProperty("username").GetString()}");
        }
        Console.WriteLine(new string('=', 50));
    }

    static void PrintPrivateMessage(JsonElement element)
    {
        Console.WriteLine($"\n[私聊] {element.GetProperty("senderName").GetString()} ({element.GetProperty("timestamp").GetString()}):");
        Console.WriteLine($"  {element.GetProperty("content").GetString()}");
    }

    static void PrintPublicMessage(JsonElement element)
    {
        Console.WriteLine($"\n[公共] {element.GetProperty("senderName").GetString()} ({element.GetProperty("timestamp").GetString()}):");
        Console.WriteLine($"  {element.GetProperty("content").GetString()}");
    }

    static void PrintSystemMessage(JsonElement element)
    {
        Console.WriteLine($"\n[系统] {element.GetProperty("timestamp").GetString()}:");
        Console.WriteLine($"  {element.GetProperty("content").GetString()}");
    }

    static async Task Send(ClientWebSocket ws, string message)
    {
        var bytes = Encoding.UTF8.GetBytes(message);
        await ws.SendAsync(new ArraySegment<byte>(bytes), WebSocketMessageType.Text, true, CancellationToken.None);
    }
}
```

step3:client form
C:\Users\wangrusheng\RiderProjects\WinFormsApp33\WinFormsApp33\Form1.cs

```csharp
using System;
using System.Drawing;
using System.Net.WebSockets;
using System.Text;
using System.Text.Json;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;
namespace WinFormsApp33
{
    

    public partial class Form1 : Form
    { 
    
     
        private readonly ClientWebSocket _ws = new ClientWebSocket();
        private readonly JsonSerializerOptions _jsonOptions = new JsonSerializerOptions
        {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };

        // 控件声明
        private ListView lstUsers;
        private RichTextBox txtChat;
        private TextBox txtInput;
        private Button btnSend;

        // 用户信息
        private readonly string _clientId;
        private readonly string _username;

        public Form1()
        {
            // 生成固定用户信息
            _clientId = Guid.NewGuid().ToString("N")[..6];
            _username = $"用户_{_clientId}";

            InitializeComponent();
            SetupUI();
            this.Load += MainForm_Load;
        }

        private void SetupUI()
        {
            // 窗体设置
            this.Text = $"{_username} - 聊天客户端";
            this.Size = new Size(800, 600);
            this.StartPosition = FormStartPosition.CenterScreen;
            this.FormClosing += MainForm_FormClosing;

            // 主布局面板
            var mainTable = new TableLayoutPanel
            {
                Dock = DockStyle.Fill,
                ColumnCount = 2,
                RowCount = 1
            };
            mainTable.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 25F));
            mainTable.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 75F));

            // 左侧用户列表
            var userPanel = new GroupBox
            {
                Text = "在线用户",
                Dock = DockStyle.Fill
            };
            lstUsers = new ListView
            {
                View = View.Details,
                FullRowSelect = true,
                Dock = DockStyle.Fill
            };
            lstUsers.Columns.Add("ID", 80);
            lstUsers.Columns.Add("用户名", 120);
            userPanel.Controls.Add(lstUsers);
            mainTable.Controls.Add(userPanel, 0, 0);

            // 右侧聊天区
            var chatPanel = new TableLayoutPanel
            {
                Dock = DockStyle.Fill,
                RowCount = 2
            };
            chatPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 85F));
            chatPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 15F));

            // 聊天记录显示
            txtChat = new RichTextBox
            {
                Dock = DockStyle.Fill,
                ReadOnly = true,
                BackColor = Color.White,
                Font = new Font("微软雅黑", 10)
            };
            chatPanel.Controls.Add(txtChat, 0, 0);

            // 输入区
            var inputPanel = new Panel
            {
                Dock = DockStyle.Fill
            };
            txtInput = new TextBox
            {
                Dock = DockStyle.Fill,
                Multiline = true,
                Font = new Font("微软雅黑", 10),
                AcceptsReturn = true
            };
            txtInput.KeyDown += TxtInput_KeyDown;
            
            btnSend = new Button
            {
                Text = "发送",
                Dock = DockStyle.Right,
                Width = 80,
                Font = new Font("微软雅黑", 10)
            };
            btnSend.Click += BtnSend_Click;
            
            inputPanel.Controls.Add(btnSend);
            inputPanel.Controls.Add(txtInput);
            chatPanel.Controls.Add(inputPanel, 0, 1);

            mainTable.Controls.Add(chatPanel, 1, 0);

            this.Controls.Add(mainTable);
        }

        private async void MainForm_Load(object sender, EventArgs e)
        {
            await ConnectToServer();
        }

        private async Task ConnectToServer()
        {
            try
            {
                await _ws.ConnectAsync(new Uri("ws://localhost:5079/ws"), CancellationToken.None);
                var initMessage = JsonSerializer.Serialize(new { username = _username });
                await SendMessage(initMessage);
                _ = Task.Run(() => ReceiveMessages());
            }
            catch (Exception ex)
            {
                MessageBox.Show($"连接失败: {ex.Message}");
                this.Close();
            }
        }

        private async Task SendMessage(string message)
        {
            var bytes = Encoding.UTF8.GetBytes(message);
            await _ws.SendAsync(new ArraySegment<byte>(bytes), 
                WebSocketMessageType.Text, 
                true, 
                CancellationToken.None);
        }

        private async Task ReceiveMessages()
        {
            var buffer = new byte[4096];
            try
            {
                while (_ws.State == WebSocketState.Open)
                {
                    var result = await _ws.ReceiveAsync(new ArraySegment<byte>(buffer), 
                        CancellationToken.None);
                    if (result.MessageType == WebSocketMessageType.Text)
                    {
                        var message = Encoding.UTF8.GetString(buffer, 0, result.Count);
                        ProcessMessage(message);
                    }
                }
            }
            catch (Exception ex)
            {
                AppendSystemMessage($"连接异常: {ex.Message}");
            }
        }

        private void ProcessMessage(string jsonMessage)
        {
            using var doc = JsonDocument.Parse(jsonMessage);
            var root = doc.RootElement;
            var type = root.GetProperty("type").GetString();

            this.Invoke((MethodInvoker)delegate
            {
                try
                {
                    switch (type)
                    {
                        case "user_list":
                            UpdateUserList(root);
                            break;
                        case "public":
                        case "private":
                        case "system":
                            AppendChatMessage(root);
                            break;
                    }
                }
                catch (Exception ex)
                {
                    AppendSystemMessage($"消息处理错误: {ex.Message}");
                }
            });
        }

        private void UpdateUserList(JsonElement element)
        {
            lstUsers.BeginUpdate();
            lstUsers.Items.Clear();
            
            foreach (var user in element.GetProperty("users").EnumerateArray())
            {
                var item = new ListViewItem(user.GetProperty("clientId").GetString());
                item.SubItems.Add(user.GetProperty("username").GetString());
                lstUsers.Items.Add(item);
            }
            
            lstUsers.EndUpdate();
        }

        private void AppendChatMessage(JsonElement message)
        {
            var type = message.GetProperty("type").GetString();
            var timestamp = message.GetProperty("timestamp").GetString();
            var content = message.GetProperty("content").GetString();
            var senderName = message.TryGetProperty("senderName", out var sn) ? sn.GetString() : "系统";

            var prefix = type switch
            {
                "public" => $"[{timestamp}] {senderName}：",
                "private" => $"[{timestamp}] 私聊来自 {senderName}：",
                "system" => $"[系统通知 {timestamp}] ",
                _ => ""
            };

            txtChat.SelectionColor = type switch
            {
                "public" => Color.Blue,
                "private" => Color.Purple,
                "system" => Color.Gray,
                _ => Color.Black
            };

            txtChat.AppendText(prefix + content + "\n");
            txtChat.ScrollToCaret();
        }

        private void AppendSystemMessage(string message)
        {
            txtChat.SelectionColor = Color.Gray;
            txtChat.AppendText($"[系统] {DateTime.Now:HH:mm:ss} {message}\n");
            txtChat.ScrollToCaret();
        }

        private async void BtnSend_Click(object sender, EventArgs e)
        {
            await SendMessageAsync();
        }

        private async void TxtInput_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.KeyCode == Keys.Enter && !e.Shift)
            {
                e.SuppressKeyPress = true;
                await SendMessageAsync();
            }
        }

        private async Task SendMessageAsync()
        {
            if (string.IsNullOrWhiteSpace(txtInput.Text)) return;

            var message = txtInput.Text.Trim();
            txtInput.Clear();

            try
            {
                if (message.StartsWith("@"))
                {
                    var parts = message.Split(new[] { ' ' }, 2);
                    if (parts.Length == 2)
                    {
                        var msg = new
                        {
                            type = "private",
                            to = parts[0][1..],
                            content = parts[1]
                        };
                        await SendMessage(JsonSerializer.Serialize(msg, _jsonOptions));
                    }
                }
                else
                {
                    var msg = new { type = "public", content = message };
                    await SendMessage(JsonSerializer.Serialize(msg, _jsonOptions));
                }
            }
            catch (Exception ex)
            {
                AppendSystemMessage($"发送失败: {ex.Message}");
            }
        }

        private void MainForm_FormClosing(object sender, FormClosingEventArgs e)
        {
            if (_ws?.State == WebSocketState.Open)
            {
                _ws.CloseAsync(WebSocketCloseStatus.NormalClosure, 
                    "Closing", 
                    CancellationToken.None).Wait();
            }
        }
    }
}
```

end