说明：
我计划用next.js+mysql实现增删改查
step1:sql

```sql
-- 用户表：存储用户基础信息
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '用户唯一标识',
    username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名（唯一）',
    email VARCHAR(100) NOT NULL UNIQUE COMMENT '邮箱（唯一）',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    status ENUM('active', 'banned', 'deleted') DEFAULT 'active' COMMENT '用户状态'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

step2:db.js C:\Users\wangrusheng\PycharmProjects\untitled4\src\lib\db.js

```typescript
import mysql from 'mysql2/promise';

const pool = mysql.createPool({
  host: 'localhost',
  user: 'root',
  password: '123456',
  database: 'db_school',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

export default pool;
```

step3:router.js C:\Users\wangrusheng\PycharmProjects\untitled4\src\app\api\users\route.js

```typescript
import { NextResponse } from 'next/server';
import pool from '@/lib/db';

// 获取所有用户或单个用户
export async function GET(request) {
  try {
    const { searchParams } = new URL(request.url);
    const id = searchParams.get('id');

    const connection = await pool.getConnection();

    if (id) {
      const [rows] = await connection.query(
        'SELECT * FROM users WHERE id = ?',
        [id]
      );
      connection.release();
      return NextResponse.json(rows[0] || {});
    } else {
      const [rows] = await connection.query('SELECT * FROM users');
      connection.release();
      return NextResponse.json(rows);
    }
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}

// 创建新用户
export async function POST(request) {
  try {
    const { username, email, status = 'active' } = await request.json();

    if (!username || !email) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    const connection = await pool.getConnection();
    const [result] = await connection.query(
      'INSERT INTO users (username, email, status) VALUES (?, ?, ?)',
      [username, email, status]
    );

    connection.release();
    return NextResponse.json({
      id: result.insertId,
      message: 'User created successfully'
    }, { status: 201 });

  } catch (error) {
    if (error.code === 'ER_DUP_ENTRY') {
      return NextResponse.json(
        { error: 'Username or email already exists' },
        { status: 409 }
      );
    }
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}

// 更新用户
export async function PUT(request) {
  try {
    const { id, ...updateData } = await request.json();

    if (!id) {
      return NextResponse.json(
        { error: 'User ID required' },
        { status: 400 }
      );
    }

    const connection = await pool.getConnection();
    const [result] = await connection.query(
      'UPDATE users SET ? WHERE id = ?',
      [updateData, id]
    );

    connection.release();

    if (result.affectedRows === 0) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    return NextResponse.json({ message: 'User updated successfully' });

  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}

// 删除用户
export async function DELETE(request) {
  try {
    const { searchParams } = new URL(request.url);
    const id = searchParams.get('id');

    if (!id) {
      return NextResponse.json(
        { error: 'User ID required' },
        { status: 400 }
      );
    }

    const connection = await pool.getConnection();
    const [result] = await connection.query(
      'DELETE FROM users WHERE id = ?',
      [id]
    );

    connection.release();

    if (result.affectedRows === 0) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      );
    }

    return NextResponse.json({ message: 'User deleted successfully' });

  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

step4:postman

```bash
 获取所有用户

  http://localhost:3002/api/users



[
    {
        "id": 1,
        "username": "张飞",
        "email": "zhangfei@example.com",
        "created_at": "2025-03-24T02:56:52.000Z",
        "updated_at": "2025-03-24T02:56:52.000Z",
        "status": "active"
    },
    {
        "id": 2,
        "username": "刘备",
        "email": "liubei@example.com",
        "created_at": "2025-03-24T02:56:52.000Z",
        "updated_at": "2025-03-24T02:56:52.000Z",
        "status": "active"
    },
    {
        "id": 3,
        "username": "关羽",
        "email": "guanyu@example.com",
        "created_at": "2025-03-24T02:56:52.000Z",
        "updated_at": "2025-03-24T02:56:52.000Z",
        "status": "active"
    }
]

获取单个用户

GET http://localhost:3000/api/users?id=1



{
    "id": 1,
    "username": "张飞",
    "email": "zhangfei@example.com",
    "created_at": "2025-03-24T02:56:52.000Z",
    "updated_at": "2025-03-24T02:56:52.000Z",
    "status": "active"
}


创建用户

 POST http://localhost:3000/api/users
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "status": "active"
}




{
    "id": 4,
    "message": "User created successfully"
}



更新用户


PUT http://localhost:3000/api/users
Content-Type: application/json

{
  "id": 1,
  "username": "new_username",
  "email": "new@example.com"
}
{
    "message": "User updated successfully"
}

删除用户

DELETE http://localhost:3000/api/users?id=1

{
    "message": "User deleted successfully"
}

```
step6:packages.json C:\Users\wangrusheng\PycharmProjects\untitled4\package.json

```bash
{
  "name": "untitled4",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "@heroicons/react": "^2.2.0",
    "mysql2": "^3.14.0",
    "next": "15.2.3",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@eslint/eslintrc": "^3",
    "@tailwindcss/postcss": "^4",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "autoprefixer": "^10.4.21",
    "eslint": "^9",
    "eslint-config-next": "15.2.3",
    "postcss": "^8.5.3",
    "tailwindcss": "^4.0.15",
    "typescript": "^5"
  }
}

```
总结：
1.就是一个基于nodejs弄出来的框架，写写api还可以，
2.千万别用来写界面，巨麻烦，我想弄个界面，对user表增删改查，居然让我创建一堆的文件
3.构建前端页面，比较慢，很麻烦，访问路由，居然需要自己手动拼接路径，体验感比angular差远了
3.总体来讲，我的评价是：写api不如fastapi，写界面不如angular，玩具一个
end