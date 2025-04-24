说明：
朋友圈

1.内容列表
2.对内容进行评论
3.发表和删除内容
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/41428663593c435fa1f0acfae9cf2a08.png#pic_center)

step1:sql

```sql

-- 用户表（存储用户基本信息）
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    avatar_url VARCHAR(255) COMMENT '头像地址',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 朋友圈内容表（核心内容存储）
CREATE TABLE posts (
    post_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    content TEXT NOT NULL COMMENT '动态内容',
    image_urls JSON COMMENT '图片地址数组',
    like_count INT DEFAULT 0 COMMENT '点赞数',
    comment_count INT DEFAULT 0 COMMENT '评论数',
    is_deleted TINYINT(1) DEFAULT 0 COMMENT '是否删除',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 评论表（评论系统核心）
CREATE TABLE comments (
    comment_id INT AUTO_INCREMENT PRIMARY KEY,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    parent_id INT DEFAULT 0 COMMENT '父评论ID（支持二级评论）',
    content VARCHAR(1000) NOT NULL,
    is_deleted TINYINT(1) DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (post_id) REFERENCES posts(post_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 点赞关系表（可选）
CREATE TABLE likes (
    like_id INT AUTO_INCREMENT PRIMARY KEY,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uniq_post_user (post_id, user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


INSERT INTO users (username, avatar_url) VALUES
('张三', 'https://randomuser.me/api/portraits/men/1.jpg'),
('李四', 'https://randomuser.me/api/portraits/men/2.jpg'),
('王五', 'https://randomuser.me/api/portraits/men/3.jpg'),
('赵六', 'https://randomuser.me/api/portraits/men/4.jpg'),
('陈七', 'https://randomuser.me/api/portraits/men/5.jpg'),
('周八', 'https://randomuser.me/api/portraits/men/6.jpg'),
('吴九', 'https://randomuser.me/api/portraits/men/1.jpg'),
('郑十', 'https://randomuser.me/api/portraits/men/2.jpg'),
('孙十一', 'https://randomuser.me/api/portraits/men/3.jpg'),
('刘十二', 'https://randomuser.me/api/portraits/men/4.jpg');



INSERT INTO posts (user_id, content, image_urls) VALUES
(1, '今天天气真好！', '["https://randomuser.me/api/portraits/men/1.jpg","https://randomuser.me/api/portraits/men/2.jpg"]'),
(2, '打卡新开的咖啡店☕️', '["https://randomuser.me/api/portraits/men/3.jpg","https://randomuser.me/api/portraits/men/4.jpg"]'),
(3, '周末爬山，累并快乐着🏔️', '["https://randomuser.me/api/portraits/men/5.jpg","https://randomuser.me/api/portraits/men/6.jpg"]'),
(4, '深夜放毒，自制火锅🍲', '["https://randomuser.me/api/portraits/men/4.jpg","https://randomuser.me/api/portraits/men/2.jpg"]'),
(5, '新书到货，开读！📚', NULL),
(6, '健身打卡 Day 30💪', '["https://randomuser.me/api/portraits/men/3.jpg"]'),
(7, '旅行回忆 #北海', '["https://randomuser.me/api/portraits/men/4.jpg","https://randomuser.me/api/portraits/men/5.jpg"]'),
(8, '加班到凌晨，晚安🌙', NULL),
(9, '今日份早餐🥐', '["https://randomuser.me/api/portraits/men/4.jpg"]'),
(10, '宠物日常🐶', '["https://randomuser.me/api/portraits/men/6.jpg"]');

INSERT INTO comments (post_id, user_id, parent_id, content) VALUES
(1, 2, 0, '确实不错！'),
(1, 3, 0, '一起出去玩吧~'),
(2, 1, 0, '这家咖啡好喝吗？'),
(2, 4, 3, '超好喝！推荐拿铁！'),
(3, 5, 0, '风景太美了！'),
(4, 6, 0, '饿了……'),
(5, 7, 0, '什么书呀？'),
(6, 8, 0, '坚持就是胜利！'),
(7, 9, 0, '我也想去西藏！'),
(10, 10, 0, '狗狗好可爱！');


INSERT INTO likes (post_id, user_id) VALUES
(1, 2), (1, 3), (1, 4),
(2, 1), (2, 5),
(3, 6), (3, 7),
(4, 8), (4, 9),
(5, 10);
```

step2:fastapi

```python
from fastapi import FastAPI, HTTPException, Depends
import pymysql.cursors
from enum import Enum
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional, List
import json

app = FastAPI()

# CORS配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 数据库配置
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


def db_execute(query: str, params=None, fetch=True):
    try:
        # 转换所有JSON参数
        processed_params = []
        for param in params:
            if isinstance(param, (list, dict)):
                processed_params.append(json.dumps(param))
            else:
                processed_params.append(param)

        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            print(f"Executing query: {query % tuple(processed_params)}")  # 调试输出
            cursor.execute(query, params or ())
            if fetch:
                result = cursor.fetchall()
            else:
                result = cursor.lastrowid
            connection.commit()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Database error: {str(e)}")


# 数据模型
class PostCreate(BaseModel):
    user_id: int
    content: str
    image_urls: Optional[List[str]] = None


class CommentCreate(BaseModel):
    user_id: int
    content: str
    parent_id: Optional[int] = 0


class PostStatus(str, Enum):
    available = "available"
    deleted = "deleted"

class PostCreate(BaseModel):
    user_id: int
    content: str
    image_urls: Optional[List[str]] = Field(
        default=None,
        example=["img1.jpg", "img2.jpg"],
        description="图片URL列表"
    )

# 接口实现
@app.get("/posts")
async def get_posts(
        user_id: Optional[int] = None,
        page: int = 1,
        page_size: int = 10,
        order_by: str = "newest"
):
    """获取朋友圈列表"""
    offset = (page - 1) * page_size
    base_query = """
    SELECT 
        p.*, 
        u.username,
        u.avatar_url,
        COUNT(c.comment_id) as comment_count
    FROM posts p
    JOIN users u ON p.user_id = u.user_id
    LEFT JOIN comments c ON p.post_id = c.post_id
    WHERE p.is_deleted = 0
    """
    params = []

    if user_id:
        base_query += " AND p.user_id = %s"
        params.append(user_id)

    base_query += " GROUP BY p.post_id"

    # 排序方式
    order_mapping = {
        "newest": "p.created_at DESC",
        "hottest": "p.like_count DESC"
    }
    base_query += f" ORDER BY {order_mapping.get(order_by, 'p.created_at DESC')}"

    base_query += " LIMIT %s OFFSET %s"
    params.extend([page_size, offset])

    return {"data": db_execute(base_query, params)}


@app.post("/posts")
async def create_post(post: PostCreate):
    # 验证用户存在
    user_exists = db_execute("SELECT user_id FROM users WHERE user_id = %s", (post.user_id,))
    if not user_exists:
        raise HTTPException(status_code=404, detail="User not found")

    # 转换图片列表为JSON字符串
    image_json = json.dumps(post.image_urls) if post.image_urls else None

    query = """
    INSERT INTO posts 
    (user_id, content, image_urls)
    VALUES (%s, %s, %s)
    """
    try:
        post_id = db_execute(
            query,
            (post.user_id, post.content, image_json),  # 使用转换后的值
            fetch=False
        )
        return {"post_id": post_id, "message": "Post created successfully"}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.delete("/posts/{post_id}")
async def delete_post(post_id: int, user_id: int):
    """删除朋友圈（软删除）"""
    # 验证权限
    post_owner = db_execute(
        "SELECT user_id FROM posts WHERE post_id = %s",
        (post_id,)
    )
    if not post_owner or post_owner[0]['user_id'] != user_id:
        raise HTTPException(status_code=403, detail="No permission to delete this post")

    db_execute(
        "UPDATE posts SET is_deleted = 1 WHERE post_id = %s",
        (post_id,),
        fetch=False
    )
    return {"message": "Post deleted successfully"}


@app.post("/posts/{post_id}/comments")
async def create_comment(post_id: int, comment: CommentCreate):
    """添加评论"""
    # 验证帖子存在
    post_exists = db_execute(
        "SELECT post_id FROM posts WHERE post_id = %s AND is_deleted = 0",
        (post_id,)
    )
    if not post_exists:
        raise HTTPException(status_code=404, detail="Post not found")

    # 验证用户存在
    user_exists = db_execute(
        "SELECT user_id FROM users WHERE user_id = %s",
        (comment.user_id,)
    )
    if not user_exists:
        raise HTTPException(status_code=404, detail="User not found")

    # 验证父评论
    if comment.parent_id != 0:
        parent_exists = db_execute(
            "SELECT comment_id FROM comments WHERE comment_id = %s",
            (comment.parent_id,)
        )
        if not parent_exists:
            raise HTTPException(status_code=404, detail="Parent comment not found")

    # 插入评论
    query = """
    INSERT INTO comments 
    (post_id, user_id, parent_id, content)
    VALUES (%s, %s, %s, %s)
    """
    try:
        db_execute(
            query,
            (post_id, comment.user_id, comment.parent_id, comment.content),
            fetch=False
        )
        # 更新评论计数
        db_execute(
            "UPDATE posts SET comment_count = comment_count + 1 WHERE post_id = %s",
            (post_id,),
            fetch=False
        )
        return {"message": "Comment added successfully"}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))


@app.get("/posts/{post_id}/comments")
async def get_comments(post_id: int, page: int = 1, page_size: int = 10):
    """获取评论列表"""
    offset = (page - 1) * page_size
    query = """
    SELECT 
        c.*,
        u.username,
        u.avatar_url,
        COUNT(r.comment_id) as reply_count
    FROM comments c
    JOIN users u ON c.user_id = u.user_id
    LEFT JOIN comments r ON c.comment_id = r.parent_id
    WHERE c.post_id = %s AND c.is_deleted = 0
    GROUP BY c.comment_id
    ORDER BY c.created_at DESC
    LIMIT %s OFFSET %s
    """
    return {"data": db_execute(query, (post_id, page_size, offset))}


@app.get("/users/{user_id}/posts")
async def get_user_posts(
        user_id: int,
        page: int = 1,
        page_size: int = 10
):
    """获取用户个人朋友圈"""
    offset = (page - 1) * page_size
    query = """
    SELECT 
        p.*,
        u.username,
        u.avatar_url
    FROM posts p
    JOIN users u ON p.user_id = u.user_id
    WHERE p.user_id = %s AND p.is_deleted = 0
    ORDER BY p.created_at DESC
    LIMIT %s OFFSET %s
    """
    return {"data": db_execute(query, (user_id, page_size, offset))}


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:postman

```bash
get ： http://localhost:8000/posts?page=1&page_size=5&order_by=hottest


{
    "data": [
        {
            "post_id": 1,
            "user_id": 1,
            "content": "今天天气真好！",
            "image_urls": "[\"https://example.com/posts/1_1.jpg\"]",
            "like_count": 0,
            "comment_count": 0,
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:22",
            "updated_at": "2025-03-19T03:50:22",
            "username": "张三",
            "avatar_url": "https://example.com/avatars/1.jpg",
            ".comment_count": 2
        },
        {
            "post_id": 2,
            "user_id": 2,
            "content": "打卡新开的咖啡店☕️",
            "image_urls": "[\"https://example.com/posts/2_1.jpg\", \"https://example.com/posts/2_2.jpg\"]",
            "like_count": 0,
            "comment_count": 0,
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:22",
            "updated_at": "2025-03-19T03:50:22",
            "username": "李四",
            "avatar_url": "https://example.com/avatars/2.jpg",
            ".comment_count": 2
        },
        {
            "post_id": 3,
            "user_id": 3,
            "content": "周末爬山，累并快乐着🏔️",
            "image_urls": "[\"https://example.com/posts/3_1.jpg\"]",
            "like_count": 0,
            "comment_count": 0,
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:22",
            "updated_at": "2025-03-19T03:50:22",
            "username": "王五",
            "avatar_url": "https://example.com/avatars/3.jpg",
            ".comment_count": 1
        },
        {
            "post_id": 4,
            "user_id": 4,
            "content": "深夜放毒，自制火锅🍲",
            "image_urls": "[\"https://example.com/posts/4_1.jpg\", \"https://example.com/posts/4_2.jpg\"]",
            "like_count": 0,
            "comment_count": 0,
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:22",
            "updated_at": "2025-03-19T03:50:22",
            "username": "赵六",
            "avatar_url": "https://example.com/avatars/4.jpg",
            ".comment_count": 1
        },
        {
            "post_id": 5,
            "user_id": 5,
            "content": "新书到货，开读！📚",
            "image_urls": null,
            "like_count": 0,
            "comment_count": 0,
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:22",
            "updated_at": "2025-03-19T03:50:22",
            "username": "陈七",
            "avatar_url": "https://example.com/avatars/5.jpg",
            ".comment_count": 1
        }
    ]
}

POST http://localhost:8000/posts
Content-Type: application/json

{
    "user_id": 1,
    "content": "周末去爬山！",
    "image_urls": ["mountain.jpg", "sunset.jpg"]
}

{
    "post_id": 12,
    "message": "Post created successfully"
}

DELETE http://localhost:8000/posts/123?user_id=1

{
    "message": "Post deleted successfully"
}


POST http://localhost:8000/posts/1/comments
Body (JSON):
{
    "user_id": 2,
    "content": "风景真美！",
    "parent_id": 0
}

get http://localhost:8000/posts/2/comments?page=1&page_size=3



{
    "user_id": 2,
    "content": "风景真美！",
    "parent_id": 0
}

{
    "data": [
        {
            "comment_id": 3,
            "post_id": 2,
            "user_id": 1,
            "parent_id": 0,
            "content": "这家咖啡好喝吗？",
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:24",
            "updated_at": "2025-03-19T03:50:24",
            "username": "张三",
            "avatar_url": "https://example.com/avatars/1.jpg",
            "reply_count": 1
        },
        {
            "comment_id": 4,
            "post_id": 2,
            "user_id": 4,
            "parent_id": 3,
            "content": "超好喝！推荐拿铁！",
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:24",
            "updated_at": "2025-03-19T03:50:24",
            "username": "赵六",
            "avatar_url": "https://example.com/avatars/4.jpg",
            "reply_count": 0
        }
    ]
}



 get ： http://localhost:8000/users/1/posts?page=1&page_size=10


 {
    "data": [
        {
            "post_id": 1,
            "user_id": 1,
            "content": "今天天气真好！",
            "image_urls": "[\"https://example.com/posts/1_1.jpg\"]",
            "like_count": 0,
            "comment_count": 1,
            "is_deleted": 0,
            "created_at": "2025-03-19T03:50:22",
            "updated_at": "2025-03-19T04:18:12",
            "username": "张三",
            "avatar_url": "https://example.com/avatars/1.jpg"
        }
    ]
}
```

step4:service

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable, catchError, throwError } from 'rxjs';

// firend.service.ts
export interface Post {
  post_id: number;
  user_id: number;
  content: string;
  image_urls: string;  // 修正为string类型
  like_count: number;
  comment_count: number;
  is_deleted: boolean;
  created_at: string;
  updated_at: string;
  username: string;
  avatar_url: string;
}

export interface DisplayPost extends Post {
  images: string[];  // 转换后的图片数组
}


export interface Comment {
  comment_id: number;
  post_id: number;
  user_id: number;
  parent_id: number;
  content: string;
  is_deleted: boolean;
  created_at: string;
  updated_at: string;
  username: string;
  avatar_url: string;
  reply_count: number;
}



@Injectable({
  providedIn: 'root'
})
export class FirendService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  // 获取朋友圈列表
  getPosts(params: {
    page?: number;
    page_size?: number;
    order_by?: 'newest' | 'hottest';
    user_id?: number;
  }): Observable<{ data: Post[] }> {
    let httpParams = new HttpParams();

    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined) {
        httpParams = httpParams.set(key, value.toString());
      }
    });

    return this.http.get<{ data: Post[] }>(`${this.apiUrl}/posts`, { params: httpParams })
      .pipe(
        catchError(this.handleError)
      );
  }

  // 创建新帖子
  createPost(postData: {
    user_id: number;
    content: string;
    image_urls?: string[];
  }): Observable<{ post_id: number; message: string }> {
    return this.http.post<{ post_id: number; message: string }>(
      `${this.apiUrl}/posts`,
      postData
    ).pipe(
      catchError(this.handleError)
    );
  }

  // 删除帖子
  deletePost(postId: number, userId: number): Observable<{ message: string }> {
    return this.http.delete<{ message: string }>(
      `${this.apiUrl}/posts/${postId}`,
      { params: { user_id: userId.toString() } }
    ).pipe(
      catchError(this.handleError)
    );
  }

  // 添加评论
  addComment(postId: number, commentData: {
    user_id: number;
    content: string;
    parent_id?: number;
  }): Observable<{ message: string }> {
    return this.http.post<{ message: string }>(
      `${this.apiUrl}/posts/${postId}/comments`,
      commentData
    ).pipe(
      catchError(this.handleError)
    );
  }

  // 获取评论列表
  getComments(postId: number, params: {
    page?: number;
    page_size?: number;
  }): Observable<{ data: Comment[] }> {
    let httpParams = new HttpParams();

    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined) {
        httpParams = httpParams.set(key, value.toString());
      }
    });

    return this.http.get<{ data: Comment[] }>(
      `${this.apiUrl}/posts/${postId}/comments`,
      { params: httpParams }
    ).pipe(
      catchError(this.handleError)
    );
  }

  // 获取用户帖子
  getUserPosts(userId: number, params: {
    page?: number;
    page_size?: number;
  }): Observable<{ data: Post[] }> {
    let httpParams = new HttpParams();

    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined) {
        httpParams = httpParams.set(key, value.toString());
      }
    });

    return this.http.get<{ data: Post[] }>(
      `${this.apiUrl}/users/${userId}/posts`,
      { params: httpParams }
    ).pipe(
      catchError(this.handleError)
    );
  }

  private handleError(error: any) {
    console.error('API Error:', error);
    let errorMessage = 'An unknown error occurred!';
    if (error.error instanceof ErrorEvent) {
      errorMessage = `Client-side error: ${error.error.message}`;
    } else if (error.status) {
      errorMessage = `Server error ${error.status}: ${error.error?.detail || error.message}`;
    }
    return throwError(() => new Error(errorMessage));
  }
}

```

step5:ts  == C:\Users\wangrusheng\WebstormProjects\untitled5\src\app\user\user.component.ts

```typescript
import { Component, OnInit  } from '@angular/core';
import { FirendService,Post,Comment,DisplayPost } from '../service/firend.service';
import {FormsModule} from '@angular/forms';
import {NgForOf, NgIf} from '@angular/common';




@Component({
  selector: 'app-post',
  templateUrl: './user.component.html',
  imports: [
    FormsModule,
    NgForOf,
    NgIf
  ],
  styleUrl: './user.component.css'
})
export class UserComponent  implements OnInit {

  posts: DisplayPost[] = [];  // 使用扩展后的类型

  // 输入绑定参数
  inputPostId: number = 1;  // 1 or 2
  inputUserId: number = 1;  //1 or 2
  commentPostId: number = 3;  // 3 or  4

  // 控制台输出显示
  consoleOutput: string = '';

  // 在原有方法中添加日志记录
  private log(message: string) {
    this.consoleOutput += `${new Date().toLocaleTimeString()}: ${message}\n`;
  }



  constructor(private firendService: FirendService) {}

  ngOnInit() {
    this.firendService.getPosts({ order_by: 'newest' }).subscribe({
      next: (res) => {
        console.error(res)
        this.posts = res.data.map(post => ({
          ...post,
          images: post.image_urls ? JSON.parse(post.image_urls) : []
        }));
      },
      error: (err) => console.error('加载失败:', err)
    });
  }

  // 新增时间格式化方法
  formatTime(timeString: string): string {
    const options: Intl.DateTimeFormatOptions = {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit'
    };
    return new Date(timeString).toLocaleString('zh-CN', options);
  }


  // 获取热门帖子
  loadHotPosts(): void {
    this.firendService.getPosts({
      page: 1,
      page_size: 5,
      order_by: 'hottest'
    }).subscribe({
      next: (res) => console.log('热门帖子:', res.data),
      error: (err) => console.error(err)
    });
  }

  // 创建新帖子
  createNewPost(): void {
    const newPost = {
      user_id: 1,
      content: '周末去爬山！',
      image_urls: ['mountain.jpg', 'sunset.jpg']
    };

    this.firendService.createPost(newPost).subscribe({
      next: (res) => console.log('创建成功:', res.post_id),
      error: (err) => console.error(err)
    });
  }

  // 删除帖子
  deletePost(postId: number): void {
    this.firendService.deletePost(postId, 1).subscribe({
      next: () => console.log('删除成功'),
      error: (err) => console.error(err)
    });
  }

  // 添加评论
  addComment(postId: number): void {
    const newComment = {
      user_id: 2,
      content: '风景真美！',
      parent_id: 0
    };

    this.firendService.addComment(postId, newComment).subscribe({
      next: () => console.log('评论成功'),
      error: (err) => console.error(err)
    });
  }

  // 获取评论
  loadComments(postId: number): void {
    this.firendService.getComments(postId, {
      page: 1,
      page_size: 3
    }).subscribe({
      next: (res) => console.log('评论列表:', res.data),
      error: (err) => console.error(err)
    });
  }

  // 获取用户帖子
  loadUserPosts(userId: number): void {
    this.firendService.getUserPosts(userId, {
      page: 1,
      page_size: 10
    }).subscribe({
      next: (res) => console.log('用户帖子:', res.data),
      error: (err) => console.error(err)
    });
  }
}

```

step6:html

```xml
<div class="test-container">

  <!-- 朋友圈内容列表 -->
  <div class="post-grid">
    <div *ngFor="let post of posts" class="post-card">
      <div class="post-header">
        <img [src]="post.avatar_url" alt="头像" class="avatar">
        <div class="user-info">
          <h3>{{ post.username }}</h3>
          <span class="post-time">{{ formatTime(post.created_at) }}</span>
        </div>
      </div>

      <div class="post-content">
        <p>{{ post.content }}</p>
        <div class="image-grid" *ngIf="post.images.length">
          <img *ngFor="let img of post.images" [src]="img" alt="动态图片">
        </div>
      </div>

      <div class="interaction">
        <span class="likes">❤ {{ post.like_count }} 赞</span>
        <span class="comments">💬 {{ post.comment_count }} 评论</span>
      </div>
    </div>
  </div>

  <!-- 测试按钮区域 -->
  <div class="test-section">
    <h3>帖子操作测试</h3>

    <div class="test-group">
      <button (click)="loadHotPosts()">加载热门帖子</button>
      <button (click)="createNewPost()">创建新帖子</button>
    </div>

    <div class="test-group">
      <input type="number" [(ngModel)]="inputPostId" placeholder="帖子ID">
      <button (click)="deletePost(inputPostId)">删除帖子</button>
      <button (click)="loadComments(inputPostId)">加载评论</button>
    </div>

    <div class="test-group">
      <input type="number" [(ngModel)]="inputUserId" placeholder="用户ID">
      <button (click)="loadUserPosts(inputUserId)">加载用户帖子</button>
    </div>

    <div class="test-group">
      <input type="number" [(ngModel)]="commentPostId" placeholder="帖子ID">
      <button (click)="addComment(commentPostId)">添加评论</button>
    </div>
  </div>

  <!-- 结果显示区域 -->
  <div class="result-area">
    <h4>控制台输出：</h4>
    <pre>{{ consoleOutput }}</pre>
  </div>
</div>

```

step7:css

```css
.test-container {
  padding: 20px;
  font-family: Arial, serif;
}

.test-section {
  margin-bottom: 30px;
  padding: 15px;
  border: 1px solid #eee;
}

.test-group {
  margin: 10px 0;
  display: flex;
  gap: 10px;
  align-items: center;
}

button {
  padding: 8px 15px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: opacity 0.3s;
}

button:hover {
  opacity: 0.8;
}

input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 120px;
}

.result-area {
  margin-top: 20px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 6px;
}

pre {
  white-space: pre-wrap;
  word-wrap: break-word;
}
.content-container {
  display: grid;
  grid-template-columns: 3fr 1fr;
  gap: 2rem;
  padding: 2rem;
}

.post-grid {
  display: grid;
  gap: 1.5rem;
}

.post-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  padding: 1.5rem;
  transition: transform 0.2s;
}

.post-card:hover {
  transform: translateY(-3px);
}

.post-header {
  display: flex;
  align-items: center;
  margin-bottom: 1rem;
}

.avatar {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  margin-right: 1rem;
}

.user-info h3 {
  margin: 0;
  font-size: 1.1rem;
  color: #2d3748;
}

.post-time {
  font-size: 0.85rem;
  color: #718096;
}

.post-content p {
  margin: 0 0 1rem 0;
  line-height: 1.6;
  color: #4a5568;
}

.image-grid {
  display: grid;
  gap: 0.5rem;
  margin-top: 1rem;
}

/* 不同图片数量的布局 */
.image-grid img {
  width: 100%;
  border-radius: 8px;
  object-fit: cover;
}

.image-grid img:only-child {
  max-height: 400px;
}

.image-grid img:nth-child(2) {
  grid-template-columns: repeat(2, 1fr);
}

.image-grid img:nth-child(3) {
  grid-template-columns: repeat(3, 1fr);
}

.interaction {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid #eee;
  display: flex;
  gap: 1.5rem;
}

.likes, .comments {
  display: flex;
  align-items: center;
  gap: 0.3rem;
  color: #718096;
  cursor: pointer;
  transition: color 0.2s;
}

.likes:hover { color: #f56565; }
.comments:hover { color: #48bb78; }



```

end