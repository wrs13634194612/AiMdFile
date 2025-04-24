说明：fastapi+angular评论和回复

效果图:
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/86cbfc3c35ce4ef9953c9a3ec509dd0f.png#pic_center)

step1:sql

```sql
show databases;
DROP TABLE users;
SHOW CREATE TABLE db_school.users;
show tables;
use db_school;
SELECT * FROM db_school.jewelry_categories;
CREATE DATABASE db_school;
select *from users
-- 用户表：存储用户基础信息
CREATE TABLE users (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '用户唯一标识',
    username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名（唯一）',
    email VARCHAR(100) NOT NULL UNIQUE COMMENT '邮箱（唯一）',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    status ENUM('active', 'banned', 'deleted') DEFAULT 'active' COMMENT '用户状态'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

-- 评论表：存储用户对内容的评论
CREATE TABLE comments (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '评论唯一标识',
    user_id INT UNSIGNED NOT NULL COMMENT '发表用户ID',
    content TEXT NOT NULL COMMENT '评论内容',
    status ENUM('visible', 'deleted', 'hidden') DEFAULT 'visible' COMMENT '评论状态',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='评论表';

-- 回复表：存储对评论的回复
CREATE TABLE replies (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '回复唯一标识',
    comment_id INT UNSIGNED NOT NULL COMMENT '关联评论ID',
    user_id INT UNSIGNED NOT NULL COMMENT '回复用户ID',
    content TEXT NOT NULL COMMENT '回复内容',
    status ENUM('visible', 'deleted', 'hidden') DEFAULT 'visible' COMMENT '回复状态',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (comment_id) REFERENCES comments(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='回复表';

-- 回复子表：存储对回复的再回复
CREATE TABLE sub_replies (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '子回复唯一标识',
    reply_id INT UNSIGNED NOT NULL COMMENT '关联回复ID',
    user_id INT UNSIGNED NOT NULL COMMENT '回复用户ID',
    reply_to_user_id INT UNSIGNED NOT NULL COMMENT '被回复用户ID',
    content TEXT NOT NULL COMMENT '回复内容',
    status ENUM('visible', 'deleted', 'hidden') DEFAULT 'visible' COMMENT '回复状态',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    FOREIGN KEY (reply_id) REFERENCES replies(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (reply_to_user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='回复子表';

-- 索引优化
CREATE INDEX idx_comments_user ON comments(user_id);
CREATE INDEX idx_comments_status ON comments(status);
CREATE INDEX idx_replies_comment ON replies(comment_id);
CREATE INDEX idx_subreplies_reply ON sub_replies(reply_id);


-- 插入用户数据（10条）
INSERT INTO users (username, email, status) VALUES
('张飞', 'zhangfei@example.com', 'active'),
('刘备', 'liubei@example.com', 'active'),
('关羽', 'guanyu@example.com', 'active');
 
```

step2:fastapi

```python
from typing import Dict, List, Optional
from datetime import datetime
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel
import pymysql.cursors

# ---------------------- FastAPI 初始化 ----------------------
app = FastAPI(title="学校评论系统 API", version="1.0.0")

# 允许跨域请求
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# ---------------------- 数据库配置 ----------------------
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


# ---------------------- Pydantic 模型 ----------------------
class CommentCreate(BaseModel):
    user_id: int
    content: str
    status: str = 'visible'


class ReplyCreate(BaseModel):
    user_id: int
    content: str
    status: str = 'visible'


class SubReplyCreate(BaseModel):
    user_id: int
    reply_to_user_id: int
    content: str
    status: str = 'visible'


# ---------------------- 数据库操作核心函数 ----------------------
def execute_query(query: str, params=None, fetch: bool = True) -> Optional[List[Dict]]:
    """执行 SQL 查询并返回结果"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall() if fetch else None
            connection.commit()
        return result
    except Exception as e:
        connection.rollback()
        raise HTTPException(
            status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
            detail=f"数据库操作失败: {str(e)}"
        )
    finally:
        connection.close()


# ---------------------- API 端点 ----------------------
@app.get("/comments", response_model=List[Dict], summary="获取所有评论")
def get_all_comments():
    """
    获取所有评论（按时间倒序排列），包含：
    - 评论基本信息
    - 关联的用户名
    """
    try:
        query = """
            SELECT c.*, u.username AS user_username 
            FROM comments c
            JOIN users u ON c.user_id = u.id
            ORDER BY c.created_at DESC
        """
        comments = execute_query(query)
        return [{k: v.isoformat() if isinstance(v, datetime) else v for k, v in item.items()}
                for item in comments]
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@app.get("/comments/{comment_id}", response_model=Dict, summary="获取评论详情")
def get_comment_detail(comment_id: int):
    """
    获取指定评论的完整信息，包含：
    - 评论基本信息
    - 所有直接回复
    - 每个回复的子回复
    """
    try:
        # 获取基础评论信息
        comment_query = """
            SELECT c.*, u.username AS user_username 
            FROM comments c
            JOIN users u ON c.user_id = u.id
            WHERE c.id = %s
            ORDER BY c.created_at ASC
        """
        comment = execute_query(comment_query, (comment_id,))
        if not comment:
            raise HTTPException(status_code=404, detail="评论不存在")
        comment_data = comment[0]

        # 获取关联回复
        replies_query = """
            SELECT r.*, u.username AS user_username
            FROM replies r
            JOIN users u ON r.user_id = u.id
            WHERE r.comment_id = %s
            ORDER BY r.created_at ASC
        """
        replies = execute_query(replies_query, (comment_id,))

        # 批量获取子回复
        sub_replies_dict = {}
        if replies:
            reply_ids = tuple(reply["id"] for reply in replies)
            sub_query = """
                SELECT sr.*, 
                    u.username AS user_username,
                    ru.username AS reply_to_username
                FROM sub_replies sr
                JOIN users u ON sr.user_id = u.id
                JOIN users ru ON sr.reply_to_user_id = ru.id
                WHERE sr.reply_id IN %s
                ORDER BY sr.created_at ASC
            """
            sub_replies = execute_query(sub_query, (reply_ids,))
            for sr in sub_replies:
                sub_replies_dict.setdefault(sr["reply_id"], []).append(sr)

        # 构建嵌套结构
        comment_data["replies"] = []
        for reply in replies:
            reply["sub_replies"] = sub_replies_dict.get(reply["id"], [])
            comment_data["replies"].append(reply)

        # 转换日期格式
        def convert_dates(obj):
            if isinstance(obj, datetime):
                return obj.isoformat()
            return obj

        return {k: convert_dates(v) for k, v in comment_data.items()}
    except HTTPException as he:
        raise he
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"服务器错误: {str(e)}")


@app.post("/comments", status_code=status.HTTP_201_CREATED, summary="创建新评论")
def create_comment(comment: CommentCreate):
    """创建新的评论条目"""
    try:
        query = "INSERT INTO comments (user_id, content, status) VALUES (%s, %s, %s)"
        execute_query(query, (comment.user_id, comment.content, comment.status), fetch=False)
        return {"message": "评论创建成功"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@app.post("/comments/{comment_id}/replies", status_code=201, summary="创建回复")
def create_reply(comment_id: int, reply: ReplyCreate):
    """在指定评论下创建回复"""
    try:
        query = "INSERT INTO replies (comment_id, user_id, content, status) VALUES (%s, %s, %s, %s)"
        execute_query(query, (comment_id, reply.user_id, reply.content, reply.status), fetch=False)
        return {"message": "回复创建成功"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@app.post("/replies/{reply_id}/subreplies", status_code=201, summary="创建子回复")
def create_subreply(reply_id: int, subreply: SubReplyCreate):
    """在指定回复下创建子回复"""
    try:
        query = """
            INSERT INTO sub_replies 
                (reply_id, user_id, reply_to_user_id, content, status) 
            VALUES (%s, %s, %s, %s, %s)
        """
        params = (reply_id, subreply.user_id, subreply.reply_to_user_id, subreply.content, subreply.status)
        execute_query(query, params, fetch=False)
        return {"message": "子回复创建成功"}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step2.1:fastapi 测试脚本

```python
from typing import Dict, List, Optional
from collections import defaultdict
import json
import pymysql.cursors
from datetime import datetime  # 新增导入

# 数据库连接配置
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


# ---------------------- 通用数据库操作函数 ----------------------
def execute_query(query: str, params=None, fetch: bool = True) -> Optional[List[Dict]]:
    """执行 SQL 查询并返回结果"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall() if fetch else None
            connection.commit()
        return result
    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"数据库操作失败: {str(e)}")
    finally:
        connection.close()


def get_comment_with_replies(comment_id: int) -> Optional[Dict]:
    try:
        # 查询基础评论信息
        comment_query = """
            SELECT c.*, u.username AS user_username 
            FROM comments c
            JOIN users u ON c.user_id = u.id
            WHERE c.id = %s
        """
        comment = execute_query(comment_query, (comment_id,))
        if not comment:
            return None
        comment_data = comment[0]

        # 转换datetime字段为字符串
        def convert_datetime(obj):
            if isinstance(obj, datetime):
                return obj.isoformat()
            return obj

        comment_data = {k: convert_datetime(v) for k, v in comment_data.items()}
        comment_data["replies"] = []

        # 查询关联回复
        replies_query = """
            SELECT r.*, u.username AS user_username
            FROM replies r
            JOIN users u ON r.user_id = u.id
            WHERE r.comment_id = %s
        """
        replies = execute_query(replies_query, (comment_id,))

        # 批量查询子回复
        reply_ids = [reply["id"] for reply in replies]
        sub_replies_dict = defaultdict(list)

        if reply_ids:
            sub_query = """
                SELECT sr.*, 
                    u.username AS user_username,
                    ru.username AS reply_to_username
                FROM sub_replies sr
                JOIN users u ON sr.user_id = u.id
                JOIN users ru ON sr.reply_to_user_id = ru.id
                WHERE sr.reply_id IN %s
            """
            sub_replies = execute_query(sub_query, (tuple(reply_ids),))
            for sr in sub_replies:
                sr = {k: convert_datetime(v) for k, v in sr.items()}
                sub_replies_dict[sr["reply_id"]].append(sr)

        # 构建嵌套结构
        for reply in replies:
            reply = {k: convert_datetime(v) for k, v in reply.items()}
            reply["sub_replies"] = sub_replies_dict.get(reply["id"], [])
            comment_data["replies"].append(reply)

        return comment_data

    except Exception as e:
        raise RuntimeError(f"查询失败: {str(e)}")
# ---------------------- 新增数据插入函数 ----------------------
def insert_comment(user_id: int, content: str, status: str = 'visible') -> None:
    """插入评论数据"""
    query = "INSERT INTO comments (user_id, content, status) VALUES (%s, %s, %s)"
    params = (user_id, content, status)
    execute_query(query, params, fetch=False)

def insert_reply(comment_id: int, user_id: int, content: str, status: str = 'visible') -> None:
    """插入回复数据"""
    query = "INSERT INTO replies (comment_id, user_id, content, status) VALUES (%s, %s, %s, %s)"
    params = (comment_id, user_id, content, status)
    execute_query(query, params, fetch=False)

def insert_sub_reply(reply_id: int, user_id: int, reply_to_user_id: int, content: str, status: str = 'visible') -> None:
    """插入子回复数据"""
    query = """
        INSERT INTO sub_replies 
            (reply_id, user_id, reply_to_user_id, content, status) 
        VALUES (%s, %s, %s, %s, %s)
    """
    params = (reply_id, user_id, reply_to_user_id, content, status)
    execute_query(query, params, fetch=False)

# 1. 新增获取所有评论的函数
def get_all_comments() -> Optional[List[Dict]]:
    """获取所有评论及其用户信息，按创建时间倒序排列"""
    try:
        query = """
            SELECT c.*, u.username AS user_username 
            FROM comments c
            JOIN users u ON c.user_id = u.id
            ORDER BY c.created_at DESC
        """
        comments = execute_query(query)
        if not comments:
            return []
        # 转换datetime字段为字符串
        converted = []
        for comment in comments:
            converted_comment = {
                k: v.isoformat() if isinstance(v, datetime) else v
                for k, v in comment.items()
            }
            converted.append(converted_comment)
        return converted
    except Exception as e:
        raise RuntimeError(f"获取所有评论失败: {str(e)}")
# 2. 新增根据ID获取评论的函数
def get_comment_by_id(comment_id: int) -> Optional[Dict]:
    """根据评论ID获取单个评论信息"""
    try:
        query = """
            SELECT c.*, u.username AS user_username 
            FROM comments c
            JOIN users u ON c.user_id = u.id
            WHERE c.id = %s
        """
        result = execute_query(query, (comment_id,))
        if not result:
            return None
        comment = result[0]
        # 转换datetime字段为字符串
        converted_comment = {
            k: v.isoformat() if isinstance(v, datetime) else v
            for k, v in comment.items()
        }
        return converted_comment
    except Exception as e:
        raise RuntimeError(f"获取评论失败: {str(e)}")
# 使用示例
if __name__ == "__main__":
    try:

        # 插入示例评论
        insert_comment(1, '这是用户1的评论内容，欢迎大家讨论！')
        insert_comment(10, '用户10的最后一个评论。')

        # 插入示例回复
        insert_reply(1, 2, '用户2回复用户1：同意你的观点！')
        insert_reply(9, 1, '用户1回复用户9：测试回复。')

        # 插入示例子回复
        insert_sub_reply(1, 3, 2, '用户3回复用户2：具体哪里同意？')
        insert_sub_reply(9, 2, 10, '用户2回复用户10：我不同意。')



        print("数据插入成功")



        comment = get_comment_with_replies(21)
        print("comment_wrs:",comment)

        print(json.dumps(comment, indent=2, ensure_ascii=False))

        # 测试获取所有评论
        print("=== 所有评论 ===")
        all_comments = get_all_comments()
        print(json.dumps(all_comments, indent=2, ensure_ascii=False))

        # 测试根据ID获取评论
        print("\n=== 评论ID=1 ===")
        comment = get_comment_by_id(1)
        print(json.dumps(comment, indent=2, ensure_ascii=False))
    except RuntimeError as e:
        print(f"操作失败: {str(e)}")
    except Exception as e:
        print(f"未知错误: {str(e)}")
```


step3:postman

```bash
接口1：查询所有评论
方法：GET
http://localhost:8000/comments
接口2：根据ID查询评论
方法：GET
http://localhost:8000/comments/1
接口3：创建新评论：
POST http://localhost:8000/comments
Content-Type: application/json
{
  "user_id": 1,
  "content": "大飞来了"
}
 {
    "message": "评论创建成功"
}
在评选详情页面 
1. 新增一个按钮，开始回复
2.点击开始回复按钮，出现弹窗，
3.输入评论的内容，状态默认visible
4.开始调用后端的post请求接口
5.请求成功刷新页面

接口4：在指定评论下创建回复：post http://localhost:8000/comments/1/replies
{
    "user_id": 2,
    "content": "我提议 今天我们三兄弟 桃园结义吧",
    "status": "visible"
}

{
    "message": "回复创建成功"
}
接口5：创建子回复：
POST  
http://localhost:8000/replies/1/subreplies
{
  "user_id": 3,
  "reply_to_user_id": 2,
  "content": "大哥，关某愿意追随大哥，生死与共"
}
{
    "message": "子回复创建成功"
}
```

step4:评论页C:\Users\wangrusheng\PycharmProjects\untitled\src\app\user\user.component.ts

```typescript
// user.component.ts
import { Component, OnInit } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import {FormsModule, ReactiveFormsModule} from '@angular/forms';

interface Comment {
  id: number;
  user_id: number;
  content: string;
  status: string;
  created_at: string;
  updated_at: string;
  user_username: string;
}
// 定义传递值接口
interface DialogParams {
  flag: boolean;
  message: string;
  count: number;
}

@Component({
  selector: 'app-user',
  standalone: true,
  imports: [CommonModule, RouterModule, FormsModule, ReactiveFormsModule],
  templateUrl: './user.component.html',
  styleUrls: ['./user.component.css']
})
export class UserComponent implements OnInit {
  comments: Comment[] = [];
  isLoading = false;
  errorMessage = '';

  // 分页相关属性
  currentPage = 1;
  itemsPerPage = 5;
  totalItems = 0;


  showReplyModal = false;
  newReplyContent = '';
  selectedStatus: 'visible' | 'deleted' | 'hidden' = 'visible';
  newUserId: number | null = null;
  newReplyUserId: number | null = null;
  submitError = '';
  isSubmitting = false;

  // 修改为对象类型存储多个值
  passedData: DialogParams = {
    flag: false,
    message: '',
    count: 0
  };

  constructor(private http: HttpClient) {}

  ngOnInit(): void {
    this.loadComments();
  }

  loadComments(page = 1): void {
    this.isLoading = true;
    this.currentPage = page;

    const params = new HttpParams()
      .set('page', page.toString())
      .set('page_size', this.itemsPerPage.toString());

    this.http.get<Comment[]>('http://localhost:8000/comments', { params })
      .subscribe({
        next: (response) => {
          // 实际开发中应从接口返回分页信息，这里模拟分页
          this.totalItems = response.length;
          this.comments = response.slice(
            (this.currentPage - 1) * this.itemsPerPage,
            this.currentPage * this.itemsPerPage
          );
          this.isLoading = false;
          console.log("UserComponent-coments:",this.comments)
        },
        error: (err) => {
          this.errorMessage = '加载评论失败，请稍后重试';
          this.isLoading = false;
          console.error('API Error:', err);
        }
      });
  }

  get totalPages(): number {
    return Math.ceil(this.totalItems / this.itemsPerPage);
  }

  prevPage(): void {
    if (this.currentPage > 1) {
      this.currentPage--;
      this.loadComments(this.currentPage);
    }
  }

  nextPage(): void {
    if (this.currentPage < this.totalPages) {
      this.currentPage++;
      this.loadComments(this.currentPage);
    }
  }

  // 修改方法接收对象参数
  openReplyModal(params: DialogParams = { flag: false, message: '默认消息', count: 0 }): void {
    this.showReplyModal = true;
    this.newReplyContent = '';
    this.selectedStatus = 'visible';
    this.newUserId = null;
    this.newReplyUserId = null;
    this.submitError = '';
    this.passedData = { ...params }; // 使用展开运算符保持数据不可变
  }

  closeReplyModal(): void {
    this.showReplyModal = false;
    this.passedData = { flag: false, message: '', count: 0 };
  }

  submitReply(): void {
    if (!this.newUserId || isNaN(this.newUserId)) {
      this.submitError = '请输入有效的用户ID';
      return;
    }
    if (!this.newReplyContent.trim()) {
      this.submitError = '请输入回复内容';
      return;
    }

    this.isSubmitting = true;
    const url2 = `http://localhost:8000/comments`;
    const body = {
      user_id: this.newUserId,
      content: this.newReplyContent
    };
    console.log('submitReply_body:', body);
    this.http.post(url2, body).subscribe({
      next: () => {
        this.isSubmitting = false;
        this.showReplyModal = false;
        this.loadComments(); // 刷新数据
      },
      error: (err) => {
        this.isSubmitting = false;
        this.submitError = '提交失败，请稍后重试';
        console.error('子回复创建失败:', err);
      }
    });
  }


}
<!-- user.component.html -->
<div class="comments-container">
  <!-- 加载状态 -->
  <div *ngIf="isLoading" class="loading">
    <div class="spinner"></div>
    正在加载评论...
  </div>

  <!-- 错误提示 -->
  <div *ngIf="errorMessage" class="error">
    {{ errorMessage }}
    <button (click)="loadComments()">重试</button>
  </div>

  <!-- 评论列表 -->
  <div *ngIf="!isLoading && !errorMessage">
    <h2>用户评论（共 {{ totalItems }} 条）</h2>
    <button class="detail-btn"
            (click)="openReplyModal({flag: false, message: '普通消息', count: 10})">
      新增评论
    </button>
    <div class="comment-list">
      <div *ngFor="let comment of comments" class="comment-card">
        <div class="comment-header">
          <span class="username">{{ comment.user_username }}</span>
          <span class="time">{{ comment.created_at | date: 'yyyy-MM-dd HH:mm' }}</span>
        </div>
        <p class="content">{{ comment.content }}</p>
        <div class="comment-footer">
          <button
            [routerLink]="['/comments', comment.id]"
            class="detail-btn">
            查看详情
          </button>
        </div>
      </div>
    </div>

    <!-- 分页控件 -->
    <div class="pagination">
      <button
        (click)="prevPage()"
        [disabled]="currentPage === 1">
        上一页
      </button>

      <span class="page-info">
        第 {{ currentPage }} 页 / 共 {{ totalPages }} 页
      </span>

      <button
        (click)="nextPage()"
        [disabled]="currentPage === totalPages">
        下一页
      </button>
    </div>
  </div>


  <!-- 弹窗内容 -->
  <div class="modal-overlay" *ngIf="showReplyModal">
    <div class="modal-content">
      <h3>创建评论</h3>

      <!-- 修改展示多个参数 -->
      <div class="passed-values" *ngIf="this.passedData.flag">
        <div>接收到的参数：</div>
        <div>Flag: {{ passedData.flag ? 'TRUE' : 'FALSE' }}</div>
        <div>Message: {{ passedData.message }}</div>
        <div>Count: {{ passedData.count }}</div>
      </div>

      <form (ngSubmit)="submitReply()">
        <!-- 原有表单内容保持不变 -->
        <div class="form-group">
          <label>回复内容：</label>
          <textarea
            [(ngModel)]="newReplyContent"
            name="content"
            required
            rows="4"
          ></textarea>
        </div>

        <div class="form-group"  *ngIf="this.passedData.flag">
          <label>状态：</label>
          <select
            [(ngModel)]="selectedStatus"
            name="status"
            class="status-select"
          >
            <option value="visible">Visible</option>
            <option value="deleted">Deleted</option>
            <option value="hidden">Hidden</option>
          </select>
        </div>

        <div class="form-group">
          <label>(user_id)回复用户ID：</label>
          <input
            type="number"
            [(ngModel)]="newUserId"
            name="userId"
            required
            class="user-id-input"
          >
        </div>
        <div class="form-group"  *ngIf="this.passedData.flag">
          <label>(reply_to_user_id)被回复用户ID：</label>
          <input
            type="number"
            [(ngModel)]="newReplyUserId"
            name="userId"
            required
            class="user-id-input"
          >
        </div>
        <div *ngIf="submitError" class="error-message">{{ submitError }}</div>

        <div class="button-group">
          <button
            type="button"
            (click)="closeReplyModal()"
            class="cancel-btn"
          >
            取消
          </button>
          <button
            type="submit"
            [disabled]="isSubmitting"
            class="submit-btn">
            {{ isSubmitting ? '提交中...' : '提交' }}
          </button>
        </div>
      </form>
    </div>
  </div>



</div>
/* user.component.css */
.comments-container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.loading {
  text-align: center;
  padding: 2rem;
  color: #666;
}

.spinner {
  display: inline-block;
  width: 2rem;
  height: 2rem;
  border: 3px solid #f3f3f3;
  border-radius: 50%;
  border-top-color: #2196F3;
  animation: spin 1s linear infinite;
  margin-bottom: 1rem;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error {
  background: #ffebee;
  color: #b71c1c;
  padding: 1rem;
  border-radius: 4px;
  text-align: center;
}

.comment-list {
  margin-top: 1.5rem;
}

.comment-card {
  background: white;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1rem;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.comment-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
  color: #666;
}

.content {
  font-size: 1.1rem;
  line-height: 1.6;
  color: #333;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  margin: 2rem 0;
}

button {
  padding: 0.5rem 1.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: #f5f5f5;
  cursor: pointer;
  transition: all 0.2s;
}

button:hover:not(:disabled) {
  background: #2196F3;
  color: white;
  border-color: transparent;
}

button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.page-info {
  color: #666;
}


.comment-footer {
  margin-top: 1rem;
  text-align: right;
}

.detail-btn {
  background: #2196F3;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  transition: opacity 0.2s;
}

.detail-btn:hover {
  opacity: 0.9;
}
.comment-container {
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
}

.loading, .error {
  text-align: center;
  padding: 20px;
  color: #666;
}

.main-comment {
  background: white;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.replies-section {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.reply-item {
  margin: 15px 0;
  padding: 15px;
  border-left: 3px solid #eee;
}

.sub-replies {
  margin-left: 30px;
  border-left: 2px solid #ddd;
  padding-left: 15px;
}

.username {
  font-weight: bold;
  color: #2c3e50;
  margin-right: 10px;
}

.reply-to {
  color: #666;
  font-size: 0.9em;
  margin: 0 5px;
}

.time {
  color: #95a5a6;
  font-size: 0.85em;
}

.content {
  margin: 8px 0;
  color: #34495e;
  line-height: 1.6;
}
/* 确保容器可见 */
.comment-container {
  min-height: 300px;  /* 保证最小高度 */
  position: relative; /* 用于加载层定位 */
}

/* 增强加载状态显示 */
.loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.2em;
}

/* 确保内容层级 */
.main-comment {
  position: relative;
  z-index: 1;
}
/* 新增样式 */
.reply-button {
  margin-top: 15px;
  padding: 8px 16px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 25px;
  border-radius: 8px;
  width: 500px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.status-select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.user-id-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.error-message {
  color: #dc3545;
  margin-bottom: 15px;
}

.button-group {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.cancel-btn {
  padding: 8px 16px;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-btn {
  padding: 8px 16px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-btn:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}
/*fenge分割线*/
/* dialog-test.component.css */
/* 新增样式 */
.passed-value {
  margin-bottom: 15px;
  padding: 10px;
  border-radius: 4px;
  font-weight: bold;
}

.true-value {
  background-color: #e8f5e9;
  color: #2e7d32;
}

.false-value {
  background-color: #ffebee;
  color: #c62828;
}

.reply-button {
  margin-right: 10px;
  padding: 8px 16px;
}
/* 新增传递值样式 */
.passed-values {
  padding: 10px;
  margin-bottom: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background-color: #f8f9fa;
}

.passed-values div:first-child {
  font-weight: bold;
  margin-bottom: 8px;
}

.passed-values div:not(:first-child) {
  margin: 4px 0;
  color: #666;
}

```

step5:回复页C:\Users\wangrusheng\PycharmProjects\untitled\src\app\user-detail\user-detail.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ActivatedRoute } from '@angular/router';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, catchError, throwError } from 'rxjs';
import {FormsModule} from '@angular/forms';

// 定义类型接口
interface CommentWithReplies {
  id: number;
  user_id: number;
  content: string;
  status: string;
  created_at: string;
  updated_at: string;
  user_username: string;
  replies?: Reply[];
}

interface Reply {
  id: number;
  comment_id: number;
  user_id: number;
  content: string;
  status: string;
  created_at: string;
  updated_at: string;
  user_username: string;
  sub_replies?: SubReply[];
}

interface SubReply {
  id: number;
  reply_id: number;
  user_id: number;
  reply_to_user_id: number;
  content: string;
  status: string;
  created_at: string;
  updated_at: string;
  user_username: string;
  reply_to_username: string;
}

// 定义传递值接口
interface DialogParams {
  flag: boolean;
  message: string;
  count: number;
}

@Component({
  selector: 'app-user-detail',
  templateUrl: './user-detail.component.html',
  imports: [
    CommonModule,
    FormsModule
  ],
  styleUrls: ['./user-detail.component.css'],
  standalone: true
})
export class UserDetailComponent implements OnInit {
  commentId!: number;
  replyId!: number;
  commentDetail!: CommentWithReplies;
  isLoading = true;
  errorMessage = '';

    // 新增变量
  showReplyModal = false;
  newReplyContent = '';
  selectedStatus: 'visible' | 'deleted' | 'hidden' = 'visible';
  newUserId: number | null = null;
  newReplyUserId: number | null = null;
  submitError = '';
  isSubmitting = false;
  // 新增传递值变量
  // 修改为对象类型存储多个值
  passedData: DialogParams = {
    flag: false,
    message: '',
    count: 0
  };

  constructor(
    private route: ActivatedRoute,
    private http: HttpClient
  ) {}

  ngOnInit(): void {
    this.commentId = Number(this.route.snapshot.paramMap.get('id'));
    this.loadCommentDetail();
  }

  private loadCommentDetail(): void {
    this.http.get<CommentWithReplies>(`http://localhost:8000/comments/${this.commentId}`)
      .pipe(
        catchError(this.handleError)
      )
      .subscribe({
        next: (response) => {
          // 如果接口返回的是包裹对象（根据实际情况选择）
          // this.commentDetail = response.data;
          // 如果直接返回数据对象
          this.commentDetail = response;
          this.isLoading = false;
        },
        error: (err) => {
          this.errorMessage = '加载评论详情失败，请稍后重试';
          this.isLoading = false;
          console.error('获取评论详情失败:', err);
        }
      });
  }

  private handleError(error: HttpErrorResponse) {
    let errorMessage = '发生未知错误';
    if (error.error instanceof ErrorEvent) {
      errorMessage = `客户端错误：${error.error.message}`;
    } else {
      errorMessage = `服务端错误：${error.status}\n${error.message}`;
    }
    return throwError(() => new Error(errorMessage));
  }

   // 新增方法
  openReplyModal(params: DialogParams = { flag: false, message: '默认消息', count: 0 }): void {
    this.showReplyModal = true;
    this.newReplyContent = '';
    this.selectedStatus = 'visible';
    this.newUserId = null;
    this.newReplyUserId = null;
    this.submitError = '';
    this.passedData = { ...params }; // 使用展开运算符保持数据不可变
  }

  closeReplyModal(): void {
    this.showReplyModal = false;
  }

  submitReply(): void {
    if (!this.newUserId || isNaN(this.newUserId)) {
      this.submitError = '请输入有效的用户ID';
      return;
    }
    if (!this.newReplyContent.trim()) {
      this.submitError = '请输入回复内容';
      return;
    }

    this.isSubmitting = true;
    if (this.passedData.flag) {
        const url2 = `http://localhost:8000/replies/${this.passedData.count}/subreplies`;
        const body = {
          user_id: this.newUserId,
          content: this.newReplyContent,
          reply_to_user_id: this.newReplyUserId,
          status: this.selectedStatus
         };
        console.log('submitReply_body:', body);
        this.http.post(url2, body).subscribe({
        next: () => {
            this.isSubmitting = false;
            this.showReplyModal = false;
            this.loadCommentDetail(); // 刷新数据
        },
        error: (err) => {
            this.isSubmitting = false;
            this.submitError = '提交失败，请稍后重试';
            console.error('子回复创建失败:', err);
          }
        });
    } else {
      const url = `http://localhost:8000/comments/${this.commentId}/replies`;
      const body = {
        user_id: this.newUserId,
        content: this.newReplyContent,
        status: this.selectedStatus
      };

      this.http.post(url, body).subscribe({
      next: () => {
          this.isSubmitting = false;
          this.showReplyModal = false;
          this.loadCommentDetail(); // 刷新数据
      },
      error: (err) => {
          this.isSubmitting = false;
          this.submitError = '提交失败，请稍后重试';
          console.error('回复创建失败:', err);
        }
      });
    }
  }
}
<div class="comment-container">
  <!-- 加载状态 -->
  <div *ngIf="isLoading" class="loading">加载中...</div>

  <!-- 错误提示 -->
  <div *ngIf="errorMessage" class="error">{{ errorMessage }}</div>

  <!-- 评论详情 -->
  <div *ngIf="commentDetail && !isLoading">
    <div class="main-comment">
      <h2>评论详情</h2>
      <div class="comment-header">
        <span class="username">{{ commentDetail.user_username }}</span>
        <span class="time">{{ commentDetail.created_at | date:'yyyy-MM-dd HH:mm' }}</span>
      </div>
      <p class="content">{{ commentDetail.content }}</p>
      <button class="reply-button" (click)="openReplyModal({flag: false, message: '普通消息', count: 0})">开始回复</button>
    </div>

    <!-- 回复列表 -->
    <div class="replies-section">
      <h3>全部回复（{{ commentDetail.replies?.length || 0 }}）</h3>
      <div class="reply-list">
        <!-- 主回复 -->
        <div *ngFor="let reply of commentDetail.replies" class="reply-item">
          <div class="reply-main">
            <div class="reply-header">
              <span class="username">{{ reply.user_username }}</span>
              <span class="time">{{ reply.created_at | date:'yyyy-MM-dd HH:mm' }}</span>
            </div>
            <p class="content">{{ reply.content }}</p>
            <!-- 新增的回复按钮 -->
            <button class="reply-button" (click)="openReplyModal({flag: true, message: '重要消息', count: reply.id})">开始回复</button>
          </div>

          <!-- 子回复 -->
          <div *ngIf="reply.sub_replies?.length" class="sub-replies">
            <div *ngFor="let subReply of reply.sub_replies" class="sub-reply-item">
              <div class="reply-header">
                <span class="username">{{ subReply.user_username }}</span>
                <span class="reply-to">回复 {{ subReply.reply_to_username }}</span>
                <span class="time">{{ subReply.created_at | date:'yyyy-MM-dd HH:mm' }}</span>
              </div>
              <p class="content">{{ subReply.content }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
    <!-- 新增回复弹窗 -->
  <div class="modal-overlay" *ngIf="showReplyModal">
    <div class="modal-content">
      <h3>创建回复</h3>
      <form (ngSubmit)="submitReply()">
        <div class="form-group">
          <label>回复内容：</label>
          <textarea
            [(ngModel)]="newReplyContent"
            name="content"
            required
            rows="4"
          ></textarea>
        </div>

        <div class="form-group">
          <label>状态：</label>
          <select
            [(ngModel)]="selectedStatus"
            name="status"
            class="status-select"
          >
            <option value="visible">Visible</option>
            <option value="deleted">Deleted</option>
            <option value="hidden">Hidden</option>
          </select>
        </div>
        <div class="form-group">
          <label>(user_id)回复用户ID：</label>
          <input
            type="number"
            [(ngModel)]="newUserId"
            name="userId"
            required
            class="user-id-input"
          >
        </div>
        <div class="form-group" *ngIf="this.passedData.flag">
          <label>(reply_to_user_id)被回复用户ID：</label>
          <input
            type="number"
            [(ngModel)]="newReplyUserId"
            name="userId"
            required
            class="user-id-input"
          >
        </div>

        <div *ngIf="submitError" class="error-message">{{ submitError }}</div>

        <div class="button-group">
          <button
            type="button"
            (click)="closeReplyModal()"
            class="cancel-btn"
          >
            取消
          </button>
          <button
            type="submit"
            [disabled]="isSubmitting"
            class="submit-btn"
          >
            {{ isSubmitting ? '提交中...' : '提交' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</div>
.comment-container {
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
}

.loading, .error {
  text-align: center;
  padding: 20px;
  color: #666;
}

.main-comment {
  background: white;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.replies-section {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.reply-item {
  margin: 15px 0;
  padding: 15px;
  border-left: 3px solid #eee;
}

.sub-replies {
  margin-left: 30px;
  border-left: 2px solid #ddd;
  padding-left: 15px;
}

.username {
  font-weight: bold;
  color: #2c3e50;
  margin-right: 10px;
}

.reply-to {
  color: #666;
  font-size: 0.9em;
  margin: 0 5px;
}

.time {
  color: #95a5a6;
  font-size: 0.85em;
}

.content {
  margin: 8px 0;
  color: #34495e;
  line-height: 1.6;
}
/* 确保容器可见 */
.comment-container {
  min-height: 300px;  /* 保证最小高度 */
  position: relative; /* 用于加载层定位 */
}

/* 增强加载状态显示 */
.loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-size: 1.2em;
}

/* 确保内容层级 */
.main-comment {
  position: relative;
  z-index: 1;
}
/* 新增样式 */
.reply-button {
  margin-top: 15px;
  padding: 8px 16px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 25px;
  border-radius: 8px;
  width: 500px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.status-select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.user-id-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.error-message {
  color: #dc3545;
  margin-bottom: 15px;
}

.button-group {
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}

.cancel-btn {
  padding: 8px 16px;
  background-color: #6c757d;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-btn {
  padding: 8px 16px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-btn:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}

```


end