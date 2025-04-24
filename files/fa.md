说明：
我计划用nodejs来实现操作mysql中的表，增删改查的功能，
包括：
1.连接mysql数据库，
2.创建数据表，
3.写增删改查的方法，
4.postman里面去请求访问数据，
5.在angular里面，用表格展示所有数据，添加数据，删除数据等功能

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a147251ed9ff4a21a0118d9a4f8c84fd.png#pic_center)

step0: 打开终端，进入项目目录untitled4，使用npm导入模块

```bash
npm install express body-parser --save
npm install mysql2 dotenv --save     
 npm install cors --save  
 npm install dotenv --save        
 npm install mysql --save  
```

step1:在package.json里面，验证刚刚导入的模块C:\Users\Administrator\WebstormProjects\untitled4\package.json

```bash
  "body-parser": "^1.20.3",
    "cors": "^2.8.5",
    "dotenv": "^16.4.7",
    "express": "^4.21.2",
    "mysql": "^2.18.1",
    "mysql2": "^3.12.0",
```

step2:配置数据库参数，用户名，密码，C:\Users\Administrator\WebstormProjects\untitled4\.env

```bash
# .env 文件内容（放在项目根目录）
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=123456
DB_NAME=db_spring

```

step3:连接数据库 C:\Users\Administrator\WebstormProjects\untitled4\db.js

```javascript
// db.js
const mysql = require('mysql2');
require('dotenv').config();

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  port: process.env.DB_PORT,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

module.exports = pool.promise(); // 返回 Promise 风格的池对象

```

step4:写一个测试类，看看数据库是否连接成功C:\Users\Administrator\WebstormProjects\untitled4\test-connection.js

```javascript
// test-connection.js
const db = require('./db');

async function testConnection() {
  try {
    // 执行简单查询
    const [rows] = await db.query('SELECT 1 + 1 AS solution');
    console.log('✅ 连接成功！结果:', rows[0][0]);
  } catch (err) {
    console.error('❌ 连接失败:', err);
  } finally {
    // 关闭连接池
    await db.end();
  }
}

testConnection();

```

step5:在终端运行刚刚的 test-connection.js

```bash
PS C:\Users\Administrator\WebstormProjects\untitled4> node test-connection.js
✅ 连接成功！结果: undefined

```

step6:连接上数据库以后，开始创建数据库表，C:\Users\Administrator\WebstormProjects\untitled4\crud.js

```javascript
const db = require('./db');

// 创建表（仅需运行一次）
async function initializeTable() {
  const sql = `
    CREATE TABLE IF NOT EXISTS users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(100) NOT NULL,
      email VARCHAR(100) UNIQUE NOT NULL,
      age INT DEFAULT 0
    );
  `;
  await db.query(sql);
  console.log('✅ 用户表已创建！');
}

// 查询所有用户
async function getAllUsers() {
  const [rows] = await db.query('SELECT * FROM users');
  return rows;
}

// 查询单个用户（根据ID）
async function getUserById(id) {
  const [rows] = await db.query('SELECT * FROM users WHERE id = ?', [id]);
  return rows[0]; // 返回单个用户对象或 null
}

// 插入用户
async function createUser(user) {
  const [result] = await db.query(
    'INSERT INTO users (name, email, age) VALUES (?, ?, ?)',
    [user.name, user.email, user.age]
  );
  return { id: result.insertId, ...user };
}

// 更新用户
async function updateUser(id, user) {
  const [result] = await db.query(
    'UPDATE users SET name = ?, email = ?, age = ? WHERE id = ?',
    [user.name, user.email, user.age, id]
  );
  return { id, ...user };
}

// 删除用户
async function deleteUser(id) {
  const [result] = await db.query('DELETE FROM users WHERE id = ?', [id]);
  return { affectedRows: result.affectedRows };
}

module.exports = {
  initializeTable,
  getAllUsers,
  getUserById,
  createUser,
  updateUser,
  deleteUser,
};

```

step7:数据库表，创建完成后，开始写后端请求类，处理get和post请求C:\Users\Administrator\WebstormProjects\untitled4\spring.js

```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const crud = require('./crud');

const app = express();
app.use(bodyParser.json());
// 开启跨域支持（允许所有来源访问）
app.use(cors());

// 初始化表（运行前执行一次）
(async () => {
  await crud.initializeTable();
})();

// API 路由
app.get('/users', async (req, res) => {
  const users = await crud.getAllUsers();
  res.json(users);
});

// 新增：根据ID查询用户
app.get('/users/:id', async (req, res) => {
  const id = parseInt(req.params.id);
  try {
    const user = await crud.getUserById(id);
    if (user) {
      res.status(200).json(user);
    } else {
      res.status(404).json({ message: '用户不存在' });
    }
  } catch (err) {
    console.error('查询用户错误:', err);
    res.status(500).json({ error: '服务器内部错误' });
  }
});

app.post('/users', async (req, res) => {
  const newUser = await crud.createUser(req.body);
  res.status(201).json(newUser);
});

app.put('/users/:id', async (req, res) => {
  const id = parseInt(req.params.id);
  const updatedUser = await crud.updateUser(id, req.body);
  res.json(updatedUser);
});

app.delete('/users/:id', async (req, res) => {
  const id = parseInt(req.params.id);
  const result = await crud.deleteUser(id);
  res.json({ message: `Deleted ${result.affectedRows} user(s)` });
});

// 启动服务器
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Server is running on port ${PORT}`);
});

```

step8:到这里，咱们的nodejs，后端部分和mysql部分就写完了，接下来，用postman测试，访问请求接口，看看是否能成功调用mysql表中的数据

查询具体数据

```bash
get请求 http://192.168.0.113:3000/users/3
{
    "id": 3,
    "name": "张仲景",
    "email": "zhangsan@example.com",
    "age": 108
}
```
查询所有

```bash
get请求 http://192.168.0.113:3000/users
[
    {
        "id": 1,
        "name": "王婆婆",
        "email": "wangwu@example.com",
        "age": 30
    },
    {
        "id": 3,
        "name": "张仲景",
        "email": "zhangsan@example.com",
        "age": 108
    },
    {
        "id": 13,
        "name": "廖化",
        "email": "liaohua@example.com",
        "age": 28
    },
    {
        "id": 14,
        "name": "黄波",
        "email": "bozhong@example.com",
        "age": 23
    },
    {
        "id": 15,
        "name": "周瑜",
        "email": "zhaouyun@example.com",
        "age": 52
    },
    {
        "id": 16,
        "name": "刘能",
        "email": "znengoliu@example.com",
        "age": 65
    }
]
```
新增数据

```bash
post请求，http://192.168.0.113:3000/users
{
    "id": 17,
    "name": "仲康",
    "email": "zhankangn@example.com",
    "age": 16
}
```
删除数据

```bash
delete请求，http://192.168.0.113:3000/users/17
{
    "message": "Deleted 1 user(s)"
}
```
修改数据

```bash
http://192.168.0.113:3000/users/16
{
    "id": 16,
    "name": "康男",
    "email": "zjmnangn@example.com",
    "age": 19
}
```

step9:postman测试通过，表示后端完全写完了，接下来写angular前端代码

step10:在app.config里面配置provideHttpClient
C:\Users\Administrator\WebstormProjects\untitled4\src\app\app.config.ts
```javascript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';
import { provideIndexedDb } from 'ngx-indexed-db';
import { DB_CONFIG } from './db.config';

import { provideAnimations } from '@angular/platform-browser/animations';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';



export const appConfig: ApplicationConfig = {
  providers: [provideIndexedDb(DB_CONFIG),provideHttpClient(),provideZoneChangeDetection({ eventCoalescing: true }), provideRouter(routes),provideAnimations(), provideAnimationsAsync()]
};

```

step11:写一个http的封装类，增删改查的方法，
C:\Users\Administrator\WebstormProjects\untitled4\src\app\springboot\user.service.ts
```javascript
// src/app/user.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private apiUrl = 'http://localhost:3000/users';

  constructor(private http: HttpClient) {}

  // 查询所有用户
  getAllUsers(): Observable<any> {
    return this.http.get(this.apiUrl);
  }

  // 创建用户
  createUser(user: any): Observable<any> {
    return this.http.post(this.apiUrl, user);
  }

  // 获取单个用户（新增）
  getUserById(id: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/${id}`);
  }

  // 更新用户
  updateUser(id: number, user: any): Observable<any> {
    return this.http.put(`${this.apiUrl}/${id}`, user);
  }

  // 删除用户
  deleteUser(id: number): Observable<any> {
    return this.http.delete(`${this.apiUrl}/${id}`);
  }
}

```

step12:C:\Users\Administrator\WebstormProjects\untitled4\src\app\springboot\springboot.component.ts

```javascript
// src/app/springboot.component.ts
import { Component, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { NgForm } from '@angular/forms';
import {NgForOf} from '@angular/common';
import { UserService } from './user.service';

@Component({
  selector: 'app-springboot',
  imports: [
    FormsModule,
    NgForOf
  ],
  templateUrl: './springboot.component.html',
  styleUrls: ['./springboot.component.css']
})
export class SpringbootComponent implements OnInit {
  users: any[] = [];
  isLoading = false;
  error: string = '';
  selectedUser: any = {};
  isEditMode: boolean = false;

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.isLoading = true;
    this.userService.getAllUsers().subscribe(
      (data) => {
        this.users = data;
        this.isLoading = false;
      },
      (err) => {
        this.error = '加载用户失败';
        this.isLoading = false;
      }
    );
  }

  // 添加/修改用户
  submitForm(userForm: NgForm): void {
    if (userForm.invalid) return;

    if (this.isEditMode) {
      this.updateUser(this.selectedUser.id, this.selectedUser);
    } else {
      this.createUser(this.selectedUser);
    }

    // 重置表单和状态
    userForm.reset();
    this.selectedUser = {};
    this.isEditMode = false;
  }

  // 创建新用户
  createUser(user: any): void {
    this.userService.createUser(user).subscribe(
      (newUser) => {
        this.users.push(newUser);
      },
      (err) => {
        this.error = '创建用户失败';
      }
    );
  }

  // 编辑用户
  editUser(id: number): void {
    this.userService.getUserById(id).subscribe(
      (user) => {
        this.selectedUser = user;
        this.isEditMode = true;
      },
      (err) => {
        this.error = '加载用户失败';
      }
    );
  }

  // 更新用户
  updateUser(id: number, user: any): void {
    this.userService.updateUser(id, user).subscribe(() => {
      // 替换原用户数据
      this.users = this.users.map(u =>
        u.id === id ? { ...u, name: user.name, email: user.email, age: user.age } : u
      );
    });
  }

  // 删除用户
  deleteUser(id: number): void {
    this.userService.deleteUser(id).subscribe(() => {
      this.users = this.users.filter(u => u.id !== id);
    });
  }

  // 重置表单
  resetForm(): void {
    this.selectedUser = {};
    this.isEditMode = false;
  }
}

```

step13: C:\Users\Administrator\WebstormProjects\untitled4\src\app\springboot\springboot.component.html

```xml
<div class="container">
  <h2>用户管理</h2>

  <!-- 添加/编辑用户表单 -->
  <div class="card mb-3">
    <div class="card-header">
      <h3 class="card-title">添加/编辑用户</h3>
    </div>
    <div class="card-body">
      <form #userForm="ngForm" (ngSubmit)="submitForm(userForm)">
        <div class="form-group">
          <input type="text" class="form-control" name="name" [(ngModel)]="selectedUser.name" required>
        </div>
        <div class="form-group">
          <input type="email" class="form-control" name="email" [(ngModel)]="selectedUser.email" required>
        </div>
        <div class="form-group">
          <input type="number" class="form-control" name="age" [(ngModel)]="selectedUser.age" min="0">
        </div>
        <button type="submit" class="btn btn-modern">
          {{ isEditMode ? '保存' : '添加' }}
        </button>
        <button type="reset" class="btn btn-secondary-modern" (click)="resetForm()">
          清空
        </button>
      </form>
    </div>
  </div>

  <!-- 用户列表 -->
  <table class="table table-striped">
    <thead>
    <tr>
      <th>ID</th>
      <th>姓名</th>
      <th>邮箱</th>
      <th>年龄</th>
      <th>操作</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let user of users">
      <td>{{ user.id }}</td>
      <td>{{ user.name }}</td>
      <td>{{ user.email }}</td>
      <td>{{ user.age }}</td>
      <td>
        <button (click)="editUser(user.id)" class="btn btn-warning btn-sm">
          修改
        </button>
        <button (click)="deleteUser(user.id)" class="btn btn-danger btn-sm">
          删除
        </button>
      </td>
    </tr>
    </tbody>
  </table>
</div>

```

step14: C:\Users\Administrator\WebstormProjects\untitled4\src\app\springboot\springboot.component.css

```css
/* 容器全局样式 */
.container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 12px;  /* 整体圆角 */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);  /* 柔和阴影 */
}

/* 卡片组件增强 */
.card {
  border: none;
  border-radius: 10px;  /* 卡片圆角 */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);  /* 层级阴影 */
  background: white;
}
.card-header {
  background: linear-gradient(135deg, #6c5ce7, #4b4ced);  /* 渐变背景 */
  color: white;
  border-radius: 10px 10px 0 0;  /* 顶部圆角 */
}

/* 按钮现代化改造 */
.btn {
  border: none;
  border-radius: 8px;  /* 统一圆角 */
  padding: 10px 25px;
  transition: all 0.3s ease;  /* 平滑动效 */
  font-weight: 500;
}
.btn-primary {
  background: linear-gradient(45deg, #4b4ced, #6c5ce7);  /* 渐变背景 */
  color: white;
  box-shadow: 0 4px 6px rgba(75, 76, 237, 0.2);  /* 投影增强 */
}
.btn-primary:hover {
  transform: translateY(-2px);  /* 悬停微抬升 */
  box-shadow: 0 6px 12px rgba(75, 76, 237, 0.3);
}
.btn-secondary {
  background: #e9ecef;
  color: #495057;
}
.btn-sm {
  padding: 6px 15px;
  font-size: 0.9em;
}

/* 表格现代化设计 */
.table {
  border-collapse: separate;
  border-spacing: 0;
  background: white;
  border-radius: 10px;  /* 表格整体圆角 */
  overflow: hidden;  /* 隐藏溢出圆角 */
}
.table thead th {
  background: #f8f9fa;
  color: #495057;
  font-weight: 600;
  border-bottom: 2px solid #e9ecef;  /* 表头分割线 */
}
.table tbody tr:hover {
  background: #f8f9fa;  /* 行悬停高亮 */
}
.table-striped tbody tr:nth-of-type(odd) {
  background-color: rgba(241, 243, 245, 0.5);  /* 柔和条纹 */
}
.table td, .table th {
  padding: 15px;
  border-top: 1px solid #e9ecef;  /* 单元格分割线 */
}

/* 输入框微调 */
.form-control {
  border-radius: 6px;  /* 输入框圆角 */
  border: 1px solid #dee2e6;
  padding: 10px 15px;
}

```

end