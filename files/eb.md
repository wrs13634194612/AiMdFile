说明：
fastapi+angular外卖系统


1.美食分类（粥，粉，面，炸鸡，炒菜，西餐，奶茶等等）

2.商家列表 （kfc，兰州拉面，湘菜馆，早餐店，重庆小面，潮汕砂锅粥，蜜雪冰城等等）

商家item：
商家店名，评分，月销量，人均价格，起送价格，配送费价格，店铺位置，商家标签，商家分类



3.商家详情页



商家店名，评分，月销量 

菜品分类（比如炒饭，拉面，盖饭，单人套餐，双人套餐，米线，酒水饮料）

菜品列表
菜品item：
菜品名，评分，月销量，菜品标签，菜品分类，菜品价格

显示评价列表，包括用户评分和评价内容

商家详情，包括店铺位置，商家联系电话，营业时间，订餐注意事项，店铺政策

4.结算页

填写用户名，用户联系电话，收货地址，
商品详细清单 店铺名，菜品名，购买数量，菜品单价，配送费，打包费，总金额，提交订单功能

5.订单详情页

订单单号，订单状态（已完成，已退款，待接单，待配送），店铺名称，菜品名称，购买数量，菜品单价，配送费，打包费，总金额 

配送信息，订单下单时间，配送完成时间，配送地址，骑手名，骑手联系电话，支付方式，备注信息等等


6.评价表
用户对菜品的评价
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/b4d353275db74968a3391f2400ddd11c.png#pic_center)
json:

```bash
/* Merchant List Endpoint - GET http://localhost:8000/merchants */
{
    "data": [
        {
            "id": 7,
            "name": "Haidilao Hot Pot",
            "rating": 4.9,
            "monthly_sales": 4000,
            "avg_price": 80.0,
            "min_order_price": 50.0,
            "delivery_fee": 0.0,
            "location": "3F Food Court",
            "tags": "Hot Pot,24/7",
            "contact_phone": "13800138444",
            "business_hours": "24/7",
            "notice": "Free delivery",
            "policy": "Service first",
            "categories": "Hot Pot"
        },
        // Other merchants follow similar pattern...
    ]
}

/* Food Categories Endpoint - GET http://localhost:8000/categories */
{
    "data": [
        {"id": 7, "name": "Bubble Tea"},
        {"id": 10, "name": "Japanese"},
        {"id": 8, "name": "Hot Pot"},
        // Other categories...
    ]
}

/* Merchant Details - GET http://localhost:8000/merchants/1 */
{
    "merchant": {
        "id": 1,
        "name": "KFC",
        "rating": 4.5,
        "monthly_sales": 2000,
        "avg_price": 35.0,
        "min_order_price": 20.0,
        "delivery_fee": 5.0,
        "location": "People's Square, City Center",
        "tags": "Fast Food,24/7",
        "contact_phone": "13800138000",
        "business_hours": "07:00-23:00",
        "notice": "Order during business hours",
        "policy": "Unconditional refund"
    },
    "categories": [
        {
            "id": 1,
            "name": "Fried Chicken Meals",
            "dishes": "[{\"id\": 1, \"name\": \"Spicy Chicken Burger Meal\", \"tags\": \"Popular\", \"price\": 35.00, \"sales\": 800, \"rating\": 4.50}]"
        }
    ],
    "reviews": [
        {
            "rating": 4.8,
            "content": "The burger was delicious!",
            "review_time": "2024-03-01T13:00:00",
            "dish_name": "Spicy Chicken Burger Meal"
        }
    ]
}

/* Create Order - POST http://localhost:8000/orders */
Request Body:
{
    "customer_name": "Wang Xiaoming",
    "customer_phone": "13800138000",
    "delivery_address": "No.123 Zhangjiang Road, Pudong, Shanghai",
    "merchant_id": 1,
    "payment_method": "Alipay",
    "items": [
        {"dish_id": 1, "quantity": 2, "price": 35.00},
        {"dish_id": 2, "quantity": 1, "price": 40.00}
    ],
    "note": "Need invoice, no chili please"
}

Response:
{"order_id": 14}

/* Order Details - GET http://localhost:8000/orders/1 */
{
    "order": {
        "id": 1,
        "customer_name": "Zhang San",
        "status": "Completed",
        "payment_method": "WeChat Pay",
        "merchant_name": "KFC"
    },
    "items": [
        {"dish_name": "Spicy Chicken Burger Meal", "quantity": 2},
        {"dish_name": "Orleans Wings Meal", "quantity": 1}
    ]
}

/* Dish Reviews - GET http://localhost:8000/dishes/1/reviews */
[
    {
        "rating": 4.8,
        "content": "Perfectly crispy chicken!",
        "review_time": "2024-03-01T13:00:00",
        "quantity": 2
    }
]
```

网络请求： C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\services\merchant.service.ts

```typescript
// merchant.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class MerchantService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  getCategories(): Observable<any> {
    return this.http.get(`${this.apiUrl}/categories`);
  }

  getMerchants(categoryId?: number, page = 1, sortBy = 'rating'): Observable<any> {
    const params: any = {
      page: page.toString(),
      sort_by: sortBy
    };
    if (categoryId) params.category_id = categoryId.toString();

    return this.http.get(`${this.apiUrl}/merchants`, { params });
  }

  getMerchantDetail(id: number): Observable<any> {
    console.log("getMerchantDetail",id)
    return this.http.get(`${this.apiUrl}/merchants/${id}`);
  }

  createOrder(orderData: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/orders`, orderData);
  }

  getOrderDetail(id: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/orders/${id}`);
  }

  submitReview(reviewData: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/reviews`, reviewData);
  }
}

```

step1:sql

```sql

-- 1. 美食分类表
CREATE TABLE category (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE
);

select *from category

-- 2. 商家表
CREATE TABLE merchant (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    rating DECIMAL(3,2) DEFAULT 0.00,
    monthly_sales INT DEFAULT 0,
    avg_price DECIMAL(8,2) DEFAULT 0.00,
    min_order_price DECIMAL(8,2) DEFAULT 0.00,
    delivery_fee DECIMAL(8,2) DEFAULT 0.00,
    location VARCHAR(255),
    tags VARCHAR(255),
    contact_phone VARCHAR(20),
    business_hours VARCHAR(100),
    notice TEXT,
    policy TEXT
);

-- 商家分类关联表
CREATE TABLE merchant_category (
    merchant_id INT,
    category_id INT,
    PRIMARY KEY (merchant_id, category_id),
    FOREIGN KEY (merchant_id) REFERENCES merchant(id),
    FOREIGN KEY (category_id) REFERENCES category(id)
);

-- 3. 菜品分类表
CREATE TABLE dish_category (
    id INT PRIMARY KEY AUTO_INCREMENT,
    merchant_id INT NOT NULL,
    name VARCHAR(50) NOT NULL,
    FOREIGN KEY (merchant_id) REFERENCES merchant(id)
);

-- 4. 菜品表
CREATE TABLE dish (
    id INT PRIMARY KEY AUTO_INCREMENT,
    merchant_id INT NOT NULL,
    category_id INT NOT NULL,
    name VARCHAR(100) NOT NULL,
    rating DECIMAL(3,2) DEFAULT 0.00,
    monthly_sales INT DEFAULT 0,
    tags VARCHAR(255),
    price DECIMAL(8,2) NOT NULL,
    FOREIGN KEY (merchant_id) REFERENCES merchant(id),
    FOREIGN KEY (category_id) REFERENCES dish_category(id)
);

-- 5. 订单表
CREATE TABLE `order` (
    id INT PRIMARY KEY AUTO_INCREMENT,
    customer_name VARCHAR(100) NOT NULL,
    customer_phone VARCHAR(20) NOT NULL,
    delivery_address VARCHAR(255) NOT NULL,
    merchant_id INT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    delivery_fee DECIMAL(8,2) DEFAULT 0.00,
    packing_fee DECIMAL(8,2) DEFAULT 0.00,
    status ENUM('已完成','已退款','待接单','待配送') DEFAULT '待接单',
    order_time DATETIME NOT NULL,
    complete_time DATETIME,
    rider_name VARCHAR(100),
    rider_phone VARCHAR(20),
    payment_method VARCHAR(50),
    note TEXT,
    FOREIGN KEY (merchant_id) REFERENCES merchant(id)
);


select *from db_school.order;

-- 6. 订单明细表
CREATE TABLE order_item (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    dish_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(8,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES `order`(id),
    FOREIGN KEY (dish_id) REFERENCES dish(id)
);

-- 7. 评价表
CREATE TABLE review (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_item_id INT NOT NULL UNIQUE,
    rating DECIMAL(3,2) NOT NULL,
    content TEXT,
    review_time DATETIME NOT NULL,
    FOREIGN KEY (order_item_id) REFERENCES order_item(id)
);

-- Insert food categories
INSERT INTO category (name) VALUES
('Porridge'),('Rice Noodles'),('Noodles'),('Fried Chicken'),('Stir-Fry'),('Western Food'),('Bubble Tea');

INSERT INTO category (name) VALUES
('Hot Pot'), ('BBQ'), ('Japanese Cuisine');


-- Insert merchant data
INSERT INTO merchant (name, rating, monthly_sales, avg_price, min_order_price, delivery_fee, location, tags, contact_phone, business_hours, notice, policy) VALUES
('KFC', 4.5, 2000, 35.00, 20.00, 5.00, 'People''s Square, City Center', 'Fast Food,24/7 Service', '13800138000', '07:00-23:00', 'Please order during business hours', 'Unconditional refund'),
('Lanzhou Beef Noodles', 4.7, 1500, 20.00, 15.00, 3.00, '100 Zhongshan Road', 'Noodles,Halal', '13900139000', '08:00-22:00', 'Advance reservation recommended', 'Free packaging'),
('Mixue Ice Cream & Tea', 4.6, 3000, 10.00, 0.00, 0.00, '50 Jiefang Road', 'Bubble Tea,Desserts', '13700137000', '09:00-22:00', 'Minimum order ¥10', 'Made fresh');

-- Additional merchants...
INSERT INTO merchant (name, rating, monthly_sales, avg_price, min_order_price, delivery_fee, location, tags, contact_phone, business_hours, notice, policy) VALUES
('McDonald''s', 4.6, 2500, 30.00, 20.00, 4.00, '5 Commercial Street', 'Fast Food,Burgers', '13800138111', '07:00-23:30', '24/7 delivery', 'Coupons accepted'),
('Pizza Hut', 4.4, 1800, 50.00, 30.00, 6.00, 'City Center Plaza', 'Pizza,Western Food', '13800138222', '10:00-22:00', 'Happy Hour deals', 'Utensils provided'),
('Starbucks', 4.7, 3000, 35.00, 0.00, 5.00, '1F Shopping Mall', 'Coffee,Desserts', '13800138333', '08:00-21:00', 'Takeaway discounts', 'Eco-cup discount'),
('Haidilao Hot Pot', 4.9, 4000, 80.00, 50.00, 0.00, '3F Food Court', 'Hot Pot,24/7', '13800138444', '24/7', 'Free delivery', 'Service first'),
('Shaxian Snacks', 4.3, 1200, 15.00, 10.00, 2.00, 'Community Street', 'Snacks,Economy', '13800138555', '06:30-22:00', 'Large portions', 'Free rice refill'),
('Zhen Gongfu', 4.5, 1500, 25.00, 15.00, 3.00, 'Near Train Station', 'Fast Food,Steamed Dishes', '13800138666', '09:00-21:00', 'Quick service', 'Self-pickup available'),
('Burger King', 4.6, 2000, 40.00, 25.00, 5.00, 'Block B Commercial Area', 'Burgers,Fries', '13800138777', '08:00-24:00', 'King Meal deals', 'Reward points');

-- Special noodle merchants
INSERT INTO merchant (name, rating, monthly_sales, avg_price, min_order_price, delivery_fee, location, tags, contact_phone, business_hours, notice, policy)
VALUES (
    'Liuzhou Luosifen',
    4.8,
    1800,
    18.00,
    15.00,
    2.50,
    'No.8 Food Street',
    'Luosifen,Authentic',
    '13811112222',
    '09:00-22:00',
    'Customizable spiciness',
    'Free extra bamboo shoots'
);

-- Link categories (assuming Rice Noodles category_id=2)
INSERT INTO merchant_category (merchant_id, category_id)
VALUES
    ((SELECT id FROM merchant WHERE name='Liuzhou Luosifen'), 2),
    ((SELECT id FROM merchant WHERE name='Nanning Old Friend Noodles'), 2);

-- Insert dish categories
INSERT INTO dish_category (merchant_id, name) VALUES
(1,'Fried Chicken Meals'),(1,'Burger Meals'),
(2,'Signature Noodles'),(2,'Rice Bowls'),
(3,'Classic Bubble Tea'),(3,'Summer Specials');

-- Insert dishes
INSERT INTO dish (merchant_id, category_id, name, rating, monthly_sales, tags, price) VALUES
(1,1,'Spicy Chicken Burger Meal',4.5,800,'Popular',35.00),
(1,2,'Orleans Wings Meal',4.6,600,'New',40.00),
(2,3,'Beef Noodles',4.8,500,'Signature',18.00),
(2,4,'Braised Beef Rice Bowl',4.7,300,'Recommended',22.00),
(3,5,'Pearl Milk Tea',4.7,1500,'Classic',8.00),
(3,6,'Strawberry Sundae',4.8,1000,'Seasonal',6.00);

-- Insert orders
INSERT INTO `order` (customer_name, customer_phone, delivery_address, merchant_id, total_amount, delivery_fee, packing_fee, status, order_time, payment_method) VALUES
('Zhang San','13800001111','Tech Park Building 1',1,77.00,5.00,2.00,'Completed','2024-03-01 12:00:00','WeChat Pay'),
('Li Si','13900002222','University Dorm 3',2,40.00,3.00,1.00,'Pending Delivery','2024-03-02 18:30:00','Alipay');

-- Insert reviews
INSERT INTO review (order_item_id, rating, content, review_time) VALUES
(1,4.8,'Burgers were delicious!','2024-03-01 13:00:00'),
(3,4.9,'Very authentic noodles','2024-03-02 19:00:00');
```

step2:fastapi

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import pymysql.cursors
from typing import List, Optional
from pydantic import BaseModel
from datetime import datetime
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

# 基础数据模型
class Category(BaseModel):
    id: int
    name: str

class MerchantBase(BaseModel):
    name: str
    rating: float
    monthly_sales: int
    avg_price: float

# API 请求模型
class CreateOrderItem(BaseModel):
    dish_id: int
    quantity: int
    price: float

class CreateOrder(BaseModel):
    customer_name: str
    customer_phone: str
    delivery_address: str
    merchant_id: int
    items: List[CreateOrderItem]
    payment_method: str
    note: Optional[str] = None

def db_query(query: str, params=None, fetch_one=False):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchone() if fetch_one else cursor.fetchall()
        connection.commit()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))



# 1. 获取所有美食分类
@app.get("/categories")
async def get_categories():
    """获取所有美食分类"""
    query = "SELECT id, name FROM category"
    return {"data": db_query(query)}


# 2. 商家列表接口
@app.get("/merchants")
async def get_merchants(
        category_id: Optional[int] = None,
        page: int = 1,
        page_size: int = 10,
        sort_by: str = 'rating'
):
    """获取商家列表（支持分类筛选和排序）"""
    offset = (page - 1) * page_size
    base_query = """
        SELECT m.*, GROUP_CONCAT(c.name) AS categories 
        FROM merchant m
        LEFT JOIN merchant_category mc ON m.id = mc.merchant_id
        LEFT JOIN category c ON mc.category_id = c.id
    """
    where = []
    params = []

    if category_id:
        where.append("mc.category_id = %s")
        params.append(category_id)

    if where:
        base_query += " WHERE " + " AND ".join(where)

    base_query += " GROUP BY m.id"

    # 排序逻辑
    sort_columns = {
        'rating': 'm.rating DESC',
        'sales': 'm.monthly_sales DESC',
        'price': 'm.avg_price ASC'
    }
    order_by = sort_columns.get(sort_by, 'm.rating DESC')
    base_query += f" ORDER BY {order_by} LIMIT %s OFFSET %s"

    params.extend([page_size, offset])
    return {"data": db_query(base_query, params)}


# 3. 商家详情
@app.get("/merchants/{merchant_id}")
async def get_merchant_detail(merchant_id: int):
    """获取商家详细信息"""
    # 商家基本信息
    merchant = db_query("SELECT * FROM merchant WHERE id = %s", (merchant_id,), True)
    if not merchant:
        raise HTTPException(404, "商家不存在")

    # 菜品分类及菜品
    categories = db_query("""
    SELECT dc.id, dc.name, 
    (SELECT JSON_ARRAYAGG(
        JSON_OBJECT('id',d.id,'name',d.name,'price',d.price,
                    'rating',d.rating,'sales',d.monthly_sales,'tags',d.tags)
    ) FROM dish d WHERE d.category_id = dc.id) AS dishes
    FROM dish_category dc WHERE dc.merchant_id = %s
    """, (merchant_id,))

    # 评价
    reviews = db_query("""
    SELECT r.rating, r.content, r.review_time, d.name AS dish_name
    FROM review r
    JOIN order_item oi ON r.order_item_id = oi.id
    JOIN dish d ON oi.dish_id = d.id
    WHERE d.merchant_id = %s ORDER BY r.review_time DESC LIMIT 10
    """, (merchant_id,))

    return {
        "merchant": merchant,
        "categories": categories,
        "reviews": reviews
    }


# 4. 创建订单接口
@app.post("/orders")
async def create_order(order_data: CreateOrder):
    """创建新订单"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 计算总金额
            total = sum(item.price * item.quantity for item in order_data.items)

            # 获取商家配送费
            cursor.execute("SELECT delivery_fee, min_order_price FROM merchant WHERE id = %s",
                           (order_data.merchant_id,))
            merchant_info = cursor.fetchone()
            if not merchant_info:
                raise HTTPException(status_code=404, detail="Merchant not found")

            if total < merchant_info['min_order_price']:
                raise HTTPException(
                    status_code=400,
                    detail=f"订单金额需达到{merchant_info['min_order_price']}元"
                )

            # 创建订单
            order_query = """
                INSERT INTO `order` (
                    customer_name, customer_phone, delivery_address,
                    merchant_id, total_amount, delivery_fee, 
                    status, order_time, payment_method, note
                ) VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s,%s)
            """
            order_params = (
                order_data.customer_name,
                order_data.customer_phone,
                order_data.delivery_address,
                order_data.merchant_id,
                total,
                merchant_info['delivery_fee'],
                '待接单',
                datetime.now(),
                order_data.payment_method,
                order_data.note
            )
            cursor.execute(order_query, order_params)
            order_id = cursor.lastrowid

            # 创建订单明细
            for item in order_data.items:
                cursor.execute("""
                    INSERT INTO order_item 
                    (order_id, dish_id, quantity, price)
                    VALUES (%s,%s,%s,%s)
                """, (order_id, item.dish_id, item.quantity, item.price))

            connection.commit()
            return {"order_id": order_id}
    except Exception as e:
        connection.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        connection.close()

# 5. 订单详情接口
@app.get("/orders/{order_id}")
async def get_order_detail(order_id: int):
    """获取订单详情"""
    order_query = """
        SELECT o.*, m.name AS merchant_name 
        FROM `order` o
        JOIN merchant m ON o.merchant_id = m.id
        WHERE o.id = %s
    """
    order = db_query(order_query, (order_id,), fetch_one=True)
    if not order:
        raise HTTPException(status_code=404, detail="订单不存在")

    items_query = """
        SELECT oi.*, d.name AS dish_name 
        FROM order_item oi
        JOIN dish d ON oi.dish_id = d.id
        WHERE oi.order_id = %s
    """
    items = db_query(items_query, (order_id,))

    return {"order": order, "items": items}

# 6. 获取菜品评价
@app.get("/dishes/{dish_id}/reviews")
async def get_dish_reviews(dish_id: int):
    """获取菜品评价"""
    return db_query("""
    SELECT r.rating, r.content, r.review_time, oi.quantity 
    FROM review r
    JOIN order_item oi ON r.order_item_id = oi.id
    WHERE oi.dish_id = %s
    ORDER BY r.review_time DESC
    """, (dish_id,))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)

```

step3:商家列表  C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\merchants-list\merchants-list.component.html

```typescript
import { Component, OnInit } from '@angular/core';

import { MerchantService} from '../services/merchant.service';
import {NgForOf, NgIf} from '@angular/common';
import {RouterLink} from '@angular/router';


@Component({
  selector: 'app-merchants-list',
  imports: [
    NgForOf,
    NgIf,
    RouterLink
  ],
  templateUrl: './merchants-list.component.html',
  styleUrl: './merchants-list.component.css'
})
export class MerchantsListComponent  implements OnInit {
  categories: any[] = [];
  merchants: any[] = [];
  selectedCategoryId: number | null = null;
  isLoading = false;

  constructor(private merchantService: MerchantService) { }

  ngOnInit() {
    this.loadCategories();
    this.loadMerchants(); // 初始加载全部商家
  }

  // 加载分类数据
  loadCategories() {
    this.merchantService.getCategories().subscribe(res => {
      this.categories = res.data;
    });
  }

  // 加载商家数据
  loadMerchants(categoryId?: number) {
    this.isLoading = true;
    this.merchantService.getMerchants(categoryId).subscribe({
      next: (res) => {
        this.merchants = res.data;
        this.isLoading = false;
      },
      error: () => {
        this.isLoading = false;
        alert('加载商家失败，请重试');
      }
    });
  }

  // 分类选择处理
  selectCategory(categoryId: number | null) {
    this.selectedCategoryId = categoryId;
    this.loadMerchants(categoryId ?? undefined);
  }
}
<!-- takeout.component.html -->
<div class="container">
  <!-- 左侧分类导航 -->
  <div class="categories-sidebar">
    <h3>美食分类</h3>
    <ul>
      <li
        *ngFor="let category of categories"
        [class.active]="selectedCategoryId === category.id"
        (click)="selectCategory(category.id)">
        {{category.name}}
      </li>
      <li
        [class.active]="!selectedCategoryId"
        (click)="selectCategory(null)">
        全部商家
      </li>
    </ul>
  </div>

  <!-- 右侧商家列表 -->
  <div class="merchant-list">
    <!-- 加载状态 -->
    <div *ngIf="isLoading" class="loading">加载中...</div>

    <!-- 商家列表 -->
    <div *ngFor="let merchant of merchants" class="merchant-card">
      <div class="merchant-header">
        <h3>{{merchant.name}}</h3>
        <span class="rating">{{merchant.rating}} ★</span>
      </div>

      <div class="merchant-info">
        <div class="meta">
          <span>月售 {{merchant.monthly_sales}}</span>
          <span>起送 ¥{{merchant.min_order_price}}</span>
          <span>配送 ¥{{merchant.delivery_fee}}</span>
        </div>
        <a [routerLink]="['/merchants-detail', merchant.id]" class="detail-link">查看详情</a>

        <div class="tags">
          <span *ngFor="let tag of merchant.tags.split(',')">{{tag}}</span>
        </div>
      </div>
    </div>
  </div>
</div>
/* takeout.component.css */
.container {
  display: grid;
  grid-template-columns: 200px 1fr;
  gap: 20px;
  padding: 20px;
}

.categories-sidebar {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 8px;
}

.categories-sidebar ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.categories-sidebar li {
  padding: 10px;
  cursor: pointer;
  border-radius: 4px;
  margin-bottom: 5px;
  transition: all 0.2s;
}

.categories-sidebar li:hover {
  background: #eee;
}

.categories-sidebar li.active {
  background: #007bff;
  color: white;
}

.merchant-list {
  display: grid;
  gap: 15px;
}

.merchant-card {
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.merchant-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.rating {
  color: #ffd700;
  font-weight: bold;
}

.meta span {
  margin-right: 15px;
  color: #666;
}

.tags span {
  display: inline-block;
  background: #f0f0f0;
  padding: 4px 8px;
  border-radius: 4px;
  margin: 5px 5px 0 0;
  font-size: 0.9em;
}

.loading {
  text-align: center;
  padding: 20px;
  color: #666;
}

```

step4:商家详情  C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\merchants-detail\merchants-detail.component.css

```typescript
// merchants-detail.component.ts
import { Component, OnInit } from '@angular/core';
import {ActivatedRoute, RouterLink} from '@angular/router';
import { MerchantService } from '../services/merchant.service';
import { DatePipe, NgFor, NgIf } from '@angular/common';

@Component({
  selector: 'app-merchants-detail',
  standalone: true,
  imports: [NgIf, NgFor, DatePipe, RouterLink],
  templateUrl: './merchants-detail.component.html',
  styleUrls: ['./merchants-detail.component.css']
})
export class MerchantsDetailComponent implements OnInit {
  merchantDetail: any;
  isLoading = true;
  errorMessage = '';

  constructor(
    private route: ActivatedRoute,
    private merchantService: MerchantService
  ) { }

  ngOnInit() {
    const id = this.route.snapshot.paramMap.get('id');
    if (id) {
      this.loadMerchantDetail(+id);
    } else {
      this.errorMessage = '无效的商家ID';
      this.isLoading = false;
    }
  }

  loadMerchantDetail(id: number) {
    this.merchantService.getMerchantDetail(id).subscribe({
      next: (res) => {
        this.merchantDetail = {
          ...res,
          categories: res.categories.map((cat: any) => ({
            ...cat,
            dishes: JSON.parse(cat.dishes)
          }))
        };
        this.isLoading = false;
      },
      error: (err) => {
        this.errorMessage = '加载商家详情失败，请稍后重试';
        this.isLoading = false;
      }
    });
  }

  getRatingStars(rating: number): string {
    return '★'.repeat(Math.round(rating));
  }
}

<!-- merchants-detail.component.html -->
<div class="merchant-container">
  <!-- 加载状态 -->
  <div *ngIf="isLoading" class="loading">加载中...</div>

  <!-- 错误提示 -->
  <div *ngIf="errorMessage" class="error">{{ errorMessage }}</div>

  <!-- 商家详情内容 -->
  <div *ngIf="merchantDetail" class="merchant-content">
    <!-- 商家基本信息 -->
    <section class="merchant-info">
      <h1>{{ merchantDetail.merchant.name }}</h1>
      <div class="stats">
        <span class="rating">
          {{ getRatingStars(merchantDetail.merchant.rating) }}
          <em>({{ merchantDetail.merchant.rating }})</em>
        </span>
        <span>月售 {{ merchantDetail.merchant.monthly_sales }}</span>
        <span>人均 ¥{{ merchantDetail.merchant.avg_price }}</span>
      </div>

      <div class="details">
        <div class="detail-item">
          <label>起送价:</label>
          <span>¥{{ merchantDetail.merchant.min_order_price }}</span>
        </div>
        <div class="detail-item">
          <label>配送费:</label>
          <span>¥{{ merchantDetail.merchant.delivery_fee }}</span>
        </div>
        <div class="detail-item">
          <label>营业时间:</label>
          <span>{{ merchantDetail.merchant.business_hours }}</span>
        </div>
        <div class="detail-item">
          <label>联系电话:</label>
          <a href="tel:{{ merchantDetail.merchant.contact_phone }}">
            {{ merchantDetail.merchant.contact_phone }}
          </a>
        </div>
        <div class="detail-item">
          <label>地址:</label>
          <address>{{ merchantDetail.merchant.location }}</address>
        </div>
      </div>

      <div class="tags">
        <span *ngFor="let tag of merchantDetail.merchant.tags.split(',')"
              class="tag">
          {{ tag }}
        </span>
      </div>
    </section>

    <!-- 菜品分类 -->

    <!-- merchants-detail.component.html -->
    <section *ngFor="let category of merchantDetail.categories" class="menu-category">
      <h2>{{ category.name }}</h2>
      <div class="dishes">
        <div *ngFor="let dish of category.dishes" class="dish-card">
          <!-- 其他菜品信息 -->
          <div class="dish-info">
            <h3>{{ dish.name }}</h3>
            <div class="dish-meta">
              <span class="price">¥{{ dish.price }}</span>
              <span class="sales">月售 {{ dish.sales }}</span>
              <span class="dish-rating">
                {{ getRatingStars(dish.rating) }}
              </span>
            </div>
            <div class="dish-tags">
              <span *ngFor="let tag of dish.tags.split(',')"
                    class="tag">
                {{ tag }}
              </span>
            </div>
          <button class="book-button"
                  [routerLink]="['/merchants-order']"
                  [state]="{
                orderData: {
                  dish: dish,
                  merchant: merchantDetail.merchant
                }
              }">
            立即购买
          </button>
        </div>
      </div>
      </div>
    </section>


    <!-- 用户评价 -->
    <section class="reviews">
      <h2>用户评价 ({{ merchantDetail.reviews.length }})</h2>
      <div *ngFor="let review of merchantDetail.reviews" class="review-card">
        <div class="review-header">
          <span class="rating">{{ getRatingStars(review.rating) }}</span>
          <span class="dish">{{ review.dish_name }}</span>
          <time [title]="review.review_time | date:'long'">
            {{ review.review_time | date:'yyyy-MM-dd HH:mm' }}
          </time>
        </div>
        <p class="content">{{ review.content }}</p>
      </div>
    </section>
  </div>
</div>


/* merchants-detail.component.css */
.merchant-container {
  max-width: 1200px;
  margin: 20px auto;
  padding: 20px;
}

.loading, .error {
  text-align: center;
  padding: 40px;
  font-size: 1.2rem;
}

.error {
  color: #dc3545;
}

.merchant-content {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  padding: 24px;
}

.merchant-info h1 {
  color: #333;
  margin-bottom: 16px;
}

.stats {
  display: flex;
  gap: 20px;
  margin-bottom: 24px;
  color: #666;
}

.rating {
  color: #ffd700;
  font-size: 1.2em;
}
.rating em {
  color: #666;
  font-style: normal;
  font-size: 0.9em;
  margin-left: 8px;
}

.details {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 16px;
  margin-bottom: 24px;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #eee;
}

.detail-item label {
  color: #999;
}

.tags {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.tag {
  background: #f0f0f0;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 0.9em;
  color: #666;
}

/* 菜品分类样式 */
.menu-category {
  margin: 32px 0;
  border-top: 2px solid #eee;
  padding-top: 24px;
}

.menu-category h2 {
  color: #444;
  margin-bottom: 16px;
}

.dishes {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.dish-card {
  border: 1px solid #eee;
  border-radius: 8px;
  padding: 16px;
  transition: transform 0.2s;
}

.dish-card:hover {
  transform: translateY(-2px);
}

.dish-meta {
  display: flex;
  gap: 12px;
  margin: 8px 0;
  color: #666;
}

.price {
  color: #e4393c;
  font-weight: bold;
}

.dish-rating {
  color: #ffd700;
}

/* 评价样式 */
.reviews {
  margin-top: 40px;
}

.review-card {
  padding: 16px;
  border: 1px solid #eee;
  border-radius: 8px;
  margin-bottom: 16px;
}

.review-header {
  display: flex;
  gap: 12px;
  align-items: center;
  margin-bottom: 8px;
  color: #666;
}

.review-header time {
  font-size: 0.9em;
  color: #999;
}

.content {
  color: #333;
  line-height: 1.6;
}

```

step5:立即购买C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\merchants-order\merchants-order.component.css

```typescript
// merchants-order.component.ts
import { Component, OnInit } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
import {FormBuilder, FormGroup, ReactiveFormsModule, Validators} from '@angular/forms';
import {Location, NgIf} from '@angular/common';
import { MerchantService } from '../services/merchant.service';

// 类型定义
interface Dish {
  id: number;
  name: string;
  price: number;
  // 其他字段根据实际API响应添加
}

interface Merchant {
  id: number;
  name: string;
  delivery_fee: number;
  location: string;
  // 其他字段根据实际API响应添加
}

interface OrderState {
  orderData: {
    dish: Dish;
    merchant: Merchant;
  };
}

@Component({
  selector: 'app-merchants-order',
  templateUrl: './merchants-order.component.html',
  imports: [
    ReactiveFormsModule,
    NgIf
  ],
  styleUrls: ['./merchants-order.component.css']
})
export class MerchantsOrderComponent implements OnInit {
  orderForm: FormGroup;
  selectedDish?: Dish;
  merchant?: Merchant;
  loading = false;

  constructor(
    private fb: FormBuilder,
    private location: Location,
    private router: Router,
    private route: ActivatedRoute,
    private merchantService: MerchantService
  ) {
    this.orderForm = this.fb.group({
      customer_name: ['', [Validators.required, Validators.minLength(2)]],
      customer_phone: ['', [Validators.required, Validators.pattern(/^1[3-9]\d{9}$/)]],
      delivery_address: ['', [Validators.required, Validators.minLength(5)]],
      payment_method: ['alipay', Validators.required],
      note: [''],
      quantity: [1, [Validators.required, Validators.min(1), Validators.max(10)]]
    });
  }

  ngOnInit(): void {
    this.handleRouteState();
    this.setupFormListeners();
  }

  private handleRouteState(): void {
    const state = this.location.getState() as OrderState;
    console.log('路由状态:', state);

    if (state?.orderData?.dish && state?.orderData?.merchant) {
      this.selectedDish = state.orderData.dish;
      this.merchant = state.orderData.merchant;
      this.initFormWithFallbackData();
    } else {
      console.warn('无效的路由状态，尝试通过参数加载');
      this.loadFromQueryParams();
    }
  }

  private loadFromQueryParams(): void {
    this.route.queryParams.subscribe(params => {
      if (params['merchantId'] && params['dishId']) {
        this.merchantService.getMerchantDetail(params['merchantId']).subscribe({
          next: merchant => {
            this.merchant = merchant;
            this.selectedDish = this.findDishInCategories(merchant.categories, params['dishId']);
            if (!this.selectedDish) {
              this.redirectToMerchantList();
            }
          },
          error: () => this.redirectToMerchantList()
        });
      } else {
        this.redirectToMerchantList();
      }
    });
  }

  private findDishInCategories(categories: any[], dishId: string): Dish | undefined {
    for (const category of categories) {
      const dishes = JSON.parse(category.dishes);
      const found = dishes.find((d: any) => d.id === +dishId);
      if (found) return found;
    }
    return undefined;
  }

  private initFormWithFallbackData(): void {
    this.orderForm.patchValue({
      delivery_address: this.merchant?.location || ''
    });
  }

  private setupFormListeners(): void {
    this.orderForm.get('quantity')?.valueChanges.subscribe(value => {
      if (value > 10) {
        this.orderForm.get('quantity')?.setValue(10);
      }
    });
  }

  get totalAmount(): string {
    if (!this.selectedDish || !this.merchant) return '0.00';
    const quantity = this.orderForm.get('quantity')?.value || 1;
    return (this.selectedDish.price * quantity + this.merchant.delivery_fee).toFixed(2);
  }

  onSubmit(): void {
    if (this.orderForm.invalid || !this.selectedDish || !this.merchant) return;

    const orderData = {
      ...this.orderForm.value,
      merchant_id: this.merchant.id,
      items: [{
        dish_id: this.selectedDish.id,
        quantity: this.orderForm.value.quantity,
        price: this.selectedDish.price
      }]
    };

    this.loading = true;
    this.merchantService.createOrder(orderData).subscribe({
      next: (res) => {
        this.router.navigate(['/merchants-order-detail', res.order_id]);
        this.loading = false;
      },
      error: (err) => {
        console.error('订单提交失败:', err);
        alert(`提交失败: ${err.error?.detail || '服务器错误'}`);
        this.loading = false;
      }
    });
  }

  private redirectToMerchantList(): void {
    this.router.navigate(['/merchants'], {
      queryParamsHandling: 'preserve'
    });
  }
}

<!-- merchants-order.component.html -->
<div class="order-container">
  <h2>确认订单</h2>

  <form *ngIf="merchant && selectedDish" [formGroup]="orderForm" (ngSubmit)="onSubmit()">
    <!-- 商家信息 -->
    <div class="merchant-info">
      <h3>{{ merchant.name }}</h3>
      <p>{{ merchant.location }}</p>
    </div>

    <!-- 商品信息 -->
    <div class="dish-info">
      <h4>{{ selectedDish.name }}</h4>
      <div class="price-quantity">
        <span class="price">¥{{ selectedDish.price }}</span>
        <div class="quantity-control">
          <label>数量：</label>
          <input type="number" formControlName="quantity" min="1">
        </div>
      </div>
    </div>

    <!-- 订单表单 -->
    <div class="form-section">
      <div class="form-group">
        <label>收货人姓名</label>
        <input type="text" formControlName="customer_name">
        <div *ngIf="orderForm.get('customer_name')?.errors?.['required']" class="error">
          姓名不能为空
        </div>
      </div>

      <div class="form-group">
        <label>联系电话</label>
        <input type="tel" formControlName="customer_phone">
        <div *ngIf="orderForm.get('customer_phone')?.errors?.['required']" class="error">
          电话不能为空
        </div>
        <div *ngIf="orderForm.get('customer_phone')?.errors?.['pattern']" class="error">
          请输入有效的手机号
        </div>
      </div>

      <div class="form-group">
        <label>配送地址</label>
        <textarea formControlName="delivery_address"></textarea>
        <div *ngIf="orderForm.get('delivery_address')?.errors?.['required']" class="error">
          地址不能为空
        </div>
      </div>

      <div class="form-group">
        <label>支付方式</label>
        <select formControlName="payment_method">
          <option value="alipay">支付宝</option>
          <option value="wechat">微信支付</option>
          <option value="cash">货到付款</option>
        </select>
      </div>

      <div class="form-group">
        <label>备注</label>
        <textarea formControlName="note"></textarea>
      </div>
    </div>

    <!-- 费用汇总 -->
    <div class="price-summary">
      <div class="summary-item">
        <span>商品总额：</span>
<!--        <span>¥{{ (selectedDish.price * orderForm.value.quantity).toFixed(2) }}</span>-->
      </div>
      <div class="summary-item">
        <span>配送费：</span>
        <span>¥{{ merchant.delivery_fee }}</span>
      </div>
      <div class="summary-item total">
        <span>总计：</span>
        <span>¥{{ totalAmount }}</span>
      </div>
    </div>

    <button type="submit" [disabled]="orderForm.invalid || loading" class="submit-btn">
      {{ loading ? '提交中...' : '提交订单' }}
    </button>
  </form>
</div>


.order-container {
  max-width: 600px;
  margin: 20px auto;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.merchant-info {
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.dish-info {
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.error {
  color: #dc3545;
  font-size: 0.9em;
  margin-top: 5px;
}

.price-summary {
  margin-top: 20px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 4px;
}

.summary-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.total {
  font-weight: bold;
  font-size: 1.1em;
}

.submit-btn {
  width: 100%;
  padding: 12px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 20px;
}

.submit-btn:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

```

step6:订单详情C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\merchants-order-detail\merchants-order-detail.component.css

```typescript
// merchants-order-detail.component.ts
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { MerchantService } from '../services/merchant.service';
import { DatePipe, CurrencyPipe, NgIf, NgFor } from '@angular/common';

interface OrderItem {
  id: number;
  dish_name: string;
  quantity: number;
  price: number;
}

interface OrderDetail {
  id: number;
  customer_name: string;
  customer_phone: string;
  delivery_address: string;
  total_amount: number;
  delivery_fee: number;
  packing_fee: number;
  status: string;
  order_time: string;
  complete_time: string | null;
  rider_name: string | null;
  rider_phone: string | null;
  payment_method: string;
  note: string | null;
  merchant_name: string;
}

@Component({
  selector: 'app-merchants-order-detail',
  standalone: true,
  imports: [NgIf, NgFor, DatePipe, CurrencyPipe],
  templateUrl: './merchants-order-detail.component.html',
  styleUrls: ['./merchants-order-detail.component.css']
})
export class MerchantsOrderDetailComponent implements OnInit {
  orderId!: number;
  orderId2!: number;

  order!: OrderDetail;
  items: OrderItem[] = [];
  isLoading = true;
  errorMessage = '';

  constructor(
    private route: ActivatedRoute,
    private router: Router,
    private merchantService: MerchantService
  ) {}

  ngOnInit(): void {
    this.route.params.subscribe(params => {
      this.orderId = +params['order_id'];
      this.orderId2 = params['order_id'];
      console.log('从路由获取的order_id:', this.orderId+"==="+this.orderId2);

      this.loadOrderDetail();
    });
  }

  loadOrderDetail(): void {
    this.isLoading = true;
    this.merchantService.getOrderDetail(this.orderId).subscribe({
      next: (res) => {
        this.order = res.order;
        this.items = res.items;
        this.isLoading = false;
      },
      error: (err) => {
        console.error('加载失败:', err);
        this.errorMessage = '无法加载订单详情，请稍后重试';
        this.isLoading = false;
      }
    });
  }

  getStatusClass(): string {
    switch(this.order.status) {
      case '已完成': return 'status-completed';
      case '待接单': return 'status-pending';
      case '配送中': return 'status-delivering';
      default: return 'status-default';
    }
  }
}

<!-- merchants-order-detail.component.html -->
<div class="order-detail-container">
  <!-- 加载状态 -->
  <div *ngIf="isLoading" class="loading">加载中...</div>

  <!-- 错误提示 -->
  <div *ngIf="errorMessage" class="error">{{ errorMessage }}</div>

  <!-- 订单详情 -->
  <div *ngIf="order" class="order-content">
    <!-- 订单头信息 -->
    <div class="order-header">
      <h2>订单号：{{ order.id }}</h2>
      <span [class]="getStatusClass()">{{ order.status }}</span>
    </div>

    <!-- 基本信息 -->
    <div class="section">
      <h3>订单信息</h3>
      <div class="info-grid">
        <div class="info-item">
          <label>下单时间：</label>
          <span>{{ order.order_time | date:'yyyy-MM-dd HH:mm' }}</span>
        </div>
        <div class="info-item">
          <label>商家名称：</label>
          <span>{{ order.merchant_name }}</span>
        </div>
        <div class="info-item">
          <label>支付方式：</label>
          <span>{{ order.payment_method }}</span>
        </div>
      </div>
    </div>

    <!-- 商品清单 -->
    <div class="section">
      <h3>商品清单</h3>
      <table class="item-table">
        <thead>
        <tr>
          <th>商品名称</th>
          <th>单价</th>
          <th>数量</th>
          <th>小计</th>
        </tr>
        </thead>
        <tbody>
        <tr *ngFor="let item of items">
          <td>{{ item.dish_name }}</td>
          <td>{{ item.price | currency:'CNY':'symbol':'1.2-2' }}</td>
          <td>{{ item.quantity }}</td>
          <td>{{ item.price * item.quantity | currency:'CNY':'symbol':'1.2-2' }}</td>
        </tr>
        </tbody>
      </table>
    </div>

    <!-- 费用明细 -->
    <div class="section">
      <h3>费用明细</h3>
      <div class="fee-details">
        <div class="fee-item">
          <span>商品总额：</span>
          <span>{{ order.total_amount - order.delivery_fee - order.packing_fee | currency:'CNY':'symbol':'1.2-2' }}</span>
        </div>
        <div class="fee-item">
          <span>配送费：</span>
          <span>{{ order.delivery_fee | currency:'CNY':'symbol':'1.2-2' }}</span>
        </div>
        <div class="fee-item">
          <span>包装费：</span>
          <span>{{ order.packing_fee | currency:'CNY':'symbol':'1.2-2' }}</span>
        </div>
        <div class="fee-item total">
          <span>实付金额：</span>
          <span>{{ order.total_amount | currency:'CNY':'symbol':'1.2-2' }}</span>
        </div>
      </div>
    </div>

    <!-- 配送信息 -->
    <div class="section">
      <h3>配送信息</h3>
      <div class="delivery-info">
        <p>收货人：{{ order.customer_name }}</p>
        <p>联系电话：{{ order.customer_phone }}</p>
        <p>配送地址：{{ order.delivery_address }}</p>
        <div *ngIf="order.rider_name" class="rider-info">
          <p>骑手：{{ order.rider_name }}</p>
          <p>骑手电话：{{ order.rider_phone }}</p>
        </div>
      </div>
    </div>

    <!-- 备注信息 -->
    <div *ngIf="order.note" class="section">
      <h3>订单备注</h3>
      <p class="note">{{ order.note }}</p>
    </div>
  </div>
</div>

/* merchants-order-detail.component.css */
.order-detail-container {
  max-width: 1200px;
  margin: 20px auto;
  padding: 20px;
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.loading, .error {
  text-align: center;
  padding: 40px;
  font-size: 1.2rem;
}

.error {
  color: #dc3545;
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.section {
  margin-bottom: 30px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 6px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 15px;
}

.item-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

.item-table th,
.item-table td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #eee;
}

.item-table th {
  background: #f8f9fa;
}

.fee-details {
  max-width: 400px;
}

.fee-item {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.fee-item.total {
  font-weight: bold;
  border-top: 1px solid #eee;
  padding-top: 10px;
  font-size: 1.1em;
}

.note {
  color: #666;
  line-height: 1.6;
}

/* 状态标签样式 */
.status-completed { color: #28a745; }
.status-pending { color: #ffc107; }
.status-delivering { color: #17a2b8; }
.status-default { color: #6c757d; }

```

end