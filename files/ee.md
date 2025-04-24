说明：fastapi+angular在线订电影票系统
1.电影列表
电影名  评分  主演  导演  标签  类型


2.电影详情页
电影名  类型 上映时间 标签 类型
主演 导演 时长
评分
剧情简介
用户评论列表



3.影院列表
影院名 地址  类型 标签  价格

4.对应影院的日期 场次列表
电影开场时间，结束时间，类型（国语2D，英文版3D），价格

5.购票，提交订单
影院名称  座位号  场次信息 电影开场时间 电影类型 购买人姓名，联系电话 

6.订单记录  
订单日期，实付金额 ，用户名，电话号码 座位信息 影院名，放映厅名（8号厅）
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d8d52b6e3cd143d0b43b1fc9354c84b4.png#pic_center)

step1:建表。添加数据，查询 mysql

```sql

-- 电影相关表
CREATE TABLE movie (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,       -- 电影名
    director VARCHAR(255) NOT NULL,   -- 导演（简化为字符串存储）
    duration INT NOT NULL,            -- 时长（分钟）
    release_date DATE,                -- 上映时间
    description TEXT,                 -- 剧情简介
    rating DECIMAL(3,1) DEFAULT 0.0   -- 评分（10分制）
);

CREATE TABLE genre (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL -- 类型名称（如：喜剧/动作）
);

CREATE TABLE movie_genre (
    movie_id INT NOT NULL,
    genre_id INT NOT NULL,
    PRIMARY KEY (movie_id, genre_id),
    FOREIGN KEY (movie_id) REFERENCES movie(id),
    FOREIGN KEY (genre_id) REFERENCES genre(id)
);

CREATE TABLE actor (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL       -- 演员姓名
);

CREATE TABLE movie_actor (
    movie_id INT NOT NULL,
    actor_id INT NOT NULL,
    PRIMARY KEY (movie_id, actor_id),
    FOREIGN KEY (movie_id) REFERENCES movie(id),
    FOREIGN KEY (actor_id) REFERENCES actor(id)
);

-- 影院相关表
CREATE TABLE cinema (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,      -- 影院名称
    address VARCHAR(255) NOT NULL,    -- 地址
    tags VARCHAR(255)                 -- 标签（如：IMAX/杜比，逗号分隔存储）
);

CREATE TABLE hall (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cinema_id INT NOT NULL,
    name VARCHAR(50) NOT NULL,        -- 放映厅名称（如：8号厅）
    FOREIGN KEY (cinema_id) REFERENCES cinema(id)
);

CREATE TABLE schedule (
    id INT PRIMARY KEY AUTO_INCREMENT,
    cinema_id INT NOT NULL,
    hall_id INT NOT NULL,
    movie_id INT NOT NULL,
    start_time DATETIME NOT NULL,     -- 开场时间
    end_time DATETIME NOT NULL,       -- 结束时间
    language_version VARCHAR(50),     -- 放映类型（如：国语2D）
    price DECIMAL(8,2) NOT NULL,      -- 价格
    FOREIGN KEY (cinema_id) REFERENCES cinema(id),
    FOREIGN KEY (hall_id) REFERENCES hall(id),
    FOREIGN KEY (movie_id) REFERENCES movie(id)
);

-- 订单相关表
CREATE TABLE `order` (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_name VARCHAR(255) NOT NULL,  -- 购买人姓名
    phone VARCHAR(20) NOT NULL,       -- 联系电话
    schedule_id INT NOT NULL,         -- 关联场次
    total_amount DECIMAL(10,2),       -- 实付金额
    order_time DATETIME NOT NULL,     -- 下单时间
    status TINYINT DEFAULT 1,         -- 订单状态（1-有效 0-取消）
    FOREIGN KEY (schedule_id) REFERENCES schedule(id)
);

CREATE TABLE order_seat (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    seat_number VARCHAR(20) NOT NULL, -- 座位号（如：A10/B5）
    FOREIGN KEY (order_id) REFERENCES `order`(id)
);

-- 评论表
CREATE TABLE comment (
    id INT PRIMARY KEY AUTO_INCREMENT,
    movie_id INT NOT NULL,
    user_name VARCHAR(255),           -- 评论用户（可匿名）
    content TEXT NOT NULL,            -- 评论内容
    rating DECIMAL(3,1),              -- 用户评分
    comment_time DATETIME NOT NULL,   -- 评论时间
    FOREIGN KEY (movie_id) REFERENCES movie(id)
);

-- Insert Genres
INSERT INTO genre (name) VALUES
('Comedy'), ('Action'), ('Sci-Fi'), ('Romance'), ('Thriller'),
('Horror'), ('Animation'), ('Adventure'), ('Crime'), ('Fantasy');

-- Insert Actors
INSERT INTO actor (name) VALUES
('Tim Robbins'), ('Morgan Freeman'), ('Leonardo DiCaprio'),
('Joseph Gordon-Levitt'), ('Kate Winslet'), ('Tom Hanks'),
('Matthew McConaughey'), ('Alan Rickman'), ('Robert Downey Jr.'), ('Scarlett Johansson');

-- Insert Movies
INSERT INTO movie (name, director, duration, release_date, description, rating) VALUES
('The Shawshank Redemption', 'Frank Darabont', 142, '1994-09-23', 'A wrongfully convicted banker''s journey of redemption', 9.3),
('Inception', 'Christopher Nolan', 148, '2010-07-16', 'A mind-bending heist through dreamscapes', 9.0),
('Titanic', 'James Cameron', 195, '1997-12-19', 'Epic romance aboard the ill-fated RMS Titanic', 9.1),
('Forrest Gump', 'Robert Zemeckis', 142, '1994-07-06', 'Life story of a man with low IQ who witnesses historical events', 9.2),
('Interstellar', 'Christopher Nolan', 169, '2014-11-07', 'Space exploration to save humanity through a wormhole', 9.3),
('Léon: The Professional', 'Luc Besson', 110, '1994-09-14', 'A hitman and a young girl form an unlikely bond', 9.2),
('Avengers: Endgame', 'Joe Russo', 181, '2019-04-26', 'Superheroes'' final battle against Thanos', 8.8),
('Harry Potter and the Sorcerer''s Stone', 'Chris Columbus', 152, '2001-11-04', 'A boy''s introduction to the magical world', 8.9),
('The Lord of the Rings: The Fellowship of the Ring', 'Peter Jackson', 178, '2001-12-19', 'Quest to destroy the One Ring', 9.0),
('Zootopia', 'Byron Howard', 108, '2016-03-04', 'Anthropomorphic animal police adventure', 9.0);

-- Associate Movies with Genres (same IDs)
INSERT INTO movie_genre (movie_id, genre_id) VALUES
(1,9), (2,2), (2,3), (3,4), (4,8),
(5,3), (5,8), (6,2), (6,9), (7,2),
(7,3), (8,8), (8,10), (9,8), (9,10),
(10,1), (10,7);

-- Associate Movies with Actors (same IDs)
INSERT INTO movie_actor (movie_id, actor_id) VALUES
(1,1), (1,2), (2,3), (2,4), (3,3),
(3,5), (4,6), (5,7), (6,8), (7,9),
(7,10), (8,5), (9,3), (10,6);

-- Insert Cinemas
INSERT INTO cinema (name, address, tags) VALUES
('Wanda Cinemas (Chaoyang, Beijing)', 'Wanda Plaza, 93 Jianguo Road, Chaoyang District, Beijing', 'IMAX,Dolby'),
('UME International Cinema (Shanghai)', '388 Xijiangwan Road, Hongkou District, Shanghai', '4DX,Dolby Atmos'),
('Dadi Cinemas (Guangzhou)', '208 Tianhe Road, Tianhe District, Guangzhou', 'China Giant Screen'),
('Golden Harvest Cinema (Shenzhen)', '118 Fuhua 3rd Road, Futian District, Shenzhen', 'IMAX'),
('CGV Cinemas (Nanjing)', '1 Zhongshan South Road, Qinhuai District, Nanjing', '4DX,ScreenX'),
('Broadway Cinemas (Chengdu)', '8 Zhongshamao Street, Jinjiang District, Chengdu', 'Dolby Atmos'),
('Hengdian Cinema (Hangzhou)', '551 Wener West Road, Xihu District, Hangzhou', 'China Giant Screen'),
('Stellar Cinema (Wuhan)', 'Optics Valley Walk, Hongshan District, Wuhan', 'IMAX'),
('Pacific Theaters (Chongqing)', '26 Minquan Road, Yuzhong District, Chongqing', 'Dolby'),
('SFC Cinema (Xi''an)', '232 Xiaozhai West Road, Yanta District, Xi''an', 'IMAX,4DX');

-- Insert Halls (same structure)
INSERT INTO hall (cinema_id, name) VALUES
(1, 'Hall 1'), (1, 'IMAX Theater'), (2, 'Hall A'), (2, '4DX Hall'),
(3, 'Giant Screen Hall'), (3, 'VIP Hall'), (4, 'IMAX Theater'), (4, 'Hall 2'),
(5, 'ScreenX Hall'), (5, '4DX Hall'), (6, 'Dolby Cinema'), (6, 'Hall 1'),
(7, 'China Giant Screen'), (7, 'Hall 3'), (8, 'IMAX Theater'), (8, 'VIP Hall'),
(9, 'Dolby Cinema'), (9, 'Hall 2'), (10, 'IMAX Theater'), (10, '4DX Hall');

-- Insert Schedules
INSERT INTO schedule (cinema_id, hall_id, movie_id, start_time, end_time, language_version, price) VALUES
(1,1,1,'2023-10-01 10:00:00','2023-10-01 12:22:00','Mandarin 2D',45.00),
(1,2,2,'2023-10-01 13:00:00','2023-10-01 15:28:00','English IMAX',120.00),
(2,3,3,'2023-10-01 11:00:00','2023-10-01 14:15:00','English 3D',80.00),
(2,4,4,'2023-10-01 15:00:00','2023-10-01 17:22:00','Mandarin 2D',50.00),
(3,5,5,'2023-10-01 12:00:00','2023-10-01 14:49:00','English 2D',60.00),
(3,6,6,'2023-10-01 18:00:00','2023-10-01 19:50:00','French 2D',55.00),
(4,7,7,'2023-10-01 14:00:00','2023-10-01 17:01:00','English IMAX',110.00),
(4,8,8,'2023-10-01 10:30:00','2023-10-01 13:02:00','English 3D',75.00),
(5,9,9,'2023-10-01 16:00:00','2023-10-01 18:58:00','English 2D',65.00),
(5,10,10,'2023-10-01 09:30:00','2023-10-01 11:18:00','Mandarin 3D',40.00);

-- Insert Orders (keep original phone formats)
INSERT INTO `order` (user_name, phone, schedule_id, total_amount, order_time, status) VALUES
('John Zhang','13800000001',1,90.00,'2023-10-01 09:00:00',1),
('Mike Li','13800000002',2,240.00,'2023-10-01 12:30:00',1),
('Sarah Wang','13800000003',3,160.00,'2023-10-01 10:30:00',1),
('Emily Zhao','13800000004',4,100.00,'2023-10-01 14:30:00',1),
('David Qian','13800000005',5,120.00,'2023-10-01 11:00:00',1);

-- Insert Seats
INSERT INTO order_seat (order_id, seat_number) VALUES
(1,'A10'), (1,'B5'), (2,'C3'), (3,'D8'), (4,'E12');

-- Insert Comments
INSERT INTO comment (movie_id, user_name, content, rating, comment_time) VALUES
(1,'MovieBuff','An absolute classic, never gets old!',9.5,'2023-10-01 12:30:00'),
(2,'NolanFan','Masterpiece of layered storytelling',9.0,'2023-10-01 15:00:00'),
(3,'CinemaLover','Timeless romance, visually stunning',9.2,'2023-10-01 14:00:00'),
(4,'Runner123','Life is like a box of chocolates',9.3,'2023-10-01 16:00:00'),
(5,'SpaceGeek','Mind-blowing cosmic journey',9.4,'2023-10-01 17:00:00');
```

step2:fastapi，路径C:\Users\wangrusheng\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException, Query
from fastapi.middleware.cors import CORSMiddleware
import pymysql.cursors
from datetime import date
from pydantic import BaseModel
from typing import List

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


def query_database(query: str, params=None, fetch_one=False):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params or ())
            result = cursor.fetchone() if fetch_one else cursor.fetchall()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


# 1. 电影列表
@app.get("/movies")
async def get_movie_list(page: int = 1, size: int = 10):
    """获取电影列表"""
    offset = (page - 1) * size
    query = """
    SELECT 
        m.id,
        m.name,
        m.rating,
        m.director,
        GROUP_CONCAT(DISTINCT g.name) AS genres,
        GROUP_CONCAT(DISTINCT a.name) AS actors
    FROM movie m
    LEFT JOIN movie_genre mg ON m.id = mg.movie_id
    LEFT JOIN genre g ON mg.genre_id = g.id
    LEFT JOIN movie_actor ma ON m.id = ma.movie_id
    LEFT JOIN actor a ON ma.actor_id = a.id
    GROUP BY m.id
    LIMIT %s OFFSET %s
    """
    data = query_database(query, (size, offset))
    return {"data": data}


# 2. 电影详情
@app.get("/movies/{movie_id}")
async def get_movie_detail(movie_id: int):
    """获取电影详情"""
    # 基本信息
    query = """
    SELECT 
        m.*,
        GROUP_CONCAT(DISTINCT g.name) AS genres,
        GROUP_CONCAT(DISTINCT a.name) AS actors
    FROM movie m
    LEFT JOIN movie_genre mg ON m.id = mg.movie_id
    LEFT JOIN genre g ON mg.genre_id = g.id
    LEFT JOIN movie_actor ma ON m.id = ma.movie_id
    LEFT JOIN actor a ON ma.actor_id = a.id
    WHERE m.id = %s
    GROUP BY m.id
    """
    movie = query_database(query, (movie_id,), fetch_one=True)

    # 用户评论
    comment_query = "SELECT * FROM comment WHERE movie_id = %s ORDER BY comment_time DESC"
    comments = query_database(comment_query, (movie_id,))

    if not movie:
        raise HTTPException(status_code=404, detail="Movie not found")

    return {"data": {**movie, "comments": comments}}


# 3. 影院列表
@app.get("/cinemas")
async def get_cinema_list(city: str = None):
    """获取影院列表"""
    query = """
    SELECT 
        c.*,
        MIN(s.price) AS min_price
    FROM cinema c
    LEFT JOIN schedule s ON c.id = s.cinema_id
    WHERE (%s IS NULL OR c.address LIKE CONCAT('%%', %s, '%%'))
    GROUP BY c.id
    """
    data = query_database(query, (city, city))
    return {"data": data}


# 4. 影院场次
@app.get("/cinemas/{cinema_id}/schedules")
async def get_cinema_schedules(cinema_id: int, date: date = Query(...)):
    """获取影院指定日期的场次"""
    query = """
    SELECT 
        s.*,
        m.name AS movie_name,
        h.name AS hall_name
    FROM schedule s
    JOIN movie m ON s.movie_id = m.id
    JOIN hall h ON s.hall_id = h.id
    WHERE s.cinema_id = %s
      AND DATE(s.start_time) = %s
    ORDER BY s.start_time
    """
    data = query_database(query, (cinema_id, date))
    return {"data": data}


# 5. 提交订单
class OrderItem(BaseModel):
    schedule_id: int
    seats: List[str]
    user_name: str
    total_amount:float
    phone: str


@app.post("/orders")
async def create_order(order: OrderItem):
    """创建订单"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 创建主订单
            order_query = """
            INSERT INTO `order` 
                (user_name, phone,total_amount, schedule_id, order_time)
            VALUES (%s, %s,  %s,%s, NOW())
            """
            cursor.execute(order_query, (order.user_name, order.phone, order.total_amount,  order.schedule_id))
            order_id = cursor.lastrowid

            # 插入座位信息
            seat_query = "INSERT INTO order_seat (order_id, seat_number) VALUES (%s, %s)"
            for seat in order.seats:
                cursor.execute(seat_query, (order_id, seat))

            connection.commit()
            return {"order_id": order_id}
    except Exception as e:
        connection.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        connection.close()


# 6. 订单记录
@app.get("/orders")
async def get_order_history(
        user_name: str = Query(...),
        phone: str = Query(...)
):
    """查询订单记录"""
    query = """
    SELECT 
        o.id,
        o.order_time,
        o.total_amount,
        c.name AS cinema_name,
        h.name AS hall_name,
        m.name AS movie_name,
        s.start_time,
        GROUP_CONCAT(os.seat_number) AS seats
    FROM `order` o
    JOIN schedule s ON o.schedule_id = s.id
    JOIN cinema c ON s.cinema_id = c.id
    JOIN hall h ON s.hall_id = h.id
    JOIN movie m ON s.movie_id = m.id
    JOIN order_seat os ON o.id = os.order_id
    WHERE o.user_name = %s AND o.phone = %s
    GROUP BY o.id
    """
    data = query_database(query, (user_name, phone))
    return {"data": data}


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:验证postman

```bash


电影列表  get : http://localhost:8000/movies



{
    "data": [
        {
            "id": 1,
            "name": "The Shawshank Redemption",
            "rating": 9.3,
            "director": "Frank Darabont",
            "genres": "Crime",
            "actors": "Morgan Freeman, Tim Robbins"
        },
        {
            "id": 2,
            "name": "Inception",
            "rating": 9.0,
            "director": "Christopher Nolan",
            "genres": "Action,Sci-Fi",
            "actors": "Joseph Gordon-Levitt, Leonardo DiCaprio"
        },
        {
            "id": 3,
            "name": "Titanic",
            "rating": 9.1,
            "director": "James Cameron",
            "genres": "Romance",
            "actors": "Kate Winslet, Leonardo DiCaprio"
        },
        {
            "id": 4,
            "name": "Forrest Gump",
            "rating": 9.2,
            "director": "Robert Zemeckis",
            "genres": "Adventure",
            "actors": "Tom Hanks"
        }
    ]
}


电影详情 get http://localhost:8000/movies/1

 {
    "data": {
        "id": 1,
        "name": "The Shawshank Redemption",
        "director": "Frank Darabont",
        "duration": 142,
        "release_date": "1994-09-23",
        "description": "A wrongfully convicted banker's journey of redemption",
        "rating": 9.3,
        "genres": "Crime",
        "actors": "Morgan Freeman, Tim Robbins",
        "comments": [
            {
                "id": 11,
                "movie_id": 1,
                "user_name": "MovieBuff",
                "content": "A timeless masterpiece that gets better with every viewing",
                "rating": 9.8,
                "comment_time": "2025-03-15T22:10:00"
            }
        ]
    }
}



影院列表 get http://localhost:8000/cinemas
 {
    "data": [
        {
            "id": 1,
            "name": "Wanda Cinemas (Chaoyang, Beijing)",
            "address": "Wanda Plaza, 93 Jianguo Road, Chaoyang District, Beijing",
            "tags": "IMAX,Dolby",
            "min_price": 45.0
        },
        {
            "id": 2,
            "name": "UME International Cinema (Shanghai)",
            "address": "388 Xijiangwan Road, Hongkou District, Shanghai",
            "tags": "4DX,Dolby Atmos",
            "min_price": 50.0
        }
    ]
}



影院场次 get  http://localhost:8000/cinemas/2/schedules?date=2023-10-01

{
    "data": [
        {
            "id": 3,
            "cinema_id": 2,
            "hall_id": 3,
            "movie_id": 3,
            "start_time": "2023-10-01T11:00:00",
            "end_time": "2023-10-01T14:15:00",
            "language_version": "English 3D",
            "price": 80.0,
            "movie_name": "Titanic",
            "hall_name": "Hall A"
        }
    ]
}

提交订单  post  http://localhost:8000/orders


请求体
{
  "user_name": "John Lee",
  "phone": "13812345678",
  "schedule_id": 1,
  "total_amount": 198.00,
  "seats": ["A12", "B5", "C3"]
}

返回

{
    "order_id": 19
}


订单记录   get    http://localhost:8000/orders?user_name=李老八&phone=13812345678

{
    "data": [
        {
            "id": 19,
            "order_time": "2025-03-17T04:20:35",
            "total_amount": 198.0,
            "cinema_name": "Wanda Cinemas (Chaoyang, Beijing)",
            "hall_name": "Hall 1",
            "movie_name": "The Shawshank Redemption",
            "start_time": "2023-10-01T10:00:00",
            "seats": "A12,B5,C3"
        }
    ]
}



```

step4:路由

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class MovieService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  getMovies(page = 1, size = 10): Observable<any> {
    return this.http.get(`${this.apiUrl}/movies`, {
      params: { page, size }
    });
  }

  getMovieDetail(id: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/movies/${id}`);
  }

  getCinemas(city?: string): Observable<any> {
    let params = new HttpParams();
    if (city) params = params.set('city', city);
    return this.http.get(`${this.apiUrl}/cinemas`, { params });
  }

  getSchedules(cinemaId: number, date: string): Observable<any> {
    return this.http.get(`${this.apiUrl}/cinemas/${cinemaId}/schedules`, {
      params: { date }
    });
  }

  createOrder(order: any): Observable<any> {
    console.log("createOrder",order)
    return this.http.post(`${this.apiUrl}/orders`, order);
  }

  getOrders(userName: string, phone: string): Observable<any> {
    return this.http.get(`${this.apiUrl}/orders`, {
      params: { user_name: userName, phone }
    });
  }
}

```

```bash
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.component.html
      <a routerLink="/movie-list" routerLinkActive="active">电影列表</a>
      <a routerLink="/movie-detail" routerLinkActive="active">电影详情</a>
      <a routerLink="/movie-cinema-list" routerLinkActive="active">影院列表</a>
      <a routerLink="/movie-schedule-list" routerLinkActive="active">场次列表</a>
      <a routerLink="/movie-order" routerLinkActive="active">提交订单</a>

C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.routes.ts

  { path: 'movie-list', component: MovieListComponent },
  { path: 'movie-detail/:id', component: MovieDetailComponent },
  {
    path: 'movie-cinema-list/:id',
    component: MovieCinemaListComponent
  },

  { path: 'movie-schedule-list/:id', component: MovieScheduleListComponent },
  { path: 'movie-order', component: MovieOrderComponent },


```

step5:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\movie-list\movie-list.component.ts

```typescript
// movie-list.component.ts
import { Component, OnInit } from '@angular/core';
import { MovieService } from '../services/movie.service';
import {RouterLink} from '@angular/router';
import {NgForOf} from '@angular/common';

@Component({
  selector: 'app-movie-list',
  templateUrl: './movie-list.component.html',
  imports: [
    RouterLink,
    NgForOf
  ],
  styleUrls: ['./movie-list.component.css']
})
export class MovieListComponent implements OnInit {
  movies: any[] = [];

  constructor(private movieService: MovieService) { }

  ngOnInit() {
    this.movieService.getMovies().subscribe(res => {
      this.movies = res.data;
    });
  }

  getStarRating(rating: number): string {
    const scaled = rating / 2;
    const full = Math.floor(scaled);
    const half = scaled % 1 >= 0.5 ? 1 : 0;
    return '★'.repeat(full) + (half ? '½' : '') + '☆'.repeat(5 - full - half);
  }
}


<!-- movie-list.component.html -->
<div class="movie-grid">
  <div *ngFor="let movie of movies" class="movie-card">
    <div class="card-content">
      <h3 class="movie-title">{{ movie.name }}</h3>
      <div class="rating-section">
        <span class="rating-text">评分：{{ movie.rating }}</span>
        <span class="stars">{{ getStarRating(movie.rating) }}</span>
      </div>
      <p class="director">导演：{{ movie.director }}</p>
      <p class="genres">类型：{{ movie.genres }}</p>
      <p class="actors">主演：{{ movie.actors }}</p>
      <a [routerLink]="['/movie-detail', movie.id]" class="detail-link">查看详情</a>
    </div>
  </div>
</div>


/* movie-list.component.css */
.movie-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 2rem;
  padding: 2rem;
}

.movie-card {
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.movie-card:hover {
  transform: translateY(-5px);
}

.card-content {
  padding: 1.5rem;
}

.movie-title {
  font-size: 1.25rem;
  color: #2d3748;
  margin-bottom: 1rem;
}

.rating-section {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.75rem;
}

.rating-text {
  font-size: 0.9rem;
  color: #4a5568;
}

.stars {
  color: #f6ad55;
  font-size: 1.1rem;
}

.director, .genres, .actors {
  font-size: 0.95rem;
  color: #4a5568;
  margin-bottom: 0.5rem;
  line-height: 1.5;
}

.detail-link {
  display: inline-block;
  background: #48bb78;
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  text-decoration: none;
  font-size: 0.9rem;
  transition: background 0.2s;
}

.detail-link:hover {
  background: #48bb78;
}

```

step6:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\movie-detail\movie-detail.component.ts

```typescript
// movie-detail.component.ts
import { Component, OnInit } from '@angular/core';
import {ActivatedRoute, RouterLink} from '@angular/router';
import { MovieService } from '../services/movie.service';
import {DatePipe, NgForOf, NgIf} from '@angular/common';

@Component({
  selector: 'app-movie-detail',
  templateUrl: './movie-detail.component.html',
  imports: [
    RouterLink,
    NgForOf,
    DatePipe,
    NgIf
  ],
  styleUrls: ['./movie-detail.component.css']
})
export class MovieDetailComponent implements OnInit {
  movie: any;

  constructor(
    private route: ActivatedRoute,
    private movieService: MovieService
  ) { }

  ngOnInit() {
    const id = this.route.snapshot.paramMap.get('id');
    this.movieService.getMovieDetail(+id!).subscribe(res => {
      this.movie = res.data;
    });
  }

  getStarRating(rating: number): string {
    const scaled = rating / 2;
    const full = Math.floor(scaled);
    const half = scaled % 1 >= 0.5 ? 1 : 0;
    return '★'.repeat(full) + (half ? '½' : '') + '☆'.repeat(5 - full - half);
  }
}


<!-- movie-detail.component.html -->
<div *ngIf="movie" class="movie-detail-container">
  <div class="detail-card">
    <h1 class="movie-title">{{ movie.name }}</h1>

    <div class="basic-info">
      <div class="info-item">
        <span class="label">上映时间：</span>
        <span class="value">{{ movie.release_date | date:'yyyy-MM-dd' }}</span>
      </div>
      <div class="info-item">
        <span class="label">时长：</span>
        <span class="value">{{ movie.duration }}分钟</span>
      </div>
      <div class="info-item">
        <span class="label">评分：</span>
        <span class="rating">
          <span class="score">{{ movie.rating }}</span>
          <span class="stars">{{ getStarRating(movie.rating) }}</span>
        </span>
      </div>
    </div>

    <div class="synopsis-section">
      <h2 class="section-title">剧情简介</h2>
      <p class="synopsis-text">{{ movie.description }}</p>
    </div>

    <a [routerLink]="['/movie-cinema-list', movie.id]" class="ticket-link">立即购票</a>

    <div class="comments-section">
      <h2 class="section-title">用户评论</h2>
      <div *ngFor="let comment of movie.comments" class="comment-card">
        <div class="comment-header">
          <span class="user-name">{{ comment.user_name }}</span>
          <span class="comment-rating">
            <span class="stars">{{ getStarRating(comment.rating) }}</span>
            <span class="time">{{ comment.comment_time | date:'yyyy-MM-dd HH:mm' }}</span>
          </span>
        </div>
        <p class="comment-content">{{ comment.content }}</p>
      </div>
    </div>
  </div>
</div>


/* movie-detail.component.css */
.movie-detail-container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.detail-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 2rem;
}

.movie-title {
  font-size: 2rem;
  color: #2d3748;
  margin-bottom: 1.5rem;
}

.basic-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.info-item {
  background: #f7fafc;
  padding: 1rem;
  border-radius: 8px;
}

.label {
  font-weight: 600;
  color: #4a5568;
}

.value {
  color: #718096;
}

.rating {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.score {
  font-weight: bold;
  color: #2d3748;
}

.stars {
  color: #f6ad55;
  font-size: 1.2rem;
}

.synopsis-section {
  margin-bottom: 2rem;
}

.section-title {
  font-size: 1.5rem;
  color: #2d3748;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #e2e8f0;
}

.synopsis-text {
  color: #4a5568;
  line-height: 1.6;
  font-size: 1rem;
}

.ticket-link {
  display: inline-block;
  background: #48bb78;
  color: white;
  padding: 1rem 2rem;
  border-radius: 6px;
  text-decoration: none;
  font-weight: 500;
  transition: background 0.2s;
  margin: 1.5rem 0;
}

.ticket-link:hover {
  background: #48bb78;
}

.comment-card {
  background: #f7fafc;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.comment-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.user-name {
  font-weight: 600;
  color: #2d3748;
}

.comment-rating {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.comment-content {
  color: #4a5568;
  line-height: 1.5;
}

```

step7:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\movie-cinema-list\movie-cinema-list.component.ts

```typescript
// movie-cinema-list.component.ts
import { Component, OnInit } from '@angular/core';
import { MovieService } from '../services/movie.service';
import {RouterLink} from '@angular/router';
import {NgForOf} from '@angular/common';
import {FormsModule} from '@angular/forms';

@Component({
  selector: 'app-movie-cinema-list',
  templateUrl: './movie-cinema-list.component.html',
  imports: [
    RouterLink,
    NgForOf,
    FormsModule
  ],
  styleUrls: ['./movie-cinema-list.component.css']
})
export class MovieCinemaListComponent implements OnInit {
  cinemas: any[] = [];
  city = '';

  constructor(private movieService: MovieService) { }

  ngOnInit() {
    this.searchCinemas();
  }

  searchCinemas() {
    this.movieService.getCinemas(this.city).subscribe(res => {
      this.cinemas = res.data;
    });
  }

  formatPrice(price: number): string {
    return price ? `¥${price.toFixed(2)}` : '暂无场次';
  }
}


<!-- movie-cinema-list.component.html -->
<div class="cinema-list-container">
  <div class="search-box">
    <input [(ngModel)]="city"
           (input)="searchCinemas()"
           placeholder="输入城市名称筛选影院"
           class="search-input">
  </div>

  <div class="cinema-grid">
    <div *ngFor="let cinema of cinemas" class="cinema-card">
      <div class="card-content">
        <h3 class="cinema-name">{{ cinema.name }}</h3>
        <p class="cinema-address">{{ cinema.address }}</p>

        <div class="tags-section">
          <span *ngFor="let tag of cinema.tags.split(',')"
                class="tag">
            {{ tag }}
          </span>
        </div>

        <div class="price-section">
          <span class="price-label">最低票价：</span>
          <span class="price-value">{{ formatPrice(cinema.min_price) }}</span>
        </div>

        <a [routerLink]="['/movie-schedule-list', cinema.id]"
           class="schedule-link">
          查看场次
        </a>
      </div>
    </div>
  </div>
</div>


/* movie-cinema-list.component.css */
.cinema-list-container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.search-box {
  margin-bottom: 2rem;
  padding: 0 1rem;
}

.search-input {
  width: 100%;
  padding: 0.8rem 1.2rem;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #48bb78;
}

.cinema-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
  padding: 0 1rem;
}

.cinema-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.cinema-card:hover {
  transform: translateY(-3px);
}

.card-content {
  padding: 1.5rem;
}

.cinema-name {
  font-size: 1.25rem;
  color: #2d3748;
  margin-bottom: 0.75rem;
}

.cinema-address {
  color: #718096;
  font-size: 0.9rem;
  line-height: 1.4;
  margin-bottom: 1rem;
}

.tags-section {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.tag {
  background: #f7fafc;
  color: #48bb78;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
}

.price-section {
  display: flex;
  align-items: center;
  margin-bottom: 1.5rem;
}

.price-label {
  color: #718096;
  font-size: 0.9rem;
}

.price-value {
  color: #48bb78;
  font-size: 1.1rem;
  font-weight: 600;
  margin-left: 0.5rem;
}

.schedule-link {
  display: inline-block;
  background: #48bb78;
  color: white;
  padding: 0.6rem 1.2rem;
  border-radius: 6px;
  text-decoration: none;
  font-size: 0.9rem;
  transition: background 0.3s;
}

.schedule-link:hover {
  background: #48bb78;
}

```

step8:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\movie-schedule-list\movie-schedule-list.component.ts

```typescript
// movie-schedule-list.component.ts
import { Component, OnInit } from '@angular/core';
import {ActivatedRoute, RouterLink} from '@angular/router';
import { MovieService } from '../services/movie.service';
import {DatePipe, NgForOf} from '@angular/common';
import {FormsModule} from '@angular/forms';

interface Schedule {
  id: number;
  movie_name: string;
  hall_name: string;
  start_time: string;
  end_time: string;
  language_version: string;
  price: number;
}

interface ScheduleResponse {
  data: Schedule[];
}

@Component({
  selector: 'app-movie-schedule-list',
  templateUrl: './movie-schedule-list.component.html',
  imports: [
    NgForOf,
    DatePipe,
    RouterLink,
    FormsModule
  ],
  styleUrls: ['./movie-schedule-list.component.css']
})
export class MovieScheduleListComponent implements OnInit {
  schedules: Schedule[] = [];
  selectedDate = '2023-10-01';
  cinemaId: string | null = null;

  constructor(
    private route: ActivatedRoute,
    private movieService: MovieService
  ) { }

  ngOnInit() {
    this.cinemaId = this.route.snapshot.paramMap.get('id');
    this.loadSchedules();
  }

  loadSchedules() {
    if (this.cinemaId) {
      this.movieService.getSchedules(+this.cinemaId, this.selectedDate)
        .subscribe({
          next: (res: ScheduleResponse) => this.schedules = res.data,
          error: (err) => console.error('加载场次失败:', err)
        });
    }
  }
}


<!-- movie-schedule-list.component.html -->
<div class="schedule-container">
  <div class="date-picker">
    <input type="date"
           [(ngModel)]="selectedDate"
           (change)="loadSchedules()"
           class="date-input">
  </div>

  <div class="schedule-list">
    <div *ngFor="let schedule of schedules" class="schedule-card">
      <div class="card-header">
        <h3 class="movie-title">{{ schedule.movie_name }}</h3>
        <span class="hall-name">{{ schedule.hall_name }}</span>
      </div>

      <div class="schedule-details">
        <div class="time-section">
          <span class="start-time">{{ schedule.start_time | date:'HH:mm' }}</span>
          <span class="time-separator">-</span>
          <span class="end-time">{{ schedule.end_time | date:'HH:mm' }}</span>
        </div>

        <div class="version-price">
          <span class="language-version">{{ schedule.language_version }}</span>
          <span class="price">¥{{ schedule.price.toFixed(2) }}</span>
        </div>
      </div>

      <button [routerLink]="['/movie-order']"
              [state]="{ schedule: schedule }"
              class="book-button">
        立即购票
      </button>
    </div>
  </div>
</div>


/* movie-schedule-list.component.css */
.schedule-container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.date-picker {
  margin-bottom: 2rem;
}

.date-input {
  width: 100%;
  padding: 0.8rem;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
}

.schedule-list {
  display: grid;
  gap: 1.5rem;
}

.schedule-card {
  background: white;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.movie-title {
  font-size: 1.1rem;
  color: #2d3748;
  margin: 0;
}

.hall-name {
  background: #f7fafc;
  color: #48bb78;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.9rem;
}

.schedule-details {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.time-section {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.start-time, .end-time {
  font-size: 1.2rem;
  font-weight: 600;
  color: #2d3748;
}

.time-separator {
  color: #718096;
}

.version-price {
  text-align: right;
}

.language-version {
  display: block;
  color: #718096;
  font-size: 0.9rem;
}

.price {
  display: block;
  color: #48bb78;
  font-size: 1.4rem;
  font-weight: 600;
}

.book-button {
  width: 100%;
  background: #48bb78;
  color: white;
  padding: 0.8rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  cursor: pointer;
  transition: background 0.3s;
}

.book-button:hover {
  background: #48bb78;
}

```

step9:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\movie-order\movie-order.component.ts


```typescript
// movie-order.component.ts
import { Component } from '@angular/core';
import { MovieService } from '../services/movie.service';
import { Router } from '@angular/router';
import {DatePipe, NgForOf, NgIf} from '@angular/common';
import {FormsModule} from '@angular/forms';

interface NavigationState {
  schedule: {
    id: number;
    price: number;
    movie_name: string;
    hall_name: string;
    start_time: string;
  };
}

@Component({
  selector: 'app-movie-order',
  templateUrl: './movie-order.component.html',
  imports: [
    DatePipe,
    NgForOf,
    NgIf,
    FormsModule
  ],
  styleUrls: ['./movie-order.component.css']
})
export class MovieOrderComponent {
  order = {
    user_name: '',
    phone: '',
    seats: '',
    schedule_id: 0
  };
  orders: any[] = [];
  currentPrice = 0;

  constructor(
    private movieService: MovieService,
    private router: Router
  ) {
    const navigation = this.router.getCurrentNavigation();
    const state = navigation?.extras.state as NavigationState | undefined;

    if (state?.schedule) {
      this.order.schedule_id = state.schedule.id;
      this.currentPrice = state.schedule.price;
    } else {
      this.router.navigate(['/']);
    }
  }

  get seatCount(): number {
    return this.order.seats.split(',')
      .filter(seat => seat.trim().length > 0)
      .length;
  }

  get totalAmount(): number {
    return this.currentPrice * this.seatCount;
  }

  submitOrder() {
    const orderData = {
      ...this.order,
      total_amount: this.totalAmount,
      seats: this.order.seats.split(',').map(s => s.trim()).filter(Boolean)
    };

    if (orderData.seats.length === 0) {
      alert('请选择至少一个座位');
      return;
    }

    this.movieService.createOrder(orderData).subscribe({
      next: () => {
        this.movieService.getOrders(orderData.user_name, orderData.phone)
          .subscribe(res => this.orders = res.data);
      },
      error: (err) => alert('订单提交失败: ' + err.message)
    });
  }
}


<!-- movie-order.component.html -->
<div class="order-container">
  <div class="order-form">
    <h2 class="form-title">填写订单信息</h2>

    <form (ngSubmit)="submitOrder()" class="form">
      <div class="form-group">
        <label class="form-label">姓名</label>
        <input type="text"
               [(ngModel)]="order.user_name"
               name="userName"
               required
               class="form-input"
               placeholder="请输入真实姓名">
      </div>

      <div class="form-group">
        <label class="form-label">手机号</label>
        <input type="tel"
               [(ngModel)]="order.phone"
               name="phone"
               required
               pattern="1[3-9]\d{9}"
               class="form-input"
               placeholder="请输入11位手机号">
      </div>

      <div class="form-group">
        <label class="form-label">选择座位</label>
        <input type="text"
               [(ngModel)]="order.seats"
               name="seats"
               required
               class="form-input"
               placeholder="多个座位用逗号分隔，例：A1,B2">
        <p class="seat-hint">已选择 {{ seatCount }} 个座位</p>
      </div>

      <div class="price-summary">
        <div class="price-item">
          <span>单价：</span>
          <span class="price-value">¥{{ currentPrice.toFixed(2) }}</span>
        </div>
        <div class="price-item total">
          <span>总金额：</span>
          <span class="price-value">¥{{ totalAmount.toFixed(2) }}</span>
        </div>
      </div>

      <button type="submit" class="submit-button">提交订单</button>
    </form>
  </div>

  <div *ngIf="orders.length" class="order-history">
    <h3 class="history-title">历史订单</h3>
    <div *ngFor="let order of orders" class="history-card">
      <div class="order-header">
        <span class="cinema-name">{{ order.cinema_name }}</span>
        <span class="hall-name">{{ order.hall_name }}</span>
      </div>
      <p class="movie-info">{{ order.movie_name }}</p>
      <p class="show-time">{{ order.start_time | date:'yyyy-MM-dd HH:mm' }}</p>
      <div class="order-header">
        <span class="show-time">座位：{{ order.seats }}</span>
        <span class="hall-name">¥{{ order.total_amount.toFixed(2) }}</span>
      </div>
    </div>
  </div>
</div>


/* movie-order.component.css */
.order-container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.order-form {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 2rem;
}

.form-title {
  font-size: 1.5rem;
  color: #2d3748;
  margin-bottom: 1.5rem;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-label {
  display: block;
  color: #4a5568;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.form-input {
  width: 100%;
  padding: 0.8rem;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.3s;
}

.form-input:focus {
  outline: none;
  border-color: #48bb78;
}

.seat-hint {
  color: #718096;
  font-size: 0.9rem;
  margin-top: 0.5rem;
}

.price-summary {
  border-top: 2px solid #f7fafc;
  padding-top: 1.5rem;
  margin: 2rem 0;
}

.price-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
  font-size: 1rem;
}

.price-item.total {
  font-size: 1.1rem;
  font-weight: 600;
}

.price-value {
  color: #48bb78;
}

.submit-button {
  width: 100%;
  background: #48bb78;
  color: white;
  padding: 1rem;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background 0.3s;
}

.submit-button:hover {
  background: #48bb78;
}

.order-history {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.history-title {
  font-size: 1.25rem;
  color: #2d3748;
  margin-bottom: 1.5rem;
}

.history-card {
  border: 2px solid #f7fafc;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1rem;
}

.order-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.5rem;
}

.cinema-name {
  font-weight: 600;
  color: #2d3748;
}

.hall-name {
  color: #718096;
  font-size: 0.9rem;
}

.movie-info {
  color: #4a5568;
  margin-bottom: 0.5rem;
}

.show-time {
  color: #718096;
  font-size: 0.9rem;
  margin-bottom: 1rem;
}


```

end