说明：
我计划用fastapi+vue做一款在线小说系统，功能包括：

```bash
1.sql部分：
		1.1 mysql建表
2.后端部分：
		2.1 写接口 处理业务，对sql表进行处理
		2.2. 新增小说
		2.3. 新增章节
		2.4. 获取小说列表
		2.5. 获取章节列表
3.vue前端部分：
		3.1 小说列表，
			 3.1.1左边分类列表，右边小说列表，
			 3.1.12 点击分类列表，展示对应的小说列表
	    3.2新增小说  仿照阅文作家平台，可以输入书名，简介，创建小说
		3.3章节列表
				3.3.1 获取服务端的接口，展示章节列表
				3.3.2新增章节
				3.3.3 输入章节的内容和标题，点击发布，提交章节内容
				3.3.4 发布章节成功后，刷新左侧的章节列表
```

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/51b03171664e4bb285a5e5a981328a0d.png#pic_center)

step1:sql

```sql


show databases;

DROP TABLE users;


SHOW CREATE TABLE product_category;

show tables;

use db_school;

SELECT * FROM db_school.jewelry_categories;

CREATE DATABASE db_school;

-- 用户表（存储用户信息）
CREATE TABLE `user` (
  `user_id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `password` VARCHAR(255) NOT NULL COMMENT '加密后的密码',
  `email` VARCHAR(100) NOT NULL COMMENT '邮箱',
  `avatar` VARCHAR(255) DEFAULT NULL COMMENT '头像URL',
  `role` ENUM('user','author','admin') DEFAULT 'user' COMMENT '用户角色',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `username` (`username`),
  UNIQUE KEY `email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 小说分类表
CREATE TABLE `category` (
  `category_id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(20) NOT NULL COMMENT '分类名称',
  `description` VARCHAR(255) DEFAULT NULL,
  PRIMARY KEY (`category_id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 小说主表
CREATE TABLE `novel` (
  `novel_id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `title` VARCHAR(100) NOT NULL COMMENT '小说标题',
  `cover` VARCHAR(255) DEFAULT NULL COMMENT '封面URL',
  `author_id` INT UNSIGNED NOT NULL COMMENT '作者ID',
  `category_id` INT UNSIGNED NOT NULL COMMENT '分类ID',
  `description` TEXT COMMENT '小说简介',
  `status` ENUM('serial','finished') DEFAULT 'serial' COMMENT '连载状态',
  `word_count` INT UNSIGNED DEFAULT 0 COMMENT '总字数',
  `is_vip` TINYINT(1) DEFAULT 0 COMMENT '是否VIP作品',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` DATETIME DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`novel_id`),
  KEY `author_id` (`author_id`),
  KEY `category_id` (`category_id`),
  FOREIGN KEY (`author_id`) REFERENCES `user` (`user_id`),
  FOREIGN KEY (`category_id`) REFERENCES `category` (`category_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 章节表（建议分表或使用分区优化）
CREATE TABLE `chapter` (
  `chapter_id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `novel_id` INT UNSIGNED NOT NULL,
  `chapter_no` INT UNSIGNED NOT NULL COMMENT '章节序号',
  `title` VARCHAR(100) NOT NULL,
  `content` LONGTEXT NOT NULL COMMENT '章节内容',
  `word_count` INT UNSIGNED DEFAULT 0,
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`chapter_id`),
  UNIQUE KEY `novel_chapter` (`novel_id`,`chapter_no`),
  FOREIGN KEY (`novel_id`) REFERENCES `novel` (`novel_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

select *from chapter;

-- 标签表（多对多关系）
CREATE TABLE `tag` (
  `tag_id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(20) NOT NULL,
  PRIMARY KEY (`tag_id`),
  UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 小说标签关联表
CREATE TABLE `novel_tag` (
  `novel_id` INT UNSIGNED NOT NULL,
  `tag_id` INT UNSIGNED NOT NULL,
  PRIMARY KEY (`novel_id`,`tag_id`),
  FOREIGN KEY (`novel_id`) REFERENCES `novel` (`novel_id`),
  FOREIGN KEY (`tag_id`) REFERENCES `tag` (`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 书架/收藏表
CREATE TABLE `bookshelf` (
  `user_id` INT UNSIGNED NOT NULL,
  `novel_id` INT UNSIGNED NOT NULL,
  `last_read_chapter_id` BIGINT UNSIGNED DEFAULT NULL COMMENT '最后阅读章节',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`user_id`,`novel_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`),
  FOREIGN KEY (`novel_id`) REFERENCES `novel` (`novel_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 评论表（支持二级评论）
CREATE TABLE `comment` (
  `comment_id` BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  `user_id` INT UNSIGNED NOT NULL,
  `novel_id` INT UNSIGNED NOT NULL,
  `chapter_id` BIGINT UNSIGNED DEFAULT NULL,
  `parent_id` BIGINT UNSIGNED DEFAULT NULL COMMENT '父评论ID',
  `content` TEXT NOT NULL,
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`comment_id`),
  KEY `novel_id` (`novel_id`),
  KEY `chapter_id` (`chapter_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user` (`user_id`),
  FOREIGN KEY (`novel_id`) REFERENCES `novel` (`novel_id`),
  FOREIGN KEY (`chapter_id`) REFERENCES `chapter` (`chapter_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 插入用户数据（3位用户，包含作者和管理员）
INSERT INTO `user` (`username`, `password`, `email`, `avatar`, `role`) VALUES
('user1', '$2y$10$Eic5xYRL9J4o1NjBpFqYQe5jZ7Jzv6Uuq8XoWzRkLm6Vb0Jc7tDdK', 'user1@test.com', NULL, 'user'),
('author1', '$2y$10$o8vR7V9LqGd5VgT5h4kF4OjLz0lXl6sZ3M3r0bQwK1nY7Jc7tDdK', 'author1@test.com', '/avatar/author1.jpg', 'author'),
('admin1', '$2y$10$l9vR7V9LqGd5VgT5h4kF4OjLz0lXl6sZ3M3r0bQwK1nY7Jc7tDdK', 'admin1@test.com', NULL, 'admin');

-- 插入分类数据（2个分类）
INSERT INTO `category` (`name`, `description`) VALUES
('玄幻', '东方玄幻、异世大陆等幻想类作品'),
('都市', '现代都市背景的言情、社会类作品');

-- 插入小说数据（2本小说，作者为author1）
INSERT INTO `novel` (`title`, `cover`, `author_id`, `category_id`, `description`, `status`, `word_count`, `is_vip`) VALUES
('斗破苍穹', 'https://randomuser.me/api/portraits/men/1.jpg', 2, 1, '这里是属于斗气的世界，没有花俏艳丽的魔法...', 'finished', 5000000, 1),
('都市神医', 'https://randomuser.me/api/portraits/men/2.jpg', 2, 2, '落魄青年获得神秘传承，从此医武无双...', 'serial', 200000, 0);

-- 插入章节数据（每本小说3个章节）
INSERT INTO `chapter` (`novel_id`, `chapter_no`, `title`, `content`, `word_count`) VALUES
(1, 1, '陨落的天才', '斗之力，三段！...', 3245),
(1, 2, '斗气阁', '望着那消失在阴影中的少年...', 2987),
(2, 1, '医院奇遇', '林凡站在医院走廊里...', 4123),
(2, 2, '神秘古玉', '深夜的古玩市场...', 3789);

select *from chapter where novel_id=1;

-- 插入标签数据（3个标签）
INSERT INTO `tag` (`name`) VALUES
('热血'), ('废柴流'), ('神医');

-- 关联小说和标签
INSERT INTO `novel_tag` (`novel_id`, `tag_id`) VALUES
(1, 1), (1, 2), (2, 3);

-- 插入书架数据（用户1收藏两本小说）
INSERT INTO `bookshelf` (`user_id`, `novel_id`, `last_read_chapter_id`) VALUES
(1, 1, 1),
(1, 2, 3);

-- 插入评论数据（包含二级评论）
INSERT INTO `comment` (`user_id`, `novel_id`, `chapter_id`, `parent_id`, `content`) VALUES
(1, 1, NULL, NULL, '经典之作，百看不厌！'),
(3, 1, 1, NULL, '这个开篇真有画面感'),
(1, 1, NULL, 1, '同意！已经看了三遍了');
```

step2:test python

```python
from typing import Dict, List
import json
import pymysql
from datetime import datetime, date
from collections import defaultdict

# 数据库配置（根据实际情况修改）
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


def execute_query(query: str, params=None) -> List[Dict]:
    """执行SQL查询并返回结果"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall()
        return result
    finally:
        connection.close()


def add_novel(novel_data: Dict, tag_ids: List[int] = None) -> int:
    """
    添加新小说并关联标签
    :param novel_data: 必须包含字段：title, author_id, category_id
    :param tag_ids: 需要关联的标签ID列表
    :return: 新创建小说的ID
    """
    if tag_ids is None:
        tag_ids = []

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证作者存在
            cursor.execute("SELECT user_id FROM user WHERE user_id = %s",
                           (novel_data['author_id'],))
            if not cursor.fetchone():
                raise ValueError("作者不存在")

            # 验证分类存在
            cursor.execute("SELECT category_id FROM category WHERE category_id = %s",
                           (novel_data['category_id'],))
            if not cursor.fetchone():
                raise ValueError("分类不存在")

            # 插入小说数据
            sql = """
                INSERT INTO novel (
                    title, cover, author_id, category_id, description, 
                    status, word_count, is_vip
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
            """
            params = (
                novel_data['title'],
                novel_data.get('cover'),
                novel_data['author_id'],
                novel_data['category_id'],
                novel_data.get('description', ''),
                novel_data.get('status', 'serial'),
                novel_data.get('word_count', 0),
                novel_data.get('is_vip', False)
            )
            cursor.execute(sql, params)
            novel_id = cursor.lastrowid

            # 处理标签关联
            for tag_id in tag_ids:
                cursor.execute("SELECT tag_id FROM tag WHERE tag_id = %s", (tag_id,))
                if not cursor.fetchone():
                    raise ValueError(f"标签ID {tag_id} 不存在")
                cursor.execute(
                    "INSERT INTO novel_tag (novel_id, tag_id) VALUES (%s, %s)",
                    (novel_id, tag_id)
                )

            connection.commit()
            return novel_id
    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加小说失败: {str(e)}")
    finally:
        connection.close()



def add_chapter(chapter_data: Dict) -> int:
    """
    添加新章节到数据库
    :param chapter_data: 必须包含字段 novel_id, chapter_no, title, content
    :return: 新章节的ID
    """
    # 参数基础验证
    required_fields = ['novel_id', 'chapter_no', 'title', 'content']
    for field in required_fields:
        if field not in chapter_data:
            raise ValueError(f"缺少必要字段: {field}")

    # 字段类型验证
    if not isinstance(chapter_data['novel_id'], int) or chapter_data['novel_id'] <= 0:
        raise ValueError("novel_id 必须是正整数")

    if not isinstance(chapter_data['chapter_no'], int) or chapter_data['chapter_no'] <= 0:
        raise ValueError("chapter_no 必须是正整数")

    if not isinstance(chapter_data['title'], str) or len(chapter_data['title'].strip()) == 0:
        raise ValueError("title 不能为空")

    if not isinstance(chapter_data['content'], str) or len(chapter_data['content'].strip()) == 0:
        raise ValueError("content 不能为空")

    # 处理可选字段
    word_count = chapter_data.get('word_count', len(chapter_data['content']))
    if not isinstance(word_count, int) or word_count < 0:
        word_count = len(chapter_data['content'])  # 自动根据内容长度计算

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证小说存在
            cursor.execute(
                "SELECT novel_id FROM novel WHERE novel_id = %s",
                (chapter_data['novel_id'],))
            if not cursor.fetchone():
                raise ValueError(f"小说ID {chapter_data['novel_id']} 不存在")

            # 检查章节号是否重复
            cursor.execute(
                "SELECT chapter_id FROM chapter "
                "WHERE novel_id = %s AND chapter_no = %s",
                (chapter_data['novel_id'], chapter_data['chapter_no'])
            )
            if cursor.fetchone():
                raise ValueError("该小说已存在相同章节号")

            # 插入章节数据
            sql = """
                INSERT INTO chapter (
                    novel_id, chapter_no, title, 
                    content, word_count
                ) VALUES (%s, %s, %s, %s, %s)
            """
            params = (
                chapter_data['novel_id'],
                chapter_data['chapter_no'],
                chapter_data['title'],
                chapter_data['content'],
                word_count
            )
            cursor.execute(sql, params)
            chapter_id = cursor.lastrowid

            # 更新小说字数统计
            cursor.execute(
                "UPDATE novel SET word_count = word_count + %s "
                "WHERE novel_id = %s",
                (word_count, chapter_data['novel_id'])
            )

            connection.commit()
            return chapter_id

    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加章节失败: {str(e)}")
    finally:
        connection.close()



def generate_category_novels_json() -> List[Dict]:
    """生成包含分类及小说完整信息的JSON结构"""
    # 获取基础数据
    categories = execute_query("SELECT * FROM category ORDER BY category_id")
    novels = execute_query("""
        SELECT n.*, u.username AS author_name, c.name AS category_name 
        FROM novel n
        JOIN user u ON n.author_id = u.user_id
        JOIN category c ON n.category_id = c.category_id
    """)

    # 获取关联数据
    novel_tags = execute_query("""
        SELECT nt.novel_id, t.name 
        FROM novel_tag nt
        JOIN tag t ON nt.tag_id = t.tag_id
    """)
    chapters = execute_query("SELECT * FROM chapter ORDER BY novel_id, chapter_no")
    bookshelf_counts = execute_query("""
        SELECT novel_id, COUNT(*) AS count FROM bookshelf GROUP BY novel_id
    """)

    # 构建数据映射
    tags_map = defaultdict(list)
    for nt in novel_tags:
        tags_map[nt['novel_id']].append(nt['name'])

    chapters_map = defaultdict(list)
    for ch in chapters:
        chapters_map[ch['novel_id']].append({
            "chapter_id": ch['chapter_id'],
            "chapter_no": ch['chapter_no'],
            "title": ch['title'],
            "word_count": ch['word_count'],
            "created_at": ch['created_at'].isoformat() if ch['created_at'] else None
        })

    bookshelf_map = {bc['novel_id']: bc['count'] for bc in bookshelf_counts}

    # 构建分类结构
    category_map = defaultdict(list)
    for novel in novels:
        novel_id = novel['novel_id']
        category_map[novel['category_id']].append({
            "novel_id": novel_id,
            "title": novel['title'],
            "cover": novel['cover'],
            "author": {
                "user_id": novel['author_id'],
                "username": novel['author_name']
            },
            "description": novel['description'],
            "status": novel['status'],
            "word_count": novel['word_count'],
            "is_vip": bool(novel['is_vip']),
            "created_at": novel['created_at'].isoformat(),
            "last_updated": novel['updated_at'].isoformat() if novel['updated_at'] else None,
            "chapters": chapters_map.get(novel_id, []),
            "tags": tags_map.get(novel_id, []),
            "bookshelf_count": bookshelf_map.get(novel_id, 0)
        })

    # 构建最终JSON结构
    return [{
        "category_id": cat['category_id'],
        "name": cat['name'],
        "description": cat['description'],
        "novels": category_map.get(cat['category_id'], [])
    } for cat in categories]


def get_chapters_by_novel_id(novel_id: int) -> List[Dict]:
    """
    根据小说ID获取章节列表
    :param novel_id: 小说ID（必须大于0）
    :return: 章节列表（按章节号排序）
    """
    # 参数验证
    if not isinstance(novel_id, int) or novel_id <= 0:
        raise ValueError("小说ID必须是正整数")

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证小说存在
            cursor.execute(
                "SELECT novel_id FROM novel WHERE novel_id = %s",
                (novel_id,)
            )
            if not cursor.fetchone():
                raise ValueError("指定的小说不存在")

            # 查询章节数据
            query = """
                SELECT 
                    chapter_id,
                    chapter_no,
                    title,
                    content,
                    word_count,
                    created_at
                FROM chapter
                WHERE novel_id = %s
                ORDER BY chapter_no ASC
            """
            cursor.execute(query, (novel_id,))
            chapters = cursor.fetchall()

            # 格式化输出结果
            formatted_chapters = []
            for ch in chapters:
                formatted_chapters.append({
                    "chapter_id": ch["chapter_id"],
                    "chapter_no": ch["chapter_no"],
                    "title": ch["title"],
                    "content_preview": ch["content"][:100] + "..." if ch["content"] else "",  # 内容预览
                    "word_count": ch["word_count"],
                    "created_at": ch["created_at"].isoformat() if ch["created_at"] else None
                })
            return formatted_chapters

    except Exception as e:
        raise RuntimeError(f"获取章节列表失败: {str(e)}")
    finally:
        connection.close()



def custom_serializer(obj):
    """自定义JSON序列化处理"""
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")


# 使用示例
if __name__ == "__main__":
    try:
        # 添加新小说示例
        new_novel = {
            "title": "诡秘之主",
            "author_id": 2,
            "category_id": 1,
            "description": "迷雾笼罩的大陆，各种非凡力量开始涌现...",
            "cover": "https://randomuser.me/api/portraits/men/3.jpg",
            "status": "serial",
            "word_count": 5000,
            "is_vip": True
        }
        novel_id = add_novel(new_novel, tag_ids=[1, 3])
        print(f"成功添加小说，ID：{novel_id}")
        chapter_example = {
            "novel_id": 1,
            "chapter_no": 3,
            "title": "新的章节",
            "content": "这里是章节内容...",
            "word_count": 2500  # 可选字段
        }
        new_chapter_id = add_chapter(chapter_example)
        print(f"成功添加章节，ID：{new_chapter_id}")
        # 生成JSON示例
        result = generate_category_novels_json()
        print(json.dumps(result, indent=2, ensure_ascii=False, default=custom_serializer))

    except Exception as e:
        print(f"操作失败: {str(e)}")
```

step3: fastapi

```python
from fastapi import FastAPI,HTTPException
from fastapi.responses import Response
from typing import List, Dict
import json
from decimal import Decimal
from collections import defaultdict
import pymysql.cursors
from pydantic import BaseModel
from datetime import datetime, date
from typing import List, Optional
import uvicorn
from fastapi.middleware.cors import CORSMiddleware
from starlette import status

app = FastAPI()
# CORS配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
# 数据库配置（根据实际情况修改）
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}

# 复用之前的数据库配置和函数
# ... [保持之前的 DB_CONFIG 和函数代码不变] ...

# ======== Pydantic 请求/响应模型 ========
class AuthorInfo(BaseModel):
    user_id: int
    username: str

class ChapterInfo(BaseModel):
    chapter_id: int
    chapter_no: int
    title: str
    word_count: int
    created_at: str

class NovelInfo(BaseModel):
    novel_id: int
    title: str
    cover: Optional[str]
    author: AuthorInfo
    description: str
    status: str
    word_count: int
    is_vip: bool
    created_at: str
    last_updated: Optional[str]
    chapters: List[ChapterInfo]
    tags: List[str]
    bookshelf_count: int

class CategoryNovels(BaseModel):
    category_id: int
    name: str
    description: Optional[str]
    novels: List[NovelInfo]

class NovelCreate(BaseModel):
    title: str
    author_id: int
    category_id: int
    description: Optional[str] = ""
    cover: Optional[str] = None
    status: Optional[str] = "serial"
    word_count: Optional[int] = 0
    is_vip: Optional[bool] = False
    tags: Optional[List[int]] = []

class ChapterCreate(BaseModel):
    novel_id: int
    chapter_no: int
    title: str
    content: str
    word_count: int = None


def execute_query(query: str, params=None) -> List[Dict]:
    """执行SQL查询并返回结果"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall()
        return result
    finally:
        connection.close()


def add_novel(novel_data: Dict, tag_ids: List[int] = None) -> int:
    """
    添加新小说并关联标签
    :param novel_data: 必须包含字段：title, author_id, category_id
    :param tag_ids: 需要关联的标签ID列表
    :return: 新创建小说的ID
    """
    if tag_ids is None:
        tag_ids = []

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证作者存在
            cursor.execute("SELECT user_id FROM user WHERE user_id = %s",
                           (novel_data['author_id'],))
            if not cursor.fetchone():
                raise ValueError("作者不存在")

            # 验证分类存在
            cursor.execute("SELECT category_id FROM category WHERE category_id = %s",
                           (novel_data['category_id'],))
            if not cursor.fetchone():
                raise ValueError("分类不存在")

            # 插入小说数据
            sql = """
                INSERT INTO novel (
                    title, cover, author_id, category_id, description, 
                    status, word_count, is_vip
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
            """
            params = (
                novel_data['title'],
                novel_data.get('cover'),
                novel_data['author_id'],
                novel_data['category_id'],
                novel_data.get('description', ''),
                novel_data.get('status', 'serial'),
                novel_data.get('word_count', 0),
                novel_data.get('is_vip', False)
            )
            cursor.execute(sql, params)
            novel_id = cursor.lastrowid

            # 处理标签关联
            for tag_id in tag_ids:
                cursor.execute("SELECT tag_id FROM tag WHERE tag_id = %s", (tag_id,))
                if not cursor.fetchone():
                    raise ValueError(f"标签ID {tag_id} 不存在")
                cursor.execute(
                    "INSERT INTO novel_tag (novel_id, tag_id) VALUES (%s, %s)",
                    (novel_id, tag_id)
                )

            connection.commit()
            return novel_id
    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加小说失败: {str(e)}")
    finally:
        connection.close()

def add_chapter(chapter_data: Dict) -> int:
    """
    添加新章节到数据库
    :param chapter_data: 必须包含字段 novel_id, chapter_no, title, content
    :return: 新章节的ID
    """
    # 参数基础验证
    required_fields = ['novel_id', 'chapter_no', 'title', 'content']
    for field in required_fields:
        if field not in chapter_data:
            raise ValueError(f"缺少必要字段: {field}")

    # 字段类型验证
    if not isinstance(chapter_data['novel_id'], int) or chapter_data['novel_id'] <= 0:
        raise ValueError("novel_id 必须是正整数")

    if not isinstance(chapter_data['chapter_no'], int) or chapter_data['chapter_no'] <= 0:
        raise ValueError("chapter_no 必须是正整数")

    if not isinstance(chapter_data['title'], str) or len(chapter_data['title'].strip()) == 0:
        raise ValueError("title 不能为空")

    if not isinstance(chapter_data['content'], str) or len(chapter_data['content'].strip()) == 0:
        raise ValueError("content 不能为空")

    # 处理可选字段
    word_count = chapter_data.get('word_count', len(chapter_data['content']))
    if not isinstance(word_count, int) or word_count < 0:
        word_count = len(chapter_data['content'])  # 自动根据内容长度计算

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证小说存在
            cursor.execute(
                "SELECT novel_id FROM novel WHERE novel_id = %s",
                (chapter_data['novel_id'],))
            if not cursor.fetchone():
                raise ValueError(f"小说ID {chapter_data['novel_id']} 不存在")

            # 检查章节号是否重复
            cursor.execute(
                "SELECT chapter_id FROM chapter "
                "WHERE novel_id = %s AND chapter_no = %s",
                (chapter_data['novel_id'], chapter_data['chapter_no'])
            )
            if cursor.fetchone():
                raise ValueError("该小说已存在相同章节号")

            # 插入章节数据
            sql = """
                INSERT INTO chapter (
                    novel_id, chapter_no, title, 
                    content, word_count
                ) VALUES (%s, %s, %s, %s, %s)
            """
            params = (
                chapter_data['novel_id'],
                chapter_data['chapter_no'],
                chapter_data['title'],
                chapter_data['content'],
                word_count
            )
            cursor.execute(sql, params)
            chapter_id = cursor.lastrowid

            # 更新小说字数统计
            cursor.execute(
                "UPDATE novel SET word_count = word_count + %s "
                "WHERE novel_id = %s",
                (word_count, chapter_data['novel_id'])
            )

            connection.commit()
            return chapter_id

    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加章节失败: {str(e)}")
    finally:
        connection.close()

def generate_category_novels_json() -> List[Dict]:
    """生成包含分类及小说完整信息的JSON结构"""
    # 获取基础数据
    categories = execute_query("SELECT * FROM category ORDER BY category_id")
    novels = execute_query("""
        SELECT n.*, u.username AS author_name, c.name AS category_name 
        FROM novel n
        JOIN user u ON n.author_id = u.user_id
        JOIN category c ON n.category_id = c.category_id
    """)

    # 获取关联数据
    novel_tags = execute_query("""
        SELECT nt.novel_id, t.name 
        FROM novel_tag nt
        JOIN tag t ON nt.tag_id = t.tag_id
    """)
    chapters = execute_query("SELECT * FROM chapter ORDER BY novel_id, chapter_no")
    bookshelf_counts = execute_query("""
        SELECT novel_id, COUNT(*) AS count FROM bookshelf GROUP BY novel_id
    """)

    # 构建数据映射
    tags_map = defaultdict(list)
    for nt in novel_tags:
        tags_map[nt['novel_id']].append(nt['name'])

    chapters_map = defaultdict(list)
    for ch in chapters:
        chapters_map[ch['novel_id']].append({
            "chapter_id": ch['chapter_id'],
            "chapter_no": ch['chapter_no'],
            "title": ch['title'],
            "word_count": ch['word_count'],
            "created_at": ch['created_at'].isoformat() if ch['created_at'] else None
        })

    bookshelf_map = {bc['novel_id']: bc['count'] for bc in bookshelf_counts}

    # 构建分类结构
    category_map = defaultdict(list)
    for novel in novels:
        novel_id = novel['novel_id']
        category_map[novel['category_id']].append({
            "novel_id": novel_id,
            "title": novel['title'],
            "cover": novel['cover'],
            "author": {
                "user_id": novel['author_id'],
                "username": novel['author_name']
            },
            "description": novel['description'],
            "status": novel['status'],
            "word_count": novel['word_count'],
            "is_vip": bool(novel['is_vip']),
            "created_at": novel['created_at'].isoformat(),
            "last_updated": novel['updated_at'].isoformat() if novel['updated_at'] else None,
            "chapters": chapters_map.get(novel_id, []),
            "tags": tags_map.get(novel_id, []),
            "bookshelf_count": bookshelf_map.get(novel_id, 0)
        })

    # 构建最终JSON结构
    return [{
        "category_id": cat['category_id'],
        "name": cat['name'],
        "description": cat['description'],
        "novels": category_map.get(cat['category_id'], [])
    } for cat in categories]

def get_chapters_by_novel_id(novel_id: int) -> List[Dict]:
    """
    根据小说ID获取章节列表
    :param novel_id: 小说ID（必须大于0）
    :return: 章节列表（按章节号排序）
    """
    # 参数验证
    if not isinstance(novel_id, int) or novel_id <= 0:
        raise ValueError("小说ID必须是正整数")

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证小说存在
            cursor.execute(
                "SELECT novel_id FROM novel WHERE novel_id = %s",
                (novel_id,)
            )
            if not cursor.fetchone():
                raise ValueError("指定的小说不存在")

            # 查询章节数据
            query = """
                SELECT 
                    chapter_id,
                    chapter_no,
                    title,
                    content,
                    word_count,
                    created_at
                FROM chapter
                WHERE novel_id = %s
                ORDER BY chapter_no ASC
            """
            cursor.execute(query, (novel_id,))
            chapters = cursor.fetchall()

            # 格式化输出结果
            formatted_chapters = []
            for ch in chapters:
                formatted_chapters.append({
                    "chapter_id": ch["chapter_id"],
                    "chapter_no": ch["chapter_no"],
                    "title": ch["title"],
                    "content": ch["content"],
                    "word_count": ch["word_count"],
                    "created_at": ch["created_at"].isoformat() if ch["created_at"] else None
                })
            return formatted_chapters

    except Exception as e:
        raise RuntimeError(f"获取章节列表失败: {str(e)}")
    finally:
        connection.close()


def custom_serializer(obj):
    """自定义JSON序列化处理"""
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")


# ======== FastAPI 路由 ========
@app.post("/novels/", status_code=status.HTTP_201_CREATED)
async def create_novel(novel: NovelCreate):
    """添加新小说接口"""
    try:
        novel_data = {
            "title": novel.title,
            "author_id": novel.author_id,
            "category_id": novel.category_id,
            "description": novel.description,
            "cover": novel.cover,
            "status": novel.status,
            "word_count": novel.word_count,
            "is_vip": novel.is_vip
        }
        novel_id = add_novel(novel_data, novel.tags)
        return {"novel_id": novel_id}
    except Exception as e:
        raise HTTPException(
            status_code=400,
            detail=str(e)
        )

@app.post("/chapters/", status_code=201)
async def create_chapter(chapter: ChapterCreate):
    try:
        chapter_id = add_chapter(chapter.dict())
        return {"chapter_id": chapter_id}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.get("/categories/novels/", response_model=List[CategoryNovels])
async def get_category_novels():
    """获取分类小说列表接口"""
    try:
        result = generate_category_novels_json()
        return result
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"数据获取失败: {str(e)}"
        )

@app.get("/novels/{novel_id}/chapters", response_model=List[Dict])
async def get_novel_chapters(novel_id: int):
    try:
        return get_chapters_by_novel_id(novel_id)
    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))
    except RuntimeError as e:
        raise HTTPException(status_code=500, detail=str(e))


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step4: postman

```bash
 2. 获取分类小说列表（GET）
请求配置：

URL: GET http://localhost:8000/categories/novels/

Headers: 无特殊要求

成功响应 (HTTP 200) 示例片段：

[
    {
        "category_id": 1,
        "name": "玄幻",
        "description": "东方玄幻、异世大陆等幻想类作品",
        "novels": [
            {
                "novel_id": 1,
                "title": "斗破苍穹",
                "cover": "/cover/doupo.jpg",
                "author": {
                    "user_id": 2,
                    "username": "author1"
                },
                "description": "这里是属于斗气的世界，没有花俏艳丽的魔法...",
                "status": "finished",
                "word_count": 5000000,
                "is_vip": true,
                "created_at": "2025-03-29T15:27:59",
                "last_updated": null,
                "chapters": [
                    {
                        "chapter_id": 1,
                        "chapter_no": 1,
                        "title": "陨落的天才",
                        "word_count": 3245,
                        "created_at": "2025-03-29T15:28:03"
                    },
                    {
                        "chapter_id": 2,
                        "chapter_no": 2,
                        "title": "斗气阁",
                        "word_count": 2987,
                        "created_at": "2025-03-29T15:28:03"
                    }
                ],
                "tags": [
                    "废柴流",
                    "热血"
                ],
                "bookshelf_count": 1
            },
            {
                "novel_id": 3,
                "title": "测试小说",
                "cover": "/covers/test.jpg",
                "author": {
                    "user_id": 2,
                    "username": "author1"
                },
                "description": "这是一个测试小说",
                "status": "serial",
                "word_count": 5000,
                "is_vip": true,
                "created_at": "2025-03-29T15:44:41",
                "last_updated": null,
                "chapters": [],
                "tags": [
                    "热血",
                    "神医"
                ],
                "bookshelf_count": 0
            }
        ]
    },
    {
        "category_id": 2,
        "name": "都市",
        "description": "现代都市背景的言情、社会类作品",
        "novels": [
            {
                "novel_id": 2,
                "title": "都市神医",
                "cover": "/cover/shenyi.jpg",
                "author": {
                    "user_id": 2,
                    "username": "author1"
                },
                "description": "落魄青年获得神秘传承，从此医武无双...",
                "status": "serial",
                "word_count": 200000,
                "is_vip": false,
                "created_at": "2025-03-29T15:27:59",
                "last_updated": null,
                "chapters": [
                    {
                        "chapter_id": 3,
                        "chapter_no": 1,
                        "title": "医院奇遇",
                        "word_count": 4123,
                        "created_at": "2025-03-29T15:28:03"
                    },
                    {
                        "chapter_id": 4,
                        "chapter_no": 2,
                        "title": "神秘古玉",
                        "word_count": 3789,
                        "created_at": "2025-03-29T15:28:03"
                    }
                ],
                "tags": [
                    "神医"
                ],
                "bookshelf_count": 1
            }
        ]
    }
]

1. 添加小说（POST）
请求配置：

URL: POST http://localhost:8000/novels/

Headers:

Content-Type: application/json

Body (raw JSON):

http://localhost:8000/novels/
{
    "title": "无限恐怖",
    "author_id": 2,
    "category_id": 1,
    "description": "一个凡人修仙的传奇故事",
    "cover": "/covers/fanren.jpg",
    "status": "finished",
    "word_count": 3500000,
    "is_vip": true,
    "tags": [1, 3]
}
{
    "novel_id": 6
}

添加章节
请求配置：

Method: POST

 
Content-Type: application/json

http://localhost:8000/chapters/
{
    "novel_id": 1,
    "chapter_no": 4,
    "title": "萧家测试",
    "content": "萧炎望着测试石碑...",
    "word_count": 1500
}

{
    "chapter_id": 6
}

Postman 测试示例
请求配置：

Method: GET

URL: http://localhost:8000/novels/1/chapters

成功响应 (HTTP 200)：


[
    {
        "chapter_id": 1,
        "chapter_no": 1,
        "title": "陨落的天才",
        "content_preview": "斗之力，三段！......",
        "word_count": 3245,
        "created_at": "2025-03-29T17:58:03"
    },
    {
        "chapter_id": 2,
        "chapter_no": 2,
        "title": "斗气阁",
        "content_preview": "望着那消失在阴影中的少年......",
        "word_count": 2987,
        "created_at": "2025-03-29T17:58:03"
    },
    {
        "chapter_id": 5,
        "chapter_no": 3,
        "title": "新的章节",
        "content_preview": "这里是章节内容......",
        "word_count": 2500,
        "created_at": "2025-03-29T18:51:33"
    },
    {
        "chapter_id": 6,
        "chapter_no": 4,
        "title": "萧家测试",
        "content_preview": "萧炎望着测试石碑......",
        "word_count": 1500,
        "created_at": "2025-03-29T18:57:54"
    }
]


```

step5: 小说列表

```typescript
<template>
  <div class="container">
    <!-- 加载状态 -->
    <div v-if="loading" class="loading">加载中...</div>

    <!-- 错误提示 -->
    <div v-if="error" class="error">{{ error }}</div>

    <!-- 主内容 -->
    <div v-if="!loading && !error" class="main-layout">
      <!-- 左侧分类列表 -->
      <div class="category-list">
        <div
          v-for="category in categories"
          :key="category.category_id"
          class="category-item"
          :class="{ active: currentCategoryId === category.category_id }"
          @click="selectCategory(category.category_id)"
        >
          <h3>{{ category.name }}</h3>
          <p class="description">{{ category.description }}</p>
        </div>
      </div>

      <!-- 右侧小说列表 -->
      <div class="novel-list">
        <div
          v-for="novel in currentNovels"
          :key="novel.novel_id"
          class="novel-card"
        >
          <img :src="getCoverUrl(novel.cover)" alt="封面" class="cover">
          <div class="content">
            <h2>{{ novel.title }}</h2>
            <div class="meta">
              <span class="author">作者：{{ novel.author.username }}</span>
              <span class="status">状态：{{ novel.status === 'finished' ? '完结' : '连载' }}</span>
              <span class="word-count">字数：{{ (novel.word_count / 10000).toFixed(1) }}万字</span>
            </div>
            <p class="description">{{ novel.description }}</p>
            <div class="tags">
              <span v-for="tag in novel.tags" :key="tag" class="tag">{{ tag }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      categories: [],
      currentCategoryId: null,
      loading: true,
      error: null
    };
  },
  computed: {
    currentNovels() {
      const category = this.categories.find(c => c.category_id === this.currentCategoryId);
      return category ? category.novels : [];
    }
  },
  async created() {
    try {
      const response = await axios.get('http://localhost:8000/categories/novels/');
      this.categories = response.data;
      if (this.categories.length > 0) {
        this.currentCategoryId = this.categories[0].category_id;
      }
    } catch (error) {
      this.error = '数据加载失败，请稍后重试';
    } finally {
      this.loading = false;
    }
  },
  methods: {
    getCoverUrl(coverPath) {
      return coverPath
    },
    selectCategory(categoryId) {
      this.currentCategoryId = categoryId;
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.loading, .error {
  text-align: center;
  font-size: 18px;
  padding: 20px;
}

.main-layout {
  display: flex;
  gap: 30px;
}

.category-list {
  width: 250px;
  flex-shrink: 0;
}

.category-item {
  padding: 15px;
  margin-bottom: 15px;
  background: #f8f9fa;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.category-item:hover {
  background: #e9ecef;
}

.category-item.active {
  background: #007bff;
  color: white;
}

.category-item.active .description {
  color: #e0e0e0;
}

.description {
  font-size: 12px;
  color: #666;
  margin-top: 8px;
}

.novel-list {
  flex: 1;
}

.novel-card {
  display: flex;
  gap: 20px;
  padding: 20px;
  margin-bottom: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.cover {
  width: 120px;
  height: 160px;
  object-fit: cover;
  border-radius: 4px;
}

.content {
  flex: 1;
}

h2 {
  margin: 0 0 10px;
  color: #333;
}

.meta {
  margin-bottom: 10px;
  font-size: 14px;
  color: #666;
}

.meta span {
  margin-right: 15px;
}

.tags {
  margin-top: 10px;
}

.tag {
  display: inline-block;
  padding: 4px 8px;
  margin-right: 8px;
  background: #f0f2f5;
  border-radius: 4px;
  font-size: 12px;
  color: #666;
}
</style>
```


step6:新增小说

```typescript
<template>
  <div class="form-container">
    <form @submit.prevent="createWork">
      <!-- 作品名输入 -->
      <div class="form-item">
        <label>作品名(选填)</label>
        <input
          v-model="workName"
          type="text"
          placeholder="15字以内，请勿添加书名号等特殊符号"
          maxlength="15"
        >
        <div class="tip">当前已输入：{{ workName.length }}字</div>
      </div>

      <!-- 作品类型选择 -->
      <div class="form-item">
        <label>作品类型*</label>
        <select v-model.number="selectedGenre">
          <option value="1">玄幻</option>
          <option value="2">奇幻</option>
          <option value="3">武侠</option>
          <option value="4">都市</option>
          <option value="5">历史</option>
        </select>
        <div class="genre-description">
          {{ getGenreDescription(selectedGenre) }}
        </div>
      </div>

      <!-- 作品简介 -->
      <div class="form-item">
        <label>作品简介</label>
        <textarea
          v-model="introduction"
          placeholder="若未填，使用默认「来阅文旗下网站阅读我的更多作品吧！」"
          maxlength="500"
        ></textarea>
        <div class="tip">已输入 {{ introduction.length }}/500 字</div>
      </div>

      <!-- 提交按钮 -->
      <button type="submit" class="submit-btn">创建作品</button>
    </form>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 类型描述映射
const genreDescriptions = {
  1: '以架空世界为背景，具有明确的修炼升级体系设定，主要展现个体的玄异能力...',
  2: '西方幻想风格，包含魔法、龙族、精灵等元素的世界观...',
  3: '以中国传统武术为特色，展现江湖恩怨的门派斗争...',
  4: '现代都市背景下的现实题材或异能超能故事...',
  5: '基于历史事件或古代背景的架空演义故事...'
}

const workName = ref('')
const selectedGenre = ref(1)  // 默认选中玄幻
const introduction = ref('')

const getGenreDescription = (id) => {
  return genreDescriptions[id] || '请选择作品类型查看详细说明'
}

const createWork = async () => {
  // 验证类型选择
  if (![1, 2, 3, 4, 5].includes(selectedGenre.value)) {
    alert('请选择有效的作品类型')
    return
  }

  // 构造请求数据
  const formData = {
    title: workName.value || `未命名作品-${Date.now().toString().slice(-4)}`,
    author_id: 2,
    category_id: selectedGenre.value, // 已确保数字类型
    description: introduction.value || '来阅文旗下网站阅读我的更多作品吧！',
    cover: "https://randomuser.me/api/portraits/men/3.jpg",
    status: "finished",
    word_count: 3500000,
    is_vip: true,
    tags: [1, 3]
  }

  try {
    const response = await fetch('http://localhost:8000/novels/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    })

    if (response.status === 422) {
      const errorData = await response.json()
      alert(`数据验证失败: ${JSON.stringify(errorData.detail)}`)
      return
    }

    if (!response.ok) throw new Error(`HTTP 错误! 状态码: ${response.status}`)

    const data = await response.json()
    alert(`创建成功! 作品ID: ${data.novel_id}`)
  } catch (error) {
    console.error('创建失败:', error)
    alert('作品创建失败，请检查网络连接')
  }
}
</script>


<style scoped>
.form-container {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
  background: white;
  border-radius: 4px;
}

.form-item {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
  color: #333;
}

input, select, textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

textarea {
  height: 120px;
  resize: vertical;
}

.tip {
  color: #666;
  font-size: 12px;
  margin-top: 4px;
}

.genre-description {
  color: #888;
  font-size: 12px;
  margin-top: 8px;
  line-height: 1.4;
}

.submit-btn {
  background: #007bff;
  color: white;
  padding: 10px 25px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-btn:hover {
  background: #0056b3;
}
</style>
```


step7:新增章节

```typescript
<template>
  <!-- 模板部分保持不变 -->
  <div class="container">
    <!-- 左侧边栏 -->
    <div class="sidebar">
      <button class="new-chapter" @click="createNewChapter">新建章节</button>
      <div class="chapter-list">
        <div
          v-for="chapter in chapters"
          :key="chapter.id"
          class="chapter-item"
          :class="{ active: currentChapter?.id === chapter.id }"
          @click="selectChapter(chapter)"
        >
          <div class="chapter-title">第{{chapter.no}}章 {{ chapter.title }}</div>
          <div class="chapter-meta">
            <span class="time">{{ formatTime(chapter.createTime) }}</span>
            <span class="word-count">{{ chapter.wordCount }}字</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 右侧内容区 -->
    <div class="content-area">
      <button class="publish-btn" @click="handlePublish">发布</button>
	  <!-- 修改为文本显示 -->
      <div class="novel-title">{{ novelTitle }}</div>


	  <!-- 新增章节标题输入框 -->
      <input
        v-if="currentChapter"
        v-model="currentChapter.title"
        type="text"
        class="chapter-title-input"
        placeholder="章节标题"
      >

      <textarea
        v-if="currentChapter"
        v-model="currentChapter.content"
        class="content-editor"
        placeholder="请输入章节内容"
        @input="updateWordCount"
      ></textarea>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'

// 小说基本信息
const novelTitle = ref('重生之我成了傻社')

// 章节相关状态
const chapters = reactive([])
const currentChapter = ref(null)
let chapterId = 1

// 从后端获取章节数据
const fetchChapters = async () => {
  try {
    const response = await fetch('http://localhost:8000/novels/1/chapters')
    if (!response.ok) throw new Error('获取章节失败')
    const data = await response.json()

    // 转换数据结构
    const mappedChapters = data.map(chapter => ({
      id: chapter.chapter_id,
      no: chapter.chapter_no,
      title: chapter.title,
      content: chapter.content,
      createTime: new Date(chapter.created_at),
      wordCount: chapter.word_count
    }))

    // 更新章节列表
    chapters.splice(0, chapters.length, ...mappedChapters)

    // 默认选中第一个章节
    if (mappedChapters.length > 0) {
      currentChapter.value = mappedChapters[0]
    }
  } catch (error) {
    console.error('获取章节数据失败:', error)
    alert('章节加载失败，请稍后重试')
  }
}

// 组件挂载时获取数据
onMounted(fetchChapters)

// 创建新章节（保持原有逻辑）
const createNewChapter = () => {
  const newChapter = {
    id: chapterId++,
    title: `第${chapters.length + 1}章`,
    content: '',
    createTime: new Date(),
    wordCount: 0
  }
  chapters.push(newChapter)
  selectChapter(newChapter)
}

// 其他原有方法保持不变
const selectChapter = (chapter) => {
  console.log("selectChapter:",chapter)
  currentChapter.value = chapter
}

const updateWordCount = () => {
  if (currentChapter.value) {
    currentChapter.value.wordCount = currentChapter.value.content.replace(/\s/g, '').length
  }
}

const formatTime = (date) => {
  return new Date(date).toLocaleString('zh-CN', {
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  }).replace(/\//g, '-')
}

// 发布功能实现
const handlePublish = async () => {



  const requestBody = {
    novel_id: 1,
    chapter_no: 11,  // 建议使用实际章节编号
    title: currentChapter.value.title,  // 修改为当前章节标题
    content: currentChapter.value.content,
    word_count: currentChapter.value.wordCount
  }

  console.log("requestBody:",requestBody)

  try {
    const response = await fetch('http://localhost:8000/chapters/', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(requestBody)
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const result = await response.json()
    console.log('发布成功:', result)
    alert(`发布成功！章节ID: ${result.chapter_id}`)

    // 更新本地章节ID（如果需要）
   // currentChapter.value.id = result.chapter_id
    refreshData()
  } catch (error) {
    console.error('发布失败:', error)
    alert('发布失败，请检查网络连接后重试')
  }
}

// 在需要刷新的地方调用数据获取方法
const refreshData = async () => {
  try {
    // 清空当前数据
    chapters.splice(0, chapters.length)
    currentChapter.value = null

    // 重新获取数据
    await fetchChapters()
  } catch (error) {
    console.error('刷新失败:', error)
  }
}

</script>

<style scoped>
/* 新增章节标题输入框样式 */
.chapter-title-input {
  display: block;
  width: 100%;
  margin: 15px 0;
  padding: 10px;
  font-size: 1.1em;
  border: 1px solid #ddd;
  border-radius: 4px;
}
/* 样式部分保持不变 */
.container {
  display: flex;
  height: 100vh;
  background: white;
}

.sidebar {
  width: 300px;
  border-right: 1px solid #eee;
  padding: 20px;
  background: #f8f9fa;
}

.content-area {
  flex: 1;
  padding: 20px;
}

.chapter-list {
  margin-top: 20px;
}

.chapter-item {
  padding: 12px;
  margin: 8px 0;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  background: white;
  border: 1px solid #eee;
}

.chapter-item:hover {
  background: #e9ecef;
}

.chapter-item.active {
  border-color: #007bff;
  background: #e7f1ff;
}

.chapter-title {
  font-weight: 500;
  color: #333;
}

.chapter-meta {
  margin-top: 4px;
  font-size: 0.8em;
  color: #666;
  display: flex;
  justify-content: space-between;
}

.publish-btn {
  float: right;
  padding: 8px 20px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.novel-title {
  display: block;
  margin: 15px 0;
  padding: 8px 0;
  font-size: 1.5em;
  font-weight: bold;
  color: #333;
  border-bottom: 2px solid #007bff;
}

.content-editor {
  width: 100%;
  height: calc(100vh - 220px);
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 4px;
  resize: none;
  line-height: 1.6;
}
</style>
```

 step8:

```bash
C:\Users\wangrusheng\PycharmProjects\untitled3\src\App.vue   

 <div class="sidebar">
      <h2>路由列表</h2>
      <nav>
        <router-link to="/home" active-class="active-link">小说列表</router-link>
        <router-link to="/user" active-class="active-link">新增小说</router-link>
        <router-link to="/chapters" active-class="active-link">小说章节</router-link>
      </nav>
    </div>


C:\Users\wangrusheng\PycharmProjects\untitled3\src\router\index.js

const routes = [
  { path: '/', redirect: '/home' },
  { path: '/home', name: 'Home', component: Home },
  { path: '/user', name: 'User', component: User },
  { path: '/chapters', name: 'chapters', component: Chapters }

]
```

end