说明：
fastapi+angular在线音乐播放
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3d471131a5014e489e8167c7b9a3d8e7.png#pic_center)

step1:sql

```sql

-- 用户表
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    avatar_url VARCHAR(255),
    registration_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_login DATETIME,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 歌手表
CREATE TABLE artists (
    artist_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    alias VARCHAR(100),  -- 别名/艺名
    nationality VARCHAR(50),
    birth_date DATE,
    description TEXT,
    avatar_url VARCHAR(255),
    INDEX idx_artist_name (name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 专辑表
CREATE TABLE albums (
    album_id INT PRIMARY KEY AUTO_INCREMENT,
    artist_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    release_date DATE,
    cover_url VARCHAR(255),
    description TEXT,
    FOREIGN KEY (artist_id) REFERENCES artists(artist_id),
    INDEX idx_album_title (title)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 歌曲表（核心表）
CREATE TABLE songs (
    song_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    artist_id INT NOT NULL,
    album_id INT,
    duration_seconds INT NOT NULL,  -- 歌曲时长（秒）
    is_vip TINYINT(1) NOT NULL DEFAULT 0,
    play_count INT DEFAULT 0,
    release_date DATE,
    lyrics TEXT,
    file_path VARCHAR(255) NOT NULL,  -- 音乐文件存储路径
    FOREIGN KEY (artist_id) REFERENCES artists(artist_id),
    FOREIGN KEY (album_id) REFERENCES albums(album_id),
    INDEX idx_song_title (title),
    INDEX idx_release_date (release_date)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 分类表（流派/风格）
CREATE TABLE categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 歌曲分类关联表
CREATE TABLE song_categories (
    song_id INT NOT NULL,
    category_id INT NOT NULL,
    PRIMARY KEY (song_id, category_id),
    FOREIGN KEY (song_id) REFERENCES songs(song_id),
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 播放列表表
CREATE TABLE playlists (
    playlist_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    cover_url VARCHAR(255),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX idx_playlist_title (title)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 播放列表歌曲关联表
CREATE TABLE playlist_songs (
    playlist_id INT NOT NULL,
    song_id INT NOT NULL,
    position INT NOT NULL,  -- 歌曲在播放列表中的顺序
    added_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (playlist_id, song_id),
    FOREIGN KEY (playlist_id) REFERENCES playlists(playlist_id),
    FOREIGN KEY (song_id) REFERENCES songs(song_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 收藏表
CREATE TABLE favorites (
    user_id INT NOT NULL,
    song_id INT NOT NULL,
    added_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, song_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (song_id) REFERENCES songs(song_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 评论表
CREATE TABLE comments (
    comment_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    song_id INT NOT NULL,
    content TEXT NOT NULL,
    parent_comment_id INT,  -- 支持回复功能
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (song_id) REFERENCES songs(song_id),
    FOREIGN KEY (parent_comment_id) REFERENCES comments(comment_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;


-- 插入用户数据
INSERT INTO users (user_id, username, password_hash, email, avatar_url, registration_date, last_login) VALUES
(1, 'zhangsan', '$2a$10$N9qo8uLOickgx2ZMRZoMye3Z7g7G6W.6Uq3Mv5Q6Z7bB', 'zhangsan@example.com', '/avatars/1.jpg', '2023-01-01 10:00:00', '2023-10-01 15:00:00'),
(2, 'lisi', '$2a$10$qSNpg8LOickgx2ZMRZoMye3Z7g7G6W.6Uq3Mv5Q6Z7bB', 'lisi@example.com', '/avatars/2.jpg', '2023-02-01 11:00:00', '2023-10-02 16:00:00'),
(3, 'wangwu', '$2a$10$rT9hg8uLOickgx2ZMRZoMye3Z7g7G6W.6Uq3Mv5Q6Z7bB', 'wangwu@example.com', '/avatars/3.jpg', '2023-03-01 12:00:00', '2023-10-03 17:00:00');

-- 插入歌手数据
INSERT INTO artists (artist_id, name, alias, nationality, birth_date, description, avatar_url) VALUES
(1, '周杰伦', '周董', '中国', '1979-01-18', '著名歌手、音乐制作人，亚洲流行天王', 'https://randomuser.me/api/portraits/men/1.jpg'),
(2, 'Taylor Swift', NULL, '美国', '1989-12-13', '10次格莱美奖得主，流行音乐天后', 'https://randomuser.me/api/portraits/men/2.jpg'),
(3, 'Beyond', NULL, '中国', '1983-01-01', '香港殿堂级摇滚乐队', 'https://randomuser.me/api/portraits/men/3.jpg');

-- 插入专辑数据
INSERT INTO albums (album_id, artist_id, title, release_date, cover_url, description) VALUES
(1, 1, '范特西', '2001-09-14', 'https://randomuser.me/api/portraits/men/4.jpg', '周杰伦最具突破性的音乐作品'),
(2, 2, '1989', '2014-10-27', 'https://randomuser.me/api/portraits/men/5.jpg', '泰勒·斯威夫特首张纯流行专辑'),
(3, 3, '乐与怒', '1993-05-01', 'https://randomuser.me/api/portraits/men/6.jpg', 'Beyond乐队巅峰之作');

-- 插入歌曲数据
INSERT INTO songs (song_id, title, artist_id, album_id, duration_seconds,is_vip, play_count, release_date, lyrics, file_path) VALUES
(1, '双截棍', 1, 1, 200, 1,15000, '2001-09-14', '岩烧店的烟味弥漫...', 'https://downsc.chinaz.net/Files/DownLoad/sound1/201906/11582.mp3'),
(2, '安静', 1, 1, 240, 1,12000, '2001-09-14', '只剩下钢琴陪我弹了一天...', 'https://downsc.chinaz.net/files/download/sound1/201206/1638.mp3'),
(3, 'Shake It Off', 2, 2, 210,1, 9800000, '2014-10-27', 'Cause the players gonna play...', 'https://96.f.1ting.com/local_to_cube_202004121813/96kmp3/2021/11/18/18k_xsm/01.mp3'),
(4, 'Blank Space', 2, 2, 230, 1,12000000, '2014-10-27', 'Nice to meet you...', 'https://downsc.chinaz.net/Files/DownLoad/sound1/201906/11582.mp3'),
(5, '海阔天空', 3, 3, 300, 1,25000000, '1993-05-01', '今天我，寒夜里看雪飘过...', 'https://96.f.1ting.com/local_to_cube_202004121813/96kmp3/2021/11/18/18k_xsm/01.mp3');





-- 插入分类数据
INSERT INTO categories (category_id, name, description) VALUES
(1, '流行', '主流流行音乐'),
(2, '摇滚', '摇滚及衍生风格'),
(3, 'R&B', '节奏布鲁斯音乐'),
(4, '中国风', '融合中国传统元素的音乐');

-- 插入歌曲分类关联
INSERT INTO song_categories (song_id, category_id) VALUES
(1, 1),(1, 3),(1, 4),  -- 双截棍（流行/R&B/中国风）
(2, 1),(2, 4),        -- 安静（流行/中国风）
(3, 1),               -- Shake It Off（流行）
(4, 1),               -- Blank Space（流行）
(5, 2);               -- 海阔天空（摇滚）

-- 插入播放列表
INSERT INTO playlists (playlist_id, user_id, title, description, cover_url, created_at) VALUES
(1, 1, '我的最爱', '日常最爱听的歌曲合集', '/covers/fav1.jpg', '2023-10-01 10:00:00'),
(2, 2, '健身歌单', '运动时听的活力音乐', '/covers/gym.jpg', '2023-10-02 11:00:00'),
(3, 3, '经典收藏', '永不过时的经典作品', '/covers/classic.jpg', '2023-10-03 12:00:00');

-- 插入播放列表歌曲
INSERT INTO playlist_songs (playlist_id, song_id, position, added_time) VALUES
(1, 1, 1, '2023-10-01 10:05:00'),
(1, 5, 2, '2023-10-01 10:10:00'),
(2, 3, 1, '2023-10-02 11:15:00'),
(2, 4, 2, '2023-10-02 11:20:00'),
(3, 5, 1, '2023-10-03 12:25:00'),
(3, 1, 2, '2023-10-03 12:30:00');

-- 插入收藏数据
INSERT INTO favorites (user_id, song_id, added_time) VALUES
(1, 3, '2023-10-01 09:05:00'),
(1, 4, '2023-10-01 09:10:00'),
(2, 5, '2023-10-02 10:15:00'),
(3, 1, '2023-10-03 11:20:00');

-- 插入评论数据
INSERT INTO comments (comment_id, user_id, song_id, content, parent_comment_id, created_at) VALUES
(1, 1, 5, '每次听都热泪盈眶！', NULL, '2023-10-01 10:30:00'),
(2, 3, 5, '永远的Beyond！', 1, '2023-10-01 11:35:00'),
(3, 2, 3, '跑步时听这个超带感！', NULL, '2023-10-02 12:40:00');
```

step2:python test

```python
from typing import List, Dict, Optional
import json
from collections import defaultdict
import pymysql.cursors
from datetime import datetime, date
from decimal import Decimal

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


def generate_category_songs_json() -> List[Dict]:
    """生成分类及包含歌曲的JSON结构"""
    # 获取所有分类
    categories = execute_query("SELECT * FROM categories ORDER BY category_id")

    # 获取歌曲及其关联信息
    songs = execute_query("""
        SELECT s.*, 
               sc.category_id,
               a.name AS artist_name,
               al.title AS album_title
        FROM songs s
        JOIN song_categories sc ON s.song_id = sc.song_id
        JOIN artists a ON s.artist_id = a.artist_id
        LEFT JOIN albums al ON s.album_id = al.album_id
    """)

    # 按分类分组歌曲
    category_map = defaultdict(list)
    for song in songs:
        # 处理日期格式
        if song.get('release_date'):
            if isinstance(song['release_date'], date):
                song['release_date'] = song['release_date'].isoformat()

        # 构建歌曲结构
        song_entry = {
            'song_id': song['song_id'],
            'title': song['title'],
            'artist': {
                'id': song['artist_id'],
                'name': song['artist_name']
            },
            'album': {
                'id': song['album_id'],
                'title': song['album_title']
            } if song['album_id'] else None,
            'duration': f"{song['duration_seconds'] // 60}:{song['duration_seconds'] % 60:02}",
            'is_vip': bool(song['is_vip']),
            'play_count': song['play_count'],
            'file_path': song['file_path'],
            'release_date': song.get('release_date')
        }
        category_map[song['category_id']].append(song_entry)

    # 构建最终结构
    return [{
        "category_id": cat['category_id'],
        "name": cat['name'],
        "description": cat['description'],
        "songs": category_map.get(cat['category_id'], [])
    } for cat in categories]


def add_song(song_data: Dict, category_ids: List[int]) -> int:
    """
    添加新歌曲并关联到指定分类
    :param song_data: 包含歌曲数据的字典，必须字段：
        title, artist_id, duration_seconds, file_path
    :param category_ids: 要关联的分类ID列表
    :return: 新创建歌曲的ID
    """
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证artist存在
            cursor.execute("SELECT artist_id FROM artists WHERE artist_id = %s",
                           (song_data['artist_id'],))
            if not cursor.fetchone():
                raise ValueError("歌手不存在")

            # 验证album存在（如果提供）
            if song_data.get('album_id'):
                cursor.execute("SELECT album_id FROM albums WHERE album_id = %s",
                               (song_data['album_id'],))
                if not cursor.fetchone():
                    raise ValueError("专辑不存在")

            # 插入歌曲数据
            sql = """
                INSERT INTO songs (
                    title, artist_id, album_id, duration_seconds,
                    is_vip, play_count, release_date, lyrics, file_path
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            params = (
                song_data['title'],
                song_data['artist_id'],
                song_data.get('album_id'),
                song_data['duration_seconds'],
                song_data.get('is_vip', False),
                song_data.get('play_count', 0),
                song_data.get('release_date'),
                song_data.get('lyrics', ''),
                song_data['file_path']
            )
            cursor.execute(sql, params)
            song_id = cursor.lastrowid

            # 插入分类关联
            for cat_id in category_ids:
                cursor.execute("SELECT category_id FROM categories WHERE category_id = %s",
                               (cat_id,))
                if not cursor.fetchone():
                    raise ValueError(f"分类ID {cat_id} 不存在")

                cursor.execute(
                    "INSERT INTO song_categories (song_id, category_id) VALUES (%s, %s)",
                    (song_id, cat_id)
                )

            connection.commit()
            return song_id
    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加歌曲失败: {str(e)}")
    finally:
        connection.close()


def custom_serializer(obj):
    """自定义JSON序列化处理"""
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError(f"Type {type(obj)} not serializable")


if __name__ == "__main__":
    try:


        # 示例：添加新歌曲
        new_song = {
            'title': '测试数据',
            'artist_id': 2,
            'duration_seconds': 235,
            'file_path': '/music/nocturne.mp3',
            'album_id': 3,
            'is_vip': False,
            'release_date': '2024-05-01',
            'lyrics': '夜曲的前奏响起...'
        }
        song_id = add_song(new_song, [1, 3])
        print(f"成功添加歌曲，ID：{song_id}")
        # 示例：生成分类歌曲JSON
        result = generate_category_songs_json()
        print(json.dumps(result, indent=2, ensure_ascii=False, default=custom_serializer))
    except Exception as e:
        print(f"操作失败: {str(e)}")
```

step3:fastapi 

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


# Pydantic模型定义
class SongData(BaseModel):
    title: str
    artist_id: int
    duration_seconds: int
    file_path: str
    album_id: Optional[int] = None
    is_vip: Optional[bool] = False
    play_count: Optional[int] = 0
    release_date: Optional[date] = None
    lyrics: Optional[str] = ''


class AddSongRequest(BaseModel):
    song_data: SongData
    category_ids: List[int]


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


def generate_category_songs_json() -> List[Dict]:
    """生成分类及包含歌曲的JSON结构"""
    # 获取所有分类
    categories = execute_query("SELECT * FROM categories ORDER BY category_id")

    # 获取歌曲及其关联信息
    songs = execute_query("""
        SELECT s.*, 
               sc.category_id,
               a.name AS artist_name,
               al.title AS album_title
        FROM songs s
        JOIN song_categories sc ON s.song_id = sc.song_id
        JOIN artists a ON s.artist_id = a.artist_id
        LEFT JOIN albums al ON s.album_id = al.album_id
    """)

    # 按分类分组歌曲
    category_map = defaultdict(list)
    for song in songs:
        # 处理日期格式
        if song.get('release_date'):
            if isinstance(song['release_date'], date):
                song['release_date'] = song['release_date'].isoformat()

        # 构建歌曲结构
        song_entry = {
            'song_id': song['song_id'],
            'title': song['title'],
            'artist': {
                'id': song['artist_id'],
                'name': song['artist_name']
            },
            'album': {
                'id': song['album_id'],
                'title': song['album_title']
            } if song['album_id'] else None,
            'duration': f"{song['duration_seconds'] // 60}:{song['duration_seconds'] % 60:02}",
            'is_vip': bool(song['is_vip']),
            'play_count': song['play_count'],
            'file_path': song['file_path'],
            'release_date': song.get('release_date')
        }
        category_map[song['category_id']].append(song_entry)

    # 构建最终结构
    return [{
        "category_id": cat['category_id'],
        "name": cat['name'],
        "description": cat['description'],
        "songs": category_map.get(cat['category_id'], [])
    } for cat in categories]


def add_song(song_data: Dict, category_ids: List[int]) -> int:
    """
    添加新歌曲并关联到指定分类
    :param song_data: 包含歌曲数据的字典，必须字段：
        title, artist_id, duration_seconds, file_path
    :param category_ids: 要关联的分类ID列表
    :return: 新创建歌曲的ID
    """
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证artist存在
            cursor.execute("SELECT artist_id FROM artists WHERE artist_id = %s",
                           (song_data['artist_id'],))
            if not cursor.fetchone():
                raise ValueError("歌手不存在")

            # 验证album存在（如果提供）
            if song_data.get('album_id'):
                cursor.execute("SELECT album_id FROM albums WHERE album_id = %s",
                               (song_data['album_id'],))
                if not cursor.fetchone():
                    raise ValueError("专辑不存在")

            # 插入歌曲数据
            sql = """
                INSERT INTO songs (
                    title, artist_id, album_id, duration_seconds,
                    is_vip, play_count, release_date, lyrics, file_path
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            params = (
                song_data['title'],
                song_data['artist_id'],
                song_data.get('album_id'),
                song_data['duration_seconds'],
                song_data.get('is_vip', False),
                song_data.get('play_count', 0),
                song_data.get('release_date'),
                song_data.get('lyrics', ''),
                song_data['file_path']
            )
            cursor.execute(sql, params)
            song_id = cursor.lastrowid

            # 插入分类关联
            for cat_id in category_ids:
                cursor.execute("SELECT category_id FROM categories WHERE category_id = %s",
                               (cat_id,))
                if not cursor.fetchone():
                    raise ValueError(f"分类ID {cat_id} 不存在")

                cursor.execute(
                    "INSERT INTO song_categories (song_id, category_id) VALUES (%s, %s)",
                    (song_id, cat_id)
                )

            connection.commit()
            return song_id
    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加歌曲失败: {str(e)}")
    finally:
        connection.close()


def custom_serializer(obj):
    """自定义JSON序列化处理"""
    if isinstance(obj, (datetime, date)):
        return obj.isoformat()
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError(f"Type {type(obj)} not serializable")



# 接口路由
@app.get("/category-songs/", response_model=List[Dict])
async def get_category_songs():
    try:
        return generate_category_songs_json()
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


@app.post("/songs/")
async def create_song(request: AddSongRequest):
    try:
        # 转换Pydantic模型为字典
        song_dict = request.song_data.dict()
        # 处理可能的日期类型
        if song_dict.get('release_date'):
            song_dict['release_date'] = song_dict['release_date'].isoformat()

        song_id = add_song(
            song_data=song_dict,
            category_ids=request.category_ids
        )
        return {"song_id": song_id}
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except RuntimeError as e:
        raise HTTPException(status_code=500, detail=str(e))


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step4:postman

```bash
 GET http://localhost:8000/category-songs/
Headers:
Content-Type: application/json

响应示例：
 [
    {
        "category_id": 1,
        "name": "流行",
        "description": "主流流行音乐",
        "songs": [
            {
                "song_id": 1,
                "title": "双截棍",
                "artist": {
                    "id": 1,
                    "name": "周杰伦"
                },
                "album": {
                    "id": 1,
                    "title": "范特西"
                },
                "duration": "3:20",
                "is_vip": true,
                "play_count": 15000,
                "file_path": "/music/fantasy/nunchucks.mp3",
                "release_date": "2001-09-14"
            },
            {
                "song_id": 2,
                "title": "安静",
                "artist": {
                    "id": 1,
                    "name": "周杰伦"
                },
                "album": {
                    "id": 1,
                    "title": "范特西"
                },
                "duration": "4:00",
                "is_vip": true,
                "play_count": 12000,
                "file_path": "/music/fantasy/quiet.mp3",
                "release_date": "2001-09-14"
            },
            {
                "song_id": 3,
                "title": "Shake It Off",
                "artist": {
                    "id": 2,
                    "name": "Taylor Swift"
                },
                "album": {
                    "id": 2,
                    "title": "1989"
                },
                "duration": "3:30",
                "is_vip": true,
                "play_count": 9800000,
                "file_path": "/music/1989/shake.mp3",
                "release_date": "2014-10-27"
            },
            {
                "song_id": 4,
                "title": "Blank Space",
                "artist": {
                    "id": 2,
                    "name": "Taylor Swift"
                },
                "album": {
                    "id": 2,
                    "title": "1989"
                },
                "duration": "3:50",
                "is_vip": true,
                "play_count": 12000000,
                "file_path": "/music/1989/blank.mp3",
                "release_date": "2014-10-27"
            },
            {
                "song_id": 7,
                "title": "可达鸭",
                "artist": {
                    "id": 2,
                    "name": "Taylor Swift"
                },
                "album": {
                    "id": 3,
                    "title": "乐与怒"
                },
                "duration": "3:55",
                "is_vip": false,
                "play_count": 0,
                "file_path": "/music/nocturne.mp3",
                "release_date": "2024-05-01"
            }
        ]
    } 
]


POST http://localhost:8000/songs/
Headers:
Content-Type: application/json

Body:
{
    "song_data": {
        "title": "可达鸭",
        "artist_id": 2,
        "duration_seconds": 235,
        "file_path": "/music/nocturne.mp3",
        "album_id": 3,
        "is_vip": false,
        "release_date": "2024-05-01",
        "lyrics": "夜曲的前奏响起..."
    },
    "category_ids": [1, 3]
}

 {
    "song_id": 8
}
```

step5: hello.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';
import {NgForOf, NgIf} from '@angular/common';
import {MusicListComponent} from '../music-list/music-list.component';
import {MusicPlayerComponent} from '../music-player/music-player.component';

export interface Category {
  category_id: number;
  name: string;
  songs: Song[];
}

interface Song {
  song_id: number;
  title: string;
  file_path: string;
  duration: string;
  is_vip: boolean;
  artist: {
    name: string;
  };
}

@Component({
  selector: 'app-hello-world',
  templateUrl: './hello-world.component.html',
  imports: [
    NgIf,
    NgForOf,
    MusicListComponent,
    MusicPlayerComponent
  ],
  styleUrl: './hello-world.component.css'
})
export class HelloWorldComponent implements OnInit {
  categories: Category[] = [];
  selectedCategoryId: number | null = null;
  loading = true;
  error: string | null = null;
    // 新增当前播放歌曲属性
  currentPlayingSong: { url: string, title: string, artist: string } | null = null;

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.fetchData();
  }

  get selectedCategory(): Category | undefined {
    return this.categories.find(c => c.category_id === this.selectedCategoryId);
  }

  private fetchData() {
    this.http.get<Category[]>('http://localhost:8000/category-songs/')
      .pipe(
        catchError(err => {
          this.error = '数据加载失败，请稍后重试';
          console.error('请求失败:', err);
          return throwError(err);
        })
      )
      .subscribe({
        next: data => {
          this.categories = data;
          if (this.categories.length > 0) {
            this.selectedCategoryId = this.categories[0].category_id;
          }
        },
        error: () => this.loading = false,
        complete: () => this.loading = false
      });
  }

  selectCategory(categoryId: number) {
    this.selectedCategoryId = categoryId;
  }

  // 修改播放处理方法
  handlePlay(song: Song) {
    console.log('开始播放:', song.title);
    // 根据实际数据结构获取完整URL
    this.currentPlayingSong = {
      url: song.file_path, // 根据实际数据结构调整
      title: song.title,
      artist: song.artist.name
    };
    // 强制触发变更检测（如果必要）
    this.currentPlayingSong = {...this.currentPlayingSong};
  }

  handleFavorite(song: Song) {
    console.log('收藏歌曲:', song.title);
    // 实际收藏逻辑
  }
}

<div class="container">
  <!-- 加载状态 -->
  <div *ngIf="loading" class="loading">加载中...</div>

  <!-- 错误提示 -->
  <div *ngIf="error" class="error">{{ error }}</div>

  <div *ngIf="!loading && !error" class="music-container">
    <!-- 左侧分类列表 -->
    <div class="category-list">
      <div
        *ngFor="let category of categories"
        class="category-item"
        [class.active]="selectedCategoryId === category.category_id"
        (click)="selectCategory(category.category_id)"
      >
        {{ category.name }}
        <span class="song-count">{{ category.songs.length }}首</span>
      </div>
    </div>

    <!-- 右侧歌曲列表组件 -->
    <div class="right-section">
      <app-music-list
        [selectedCategory]="selectedCategory"
        (play)="handlePlay($event)"
        (favorite)="handleFavorite($event)">
      </app-music-list>
       <!-- 新增currentSong绑定 -->
      <app-music-player class="player-container" [currentSong]="currentPlayingSong"></app-music-player>
    </div>
  </div>
</div>

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.music-container {
  display: flex;
  gap: 20px;
  align-items: flex-start;
}

/* 左侧分类列表样式 */
.category-list {
  width: 250px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  padding: 15px;
}
/* 新增样式 */
.right-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}
.category-item {
  padding: 12px 15px;
  cursor: pointer;
  border-radius: 6px;
  margin-bottom: 6px;
  transition: all 0.3s ease;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.95em;
}

.category-item:hover {
  background-color: #f8f9fa;
}

.category-item.active {
  background-color: #007bff;
  color: white;
  font-weight: 500;
}

.song-count {
  font-size: 0.8em;
  opacity: 0.8;
}

.loading, .error {
  padding: 20px;
  text-align: center;
  font-size: 1.2em;
}

.error {
  color: #dc3545;
}

```

step6:list.ts

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { Category } from '../hello-world/hello-world.component';
import {NgForOf, NgIf} from '@angular/common';

@Component({
  selector: 'app-music-list',
  templateUrl: './music-list.component.html',
  imports: [
    NgIf,
    NgForOf
  ],
  styleUrl: './music-list.component.css'
})
export class MusicListComponent {
  @Input() selectedCategory?: Category;
  @Output() play = new EventEmitter<any>();
  @Output() favorite = new EventEmitter<any>();

  hoverIndex = -1;
}

<div class="music-list">
  <h2>{{ selectedCategory?.name }}歌曲列表</h2>

  <div class="header">
    <span class="index">序号</span>
    <span class="song">歌曲</span>
    <span class="artist">歌手</span>
    <span class="duration">时长</span>
  </div>

  <div class="song-container">
    <div
      *ngFor="let song of selectedCategory?.songs; let index = index"
      class="song-item"
      [class.even]="index % 2 === 0"
      [class.odd]="index % 2 !== 0"
      (mouseenter)="hoverIndex = index"
      (mouseleave)="hoverIndex = -1"
    >
      <span class="index">{{ index + 1 }}</span>
      <span class="song">
        {{ song.title }}
        <span *ngIf="song.is_vip" class="vip-tag">VIP</span>
      </span>
      <span class="artist">{{ song.artist.name }}</span>

      <span *ngIf="hoverIndex !== index" class="duration">{{ song.duration }}</span>

      <div *ngIf="hoverIndex === index" class="actions">
        <button class="play" (click)="play.emit(song)">▶ 播放</button>
        <button class="favorite" (click)="favorite.emit(song)">❤ 收藏</button>
      </div>
    </div>
  </div>
</div>
.music-list {
  width: 700px;
  flex: 1;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  padding: 20px;
}

h2 {
  font-size: 1.3rem;
  margin-bottom: 1.2rem;
  color: #333;
  padding-bottom: 10px;
  border-bottom: 1px solid #eee;
}

.header {
  display: flex;
  padding: 12px 15px;
  font-weight: 600;
  color: #666;
  background-color: #f8f9fa;
  border-radius: 6px;
  margin-bottom: 8px;
}

.song-container {
  border: 1px solid #eee;
  border-radius: 6px;
  overflow: hidden;
}

.song-item {
  display: flex;
  align-items: center;
  padding: 12px 15px;
  transition: all 0.2s ease;
  position: relative;
}

.even { background-color: #fcfcfc; }
.odd { background-color: #f8f9fa; }

.song-item:hover {
  background-color: #e9f5ff !important;
  transform: translateX(3px);
}

.index { width: 10%; color: #666; font-size: 0.9em; }
.song {
  width: 40%;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 8px;
}
.artist { width: 30%; color: #666; }
.duration { width: 20%; color: #888; }

.actions {
  position: absolute;
  right: 15px;
  display: flex;
  gap: 8px;
}

button {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 0.9em;
  display: flex;
  align-items: center;
  gap: 5px;
}

.play {
  background-color: #4CAF50;
  color: white;
}

.favorite {
  background-color: #ff4081;
  color: white;
}

button:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

.vip-tag {
  font-size: 0.7em;
  background: linear-gradient(45deg, #ff9800, #ff5722);
  color: white;
  padding: 2px 8px;
  border-radius: 4px;
  font-weight: bold;
}

@media (max-width: 768px) {
  .music-container {
    flex-direction: column;
  }

  .category-list {
    width: 100%;
  }

  .header, .song-item {
    font-size: 0.9em;
    padding: 10px 12px;
  }

  button {
    padding: 5px 8px;
  }
}

```

step7:plaayer.ts

```typescript
// music-player.component.ts
import { Component, ViewChild, ElementRef, AfterViewInit, Input } from '@angular/core';

@Component({
  selector: 'app-music-player',
  templateUrl: './music-player.component.html',
  styleUrls: ['./music-player.component.css']
})
export class MusicPlayerComponent implements AfterViewInit {
  @ViewChild('audioElement') audioElement!: ElementRef<HTMLAudioElement>;

     // 新增输入属性
  @Input() set currentSong(song: { url: string, title: string, artist: string } | null) {
    if (song) {
      this.playImmediately(song);
    }
  }


  playlist = [
    {
      url: 'https://downsc.chinaz.net/Files/DownLoad/sound1/201906/11582.mp3',
      title: '热爱105℃的你',
      artist: '阿肆'
    },
    {
      url: 'https://downsc.chinaz.net/files/download/sound1/201206/1638.mp3',
      title: '欢乐颂',
      artist: '贝多芬'
    },
    {
      url: 'https://96.f.1ting.com/local_to_cube_202004121813/96kmp3/2021/11/18/18k_xsm/01.mp3',
      title: '未知歌曲',
      artist: '未知艺术家'
    }
  ];

  currentTrackIndex = 0;
  isPlaying = false;
  currentTime = 0;
  totalTime = 0;
  volume = 1;

  get currentTrack() {
    return this.playlist[this.currentTrackIndex];
  }

  get progress() {
    return (this.currentTime / this.totalTime) * 100 || 0;
  }

  get volumePercent() {
    return Math.round(this.volume * 100);
  }

  ngAfterViewInit() {
    this.loadTrack();
    this.setupAudioListeners();
  }

  private setupAudioListeners() {
    const audio = this.audioElement.nativeElement;
    audio.addEventListener('timeupdate', () => this.updateTime());
    audio.addEventListener('loadedmetadata', () => this.updateDuration());
  }


    // 修改加载曲目方法
  private loadTrack() {
    const audio = this.audioElement.nativeElement;
    audio.src = this.currentTrack.url;
    audio.load();
    audio.volume = this.volume;

    if (this.isPlaying) {
      audio.play().catch(error => {
        console.error('播放失败:', error);
        this.isPlaying = false;
      });
    }
  }

  updateTime() {
    this.currentTime = this.audioElement.nativeElement.currentTime;
  }

  updateDuration() {
    this.totalTime = this.audioElement.nativeElement.duration;
  }

  // 修改切换播放状态方法
  togglePlay() {
    if (!this.currentTrack) return;
    const audio = this.audioElement.nativeElement;
    if (audio.paused) {
      audio.play().catch(error => {
        console.error('播放失败:', error);
        this.isPlaying = false;
      });
      this.isPlaying = true;
    } else {
      audio.pause();
      this.isPlaying = false;
    }
  }

  handleProgressClick(event: MouseEvent) {
    const rect = (event.currentTarget as HTMLElement).getBoundingClientRect();
    const clickPosition = (event.clientX - rect.left) / rect.width;
    this.audioElement.nativeElement.currentTime = clickPosition * this.totalTime;
  }

  previousTrack() {
    this.currentTrackIndex =
      (this.currentTrackIndex - 1 + this.playlist.length) % this.playlist.length;
    this.loadTrack();
  }

  nextTrack() {
    this.currentTrackIndex = (this.currentTrackIndex + 1) % this.playlist.length;
    this.loadTrack();
  }

  onVolumeChange(event: Event) {
    const target = event.target as HTMLInputElement;
    this.volume = parseFloat(target.value);
    this.audioElement.nativeElement.volume = this.volume;
  }

  formatTime(seconds: number): string {
    const mins = Math.floor(seconds / 60);
    const secs = Math.floor(seconds % 60);
    return `${mins}:${secs.toString().padStart(2, '0')}`;
  }


    private playImmediately(song: { url: string, title: string, artist: string }) {
    // 清空播放列表并添加新歌曲
    this.playlist = [song];
    this.currentTrackIndex = 0;

    // 立即加载并播放
    const audio = this.audioElement.nativeElement;
    audio.src = song.url;
    audio.load();

    // 处理浏览器自动播放策略
    const playPromise = audio.play();
    if (playPromise !== undefined) {
      playPromise.catch(error => {
        console.error('自动播放被阻止:', error);
        // 显示播放按钮让用户手动点击
        this.isPlaying = false;
      });
    }

    this.isPlaying = true;
  }

}
<!-- music-player.component.html -->
<div class="music-player">
  <audio #audioElement
         (ended)="nextTrack()"
         (play)="isPlaying = true"
         (pause)="isPlaying = false">
    您的浏览器不支持 audio 元素
  </audio>

  <div class="top-section">
    <div class="album-art">
      <img src="https://randomuser.me/api/portraits/men/1.jpg" alt="Album Art">
    </div>

    <div class="song-info">
      <h2 class="song-title">{{ currentTrack.title }}</h2>
      <p class="artist">{{ currentTrack.artist }}</p>
    </div>

    <div class="volume-control">
      <input type="range"
             min="0"
             max="1"
             step="0.1"
             [value]="volume"
             (input)="onVolumeChange($event)"
             class="volume-slider">
      <span class="volume-percent">{{ volumePercent }}%</span>
    </div>
  </div>

  <div class="progress-container" (click)="handleProgressClick($event)">
    <div class="progress-bar" [style.width.%]="progress"></div>
    <div class="time-display">
      <span>{{ formatTime(currentTime) }}</span>
      <span>{{ formatTime(totalTime) }}</span>
    </div>
  </div>

  <div class="controls">
    <button (click)="previousTrack()">⏮</button>
    <button class="play-pause" (click)="togglePlay()">
      {{ isPlaying ? '⏸' : '▶' }}
    </button>
    <button (click)="nextTrack()">⏭</button>
  </div>
</div>
/* music-player.component.css */
.music-player {
  width: 700px;
  flex: 1;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  padding: 20px;
}
.top-section {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 20px;
}

.album-art img {
  width: 60px;
  height: 60px;
  border-radius: 8px;
  margin: 0 auto;
  display: block;
}

.song-info {
  text-align: left;
  flex: 1;
}

.song-title {
  margin: 0;
  font-size: 16px;
}

.artist {
  margin: 4px 0 0;
  font-size: 14px;
  color: #666;
}

.progress-container {
  height: 4px;
  background: #eee;
  border-radius: 2px;
  margin: 20px 0;
  cursor: pointer;
  position: relative;
}

.progress-bar {
  height: 100%;
  background: #42b983;
  border-radius: 2px;
  transition: width 0.1s linear;
}

.time-display {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #666;
  margin-top: 8px;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin: 15px 0;
}

.controls button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background: #42b983;
  color: white;
  cursor: pointer;
  transition: opacity 0.2s;
}

.controls button:hover {
  opacity: 0.8;
}

.volume-control {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 120px;
}

.volume-slider {
  width: 80px;
}

.volume-percent {
  font-size: 14px;
  color: #666;
}

.play-pause {
  padding: 10px 25px !important;
  font-size: 1.2em;
}

```


end