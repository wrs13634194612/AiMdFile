C#和ASP.NET.Core构建RESTful.API和hello.world
1.提供RESTful API，管理用户数据，支持增删改查。
2.使用MySQL数据库存储用户信息。
3.配置详细的日志记录，包括HTTP请求/响应和自定义请求处理时间。
4.处理用户创建时的邮箱唯一性检查。
5.支持部分更新用户信息。
6.使用依赖注入管理数据库连接。
7.使用中间件进行请求日志记录和性能监控。
step0:sql

```bash
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    age INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

step1:添加依赖
C:\Users\wangrusheng\RiderProjects\WebApplication4\WebApplication4\WebApplication4.csproj

```bash
<Project Sdk="Microsoft.NET.Sdk.Web">

    <PropertyGroup>
        <TargetFramework>net9.0</TargetFramework>
        <Nullable>enable</Nullable>
        <ImplicitUsings>enable</ImplicitUsings>
    </PropertyGroup>

    <ItemGroup>
      <PackageReference Include="MySqlConnector" Version="2.4.0" />
    </ItemGroup>

</Project>

```

step2:配置日志和mysql
C:\Users\wangrusheng\RiderProjects\WebApplication4\WebApplication4\appsettings.json

```bash
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.AspNetCore.HttpLogging": "Information"
    }
  },
  "ConnectionStrings": {
    "MySQL": "Server=localhost;Database=db_school;User=root;Password=123456;"
  },
  "AllowedHosts": "*"
}

```


step3:实体类
C:\Users\wangrusheng\RiderProjects\WebApplication4\WebApplication4\User.cs

```csharp
namespace WebApplication4;

public class User
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public int Age { get; set; }
}
```


step4:接口controller
C:\Users\wangrusheng\RiderProjects\WebApplication4\WebApplication4\Program.cs

```csharp
using MySqlConnector;
using System.Data;
using Microsoft.AspNetCore.HttpLogging;
using Microsoft.Extensions.Logging;
using WebApplication4;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped(_ => 
    new MySqlConnection(builder.Configuration.GetConnectionString("MySQL")));
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

// 获取所有用户
app.MapGet("/users", async (MySqlConnection connection) =>
{
    var users = new List<User>();
    await connection.OpenAsync();
    using var cmd = new MySqlCommand("SELECT * FROM users", connection);
    using var reader = await cmd.ExecuteReaderAsync();
    
    while (await reader.ReadAsync())
    {
        users.Add(new User
        {
            Id = reader.GetInt32("id"),
            Name = reader.GetString("name"),
            Email = reader.GetString("email"),
            Age = reader.GetInt32("age")
        });
    }
    return Results.Ok(users);
});

// 创建用户
app.MapPost("/users", async (MySqlConnection connection, User user) =>
{
    await connection.OpenAsync();
    
    // 检查邮箱唯一性
    using var checkCmd = new MySqlCommand(
        "SELECT id FROM users WHERE email = @Email", connection);
    checkCmd.Parameters.AddWithValue("@Email", user.Email);
    var exists = await checkCmd.ExecuteScalarAsync();
    if (exists != null) return Results.Conflict("邮箱已存在");

    using var cmd = new MySqlCommand(
        "INSERT INTO users (name, email, age) VALUES (@Name, @Email, @Age); SELECT LAST_INSERT_ID();", 
        connection);
    
    cmd.Parameters.AddWithValue("@Name", user.Name);
    cmd.Parameters.AddWithValue("@Email", user.Email);
    cmd.Parameters.AddWithValue("@Age", user.Age);

    var newId = Convert.ToInt32(await cmd.ExecuteScalarAsync());
    return Results.Created($"/users/{newId}", new { Id = newId });
});

 
// 更新用户
app.MapPut("/users/{id}", async (MySqlConnection connection, int id, User user) =>
{
    await connection.OpenAsync();
    
    // 检查用户是否存在
    using var checkCmd = new MySqlCommand(
        "SELECT id FROM users WHERE id = @Id", connection);
    checkCmd.Parameters.AddWithValue("@Id", id);
    if (await checkCmd.ExecuteScalarAsync() == null)
        return Results.NotFound("用户不存在");

    // 构建动态更新语句
    var updates = new List<string>();
    var parameters = new List<MySqlParameter>();
    
    if (!string.IsNullOrEmpty(user.Name))
    {
        updates.Add("name = @Name");
        parameters.Add(new MySqlParameter("@Name", user.Name));
    }
    if (!string.IsNullOrEmpty(user.Email))
    {
        updates.Add("email = @Email");
        parameters.Add(new MySqlParameter("@Email", user.Email));
    }
    if (user.Age > 0)
    {
        updates.Add("age = @Age");
        parameters.Add(new MySqlParameter("@Age", user.Age));
    }

    if (updates.Count == 0)
        return Results.BadRequest("没有提供更新数据");

    var query = $"UPDATE users SET {string.Join(", ", updates)} WHERE id = @Id";
    parameters.Add(new MySqlParameter("@Id", id));

    using var cmd = new MySqlCommand(query, connection);
    cmd.Parameters.AddRange(parameters.ToArray());
    
    var affectedRows = await cmd.ExecuteNonQueryAsync();
    return Results.Ok(new { AffectedRows = affectedRows });
});

// 删除用户
app.MapDelete("/users/{id}", async (MySqlConnection connection, int id) =>
{
    await connection.OpenAsync();
    using var cmd = new MySqlCommand(
        "DELETE FROM users WHERE id = @Id", connection);
    cmd.Parameters.AddWithValue("@Id", id);
    
    var affectedRows = await cmd.ExecuteNonQueryAsync();
    return affectedRows > 0 
        ? Results.Ok(new { DeletedId = id }) 
        : Results.NotFound("用户不存在");
});

app.Run();
```

step5:postman 测试和验证

```bash


获取所有用户 http://localhost:5079/users

[
    {
        "id": 1,
        "name": "张三",
        "email": "zhangsan@example.com",
        "age": 28
    },
    {
        "id": 2,
        "name": "李四",
        "email": "lisi@outlook.com",
        "age": 35
    },
    {
        "id": 3,
        "name": "王五",
        "email": "wangwu@gmail.com",
        "age": 42
    }
]

创建用户

post:  http://localhost:5079/users

body:
{
    "name": "大聪明",
    "email": "zhugeliang@shu.com",
    "age": 54
}

result:
{
    "id": 4
}


更新用户：

put请求： http://localhost:5079/users/4
body：
{
    "name": "卧龙凤雏",
    "email": "zhugeliang@shu.com",
    "age": 54
}
result：
{
    "affectedRows": 1
}

删除用户 
delete：http://localhost:5079/users/2
result：
{
    "deletedId": 2
}
```
一个简单的mysql增删改查的接口就完成了。
////分割线 下面是helloworld版本 简化程序 直接新建web-empty 空项目，

step6:C:\Users\wangrusheng\RiderProjects\WebApplication4\WebApplication4\Program.cs

```csharp

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 定义 GET 根路径的响应
app.MapGet("/hello", () => "Hello World!");
 
app.Run();

postman请求示例 ：

http://localhost:5079/hello

result：
Hello World!
```

end