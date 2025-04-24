说明：fastapi+angular实现个人博客
博客系统：
1.文章列表
展示所有文章列表，包括文章名称 作者名称 文章摘要  和两个按钮，编辑和浏览
2.编辑文章 
编辑界面，可以修改文章标题和文章内容，然后点击保存，需要更新数据库数据
3. 浏览文章
 浏览界面 包括 文章名称  作者   发布或者修改时间， 文章分类  文章标签文章内容，底部显示对应文章的评论列表

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/04156f7619244dcb95e464b3a1c86b63.png#pic_center)

step1:首先是mysql，建表，添加数据，查询

```sql

 -- ----------------------------
-- 创建数据库
-- ----------------------------
CREATE DATABASE IF NOT EXISTS db_school
DEFAULT CHARSET = utf8mb4
DEFAULT COLLATE = utf8mb4_unicode_ci;

USE db_school;

-- ----------------------------
-- 1. 用户表（含管理员标识）
-- ----------------------------
CREATE TABLE IF NOT EXISTS users (
    user_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY COMMENT '用户ID',
    username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名',
    password_hash CHAR(60) NOT NULL COMMENT 'BCrypt加密密码',
    email VARCHAR(100) NOT NULL UNIQUE COMMENT '邮箱',
    avatar_url VARCHAR(255) DEFAULT '/default-avatar.png' COMMENT '头像地址',
    is_admin TINYINT(1) UNSIGNED DEFAULT 0 COMMENT '是否管理员',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 添加索引
CREATE INDEX idx_users_email ON users(email);

-- ----------------------------
-- 2. 分类表
-- ----------------------------
CREATE TABLE IF NOT EXISTS categories (
    category_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE COMMENT '分类名称',
    slug VARCHAR(100) NOT NULL UNIQUE COMMENT 'URL友好标识',
    description VARCHAR(200) COMMENT '分类描述',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- 3. 标签表
-- ----------------------------
CREATE TABLE IF NOT EXISTS tags (
    tag_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE COMMENT '标签名称',
    color CHAR(7) DEFAULT '#4a5568' COMMENT '标签颜色',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- 4. 文章核心表（元数据与内容分离设计）
-- ----------------------------
-- 4.1 文章元数据表
CREATE TABLE IF NOT EXISTS articles (
    article_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id INT UNSIGNED NOT NULL,
    title VARCHAR(200) NOT NULL COMMENT '文章标题',
    slug VARCHAR(200) NOT NULL UNIQUE COMMENT 'URL友好标识',
    summary VARCHAR(500) COMMENT '文章摘要',
    cover_url VARCHAR(255) COMMENT '封面图地址',
    category_id INT UNSIGNED,
    status ENUM('draft', 'published', 'archived') DEFAULT 'draft',
    view_count INT UNSIGNED DEFAULT 0 COMMENT '阅读量',
    is_top TINYINT(1) UNSIGNED DEFAULT 0 COMMENT '是否置顶',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES categories(category_id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;

-- 4.2 文章内容表（分表存储优化）
CREATE TABLE IF NOT EXISTS article_contents (
    article_id INT UNSIGNED PRIMARY KEY,
    content LONGTEXT NOT NULL COMMENT 'Markdown格式内容',
    content_hash CHAR(64) NOT NULL COMMENT '内容SHA256校验值',
    FOREIGN KEY (article_id) REFERENCES articles(article_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC;

-- 添加全文索引（英文支持）
ALTER TABLE article_contents ADD FULLTEXT INDEX idx_content (content);

-- ----------------------------
-- 5. 文章标签关联表（多对多关系）
-- ----------------------------
CREATE TABLE IF NOT EXISTS article_tags (
    article_id INT UNSIGNED NOT NULL,
    tag_id INT UNSIGNED NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (article_id, tag_id),
    FOREIGN KEY (article_id) REFERENCES articles(article_id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(tag_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- 6. 评论表（支持嵌套评论）
-- ----------------------------
CREATE TABLE IF NOT EXISTS comments (
    comment_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    article_id INT UNSIGNED NOT NULL,
    user_id INT UNSIGNED NOT NULL,
    content TEXT NOT NULL,
    parent_id BIGINT UNSIGNED DEFAULT NULL COMMENT '父评论ID',
    depth TINYINT UNSIGNED DEFAULT 0 COMMENT '评论层级',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (article_id) REFERENCES articles(article_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (parent_id) REFERENCES comments(comment_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- 插入测试数据
-- ----------------------------
-- 用户数据（密码均为123456）
INSERT INTO users (username, password_hash, email, is_admin) VALUES
('admin', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'admin@blog.com', 1),
('writer', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'writer@blog.com', 0);



-- 补充用户数据（共10条，密码均为123456）
INSERT INTO users (username, password_hash, email, is_admin) VALUES
('user3', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user3@blog.com', 0),
('user4', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user4@blog.com', 0),
('user5', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user5@blog.com', 0),
('user6', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user6@blog.com', 0),
('user7', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user7@blog.com', 0),
('user8', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user8@blog.com', 0),
('user9', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user9@blog.com', 0),
('user10', '$2a$12$5h6B6oN3q1O7eDv9QzZV0.9xTJZ8Wq1k8Lw7sB6Yf5tKjV1mYJ/Ga', 'user10@blog.com', 0);


-- 分类数据
INSERT INTO categories (name, slug) VALUES
('技术', 'tech'),
('生活', 'life'),
('旅行', 'travel');


-- 补充分类数据（共10条）
INSERT INTO categories (name, slug, description) VALUES
('编程', 'programming', '编程语言学习与实践'),
('数据库', 'database', '数据库技术与应用'),
('前端', 'frontend', '前端开发技术'),
('后端', 'backend', '服务端开发实践'),
('机器学习', 'ai', '人工智能与机器学习'),
('健康', 'health', '健康生活方式'),
('教育', 'education', '教育学习资源'),
('书籍', 'books', '好书推荐与阅读'),
('电影', 'movies', '影视文化赏析'),
('音乐', 'music', '音乐艺术欣赏');


-- 标签数据
INSERT INTO tags (name, color) VALUES
('Python', '#3572A5'),
('前端', '#E34F26'),
('数据库', '#00758F'),
('美食', '#FF6B6B');



-- 补充标签数据（共10条）
INSERT INTO tags (name, color) VALUES
('JavaScript', '#f1e05a'),
('Java', '#b07219'),
('Docker', '#0db7ed'),
('Linux', '#fcc624'),
('Git', '#f14e32'),
('React', '#61dafb'),
('Vue', '#4fc08d'),
('Node.js', '#026e00'),
('MySQL', '#00758f'),
('机器学习', '#3c873a');

-- 文章数据
INSERT INTO articles (user_id, title, slug, category_id, status) VALUES
(1, 'Python异步编程指南', 'python-async', 1, 'published'),
(2, '东京美食地图', 'tokyo-food', 3, 'published');




-- 补充文章数据（共10条）
INSERT INTO articles (user_id, title, slug, summary, category_id, status, is_top) VALUES
(3, 'JavaScript闭包详解', 'js-closure', '深入理解JavaScript闭包机制', 3, 'published', 1),
(4, 'Docker容器化部署', 'docker-deploy', '使用Docker进行项目部署', 4, 'published', 0),
(5, 'MySQL优化技巧', 'mysql-optimize', '数据库性能优化实践', 2, 'published', 1),
(6, 'React Hooks指南', 'react-hooks', 'React Hooks最佳实践', 3, 'draft', 0),
(7, 'Vue3新特性解析', 'vue3-features', 'Composition API深度解析', 3, 'published', 0),
(8, 'Linux常用命令', 'linux-commands', '系统管理员必备命令', 5, 'published', 0),
(9, 'Git高级技巧', 'git-advanced', '分支管理与冲突解决', 10, 'published', 0),
(10, '机器学习入门', 'ml-basics', '监督学习与无监督学习', 5, 'published', 0),
(1, '健康饮食指南', 'healthy-eating', '营养均衡搭配方案', 6, 'published', 0),
(2, '经典书籍推荐', 'book-recommend', '2023年必读书单', 8, 'published', 0);


-- 文章内容（包含Markdown代码块）
INSERT INTO article_contents (article_id, content, content_hash) VALUES
(1,
'## 异步函数示例\n```python\nimport asyncio\n\nasync def main():\n    print("Hello")\n    await asyncio.sleep(1)\n    print("World")\n\nasyncio.run(main())\n```',
SHA2('## 异步函数示例...', 256)),
(2,
'## 必吃拉面店\n```\n地址: 东京都新宿区xxx\n推荐: 特制酱油拉面 🍜\n```',
SHA2('## 必吃拉面店...', 256));



-- 补充文章内容（带Markdown代码示例）
INSERT INTO article_contents (article_id, content, content_hash) VALUES
(3,
'## 闭包计数器示例\n```javascript\nfunction createCounter() {\n  let count = 0;\n  return function() {\n    return ++count;\n  };\n}\nconst counter = createCounter();\nconsole.log(counter()); // 1\nconsole.log(counter()); // 2\n```',
SHA2('## 闭包计数器示例...', 256)),

(4,
'## Dockerfile示例\n```dockerfile\nFROM node:18-alpine\nWORKDIR /app\nCOPY package*.json ./\nRUN npm install\nCOPY . .\nEXPOSE 3000\nCMD ["npm", "start"]\n```',
SHA2('## Dockerfile示例...', 256)),

(5,
'## 索引优化示例\n```sql\nEXPLAIN SELECT * FROM users\nWHERE created_at > "2023-01-01"\nORDER BY user_id DESC\nLIMIT 10;\n```',
SHA2('## 索引优化示例...', 256)),

(6,
'## useState基本用法\n```jsx\nimport { useState } from "react";\nfunction Counter() {\n  const [count, setCount] = useState(0);\n  return (\n    <button onClick={() => setCount(c => c+1)}>\n      Clicked {count} times\n    </button>\n  );\n}\n```',
SHA2('## useState基本用法...', 256)),

(7,
'## setup函数示例\n```vue\n<script setup>\nimport { ref } from "vue"\nconst msg = ref("Hello Vue3!")\n</script>\n<template>\n  <h1>{{ msg }}</h1>\n</template>\n```',
SHA2('## setup函数示例...', 256)),

(8,
'## 进程查看命令\n```bash\nps aux | grep nginx\ntop -u www-data\n```',
SHA2('## 进程查看命令...', 256)),

(9,
'## 分支合并示例\n```bash\ngit checkout feature\ngit rebase main\ngit checkout main\ngit merge feature\n```',
SHA2('## 分支合并示例...', 256)),

(10,
'## 线性回归示例\n```python\nfrom sklearn.linear_model import LinearRegression\nmodel = LinearRegression()\nmodel.fit(X_train, y_train)\npredictions = model.predict(X_test)\n```',
SHA2('## 线性回归示例...', 256)),

(11,
'## 营养搭配表\n| 食物类型 | 推荐摄入量 |\n|----------|------------|\n| 蛋白质   | 每天200g   |\n| 蔬菜     | 每天500g   |\n| 碳水     | 每天300g   |\n',
SHA2('## 营养搭配表...', 256)),

(12,
'## 书单推荐\n1. 《代码整洁之道》\n2. 《深入理解计算机系统》\n3. 《人类简史》\n```\n每周阅读至少10小时\n```',
SHA2('## 书单推荐...', 256));

-- 标签关联
INSERT INTO article_tags (article_id, tag_id) VALUES
(1, 1), (1, 3),  -- Python文章关联Python和数据库标签
(2, 4);          -- 美食文章关联美食标签


-- 补充标签关联（共20条）
INSERT INTO article_tags (article_id, tag_id) VALUES
(3,2), (3,5),   -- JavaScript, 前端
(4,3), (4,5),   -- Docker, Git
(5,9),          -- MySQL
(6,6), (6,2),   -- React, JavaScript
(7,7), (7,2),   -- Vue, JavaScript
(8,4), (8,9),   -- Linux, MySQL
(9,5),          -- Git
(10,10),        -- 机器学习
(11,6),         -- 健康
(12,8);         -- 书籍

-- 评论数据（含嵌套评论）
INSERT INTO comments (article_id, user_id, content, parent_id, depth) VALUES
(1, 2, '非常实用的教程！', NULL, 0),
(1, 1, '有帮助就好 😄', 1, 1),
(2, 1, '收藏了，下次去东京一定尝试！', NULL, 0);





-- 补充评论数据（共12条）
INSERT INTO comments (article_id, user_id, content, parent_id, depth) VALUES
(3, 5, '闭包讲得很透彻！', NULL, 0),
(3, 3, '感谢支持！', 13, 1),
(4, 6, 'Docker部署确实方便', NULL, 0),
(5, 7, '索引优化效果明显', NULL, 0),
(6, 8, '什么时候发布完整版？', NULL, 0),
(7, 9, 'Vue3的Composition API真香', NULL, 0),
(8, 10, '命令总结得很全面', NULL, 0),
(9, 1, 'rebase比merge更好用', NULL, 0),
(10, 2, '机器学习入门好文', NULL, 0),
(11, 4, '健康饮食很重要', NULL, 0),
(12, 5, '书单非常经典', NULL, 0),
(5, 6, '能否讲讲联合索引？', 17, 1);


-- 包含分类名称、作者、标签列表、基础信息
SELECT
    a.article_id,
    a.title,
    u.username AS author,
    c.name AS category,
    GROUP_CONCAT(t.name) AS tags,
    a.view_count,
    a.status,
    a.is_top,
    a.created_at
FROM articles a
JOIN users u ON a.user_id = u.user_id
LEFT JOIN categories c ON a.category_id = c.category_id
LEFT JOIN article_tags at ON a.article_id = at.article_id
LEFT JOIN tags t ON at.tag_id = t.tag_id
WHERE a.status = 'published' -- 可调整状态筛选
GROUP BY a.article_id
ORDER BY a.is_top DESC, a.created_at DESC
 ; -- 分页参数


-- 获取文章基础信息
SELECT
    a.*,
    u.username,
    c.name AS category_name,
    ac.content,
    (SELECT GROUP_CONCAT(t.name)
     FROM article_tags at
     JOIN tags t ON at.tag_id = t.tag_id
     WHERE at.article_id = a.article_id) AS tags
FROM articles a
JOIN users u ON a.user_id = u.user_id
LEFT JOIN categories c ON a.category_id = c.category_id
JOIN article_contents ac ON a.article_id = ac.article_id
WHERE a.article_id = 1; -- 通过slug查询

-- 获取关联评论（支持嵌套）
SELECT
    comment_id,
    user_id,
    content,
    parent_id,
    depth,
    created_at
FROM comments
WHERE article_id = (SELECT article_id FROM articles WHERE article_id = 2)
ORDER BY parent_id ASC, created_at ASC;

START TRANSACTION;

-- 更新文章元数据
UPDATE articles
SET
    title = 'mysql新标题',
    summary = '新摘要',
    category_id = 2,
    status = 'published',
    updated_at = NOW()
WHERE article_id = 1;

-- 更新文章内容
UPDATE article_contents
SET
    content = '新的Markdown内容',
    content_hash = SHA2('新的Markdown内容', 256)
WHERE article_id = 1;

COMMIT;
```

step2:数据库mysql部分处理好了，接下来是fastapi后端接口
C:\Users\wangrusheng\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import pymysql.cursors
from pydantic import BaseModel
from pymysql import MySQLError
from datetime import datetime

app = FastAPI()

# CORS配置保持不变
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 修正数据库配置
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',  # 修正数据库名称
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


# 通用查询方法
def query_database(query: str, params=None):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall()
        connection.close()
        return result
    except MySQLError as e:
        raise HTTPException(status_code=500, detail=f"Database error: {str(e)}")


# 1. 文章列表接口
@app.get("/articles")
async def get_articles():
    """
    获取所有文章列表
    Returns: [
        {
            "article_id": int,
            "title": str,
            "author": str,
            "summary": str,
            "created_at": datetime,
            "updated_at": datetime
        }
    ]
    """
    query = """
    SELECT 
        a.article_id,
        a.title,
        u.username AS author,
        a.summary,
        a.created_at,
        a.updated_at
    FROM articles a
    JOIN users u ON a.user_id = u.user_id
    ORDER BY a.created_at DESC
    """
    return {"data": query_database(query)}


# 2. 文章详情接口
@app.get("/articles/{article_id}")
async def get_article_detail(article_id: int):
    """
    获取文章详细信息及评论
    Returns: {
        "title": str,
        "author": str,
        "created_at": datetime,
        "updated_at": datetime,
        "category": str,
        "tags": List[str],
        "content": str,
        "comments": [
            {
                "content": str,
                "commenter": str,
                "created_at": datetime
            }
        ]
    }
    """
    # 获取文章基本信息
    article_query = """
    SELECT 
        a.title,
        u.username AS author,
        a.created_at,
        a.updated_at,
        c.name AS category,
        GROUP_CONCAT(t.name) AS tags,
        ac.content
    FROM articles a
    JOIN users u ON a.user_id = u.user_id
    LEFT JOIN categories c ON a.category_id = c.category_id
    LEFT JOIN article_tags at ON a.article_id = at.article_id
    LEFT JOIN tags t ON at.tag_id = t.tag_id
    JOIN article_contents ac ON a.article_id = ac.article_id
    WHERE a.article_id = %s
    GROUP BY a.article_id
    """
    article_data = query_database(article_query, (article_id,))

    if not article_data:
        raise HTTPException(status_code=404, detail="文章未找到")

    article = article_data[0]

    # 处理标签数据
    tags = article["tags"].split(",") if article["tags"] else []

    # 获取评论数据
    comments_query = """
    SELECT 
        cm.content,
        u.username AS commenter,
        cm.created_at
    FROM comments cm
    JOIN users u ON cm.user_id = u.user_id
    WHERE cm.article_id = %s
    ORDER BY cm.created_at DESC
    """
    comments = query_database(comments_query, (article_id,))

    return {
        "data": {
            **article,
            "tags": tags,
            "comments": comments
        }
    }


# 修正后的完整代码段
class ArticleUpdate(BaseModel):
    title: str
    content: str

@app.put("/articles/{article_id}")
async def update_article(
    article_id: int,
    update_data: ArticleUpdate  # 通过模型接收请求体
):
    title = update_data.title
    content = update_data.content
    connection = None
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            # 验证文章存在
            cursor.execute("SELECT 1 FROM articles WHERE article_id = %s", (article_id,))
            if not cursor.fetchone():
                raise HTTPException(status_code=404, detail="文章不存在")

            # 更新文章表（修正这里）
            cursor.execute(
                "UPDATE articles SET title = %s, updated_at = %s WHERE article_id = %s",
                (title, datetime.now(), article_id)
            )  # 补全括号

            # 更新内容表（修正这里）
            cursor.execute(
                "UPDATE article_contents SET content = %s WHERE article_id = %s",
                (content, article_id)
            )  # 补全括号

            connection.commit()
            return {"message": "文章更新成功"}

    except MySQLError as e:
        if connection:
            connection.rollback()
        raise HTTPException(status_code=500, detail=f"数据库错误: {str(e)}")
    finally:
        if connection:
            connection.close()

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:后端接口写好了，接下来是postman测试

```bash
get请求 ：http://localhost:8000/articles/2


{
    "data": {
        "title": "东京美食地图",
        "author": "writer",
        "created_at": "2025-03-15T23:14:08",
        "updated_at": "2025-03-15T23:14:08",
        "category": "旅行",
        "tags": [
            "美食"
        ],
        "content": "## 必吃拉面店\n```\n地址: 东京都新宿区xxx\n推荐: 特制酱油拉面 🍜\n```",
        "comments": [
            {
                "content": "收藏了，下次去东京一定尝试！",
                "commenter": "admin",
                "created_at": "2025-03-15T23:14:33"
            }
        ]
    }
}

put请求 http://localhost:8000/articles/1

请求参数json

{
    "title": "新版Python异步编程指南（2024更新）",
    "content": "## 新增Python 3.12特性\n```python\nasync def coro():\n    await asyncio.sleep(1)\n```"
}

服务端返回结果：

{
    "message": "文章更新成功"
}




get请求 http://localhost:8000/articles

{
    "data": [
        {
            "article_id": 11,
            "title": "健康饮食指南",
            "author": "admin",
            "summary": "营养均衡搭配方案",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 10,
            "title": "机器学习入门",
            "author": "user10",
            "summary": "监督学习与无监督学习",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 3,
            "title": "JavaScript闭包详解",
            "author": "user3",
            "summary": "深入理解JavaScript闭包机制",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 4,
            "title": "测试Docker容器化部署",
            "author": "user4",
            "summary": "使用Docker进行项目部署",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-16T02:15:26"
        },
        {
            "article_id": 5,
            "title": "MySQL优化技巧",
            "author": "user5",
            "summary": "数据库性能优化实践",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 6,
            "title": "React Hooks指南",
            "author": "user6",
            "summary": "React Hooks最佳实践",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 7,
            "title": "angularVue3新特性解析",
            "author": "user7",
            "summary": "Composition API深度解析",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-16T02:22:03"
        },
        {
            "article_id": 8,
            "title": "Linux常用命令",
            "author": "user8",
            "summary": "系统管理员必备命令",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 9,
            "title": "Git高级技巧",
            "author": "user9",
            "summary": "分支管理与冲突解决",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 12,
            "title": "经典书籍推荐",
            "author": "writer",
            "summary": "2023年必读书单",
            "created_at": "2025-03-15T23:14:11",
            "updated_at": "2025-03-15T23:14:11"
        },
        {
            "article_id": 1,
            "title": "新版wpsPython异步编程指南（2024更新）",
            "author": "admin",
            "summary": "新摘要",
            "created_at": "2025-03-15T23:14:08",
            "updated_at": "2025-03-16T02:09:34"
        },
        {
            "article_id": 2,
            "title": "东京美食地图",
            "author": "writer",
            "summary": null,
            "created_at": "2025-03-15T23:14:08",
            "updated_at": "2025-03-15T23:14:08"
        }
    ]
}



```

step4:postman验证成功，说明后端+mysql已经完全处理好了，接下来写angular前端
安装ngx-markdown 将md转html

```bash
 npm install marked @types/marked   
```

step5:配置ngx-markdown
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.config.ts

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';
import { routes } from './app.routes';
import { provideIndexedDb } from 'ngx-indexed-db';
import { DB_CONFIG } from './db.config';
import { provideMarkdown } from 'ngx-markdown';


export const appConfig: ApplicationConfig = {
  providers: [provideMarkdown(),provideIndexedDb(DB_CONFIG),provideHttpClient(),provideZoneChangeDetection({ eventCoalescing: true }), provideRouter(routes)]
};


```

step6:添加路由

```bash
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.component.html

      <a routerLink="/article-list" routerLinkActive="active">博客列表</a>
      <a routerLink="/article-detail" routerLinkActive="active">博客详情</a>
      <a routerLink="/article-edit" routerLinkActive="active">编辑博客</a>

C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.routes.ts

  {
    path: 'article-list',
    component: ArticleListComponent,
  },


  { path: 'article-detail/:id', component: ArticleDetailComponent },
  { path: 'article-edit/:id', component: ArticleEditComponent },

```

step7:封装的网络请求
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\services\article.service.ts

```typescript
// article.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ArticleService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  getArticles(): Observable<any> {
    return this.http.get(`${this.apiUrl}/articles`);
  }

  getArticle(id: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/articles/${id}`);
  }

  updateArticle(id: number, data: { title: string, content: string }): Observable<any> {
    return this.http.put(`${this.apiUrl}/articles/${id}`, data);
  }
}

```

step8:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-list\article-list.component.ts

```typescript
// article-list.component.ts
import { Component, OnInit } from '@angular/core';
import { ArticleService } from '../services/article.service';
import {CommonModule} from '@angular/common';
import {RouterLink} from '@angular/router';

@Component({
  selector: 'app-article-list',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './article-list.component.html',
  styleUrls: ['./article-list.component.css']
})
export class ArticleListComponent implements OnInit {
  articles: any[] = [];

  constructor(private articleService: ArticleService) {}

  ngOnInit() {
    this.articleService.getArticles().subscribe({
      next: (res) => this.articles = res.data,
      error: (err) => console.error(err)
    });
  }
}

```

step9:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-list\article-list.component.html

```xml
<!-- article-list.component.html -->
<div class="container">
  <h2 class="list-title">文章列表</h2>
  <div class="article-grid">
    <div *ngFor="let article of articles" class="article-card">
      <div class="card-header">
        <h3 class="article-title">{{ article.title }}</h3>
        <span class="article-id">ID: {{ article.article_id }}</span>
      </div>
      <div class="card-body">
        <p class="author">作者：{{ article.author }}</p>
        <p class="summary">{{ article.summary || '暂无摘要' }}</p>
      </div>
      <div class="card-actions">
        <a [routerLink]="['/article-detail', article.article_id]" class="action-btn view-btn">浏览</a>
        <a [routerLink]="['/article-edit', article.article_id]" class="action-btn edit-btn">编辑</a>
      </div>
    </div>
  </div>
</div>

```

step10: C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-list\article-list.component.css

```css
/* article-list.component.css */
.container {
  padding: 2rem;
  background-color: #f5f7fa;  /* 浅灰色背景 */
}

.list-title {
  color: #2c3e50;  /* 深蓝色字体 */
  text-align: center;
  margin-bottom: 2rem;
}

.article-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);  /* 三列布局 */
  gap: 1.5rem;
  max-width: 1200px;
  margin: 0 auto;
}

.article-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  transition: transform 0.2s;
}

.article-card:hover {
  transform: translateY(-5px);
}

.card-header {
  border-bottom: 1px solid #eee;
  padding-bottom: 1rem;
  margin-bottom: 1rem;
}

.article-title {
  color: #2c3e50;
  margin: 0;
  font-size: 1.25rem;
}

.article-id {
  display: block;
  color: #7f8c8d;  /* 浅灰色 */
  font-size: 0.9rem;
  margin-top: 0.5rem;
}

.card-body {
  min-height: 120px;
}

.author {
  color: #3498db;  /* 蓝色 */
  margin: 0.5rem 0;
}

.summary {
  color: #34495e;  /* 深灰色 */
  line-height: 1.6;
  font-size: 0.95rem;
}

.card-actions {
  display: flex;
  gap: 1rem;
  margin-top: 1.5rem;
}

.action-btn {
  padding: 8px 20px;
  border-radius: 25px;  /* 圆角 */
  text-decoration: none;
  font-weight: 500;
  transition: all 0.3s;
  flex: 1;
  text-align: center;
}

.view-btn {
  background-color: #3498db;  /* 蓝色 */
  color: white;
  border: 2px solid #3498db;
}

.edit-btn {
  background-color: #2ecc71;  /* 绿色 */
  color: white;
  border: 2px solid #2ecc71;
}

.action-btn:hover {
  opacity: 0.9;
  transform: scale(0.98);
}

/* 响应式设计 */
@media (max-width: 992px) {
  .article-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 768px) {
  .article-grid {
    grid-template-columns: 1fr;
  }
}

```

step11:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-edit\article-edit.component.ts

```typescript
// article-edit.component.ts
import { Component, OnInit } from '@angular/core';
import { ArticleService } from '../services/article.service';
import { ActivatedRoute, Router } from '@angular/router';
import { FormsModule } from '@angular/forms';
import { MarkdownComponent } from 'ngx-markdown';

@Component({
  selector: 'app-article-edit',
  standalone: true,
  imports: [FormsModule, MarkdownComponent],
  templateUrl: './article-edit.component.html',
  styleUrls: ['./article-edit.component.css']
})
export class ArticleEditComponent implements OnInit {
  article: any = { title: '', content: '' };
  isSubmitting = false;

  constructor(
    private service: ArticleService,
    private route: ActivatedRoute,
    private router: Router
  ) {}

  // article-edit.component.ts
  ngOnInit() {
    // 正确获取参数并转换为数字
    const idParam = this.route.snapshot.paramMap.get('id');
    const articleId = Number(idParam);

    if (!articleId || isNaN(articleId)) {
      console.error('无效的文章ID');
      return;
    }

    this.service.getArticle(articleId).subscribe({
      next: (res) => {
        this.article = res.data;
        // 确保数据包含ID
        this.article.article_id = articleId;
      },
      error: (err) => console.error('加载失败:', err)
    });
  }

 /* ngOnInit() {
    const id = this.route.snapshot.params['id'];
    this.service.getArticle(id).subscribe({
      next: (res) => this.article = res.data,
      error: (err) => console.error(err)
    });
  }
*/

  onSubmit() {
    const articleId = Number(this.route.snapshot.paramMap.get('id'));

    if (!articleId) {
      console.error('提交时ID无效');
      return;
    }

    this.isSubmitting = true;

    this.service.updateArticle(articleId, { // 使用路由参数中的ID
      title: this.article.title,
      content: this.article.content
    }).subscribe({
      next: () => this.router.navigate(['/article-detail', articleId]),
      error: (err) => {
        console.error('保存失败:', err);
        this.isSubmitting = false;
      }
    });
  }

/*  onSubmit() {
    this.isSubmitting = true;
    this.service.updateArticle(this.article.article_id, {
      title: this.article.title,
      content: this.article.content
    }).subscribe({
      next: () => this.router.navigate(['/article-detail', this.article.article_id]),
      error: (err) => {
        console.error(err);
        this.isSubmitting = false;
      }
    });
  }*/
}

```

step12:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-edit\article-edit.component.html

```xml
<!-- article-edit.component.html -->
<div class="container">
  <h2>编辑文章</h2>
  <form (ngSubmit)="onSubmit()">
    <div class="form-group">
      <label>标题</label>
      <input type="text" [(ngModel)]="article.title" name="title" required>
    </div>

    <div class="form-group">
      <label>内容</label>
      <textarea [(ngModel)]="article.content" name="content"
                rows="15" required></textarea>
    </div>

    <div class="preview">
      <h3>预览</h3>
      <markdown [data]="article.content"></markdown>
    </div>

    <button type="submit" [disabled]="isSubmitting">
      {{ isSubmitting ? '保存中...' : '保存' }}
    </button>
  </form>
</div>

```

step13: C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-detail\article-detail.component.ts

```typescript
// article-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ArticleService } from '../services/article.service';
import { ActivatedRoute } from '@angular/router';
import { CommonModule, DatePipe } from '@angular/common';
import { MarkdownComponent } from 'ngx-markdown';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-article-detail',
  standalone: true,
  imports: [
    CommonModule,
    MarkdownComponent,
    DatePipe,
    RouterModule
  ],
  templateUrl: './article-detail.component.html',
  styleUrls: ['./article-detail.component.css']
})
export class ArticleDetailComponent implements OnInit {
  article: any = null;
  isLoading = true;
  errorMessage: string | null = null;

  constructor(
    private articleService: ArticleService,
    private route: ActivatedRoute
  ) {}

  ngOnInit(): void {
    this.loadArticle();
  }

  private loadArticle(): void {
    const articleId = this.route.snapshot.paramMap.get('id');

    if (!articleId || isNaN(+articleId)) {
      this.errorMessage = '无效的文章ID';
      this.isLoading = false;
      return;
    }

    this.articleService.getArticle(+articleId).subscribe({
      next: (res) => {
        this.article = res.data;
        this.isLoading = false;
      },
      error: (err) => {
        console.error('加载文章失败:', err);
        this.errorMessage = '无法加载文章内容，请稍后重试';
        this.isLoading = false;
      }
    });
  }

  formatTags(tagsString: string): string[] {
    return tagsString ? tagsString.split(',') : [];
  }
}

```

step14:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-detail\article-detail.component.html

```xml
<!-- article-detail.component.html -->
<div class="container">
  <!-- 加载状态 -->
  <div *ngIf="isLoading" class="loading">
    <div class="spinner"></div>
    <p>正在加载文章...</p>
  </div>

  <!-- 错误提示 -->
  <div *ngIf="errorMessage" class="error">
    {{ errorMessage }}
    <a routerLink="/article-list">返回文章列表</a>
  </div>

  <!-- 正常内容 -->
  <div *ngIf="article && !isLoading">
    <h1>{{ article.title }}</h1>

    <div class="meta-info">
      <span class="author">
        <i class="icon-user"></i>
        {{ article.author }}
      </span>
      <span class="time">
        <i class="icon-clock"></i>
        {{ article.updated_at | date: 'yyyy-MM-dd HH:mm' }}
      </span>
      <span *ngIf="article.category" class="category">
        <i class="icon-folder"></i>
        {{ article.category }}
      </span>
    </div>

    <div class="tags">
      <span *ngFor="let tag of article.tags" class="tag">
        <i class="icon-tag"></i>
        {{ tag }}
      </span>
    </div>

    <div class="content">
      <markdown [data]="article.content"></markdown>
    </div>

    <div class="comments-section">
      <h3>评论（{{ article.comments?.length || 0 }}）</h3>

      <div *ngIf="article.comments?.length === 0" class="no-comments">
        暂无评论，快来抢沙发~
      </div>

      <div *ngFor="let comment of article.comments" class="comment">
        <div class="comment-header">
          <span class="comment-author">{{ comment.commenter }}</span>
          <span class="comment-time">{{ comment.created_at | date: 'yyyy-MM-dd HH:mm' }}</span>
        </div>
        <div class="comment-content">{{ comment.content }}</div>
      </div>
    </div>
  </div>
</div>

```

step15:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\article-detail\article-detail.component.css


```css
/* 容器样式 */
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  line-height: 1.6;
}

/* 元信息样式 */
.meta-info {
  display: flex;
  gap: 1.5rem;
  color: #666;
  margin: 1rem 0;
  font-size: 0.9rem;
}

.meta-info i {
  margin-right: 0.3rem;
}

/* 标签样式 */
.tags {
  margin: 1rem 0;
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tag {
  background: #f0f0f0;
  padding: 0.3rem 0.8rem;
  border-radius: 1rem;
  font-size: 0.85rem;
}

/* Markdown内容样式 */
.content {
  margin: 2rem 0;
}

/* 评论样式 */
.comments-section {
  margin-top: 3rem;
  border-top: 1px solid #eee;
  padding-top: 2rem;
}

.comment {
  margin: 1.5rem 0;
  padding: 1rem;
  background: #f8f9fa;
  border-radius: 6px;
}

.comment-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
  color: #666;
}

/* 加载状态 */
.loading {
  text-align: center;
  padding: 3rem;
}

.spinner {
  width: 3rem;
  height: 3rem;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 错误提示 */
.error {
  color: #dc3545;
  padding: 2rem;
  text-align: center;
  border: 1px solid #ffe3e6;
  border-radius: 6px;
  background: #fff5f5;
}

```

end