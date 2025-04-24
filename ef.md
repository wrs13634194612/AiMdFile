说明：
fastapi+angular酒店预订系统


1.酒店列表
酒店名称  类型  标签  评分  地址 价格 


2.酒店详情
  2.1大床房   双床房  单间 ，每个item需要有房间信息和价格，和预定按钮  
  2.2酒店入住规则
  2.3显示用户评价  



3.预定
显示房间信息   入住时间    酒店名称 类型   入住人信息（包括姓名 电话） 提交订单按钮

4.评价
效果图:
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d403e91df8424305a27172c7f3caf545.png#pic_center)

step1:建表 添加数据，查询sql

```sql


-- 1. Hotels Table
CREATE TABLE hotels (
    hotel_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) COMMENT 'Hotel types: Business/Resort/Boutique etc.',
    tags VARCHAR(255) COMMENT 'Tags: comma-separated, e.g. WiFi,Parking,Dining',
    address VARCHAR(255),
    check_in_rules TEXT COMMENT 'Check-in policies',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Room Types Table
CREATE TABLE room_types (
    room_id INT AUTO_INCREMENT PRIMARY KEY,
    hotel_id INT NOT NULL,
    room_name VARCHAR(50) NOT NULL COMMENT 'Room type name',
    bed_type ENUM('King Bed','Twin Beds','Single Bed') NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    amenities VARCHAR(255) COMMENT 'Facilities: comma-separated',
    stock INT DEFAULT 0 COMMENT 'Total inventory',
    FOREIGN KEY (hotel_id) REFERENCES hotels(hotel_id)
);

-- 3. Users Table
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    phone VARCHAR(20) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 4. Orders Table
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    room_id INT NOT NULL,
    checkin_date DATE NOT NULL,
    checkout_date DATE NOT NULL,
    guest_name VARCHAR(50) NOT NULL,
    guest_phone VARCHAR(20) NOT NULL,
    status ENUM('Pending','Confirmed','Cancelled') DEFAULT 'Pending',
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (room_id) REFERENCES room_types(room_id)
);

-- 5. Reviews Table
CREATE TABLE reviews (
    review_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    hotel_id INT NOT NULL,
    rating TINYINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (hotel_id) REFERENCES hotels(hotel_id)
);




-- Insert Hotels
INSERT INTO hotels (name, type, tags, address, check_in_rules) VALUES
('Jinmao Grand Hyatt', 'Business', 'Meeting Rooms,Executive Lounge,Gym', '88 Century Avenue, Pudong, Shanghai', 'Check-in: 14:00, Check-out: 12:00'),
('Sanya EDITION', 'Resort', 'Private Beach,Infinity Pool,Spa', 'Haitang Bay, Sanya', 'Check-in: 15:00, Check-out: 11:00'),
('The PuLi Hotel', 'Boutique', 'Design,Artworks,Terrace', 'Gongti North Road, Chaoyang, Beijing', '24-hour check-in/out'),
('Orange Crystal', 'Business', 'Smart Rooms,Breakfast,Laundry', 'Jiaogong Road, West Lake, Hangzhou', 'Check-in: 14:00-02:00'),
('Amanfayun', 'Resort', 'Tea Plantation,SPA,Zen Meditation', 'West Lake Scenic Area, Hangzhou', 'Check-in: 13:00, Check-out: 12:00'),
('Yaduo Hotel', 'Business', 'Library,Late-night Snack,Gym', 'Chunxi Road, Jinjiang, Chengdu', 'Early check-in for members'),
('Park Hyatt', 'Business', 'Sky Bar,Swimming Pool,Observation Deck', 'Futian District, Shenzhen', 'Check-in: 15:00'),
('Club Med', 'Resort', 'Kids Club,Water Sports,All-inclusive', 'Yanshan District, Guilin', '24-hour check-in service'),
('Blossom Hill', 'Boutique', 'Ancient Town,Tea House,Courtyard', 'Pingjiang Road, Suzhou', 'Check-in: 12:00'),
('Home Inn', 'Business', 'Meeting Rooms,Self-service Laundry,Breakfast', 'Tiyu West Road, Tianhe, Guangzhou', 'Check-in: 14:00');

-- Insert Room Types
INSERT INTO room_types (hotel_id, room_name, bed_type, price, amenities, stock) VALUES
(1, 'Executive King Room', 'King Bed', 1588.00, 'Coffee Machine,BOSE Sound System,Bathtub', 15),
(1, 'Deluxe Twin Room', 'Twin Beds', 1288.00, 'Work Desk,Humidifier', 20),
(2, 'Ocean View Suite with Pool', 'King Bed', 3888.00, 'Private Pool,Butler Service', 8),
(3, 'Artist Suite', 'King Bed', 2888.00, 'Art Collections,Private Terrace', 5),
(4, 'Smart Superior Room', 'King Bed', 688.00, 'Smart Controls,Electric Curtains', 30),
(5, 'Hot Spring Villa', 'King Bed', 5888.00, 'Private Hot Spring,Courtyard', 6),
(6, 'Executive Suite', 'Twin Beds', 888.00, 'Meeting Room,Executive Floor', 12),
(7, 'Cloud View Room', 'King Bed', 2288.00, 'Floor-to-Ceiling Windows,Telescope', 18),
(8, 'Luxury Ocean View Room', 'King Bed', 2688.00, 'Balcony,Bathtub', 10),
(9, 'Courtyard View Room', 'Twin Beds', 988.00, 'Tea Set,Classical Furniture', 15);

-- Insert Users
INSERT INTO users (username, phone) VALUES
('Li Xiaoming', '13800138001'),
('Wang Fangfang', '13912345678'),
('Zhang Wei', '13611223344'),
('Chen Tingting', '13544556677'),
('Jay Chou', '18900112233'),
('Liu Yifei', '18633445566'),
('Daniel Wu', '15377889900'),
('Lin Chi-ling', '18111223344'),
('Jack Ma', '13988997766'),
('Dong Mingzhu', '13876543210');

-- Insert Orders
INSERT INTO orders (user_id, room_id, checkin_date, checkout_date, guest_name, guest_phone, status, total_amount) VALUES
(1, 1, '2025-04-01', '2025-04-03', 'Li Xiaoming', '13800138001', 'Confirmed', 3176.00),
(2, 3, '2025-03-20', '2025-03-25', 'Wang Fangfang', '13912345678', 'Pending', 19440.00),
(3, 5, '2025-05-01', '2025-05-02', 'Zhang Wei', '13611223344', 'Cancelled', 688.00),
(4, 7, '2025-03-18', '2025-03-19', 'Chen Tingting', '13544556677', 'Confirmed', 888.00),
(5, 9, '2025-06-10', '2025-06-15', 'Jay Chou', '18900112233', 'Pending', 13440.00),
(6, 2, '2025-04-20', '2025-04-21', 'Liu Yifei', '18633445566', 'Confirmed', 1288.00),
(7, 4, '2025-07-01', '2025-07-03', 'Daniel Wu', '15377889900', 'Confirmed', 5776.00),
(8, 6, '2025-08-08', '2025-08-10', 'Lin Chi-ling', '18111223344', 'Pending', 11776.00),
(9, 8, '2025-09-01', '2025-09-02', 'Jack Ma', '13988997766', 'Confirmed', 2288.00),
(10, 10, '2025-10-05', '2025-10-07', 'Dong Mingzhu', '13876543210', 'Confirmed', 1976.00);

-- Additional Orders
INSERT INTO orders (user_id, room_id, checkin_date, checkout_date, guest_name, guest_phone, status, total_amount) VALUES
(1, 2, '2025-03-18', '2025-03-20', 'Li Ming', '13812345678', 'Confirmed', 2400.00),
(1, 3, '2025-04-05', '2025-04-07', 'Li Ming', '13812345678', 'Pending', 1800.00),
(2, 1, '2025-05-01', '2025-05-03', 'Wang Fang', '13987654321', 'Cancelled', 1500.00),
(2, 4, '2025-06-10', '2025-06-12', 'Wang Fang', '13987654321', 'Confirmed', 3200.00),
(3, 5, '2025-07-15', '2025-07-17', 'Zhang Wei', '13711223344', 'Pending', 980.00),
(3, 6, '2025-08-20', '2025-08-22', 'Zhang Wei', '13711223344', 'Confirmed', 2200.00),
(4, 7, '2025-09-25', '2025-09-27', 'Chen Li', '13644556677', 'Confirmed', 2800.00),
(4, 8, '2025-10-30', '2025-11-01', 'Chen Li', '13644556677', 'Pending', 1580.00),
(5, 9, '2025-12-05', '2025-12-07', 'Zhou Jie', '13566778899', 'Confirmed', 4200.00),
(5, 10, '2026-01-10', '2026-01-12', 'Zhou Jie', '13566778899', 'Cancelled', 680.00);

-- Insert Reviews
INSERT INTO reviews (user_id, hotel_id, rating, comment) VALUES
(1, 1, 5, 'Unbeatable views and attentive service'),
(2, 2, 4, 'Great for families but limited dining options'),
(3, 4, 3, 'Soundproofing needs improvement'),
(4, 5, 5, 'Perfect zen retreat destination'),
(5, 7, 4, 'Breathtaking city skyline views'),
(6, 3, 2, 'Not worth the price'),
(7, 8, 5, 'Excellent all-inclusive resort experience'),
(8, 9, 4, 'Authentic traditional charm'),
(9, 6, 3, 'Breakfast variety could be better'),
(10, 10, 5, 'Top choice for business travel');

-- Additional Reviews
INSERT INTO reviews (user_id, hotel_id, rating, comment) VALUES
(1, 1, 5, 'Prime location with professional front desk service'),
(2, 1, 4, 'Rich breakfast options but room soundproofing could be better'),
(3, 2, 5, 'Perfect family facilities, kids loved it'),
(4, 2, 3, 'Pool water temperature was too low for comfort'),
(5, 3, 4, 'Complete business facilities with fast internet'),
(1, 3, 2, 'Room cleanliness unsatisfactory, stained bed sheets'),
(2, 4, 5, 'Spectacular ocean view with excellent butler service'),
(3, 4, 4, 'Clean beach but overpriced dining options'),
(4, 5, 5, 'Unique Republic-era charm, perfect for photos'),
(5, 5, 4, 'Average soundproofing, noticeable hallway noise');

-- Example Queries
-- 1. Hotel List Query
SELECT
    h.hotel_id,
    h.name AS hotel_name,
    h.type,
    h.tags,
    COALESCE(AVG(r.rating), 0) AS avg_rating,
    h.address,
    MIN(rt.price) AS min_price
FROM hotels h
LEFT JOIN reviews r ON h.hotel_id = r.hotel_id
LEFT JOIN room_types rt ON h.hotel_id = rt.hotel_id
GROUP BY h.hotel_id;

-- 2. Hotel Detail Query
SELECT
    room_id,
    room_name,
    bed_type,
    price,
    amenities,
    stock
FROM room_types
WHERE hotel_id = 1;

-- 2.2 Check-in Rules
SELECT check_in_rules FROM hotels WHERE hotel_id = 1;

-- 2.3 User Reviews
SELECT
    u.username,
    r.rating,
    r.comment,
    r.created_at
FROM reviews r
JOIN users u ON r.user_id = u.user_id
WHERE r.hotel_id = 1;

-- 3. Booking Information Query
SELECT
    rt.room_name,
    rt.bed_type,
    rt.price,
    o.checkin_date,
    o.checkout_date,
    h.name AS hotel_name,
    h.type AS hotel_type,
    o.guest_name,
    o.guest_phone
FROM orders o
JOIN room_types rt ON o.room_id = rt.room_id
JOIN hotels h ON rt.hotel_id = h.hotel_id
WHERE o.user_id = 1;

-- 4. Review Query
SELECT
    h.name AS hotel_name,
    u.username,
    r.rating,
    r.comment,
    r.created_at
FROM reviews r
JOIN hotels h ON r.hotel_id = h.hotel_id
JOIN users u ON r.user_id = u.user_id
WHERE r.hotel_id = 1;
```

step2:C:\Users\wangrusheng\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
import pymysql.cursors
from datetime import datetime, timedelta
from typing import List

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


def get_connection():
    return pymysql.connect(**DB_CONFIG)


class OrderCreate(BaseModel):
    user_id: int
    room_id: int
    checkin_date: str
    checkout_date: str
    guest_name: str
    guest_phone: str


# 1. 酒店列表接口
@app.get("/hotels", response_model=List[dict])
async def get_hotels():
    try:
        with get_connection() as conn:
            with conn.cursor() as cursor:
                sql = """
                SELECT 
                    h.hotel_id,
                    h.name AS hotel_name,
                    h.type,
                    h.tags,
                    h.address,
                    COALESCE(AVG(r.rating), 0) AS rating,
                    MIN(rt.price) AS min_price
                FROM hotels h
                LEFT JOIN reviews r ON h.hotel_id = r.hotel_id
                LEFT JOIN room_types rt ON h.hotel_id = rt.hotel_id
                GROUP BY h.hotel_id
                """
                cursor.execute(sql)
                results = cursor.fetchall()

                # 处理标签格式
                for hotel in results:
                    hotel['tags'] = hotel['tags'].split(',') if hotel['tags'] else []
                    hotel['price'] = hotel['min_price']
                    del hotel['min_price']

                return results
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


# 修改后的酒店详情接口
@app.get("/hotels/{hotel_id}")
async def get_hotel_detail(hotel_id: int):
    try:
        with get_connection() as conn:
            with conn.cursor() as cursor:
                # 获取酒店基本信息
                cursor.execute("""
                    SELECT 
                        h.name AS hotel_name,
                        h.address,
                        h.check_in_rules,
                        h.type AS hotel_type
                    FROM hotels h
                    WHERE h.hotel_id = %s
                """, (hotel_id,))
                hotel = cursor.fetchone()

                if not hotel:
                    raise HTTPException(status_code=404, detail="Hotel not found")

                # 获取房型信息（简化字段）
                cursor.execute("""
                    SELECT 
                        bed_type AS room_type,
                        room_name,
                        price
                    FROM room_types
                    WHERE hotel_id = %s
                """, (hotel_id,))
                rooms = cursor.fetchall()

                # 获取评价信息（简化字段）
                cursor.execute("""
                    SELECT 
                        comment,
                        rating
                    FROM reviews
                    WHERE hotel_id = %s
                    ORDER BY created_at DESC
                """, (hotel_id,))
                reviews = cursor.fetchall()

                # 合并所有信息到单个JSON对象
                return {
                    "hotel_name": hotel['hotel_name'],
                    "address": hotel['address'],
                    "check_in_rules": hotel['check_in_rules'],
                    "rooms": [
                        {
                            "type": room['room_type'],
                            "name": room['room_name'],
                            "price": float(room['price'])
                        } for room in rooms
                    ],
                    "reviews": [
                        {
                            "comment": review['comment'],
                            "rating": review['rating']
                        } for review in reviews
                    ]
                }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# 3. 创建订单接口
@app.post("/orders")
async def create_order(order: OrderCreate):
    connection = None
    try:
        connection = get_connection()
        with connection.cursor() as cursor:
            # 验证用户存在
            cursor.execute("SELECT user_id FROM users WHERE user_id = %s", (order.user_id,))
            if not cursor.fetchone():
                raise HTTPException(status_code=404, detail="User not found")

            # 获取房型信息
            cursor.execute("""
                SELECT price, stock, hotel_id 
                FROM room_types 
                WHERE room_id = %s
            """, (order.room_id,))
            room = cursor.fetchone()
            if not room:
                raise HTTPException(status_code=404, detail="Room type not found")

            # 计算入住天数
            checkin_date = datetime.strptime(order.checkin_date, "%Y-%m-%d").date()
            checkout_date = datetime.strptime(order.checkout_date, "%Y-%m-%d").date()
            if checkout_date <= checkin_date:
                raise HTTPException(status_code=400, detail="Invalid date range")
            days = (checkout_date - checkin_date).days

            # 验证库存
            if room['stock'] < 1:
                raise HTTPException(status_code=400, detail="Insufficient stock")

            # 计算总金额
            total_amount = room['price'] * days

            # 开始事务
            connection.begin()

            # 更新库存
            cursor.execute("""
                UPDATE room_types 
                SET stock = stock - 1 
                WHERE room_id = %s AND stock > 0
            """, (order.room_id,))

            if cursor.rowcount == 0:
                connection.rollback()
                raise HTTPException(status_code=400, detail="Stock update failed")

            # 创建订单
            cursor.execute("""
                INSERT INTO orders (
                    user_id, room_id, checkin_date, checkout_date,
                    guest_name, guest_phone, total_amount
                ) VALUES (%s, %s, %s, %s, %s, %s, %s)
            """, (
                order.user_id, order.room_id, checkin_date, checkout_date,
                order.guest_name, order.guest_phone, total_amount
            ))

            connection.commit()
            return {"message": "Order created successfully"}

    except Exception as e:
        if connection:
            connection.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        if connection:
            connection.close()


# 4. 获取评价接口
@app.get("/hotels/{hotel_id}/reviews")
async def get_reviews(hotel_id: int):
    try:
        with get_connection() as conn:
            with conn.cursor() as cursor:
                cursor.execute("""
                    SELECT u.username, r.rating, r.comment, r.created_at
                    FROM reviews r
                    JOIN users u ON r.user_id = u.user_id
                    WHERE r.hotel_id = %s
                    ORDER BY r.created_at DESC
                """, (hotel_id,))
                reviews = cursor.fetchall()
                return {"reviews": reviews}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:验证接口，postman

```bash
// GET http://localhost:8000/hotels
[
    {
        "hotel_id": 1,
        "hotel_name": "Jinmao Grand Hyatt",
        "type": "Business",
        "tags": [
            "Meeting Rooms",
            "Executive Lounge",
            "Gym"
        ],
        "address": "88 Century Avenue, Pudong New District, Shanghai",
        "rating": "5.0000",
        "price": "1288.00"
    },
    {
        "hotel_id": 2,
        "hotel_name": "Sanya EDITION",
        "type": "Resort",
        "tags": [
            "Private Beach",
            "Infinity Pool",
            "Spa"
        ],
        "address": "Haitang Bay, Sanya",
        "rating": "4.0000",
        "price": "3888.00"
    },
    {
        "hotel_id": 3,
        "hotel_name": "The PuLi Hotel",
        "type": "Boutique",
        "tags": [
            "Design Aesthetic",
            "Art Collection",
            "Terrace"
        ],
        "address": "Gongti North Road, Chaoyang District, Beijing",
        "rating": "2.0000",
        "price": "2888.00"
    },
    {
        "hotel_id": 4,
        "hotel_name": "Orange Crystal",
        "type": "Business",
        "tags": [
            "Smart Rooms",
            "Breakfast",
            "Laundry"
        ],
        "address": "Jiaogong Road, West Lake District, Hangzhou",
        "rating": "3.0000",
        "price": "688.00"
    },
    {
        "hotel_id": 5,
        "hotel_name": "Amanfayun",
        "type": "Resort",
        "tags": [
            "Tea Plantation",
            "SPA",
            "Zen Meditation"
        ],
        "address": "West Lake Scenic Area, Hangzhou",
        "rating": "5.0000",
        "price": "5888.00"
    },
    {
        "hotel_id": 6,
        "hotel_name": "Yaduo Hotel",
        "type": "Business",
        "tags": [
            "Library",
            "Late-night Snack",
            "Gym"
        ],
        "address": "Chunxi Road, Jinjiang District, Chengdu",
        "rating": "3.0000",
        "price": "888.00"
    },
    {
        "hotel_id": 7,
        "hotel_name": "Park Hyatt",
        "type": "Business",
        "tags": [
            "Sky Bar",
            "Swimming Pool",
            "Observation Deck"
        ],
        "address": "Futian District, Shenzhen",
        "rating": "4.0000",
        "price": "2288.00"
    },
    {
        "hotel_id": 8,
        "hotel_name": "Club Med",
        "type": "Resort",
        "tags": [
            "Kids Club",
            "Water Sports",
            "All-inclusive"
        ],
        "address": "Yanshan District, Guilin",
        "rating": "5.0000",
        "price": "2688.00"
    },
    {
        "hotel_id": 9,
        "hotel_name": "Blossom Hill",
        "type": "Boutique",
        "tags": [
            "Ancient Town",
            "Tea House",
            "Courtyard"
        ],
        "address": "Pingjiang Road, Suzhou",
        "rating": "4.0000",
        "price": "988.00"
    },
    {
        "hotel_id": 10,
        "hotel_name": "Home Inn",
        "type": "Business",
        "tags": [
            "Meeting Rooms",
            "Self-service Laundry",
            "Breakfast"
        ],
        "address": "Tiyu West Road, Tianhe District, Guangzhou",
        "rating": "5.0000",
        "price": null
    }
]

// POST http://localhost:8000/orders
Request:
{
  "user_id": 1,
  "room_id": 1,
  "checkin_date": "2024-03-01",
  "checkout_date": "2024-03-03",
  "guest_name": "Zhang San",
  "guest_phone": "13800138000"
}

Response:
{
    "message": "Order created successfully"
}

// GET http://localhost:8000/hotels/1
{
    "hotel_name": "Jinmao Grand Hyatt",
    "address": "88 Century Avenue, Pudong New District, Shanghai",
    "check_in_rules": "Check-in: 14:00, Check-out: 12:00",
    "rooms": [
        {
            "type": "King Bed",
            "name": "Executive King Room",
            "price": 1588.0
        },
        {
            "type": "Twin Beds",
            "name": "Deluxe Twin Room",
            "price": 1288.0
        }
    ],
    "reviews": [
        {
            "comment": "Prime location with professional front desk service",
            "rating": 5
        },
        {
            "comment": "Rich breakfast variety but room soundproofing needs improvement",
            "rating": 4
        },
        {
            "comment": "Unbeatable views with attentive service",
            "rating": 5
        }
    ]
}
```

step4:后端和mysql写完了 接下来 写angular
路由

```bash
 C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.component.html

   <a routerLink="/hotel-list" routerLinkActive="active">酒店列表</a>
      <a routerLink="/hotel-detail" routerLinkActive="active">酒店详情</a>
      <a routerLink="/hotel-order" routerLinkActive="active">酒店预约</a>
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.routes.ts

  { path: 'hotel-list', component: HotelListComponent },
  { path: 'hotel-detail/:id', component: HotelDetailComponent },
  {
    path: 'hotel-order/:hotelId/:roomId',
    component: HotelOrderComponent
  },


```

step5:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\hotel-list\hotel-list.component.css

```typescript
// hotel-list.component.ts
import { Component ,OnInit} from '@angular/core';
import { HotelService } from '../services/hotel.service';

import {Router, RouterLink} from '@angular/router';
import {DecimalPipe, NgForOf} from '@angular/common';

@Component({
  selector: 'app-hotel-list',
  imports: [
    NgForOf,
    RouterLink,
    DecimalPipe
  ],
  templateUrl: './hotel-list.component.html',
  styleUrl: './hotel-list.component.css'

})
export class HotelListComponent implements OnInit {
  hotels: any[] = [];

  constructor(private service: HotelService, private router: Router) {}

  ngOnInit() {
    this.service.getHotels().subscribe(data => this.hotels = data);
  }
  // 新增星级评分方法
  getStars(rating: string): number[] {
    const numericRating = parseFloat(rating);
    return Array(Math.floor(numericRating)).fill(0);
  }

}



<!-- hotel-list.component.html -->
<div class="hotel-list-container">
  <h1 class="section-title">探索精品酒店</h1>

  <div class="hotel-grid">
    <div *ngFor="let hotel of hotels" class="hotel-card">
      <div class="hotel-header">
        <div class="hotel-badge">{{hotel.type}}</div>
        <h2 class="hotel-name">{{hotel.hotel_name}}</h2>
        <div class="rating">
          <span *ngFor="let star of getStars(hotel.rating)">★</span>
        </div>
      </div>

      <div class="hotel-body">
        <div class="info-item">
          <i class="icon icon-location"></i>
          <span>{{hotel.address}}</span>
        </div>
        <div class="tag-group">
          <span *ngFor="let tag of hotel.tags" class="tag">{{tag}}</span>
        </div>
      </div>

      <div class="hotel-footer">
        <div class="price">¥{{hotel.price | number:'1.0-0'}}起</div>
        <button class="primary-btn" [routerLink]="['/hotel-detail', hotel.hotel_id]">
          查看详情
        </button>
      </div>
    </div>
  </div>
</div>


/* hotel-list.component.css */
.hotel-list-container {
  padding: 2rem 5%;
  background: #f8f9fa;
  min-height: 100vh;
}

.section-title {
  font-size: 2rem;
  color: #2d3436;
  margin-bottom: 2rem;
  font-weight: 600;
  text-align: center;
}

.hotel-grid {
  display: grid;
  gap: 1.5rem;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
}

.hotel-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s, box-shadow 0.2s;
  overflow: hidden;
}

.hotel-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.1);
}

.hotel-header {
  padding: 1.5rem;
  background: linear-gradient(135deg, #6c5ce7 0%, #a8a4e6 100%);
  color: white;
  position: relative;
}

.hotel-badge {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: rgba(255, 255, 255, 0.2);
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.875rem;
}

.hotel-name {
  font-size: 1.5rem;
  margin: 0 0 1rem;
  font-weight: 600;
}

.rating {
  display: flex;
  gap: 0.25rem;
  color: #ffd700;
}

.hotel-body {
  padding: 1.5rem;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 1rem;
  color: #636e72;
}

.icon {
  width: 20px;
  height: 20px;
  background-size: contain;
}

.icon-location {
  background: #f1f3f5;
}

.tag-group {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tag {
  background: #f1f3f5;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.875rem;
}

.hotel-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.5rem;
  border-top: 1px solid #f1f3f5;
}

.price {
  font-size: 1.25rem;
  color: #2d3436;
  font-weight: 600;
}

.primary-btn {
  background: #6c5ce7;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.2s;
}

.primary-btn:hover {
  background: #5b4bc4;
}

```

step6:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\hotel-detail\hotel-detail.component.ts

```typescript
// hotel-detail.component.ts
import { Component ,OnInit} from '@angular/core';
import { HotelService } from '../services/hotel.service';

import {ActivatedRoute, RouterLink} from '@angular/router';
import {DecimalPipe, NgForOf} from '@angular/common';

@Component({
  selector: 'app-hotel-detail',
  imports: [
    RouterLink,
    NgForOf,
    DecimalPipe
  ],
  templateUrl: './hotel-detail.component.html',
  styleUrl: './hotel-detail.component.css'
})
export class HotelDetailComponent implements OnInit {
  hotel: any;

  constructor(
    private service: HotelService,
    private route: ActivatedRoute
  ) {}

  ngOnInit() {
    this.route.params.subscribe(params => {
      this.service.getHotelDetails(params['id']).subscribe(data => {
        this.hotel = data;
        // 添加room_id到房型数据（根据实际接口调整）
        this.hotel.rooms = data.rooms.map((room: any, index: number) => ({
          ...room,
          room_id: index + 1 // 示例生成room_id，实际需要根据接口数据调整
        }));
      });
    });
  }
}
<!-- hotel-detail.component.html -->
<div class="hotel-detail-container">
  <button class="back-btn" routerLink="/hotel-list">
    ← 返回列表
  </button>

  <div class="hotel-main">
    <div class="hotel-info">
      <h1 class="hotel-title">{{hotel?.hotel_name}}</h1>
      <div class="location">
        <i class="icon icon-location"></i>
        <span>{{hotel?.address}}</span>
      </div>
    </div>

    <section class="rules-section">
      <h2 class="section-subtitle">入住须知</h2>
      <div class="rules-content">
        {{hotel?.check_in_rules}}
      </div>
    </section>

    <section class="room-section">
      <h2 class="section-subtitle">选择房型</h2>
      <div class="room-grid">
        <div *ngFor="let room of hotel?.rooms" class="room-card">
          <div class="room-header">
            <h3 class="room-type">{{room.type}}</h3>
            <div class="room-price">¥{{room.price | number:'1.0-0'}}/晚</div>
          </div>
          <div class="room-body">
            <div class="room-name">{{room.name}}</div>
            <ul class="amenities-list">
              <li *ngFor="let amenity of room.amenities?.split(',')">
                {{amenity}}
              </li>
            </ul>
          </div>
          <button class="book-btn" [routerLink]="['/hotel-order', hotel.hotel_id, room.room_id]">
            立即预订
          </button>
        </div>
      </div>
    </section>

    <section class="reviews-section">
      <h2 class="section-subtitle">用户评价（{{hotel?.reviews?.length}}）</h2>
      <div class="review-list">
        <div *ngFor="let review of hotel?.reviews" class="review-card">
          <div class="review-header">
            <div class="rating-stars">
              <span *ngFor="let star of [1,2,3,4,5]">
                {{ review.rating >= star ? '★' : '☆' }}
              </span>
            </div>
            <div class="review-rating">{{review.rating}}/5</div>
          </div>
          <p class="review-content">{{review.comment}}</p>
        </div>
      </div>
    </section>
  </div>
</div>
/* hotel-detail.component.css */
.hotel-detail-container {
  padding: 2rem 5%;
  background: #f8f9fa;
  min-height: 100vh;
}

.back-btn {
  background: none;
  border: none;
  color: #6c5ce7;
  font-size: 1rem;
  cursor: pointer;
  margin-bottom: 2rem;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  transition: background 0.2s;
}

.back-btn:hover {
  background: rgba(108, 92, 231, 0.1);
}

.hotel-main {
  max-width: 1200px;
  margin: 0 auto;
}

.hotel-info {
  text-align: center;
  margin-bottom: 3rem;
}

.hotel-title {
  font-size: 2.5rem;
  color: #2d3436;
  margin-bottom: 1rem;
}

.location {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  color: #636e72;
}

.section-subtitle {
  font-size: 1.5rem;
  color: #2d3436;
  margin-bottom: 1.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #6c5ce7;
}

.rules-section {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  margin-bottom: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.rules-content {
  line-height: 1.6;
  color: #636e72;
  white-space: pre-wrap;
}

.room-grid {
  display: grid;
  gap: 1.5rem;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
}

.room-card {
  background: white;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.room-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.room-type {
  font-size: 1.25rem;
  color: #2d3436;
  margin: 0;
}

.room-price {
  font-size: 1.25rem;
  color: #6c5ce7;
  font-weight: 600;
}

.amenities-list {
  list-style: none;
  padding: 0;
  margin: 1rem 0;
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.amenities-list li {
  background: #f1f3f5;
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  font-size: 0.875rem;
}

.book-btn {
  width: 100%;
  margin-top: 1rem;
}

.review-list {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.review-card {
  padding: 1.5rem;
  margin-bottom: 1rem;
  background: #fff;
  border-radius: 8px;
  border-left: 4px solid #6c5ce7;
}

.review-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.rating-stars {
  color: #ffd700;
  font-size: 1.25rem;
}

.review-rating {
  color: #636e72;
  font-weight: 500;
}

.review-content {
  color: #2d3436;
  line-height: 1.6;
  margin: 0;
}

```

step7: C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\hotel-order\hotel-order.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { MatSnackBar } from '@angular/material/snack-bar';
import { Router, ActivatedRoute } from '@angular/router';
import { HotelService } from '../services/hotel.service';

@Component({
  selector: 'app-booking-form',
  imports: [
    ReactiveFormsModule
  ],
  templateUrl: './hotel-order.component.html',
  styleUrl: './hotel-order.component.css'
})
export class HotelOrderComponent implements OnInit {
  bookingForm!: FormGroup;
  hotelId!: number;
  roomId!: number;

  constructor(
    private fb: FormBuilder,
    private hotelService: HotelService,
    private snackBar: MatSnackBar,
    private router: Router,
    private route: ActivatedRoute
  ) {}

  ngOnInit(): void {
    this.route.params.subscribe(params => {
      this.hotelId = +params['hotelId'];
      this.roomId = +params['roomId'];
    });
    this.initializeForm();
  }

  private initializeForm(): void {
    this.bookingForm = this.fb.group({
      checkin_date: ['', Validators.required],
      checkout_date: ['', Validators.required],
      guest_name: ['', [Validators.required, Validators.minLength(2)]],
      guest_phone: ['', [Validators.required, Validators.pattern(/^1[3-9]\d{9}$/)]]
    });
  }

  submitBooking() {
    if (this.bookingForm.valid) {
      const formData = {
        ...this.bookingForm.value,
        user_id: 1, // 根据实际用户系统修改
        room_id: this.roomId,
        hotel_id: this.hotelId
      };

      // 转换日期格式（如果需要）
      formData.checkin_date = this.formatDate(formData.checkin_date);
      formData.checkout_date = this.formatDate(formData.checkout_date);



      this.hotelService.createOrder(formData).subscribe({
        next: () => {
          alert('Booking successful!');
        },
        error: (err) => {
          console.error('预订失败:', err);
        }
      });
    }
  }

  private formatDate(dateString: string): string {
    // 如果需要特定日期格式可以在这里转换
    return new Date(dateString).toISOString().split('T')[0];
  }

  onCancel() {
    this.router.navigate(['/hotels']);
  }
}
<!-- hotel-order.component.html -->
<div class="booking-container">
  <div class="booking-card">
    <h2 class="booking-title">完成预订</h2>

    <form [formGroup]="bookingForm" (ngSubmit)="submitBooking()">
      <div class="form-group">
        <label>入住日期</label>
        <input type="date" formControlName="checkin_date">
      </div>

      <div class="form-group">
        <label>退房日期</label>
        <input type="date" formControlName="checkout_date">
      </div>

      <div class="form-group">
        <label>入住人姓名</label>
        <input type="text" formControlName="guest_name" placeholder="请输入真实姓名">
      </div>

      <div class="form-group">
        <label>联系电话</label>
        <input type="tel" formControlName="guest_phone" placeholder="11位手机号码">
      </div>

      <div class="action-buttons">
        <button type="button" class="secondary-btn" (click)="onCancel()">取消</button>
        <button type="submit" class="primary-btn" [disabled]="!bookingForm.valid">提交订单</button>
      </div>
    </form>
  </div>
</div>
/* hotel-order.component.css */
.booking-container {
  padding: 4rem 5%;
  min-height: 100vh;
  background: #f8f9fa;
  display: flex;
  justify-content: center;
}

.booking-card {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 600px;
}

.booking-title {
  font-size: 1.75rem;
  color: #2d3436;
  margin-bottom: 2rem;
  text-align: center;
}

.form-group {
  margin-bottom: 1.5rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
  color: #2d3436;
  font-weight: 500;
}

input {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #dee2e6;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.2s;
}

input:focus {
  outline: none;
  border-color: #6c5ce7;
  box-shadow: 0 0 0 3px rgba(108, 92, 231, 0.1);
}

.action-buttons {
  display: flex;
  gap: 1rem;
  margin-top: 2rem;
}

.primary-btn {
  flex: 1;
}

.secondary-btn {
  flex: 1;
  background: #f1f3f5;
  color: #2d3436;
}

.secondary-btn:hover {
  background: #dee2e6;
}

```

end