说明：
珠宝系统

1.珠宝分类（挂件，手镯，和田玉，琥珀，戒面，天然宝石，文玩杂项）

2.珠宝列表 （其中，选择不同的珠宝分类，应该加载对应的珠宝列表）

3.珠宝列表item （商家名，珠宝名称，多张珠宝图片url数组，珠宝描述，商品价格，优惠价，可以点击收藏）

4.点击珠宝列表item，跳转到珠宝详情（商家名，珠宝名称，多张珠宝图片url数组，评论表，添加查看评价，商品价格，优惠价，珠宝描述，产品政策，收藏）

5.立即购买（填写用户信息，包括用户名 ，手机号，联系地址，商品信息简介，商家名，珠宝名称，珠宝描述，价格，数量，付款金额，提交订单）

6.订单详情（订单编号，支付方式，物流单号，物流公司，下单时间，发货时间，成交时间，商品信息简介，商家名，珠宝名称，珠宝描述，价格，数量，付款金额，）

7.评论表

8.收藏表
1.珠宝分类管理：获取分类列表、创建分类
2.商家管理：获取商家列表、创建商家
3.珠宝管理：获取珠宝列表、创建珠宝、上传图片
4.用户地址管理：获取地址、创建地址
5.订单管理：创建订单、查询订单
6.评论管理：创建评论、获取评论
7.收藏管理：添加收藏、移除收藏、获取收藏列表
参考下面的代码 用postman写10个接口示例

 # 1.获取珠宝分类列表（含层级）
# 2. 获取指定分类的珠宝列表（示例：黄金项链）
# 3.珠宝详情查询
# 4.创建订单
# 5. 订单详情查询
# 6.用户收藏列表
# 7.添加评论（带购买验证）
# 8. 商品评论查询
# 9.管理端-珠宝搜索
# 10.热销商品统计
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/dad103a80dc24a42a41a6bac149f1d9b.png#pic_center)

step1:sql

```sql


show databases;

DROP TABLE users;


SHOW CREATE TABLE db_school.users;

show tables;

use db_school;

SELECT * FROM db_school.jewelry_categories;

CREATE DATABASE db_school;



-- 珠宝分类表（支持层级扩展）
CREATE TABLE db_school.jewelry_categories (
  category_id INT AUTO_INCREMENT PRIMARY KEY,
  category_name VARCHAR(50) NOT NULL UNIQUE,
  parent_id INT DEFAULT NULL COMMENT '支持多级分类',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (parent_id) REFERENCES jewelry_categories(category_id)
);

-- 商家表（独立存储商家信息）
CREATE TABLE db_school.merchants (
  merchant_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  contact_phone VARCHAR(20) NOT NULL,
  address VARCHAR(255) NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);




-- 珠宝信息表（关联商家+分类）
CREATE TABLE db_school.jewelries (
  jewelry_id INT AUTO_INCREMENT PRIMARY KEY,
  category_id INT NOT NULL,
  merchant_id INT NOT NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  original_price DECIMAL(12,2) NOT NULL,
  discounted_price DECIMAL(12,2),
  stock INT DEFAULT 0 COMMENT '库存',
  is_hot BOOLEAN DEFAULT FALSE COMMENT '热销标记',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (category_id) REFERENCES jewelry_categories(category_id),
  FOREIGN KEY (merchant_id) REFERENCES merchants(merchant_id)
);

-- 珠宝图片表（多图支持）
CREATE TABLE db_school.jewelry_images (
  image_id INT AUTO_INCREMENT PRIMARY KEY,
  jewelry_id INT NOT NULL,
  image_url VARCHAR(512) NOT NULL,
  sort_order INT DEFAULT 0,
  FOREIGN KEY (jewelry_id) REFERENCES jewelries(jewelry_id)
);

-- 用户表（增强联系方式）
CREATE TABLE db_school.users (
  user_id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) NOT NULL UNIQUE,
  phone VARCHAR(20) NOT NULL UNIQUE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 用户地址表（支持多地址）
CREATE TABLE db_school.user_addresses (
  address_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  recipient VARCHAR(50) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  address VARCHAR(255) NOT NULL,
  is_default BOOLEAN DEFAULT FALSE,
  FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- 订单表（状态追踪）
CREATE TABLE db_school.orders (
  order_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  address_id INT NOT NULL COMMENT '收货地址',
  order_number VARCHAR(50) NOT NULL UNIQUE,
  payment_method ENUM('alipay','wechat','bank') NOT NULL,
  total_amount DECIMAL(12,2) NOT NULL,
  order_status ENUM('pending','paid','shipped','completed','cancelled') DEFAULT 'pending',
  tracking_number VARCHAR(100),
  shipping_company VARCHAR(100),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  paid_at DATETIME,
  shipped_at DATETIME,
  completed_at DATETIME,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (address_id) REFERENCES user_addresses(address_id)
);

-- 订单商品快照表（防止价格变动影响）
CREATE TABLE db_school.order_items (
  item_id INT AUTO_INCREMENT PRIMARY KEY,
  order_id INT NOT NULL,
  jewelry_id INT NOT NULL,
  quantity INT NOT NULL,
  unit_price DECIMAL(12,2) NOT NULL COMMENT '下单时价格',
  FOREIGN KEY (order_id) REFERENCES orders(order_id),
  FOREIGN KEY (jewelry_id) REFERENCES jewelries(jewelry_id)
);

-- 评论表（购买验证）
CREATE TABLE db_school.reviews (
  review_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  order_id INT NOT NULL,
  jewelry_id INT NOT NULL,
  rating TINYINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
  content TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (order_id) REFERENCES orders(order_id),
  FOREIGN KEY (jewelry_id) REFERENCES jewelries(jewelry_id),
  UNIQUE (order_id) -- 一个订单只能评论一次
);

-- 收藏表
CREATE TABLE db_school.favorites (
  favorite_id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT NOT NULL,
  jewelry_id INT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(user_id),
  FOREIGN KEY (jewelry_id) REFERENCES jewelries(jewelry_id),
  UNIQUE (user_id, jewelry_id)
);


-- 1. 珠宝分类表 (包含三级分类示例)
INSERT INTO db_school.jewelry_categories (category_name, parent_id) VALUES
('项链', NULL),
('戒指', NULL),
('耳环', NULL),
('手链', NULL),
('黄金项链', 1),
('钻石项链', 1),
('翡翠戒指', 2),
('铂金戒指', 2),
('珍珠耳环', 3),
('银饰耳环', 3),
('儿童手链', 4),
('情侣手链', 4);

-- 2. 商家表
INSERT INTO db_school.merchants (name, contact_phone, address) VALUES
('周大福珠宝','13800000001',''),
('老凤祥银楼','13800000002',''),
('六福珠宝','13800000003',''),
('周生生','13800000004',''),
('谢瑞麟','13800000005',''),
('潮宏基','13800000006',''),
('明牌珠宝','13800000007',''),
('金至尊','13800000008',''),
('钻石世家','13800000009',''),
('中国黄金','13800000010','');

-- 3. 用户表
INSERT INTO db_school.users (username, phone) VALUES
('user01','13900000001'),
('user02','13900000002'),
('user03','13900000003'),
('user04','13900000004'),
('user05','13900000005'),
('user06','13900000006'),
('user07','13900000007'),
('user08','13900000008'),
('user09','13900000009'),
('user10','13900000010');


select *from jewelries;

-- 4. 珠宝信息表 (注意关联有效的category_id和merchant_id)
INSERT INTO db_school.jewelries (category_id, merchant_id, name, description, original_price, discounted_price, stock, is_hot) VALUES
(5, 1, '足金福牌项链',' 工费全免', 2999.00, 2799.00, 100, 1),
(6, 2, '星耀钻石项链','VS级白钻 ', 8999.00, NULL, 50, 0),
(7, 3, '冰种翡翠戒指',' 附证书', 15000.00, 13800.00, 10, 1),
(8, 4, '铂金素圈对戒',' 简约设计', 3999.00, 3599.00, 80, 1),
(9, 5, '南洋珍珠耳环','', 4999.00, NULL, 30, 0),
(10,6, 'S925银耳钉',' 防过敏', 199.00, 159.00, 200, 1),
(11,7, '宝宝平安手链','', 1299.00, 999.00, 60, 1),
(12,8, '情侣红绳手链','', 599.00, 399.00, 150, 0),
(5, 9, '古法黄金项链',' 哑光质感', 3599.00, 3299.00, 40, 1),
(6, 10, '群镶钻石项链','主钻0.+配钻', 12999.00, 11999.00, 20, 0);

-- 5. 珠宝图片表
INSERT INTO db_school.jewelry_images (jewelry_id, image_url, sort_order) VALUES
(1, 'https://example.com/j1_1.jpg', 1),
(1, 'https://example.com/j1_2.jpg', 2),
(2, 'https://example.com/j2_1.jpg', 1),
(3, 'https://example.com/j3_1.jpg', 1),
(4, 'https://example.com/j4_1.jpg', 1),
(5, 'https://example.com/j5_1.jpg', 1),
(6, 'https://example.com/j6_1.jpg', 1),
(7, 'https://example.com/j7_1.jpg', 1),
(8, 'https://example.com/j8_1.jpg', 1),
(9, 'https://example.com/j9_1.jpg', 1),
(10,'https://example.com/j10_1.jpg',1);



-- 清空原有数据（如果需要）
TRUNCATE TABLE db_school.jewelry_images;

-- 插入新的人像图片数据
INSERT INTO db_school.jewelry_images (jewelry_id, image_url, sort_order)
VALUES
(1, 'https://randomuser.me/api/portraits/men/1.jpg', 1),
(2, 'https://randomuser.me/api/portraits/men/3.jpg', 1),
(3, 'https://randomuser.me/api/portraits/men/4.jpg', 1),
(4, 'https://randomuser.me/api/portraits/men/5.jpg', 1),
(5, 'https://randomuser.me/api/portraits/men/6.jpg', 1),
(6, 'https://randomuser.me/api/portraits/men/1.jpg', 1),
(7, 'https://randomuser.me/api/portraits/men/2.jpg', 1),
(8, 'https://randomuser.me/api/portraits/men/4.jpg', 1),
(9, 'https://randomuser.me/api/portraits/men/5.jpg', 1),
(10, 'https://randomuser.me/api/portraits/men/6.jpg', 1);

-- 验证数据
SELECT * FROM db_school.jewelry_images
ORDER BY jewelry_id, sort_order;

-- 6. 用户地址表
INSERT INTO db_school.user_addresses (user_id, recipient, phone, address, is_default) VALUES
(1, '张三', '13900000001', '', 1),
(1, '张三', '13900000001', '', 0),
(2, '李四', '13900000002', '', 1),
(3, '王五', '13900000003', '', 1),
(4, '赵六', '13900000004', '', 1),
(5, '陈七', '13900000005', '', 1),
(6, '林八', '13900000006', '', 1),
(7, '黄九', '13900000007', '', 1),
(8, '周十', '13900000008', '', 1),
(9, '吴十一','13900000009', '', 1);


select *from db_school.orders where order_number='ORD17423849317421';

-- 7. 订单表（注意时间逻辑）
INSERT INTO db_school.orders (user_id, address_id, order_number, payment_method, total_amount, order_status, tracking_number, shipping_company) VALUES
(1, 1, '202308011001', 'alipay', 2799.00, 'completed', 'SF123456789', '顺丰速运'),
(2, 3, '202308011002', 'wechat', 13800.00, 'shipped', 'YT987654321', '圆通快递'),
(3, 4, '202308011003', 'bank', 3599.00, 'paid', NULL, NULL),
(4, 5, '202308011004', 'alipay', 159.00, 'pending', NULL, NULL),
(5, 6, '202308011005', 'wechat', 999.00, 'completed', 'JD123456789', '京东物流'),
(6, 7, '202308011006', 'bank', 399.00, 'cancelled', NULL, NULL),
(7, 8, '202308011007', 'alipay', 3299.00, 'shipped', 'ZT112233445', '中通快递'),
(8, 9, '202308011008', 'wechat', 11999.00, 'paid', NULL, NULL),
(9, 10, '202308011009', 'bank', 599.00, 'pending', NULL, NULL),
(10,2, '202308011010', 'alipay', 1999.00, 'completed', 'YZ445566778', '邮政快递');

-- 8. 订单商品表
INSERT INTO db_school.order_items (order_id, jewelry_id, quantity, unit_price) VALUES
(1, 1, 1, 2799.00),
(2, 3, 1, 13800.00),
(3, 4, 1, 3599.00),
(4, 6, 2, 159.00),
(5, 7, 1, 999.00),
(6, 8, 1, 399.00),
(7, 9, 1, 3299.00),
(8, 10,1, 11999.00),
(9, 8, 1, 599.00),
(10,5, 1, 1999.00);

-- 9. 评论表
INSERT INTO db_school.reviews (user_id, order_id, jewelry_id, rating, content) VALUES
(1, 1, 1, 5, '做工精细，非常满意！'),
(2, 2, 3, 4, '翡翠成色不错，但物流稍慢'),
(5, 5, 7, 5, '宝宝戴很好看，性价比高'),
(7, 7, 9, 5, '古法工艺很有质感'),
(10,10,5,4, '珍珠光泽不错，包装精美');

-- 10. 收藏表
INSERT INTO db_school.favorites (user_id, jewelry_id) VALUES
(1, 3),
(1, 5),
(2, 4),
(3, 2),
(4, 6),
(5, 7),
(6, 8),
(7, 9),
(8, 10),
(9, 1);

# 1.获取珠宝分类列表（含层级）

SELECT
  c1.category_name AS '一级分类',
  c2.category_name AS '二级分类'
FROM jewelry_categories c1
LEFT JOIN jewelry_categories c2
  ON c2.parent_id = c1.category_id
WHERE c1.parent_id IS NULL;

#2. 获取指定分类的珠宝列表（示例：黄金项链）

SELECT
  j.jewelry_id,
  j.name,
  m.name AS merchant_name,
  j.original_price,
  j.discounted_price,
  JSON_ARRAYAGG(ji.image_url) AS images
FROM jewelries j
JOIN merchants m USING(merchant_id)
JOIN jewelry_images ji USING(jewelry_id)
WHERE j.category_id = 5  -- 黄金项链的category_id
GROUP BY j.jewelry_id;




# 3.珠宝详情查询

SELECT
  j.name,
  m.name AS merchant_name,
  JSON_ARRAYAGG(ji.image_url) AS images,
  j.description,
  j.original_price,
  j.discounted_price,
  (SELECT COUNT(*) FROM favorites WHERE jewelry_id = j.jewelry_id) AS favorite_count
FROM jewelries j
JOIN merchants m USING(merchant_id)
JOIN jewelry_images ji USING(jewelry_id)
WHERE j.jewelry_id = 1;



#4.创建订单
-- 开启事务
START TRANSACTION;

-- 插入订单
INSERT INTO orders (
  user_id, address_id, order_number,
  payment_method, total_amount
) VALUES (
  1,
  (SELECT address_id FROM user_addresses WHERE user_id = 1 AND is_default = 1),
  'ORD202311011234',
  'alipay',
  2799.00
);

-- 插入订单商品
INSERT INTO order_items (order_id, jewelry_id, quantity, unit_price)
VALUES
  (LAST_INSERT_ID(), 1, 1, 2799.00);

COMMIT;


#5. 订单详情查询
SELECT
  o.order_number,
  o.payment_method,
  o.total_amount,
  o.tracking_number,
  o.shipping_company,
  o.created_at,
  o.paid_at,
  ua.recipient,
  ua.phone,
  ua.address,
  JSON_ARRAYAGG(
    JSON_OBJECT(
      'name', j.name,
      'price', oi.unit_price,
      'quantity', oi.quantity
    )
  ) AS items
FROM orders o
JOIN user_addresses ua USING(address_id)
JOIN order_items oi USING(order_id)
JOIN jewelries j USING(jewelry_id)
WHERE o.order_number = 'ORD202311011234';


# 6.用户收藏列表
SELECT
  j.jewelry_id,
  j.name,
  m.name AS merchant_name,
  ji.image_url AS main_image,
  j.discounted_price
FROM favorites f
JOIN jewelries j USING(jewelry_id)
JOIN merchants m USING(merchant_id)
JOIN (
  SELECT jewelry_id, MIN(image_url) AS image_url
  FROM jewelry_images GROUP BY jewelry_id
) ji USING(jewelry_id)
WHERE f.user_id = 1;

#7.添加评论（带购买验证）

INSERT INTO reviews (user_id, order_id, jewelry_id, rating, content)
SELECT
  o.user_id,
  o.order_id,
  oi.jewelry_id,
  5,
  '非常满意的购物体验'
FROM orders o
JOIN order_items oi USING(order_id)
WHERE o.order_number = 'ORD202311011234'
  AND o.order_status = 'completed'
  AND NOT EXISTS (
    SELECT 1 FROM reviews
    WHERE order_id = o.order_id
  );

#8. 商品评论查询
SELECT
  u.username,
  r.rating,
  r.content,
  r.created_at
FROM reviews r
JOIN users u USING(user_id)
WHERE r.jewelry_id = 1
ORDER BY r.created_at DESC;

# 9.管理端-珠宝搜索
SELECT
  j.jewelry_id,
  j.name,
  c.category_name,
  m.name AS merchant_name,
  j.stock,
  j.is_hot
FROM jewelries j
JOIN merchants m USING(merchant_id)
JOIN jewelry_categories c USING(category_id)
WHERE
  (:category_id IS NULL OR j.category_id = :category_id)
  AND (:merchant_id IS NULL OR j.merchant_id = :merchant_id)
ORDER BY j.created_at DESC
LIMIT 20;


# 10.热销商品统计

SELECT
  j.jewelry_id,
  j.name,
  COUNT(oi.item_id) AS sales_volume,
  SUM(oi.quantity) AS total_quantity
FROM jewelries j
LEFT JOIN order_items oi USING(jewelry_id)
GROUP BY j.jewelry_id
ORDER BY sales_volume DESC
LIMIT 10;
```

step2:fastapi

```python
from fastapi import FastAPI, HTTPException, Path,Query
import pymysql.cursors
import json
from typing import Optional, Dict, List
from pydantic import BaseModel
import time
import random
from decimal import Decimal  # 添加 Decimal 类型
from fastapi.middleware.cors import CORSMiddleware
app = FastAPI()
# CORS配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
# 数据库连接配置（保持不变）
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',  # 修正数据库名为db_school
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


def query_database(query: str, params=None):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


class OrderItemCreate(BaseModel):
    jewelry_id: int
    quantity: int

class OrderCreate(BaseModel):
    user_id: int
    payment_method: str
    items: List[OrderItemCreate]

class ReviewCreate(BaseModel):
    user_id: int
    order_number: str
    rating: int
    content: str
# 1. 获取珠宝分类列表（含层级）
@app.get("/categories", summary="获取分类层级列表")
async def get_category_tree():
    """
    获取包含父子关系的分类层级结构
    """
    query = """
    SELECT 
      c1.category_id AS parent_id,
      c1.category_name AS parent_name,
      c2.category_id AS child_id,
      c2.category_name AS child_name
    FROM jewelry_categories c1
    LEFT JOIN jewelry_categories c2 ON c2.parent_id = c1.category_id
    WHERE c1.parent_id IS NULL
    ORDER BY c1.category_id, c2.category_id
    """
    try:
        data = query_database(query)
        # 构建树形结构
        categories = {}
        for row in data:
            parent_id = row['parent_id']
            if parent_id not in categories:
                categories[parent_id] = {
                    "category_id": parent_id,
                    "category_name": row['parent_name'],
                    "children": []
                }
            if row['child_id']:
                categories[parent_id]['children'].append({
                    "category_id": row['child_id'],
                    "category_name": row['child_name']
                })
        return {"data": list(categories.values())}
    except HTTPException as e:
        return {"status": "error", "message": e.detail}


# 2. 获取指定分类的珠宝列表
@app.get("/categories/{category_id}/jewelries", summary="获取分类商品列表")
async def get_jewelries_by_category(
        category_id: int = Path(..., gt=0, description="分类ID"),
        page: int = 1,
        page_size: int = 10
):
    """
    获取指定分类下的珠宝商品列表（支持分页）
    """
    offset = (page - 1) * page_size
    query = """
    SELECT
      j.jewelry_id,
      j.name,
      m.name AS merchant_name,
      j.original_price,
      j.discounted_price,
      JSON_ARRAYAGG(ji.image_url) AS images,
      COUNT(f.favorite_id) AS favorite_count
    FROM jewelries j
    JOIN merchants m ON j.merchant_id = m.merchant_id
    LEFT JOIN jewelry_images ji ON j.jewelry_id = ji.jewelry_id
    LEFT JOIN favorites f ON j.jewelry_id = f.jewelry_id
    WHERE j.category_id = %s
    GROUP BY j.jewelry_id
    LIMIT %s OFFSET %s
    """
    try:
        data = query_database(query, (category_id, page_size, offset))
        # 转换JSON字段
        for item in data:
            if isinstance(item['images'], str):
                item['images'] = json.loads(item['images'])
        return {"data": data, "pagination": {"page": page, "page_size": page_size}}
    except HTTPException as e:
        return {"status": "error", "message": e.detail}


@app.get("/categories/jewelries", summary="获取分类商品列表")
async def get_all_jewelries(
        page: int = Query(1, gt=0, description="页码"),
        page_size: int = Query(10, gt=0, le=100, description="每页数量")
):
    """
    获取所有珠宝商品列表（支持分页）
    """
    offset = (page - 1) * page_size
    query = """
    SELECT
        j.jewelry_id,
        j.name,
        m.name AS merchant_name,
        j.original_price,
        j.discounted_price,
        JSON_ARRAYAGG(ji.image_url) AS images,
        COUNT(f.favorite_id) AS favorite_count
    FROM jewelries j
    JOIN merchants m ON j.merchant_id = m.merchant_id
    LEFT JOIN jewelry_images ji ON j.jewelry_id = ji.jewelry_id
    LEFT JOIN favorites f ON j.jewelry_id = f.jewelry_id
    GROUP BY j.jewelry_id
    LIMIT %s OFFSET %s
    """

    try:
        # 获取分页数据
        data = query_database(query, (page_size, offset))

        # 转换JSON字段
        for item in data:
            if isinstance(item['images'], str):
                item['images'] = json.loads(item['images'])

        # 获取总数用于分页（根据实际需求决定是否需要）
        # total = query_database("SELECT COUNT(*) FROM jewelries")['count']

        return {
            "data": data,
            "pagination": {
                "page": page,
                "page_size": page_size,
                # "total": total,
                # "total_pages": math.ceil(total / page_size)
            }
        }

    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"数据库查询失败: {str(e)}"
        )


# 3. 珠宝详情查询
@app.get("/jewelries/{jewelry_id}", summary="获取珠宝详情")
async def get_jewelry_detail(
        jewelry_id: int = Path(..., gt=0, description="珠宝ID")
):
    """
    获取珠宝商品详细信息（包含图片列表和收藏数）
    """
    query = """
    SELECT
      j.jewelry_id,
      j.name,
      j.description,
      j.original_price,
      j.discounted_price,
      j.stock,
      j.is_hot,
      m.name AS merchant_name,
      m.contact_phone AS merchant_phone,
      JSON_ARRAYAGG(ji.image_url) AS images,
      (SELECT COUNT(*) FROM favorites WHERE jewelry_id = j.jewelry_id) AS favorite_count,
      (SELECT AVG(rating) FROM reviews WHERE jewelry_id = j.jewelry_id) AS avg_rating
    FROM jewelries j
    JOIN merchants m ON j.merchant_id = m.merchant_id
    LEFT JOIN jewelry_images ji ON j.jewelry_id = ji.jewelry_id
    WHERE j.jewelry_id = %s
    GROUP BY j.jewelry_id
    """
    try:
        data = query_database(query, (jewelry_id,))
        if not data:
            raise HTTPException(status_code=404, detail="商品不存在")

        result = data[0]
        # 字段类型转换
        result['is_hot'] = bool(result['is_hot'])
        if isinstance(result['images'], str):
            result['images'] = json.loads(result['images'])

        return {"data": result}
    except HTTPException as e:
        return {"status": "error", "message": e.detail}


@app.post("/orders")
async def create_order(order_data: OrderCreate):
    connection = None
    try:
        connection = pymysql.connect(**DB_CONFIG)
        connection.begin()
        cursor = connection.cursor()

        # 验证用户地址
        cursor.execute(
            "SELECT address_id FROM user_addresses "
            "WHERE user_id = %s AND is_default = 1",
            (order_data.user_id,)
        )
        address = cursor.fetchone()
        if not address:
            raise HTTPException(400, "请先设置默认收货地址")
        address_id = address['address_id']

        # 生成订单号
        order_number = f"ORD{int(time.time())}{random.randint(1000, 9999)}"

        # 处理商品信息
        total_amount = Decimal('0')  # 修改为 Decimal 类型
        items_info = []
        for item in order_data.items:
            # 获取商品信息和锁库存
            cursor.execute(
                "SELECT stock, discounted_price, original_price "
                "FROM jewelries WHERE jewelry_id = %s FOR UPDATE",
                (item.jewelry_id,)
            )
            jewelry = cursor.fetchone()
            if not jewelry:
                raise HTTPException(404, f"商品不存在 ID:{item.jewelry_id}")

            if jewelry['stock'] < item.quantity:
                raise HTTPException(400, f"商品库存不足 ID:{item.jewelry_id}")

            # 计算价格
            price = Decimal(str(jewelry['discounted_price'])) if jewelry['discounted_price'] else Decimal(
                str(jewelry['original_price']))
            total_amount += price * Decimal(item.quantity)

            # 更新库存
            new_stock = jewelry['stock'] - item.quantity
            cursor.execute(
                "UPDATE jewelries SET stock = %s WHERE jewelry_id = %s",
                (new_stock, item.jewelry_id)
            )

            items_info.append({
                "jewelry_id": item.jewelry_id,
                "quantity": item.quantity,
                "unit_price": float(price)  # 保持与数据库的 DECIMAL 类型兼容
            })

        # 创建订单
        cursor.execute(
            "INSERT INTO orders "
            "(user_id, address_id, order_number, payment_method, total_amount) "
            "VALUES (%s, %s, %s, %s, %s)",
            (
                order_data.user_id,
                address_id,
                order_number,
                order_data.payment_method,
                total_amount
            )
        )
        order_id = cursor.lastrowid

        # 添加订单商品
        for item in items_info:
            cursor.execute(
                "INSERT INTO order_items "
                "(order_id, jewelry_id, quantity, unit_price) "
                "VALUES (%s, %s, %s, %s)",
                (
                    order_id,
                    item['jewelry_id'],
                    item['quantity'],
                    item['unit_price']
                )
            )

        connection.commit()
        return {"status": "success", "order_number": order_number}

    except HTTPException as he:
        if connection:
            connection.rollback()
        raise he
    except Exception as e:
        if connection:
            connection.rollback()
        raise HTTPException(500, f"创建订单失败: {str(e)}")
    finally:
        if connection:
            connection.close()


@app.get("/orders/{order_number}")
async def get_order_detail(
        order_number: str = Path(..., description="订单号"),
        user_id: int = Query(..., description="用户ID")
):
    try:
        # 查询订单基本信息
        order_query = """
        SELECT 
            o.order_number, o.payment_method, o.total_amount,
            o.order_status, o.tracking_number, o.shipping_company,
            o.created_at, o.paid_at, o.shipped_at, o.completed_at,
            ua.recipient, ua.phone, ua.address
        FROM orders o
        JOIN user_addresses ua ON o.address_id = ua.address_id
        WHERE o.order_number = %s AND o.user_id = %s
        """
        order_info = query_database(order_query, (order_number, user_id))
        if not order_info:
            raise HTTPException(404, "订单不存在或权限不足")

        # 查询订单商品
        items_query = """
        SELECT 
            j.name, oi.unit_price, oi.quantity,
            (oi.unit_price * oi.quantity) AS total_price
        FROM order_items oi
        JOIN jewelries j ON oi.jewelry_id = j.jewelry_id
        WHERE oi.order_id = (
            SELECT order_id FROM orders WHERE order_number = %s
        )
        """
        items = query_database(items_query, (order_number,))

        return {
            "status": "success",
            "order_info": order_info[0],
            "items": items
        }
    except Exception as e:
        raise HTTPException(500, f"查询失败: {str(e)}")


# 用户收藏列表
@app.get("/users/{user_id}/favorites")
async def get_favorites(
    user_id: int = Path(..., title="用户ID")
):
    query = """
    SELECT
      j.jewelry_id,
      j.name,
      m.name AS merchant_name,
      ji.image_url AS main_image,
      j.discounted_price
    FROM favorites f
    JOIN jewelries j USING(jewelry_id)
    JOIN merchants m USING(merchant_id)
    JOIN (
      SELECT jewelry_id, MIN(image_url) AS image_url
      FROM jewelry_images GROUP BY jewelry_id
    ) ji USING(jewelry_id)
    WHERE f.user_id = %s
    """
    try:
        data = query_database(query, (user_id,))
        return {"status": "success", "data": data}
    except Exception as e:
        return {"status": "error", "message": str(e)}

# 管理端珠宝搜索
@app.get("/admin/jewelries/search")
async def jewelry_search(
    category_id: Optional[int] = None,
    merchant_id: Optional[int] = None
):
    query = """
    SELECT
      j.jewelry_id,
      j.name,
      c.category_name,
      m.name AS merchant_name,
      j.stock,
      j.is_hot
    FROM jewelries j
    JOIN merchants m USING(merchant_id)
    JOIN jewelry_categories c USING(category_id)
    WHERE
      (%s IS NULL OR j.category_id = %s)
      AND (%s IS NULL OR j.merchant_id = %s)
    ORDER BY j.created_at DESC
    LIMIT 20
    """
    try:
        data = query_database(query, (category_id, category_id, merchant_id, merchant_id))
        return {"status": "success", "data": data}
    except Exception as e:
        return {"status": "error", "message": str(e)}


@app.post("/reviews")
async def create_review(review: ReviewCreate):
    try:
        # 验证订单有效性
        order_query = """
        SELECT o.order_id, o.user_id, o.order_status, oi.jewelry_id 
        FROM orders o
        JOIN order_items oi ON o.order_id = oi.order_id
        WHERE o.order_number = %s
        """
        orders = query_database(order_query, (review.order_number,))

        if not orders:
            raise HTTPException(404, "订单不存在")

        order = orders[0]

        # 验证所有权
        if order["user_id"] != review.user_id:
            raise HTTPException(403, "无权操作此订单")

        # 验证订单状态
        if order["order_status"] != "completed":
            raise HTTPException(400, "仅可评论已完成订单")

        # 检查是否已评论
        check_query = "SELECT review_id FROM reviews WHERE order_id = %s"
        if query_database(check_query, (order["order_id"],)):
            raise HTTPException(409, "该订单已评价")

        # 插入评论
        insert_query = """
        INSERT INTO reviews 
        (user_id, order_id, jewelry_id, rating, content)
        VALUES (%s, %s, %s, %s, %s)
        """
        with pymysql.connect(**DB_CONFIG) as conn:
            with conn.cursor() as cursor:
                cursor.execute(insert_query, (
                    review.user_id,
                    order["order_id"],
                    order["jewelry_id"],
                    review.rating,
                    review.content
                ))
                conn.commit()

        return {"status": "success"}

    except KeyError as e:
        raise HTTPException(400, f"数据异常: {e}")


# ======================
# 2. 商品评论查询
# ======================
@app.get("/jewelries/{jewelry_id}/reviews")
async def get_jewelry_reviews(
        jewelry_id: int = Path(..., gt=0),
        page: int = 1,
        page_size: int = 10
):
    try:
        # 基础查询
        base_query = """
        SELECT 
            u.username,
            r.rating,
            r.content,
            r.created_at
        FROM reviews r
        JOIN users u ON r.user_id = u.user_id
        WHERE r.jewelry_id = %s
        ORDER BY r.created_at DESC
        LIMIT %s OFFSET %s
        """
        offset = (page - 1) * page_size
        reviews = query_database(base_query, (jewelry_id, page_size, offset))

        # 总数查询
        count_query = "SELECT COUNT(*) AS total FROM reviews WHERE jewelry_id = %s"
        total = query_database(count_query, (jewelry_id,))[0]["total"]

        return {
            "status": "success",
            "data": reviews,
            "pagination": {
                "total": total,
                "page": page,
                "page_size": page_size
            }
        }
    except Exception as e:
        raise HTTPException(500, str(e))


# ======================
# 3. 热销商品统计
# ======================
@app.get("/analytics/hot-sales")
async def get_hot_sales(
        limit: int = 10,
        days: Optional[int] = None
):
    try:
        base_query = """
        SELECT
            j.jewelry_id,
            j.name,
            COALESCE(SUM(oi.quantity), 0) AS total_sold,
            COUNT(DISTINCT oi.order_id) AS order_count
        FROM jewelries j
        LEFT JOIN order_items oi ON j.jewelry_id = oi.jewelry_id
        """

        # 添加时间筛选
        if days:
            base_query += """
            LEFT JOIN orders o ON oi.order_id = o.order_id
            WHERE o.created_at >= DATE_SUB(NOW(), INTERVAL %s DAY)
            """
            params = (days,)
        else:
            params = None

        base_query += """
        GROUP BY j.jewelry_id
        ORDER BY total_sold DESC
        LIMIT %s
        """
        final_params = (params + (limit,)) if params else (limit,)

        results = query_database(base_query, final_params)
        return {"status": "success", "data": results}

    except Exception as e:
        raise HTTPException(500, str(e))

# 其他现有接口保持不变...

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:postman

```bash
  # 1. 获取珠宝分类列表（含层级）
http://localhost:8000/categories

  {
    "data": [
        {
            "category_id": 1,
            "category_name": "项链",
            "children": [
                {
                    "category_id": 5,
                    "category_name": "黄金项链"
                },
                {
                    "category_id": 6,
                    "category_name": "钻石项链"
                }
            ]
        },
        {
            "category_id": 2,
            "category_name": "戒指",
            "children": [
                {
                    "category_id": 7,
                    "category_name": "翡翠戒指"
                },
                {
                    "category_id": 8,
                    "category_name": "铂金戒指"
                }
            ]
        },
        {
            "category_id": 3,
            "category_name": "耳环",
            "children": [
                {
                    "category_id": 9,
                    "category_name": "珍珠耳环"
                },
                {
                    "category_id": 10,
                    "category_name": "银饰耳环"
                }
            ]
        },
        {
            "category_id": 4,
            "category_name": "手链",
            "children": [
                {
                    "category_id": 11,
                    "category_name": "儿童手链"
                },
                {
                    "category_id": 12,
                    "category_name": "情侣手链"
                }
            ]
        }
    ]
}

2.0 获取所有的珠宝列表 http://localhost:8000/categories/jewelries 

# 2. 获取指定分类的珠宝列表

 http://localhost:8000/categories/5/jewelries

 {
    "data": [
        {
            "jewelry_id": 1,
            "name": "足金福牌项链",
            "merchant_name": "周大福珠宝",
            "original_price": 2999.0,
            "discounted_price": 2799.0,
            "images": [
                "https://example.com/j1_1.jpg",
                "https://example.com/j1_2.jpg"
            ],
            "favorite_count": 2
        },
        {
            "jewelry_id": 9,
            "name": "古法黄金项链",
            "merchant_name": "钻石世家",
            "original_price": 3599.0,
            "discounted_price": 3299.0,
            "images": [
                "https://example.com/j9_1.jpg"
            ],
            "favorite_count": 1
        }
    ],
    "pagination": {
        "page": 1,
        "page_size": 10
    }
}


 http://localhost:8000/jewelries/1

 {
    "data": {
        "jewelry_id": 1,
        "name": "足金福牌项链",
        "description": "999足金福牌 工费全免",
        "original_price": 2999.0,
        "discounted_price": 2799.0,
        "stock": 98,
        "is_hot": true,
        "merchant_name": "周大福珠宝",
        "merchant_phone": "13800000001",
        "images": [
            "https://example.com/j1_1.jpg",
            "https://example.com/j1_2.jpg"
        ],
        "favorite_count": 1,
        "avg_rating": 5.0
    }
}


 http://localhost:8000/orders


 {
    "user_id": 1,
    "payment_method": "alipay",
    "items": [
        {"jewelry_id": 1, "quantity": 2},
        {"jewelry_id": 3, "quantity": 1}
    ]
}

{
    "status": "success",
    "order_number": "ORD17424341548287"
}

http://localhost:8000/orders/ORD17424341548287?user_id=1

{
    "status": "success",
    "order_info": {
        "order_number": "ORD17424341548287",
        "payment_method": "alipay",
        "total_amount": 19398.0,
        "order_status": "pending",
        "tracking_number": null,
        "shipping_company": null,
        "created_at": "2025-03-20T09:29:14",
        "paid_at": null,
        "shipped_at": null,
        "completed_at": null,
        "recipient": "张三",
        "phone": "13900000001",
        "address": "北京市朝阳区XX小区1号楼"
    },
    "items": [
        {
            "name": "足金福牌项链",
            "unit_price": 2799.0,
            "quantity": 2,
            "total_price": 5598.0
        },
        {
            "name": "冰种翡翠戒指",
            "unit_price": 13800.0,
            "quantity": 1,
            "total_price": 13800.0
        }
    ]
}



  http://localhost:8000/users/1/favorites

  {
    "status": "success",
    "data": [
        {
            "jewelry_id": 3,
            "name": "冰种翡翠戒指",
            "merchant_name": "六福珠宝",
            "main_image": "https://example.com/j3_1.jpg",
            "discounted_price": 13800.0
        },
        {
            "jewelry_id": 5,
            "name": "南洋珍珠耳环",
            "merchant_name": "谢瑞麟",
            "main_image": "https://example.com/j5_1.jpg",
            "discounted_price": null
        }
    ]
}


  http://localhost:8000/admin/jewelries/search?category_id=5&merchant_id=1

  {
    "status": "success",
    "data": [
        {
            "jewelry_id": 1,
            "name": "足金福牌项链",
            "category_name": "黄金项链",
            "merchant_name": "周大福珠宝",
            "stock": 96,
            "is_hot": 1
        }
    ]
}


  http://localhost:8000/reviews

  {
    "user_id": 1,
    "order_number": "202308011001",
    "rating": 5,
    "content": "我大飞非常满意的购物体验"
}

{
    "detail": "该订单已评价"
}

  

 商品评论查询  GET  http://localhost:8000/jewelries/5/reviews

  {
    "status": "success",
    "data": [
        {
            "username": "user10",
            "rating": 4,
            "content": "珍珠光泽不错，包装精美",
            "created_at": "2025-03-19T14:49:04"
        }
    ],
    "pagination": {
        "total": 1,
        "page": 1,
        "page_size": 10
    }
}


  http://localhost:8000/analytics/hot-sales


  {
    "status": "success",
    "data": [
        {
            "jewelry_id": 1,
            "name": "足金福牌项链",
            "total_sold": 6,
            "order_count": 4
        },
        {
            "jewelry_id": 3,
            "name": "冰种翡翠戒指",
            "total_sold": 2,
            "order_count": 2
        },
        {
            "jewelry_id": 6,
            "name": "S925银耳钉",
            "total_sold": 2,
            "order_count": 1
        },
        {
            "jewelry_id": 8,
            "name": "情侣红绳手链",
            "total_sold": 2,
            "order_count": 2
        },
        {
            "jewelry_id": 4,
            "name": "铂金素圈对戒",
            "total_sold": 1,
            "order_count": 1
        },
        {
            "jewelry_id": 5,
            "name": "南洋珍珠耳环",
            "total_sold": 1,
            "order_count": 1
        },
        {
            "jewelry_id": 7,
            "name": "宝宝平安手链",
            "total_sold": 1,
            "order_count": 1
        },
        {
            "jewelry_id": 9,
            "name": "古法黄金项链",
            "total_sold": 1,
            "order_count": 1
        },
        {
            "jewelry_id": 10,
            "name": "群镶钻石项链",
            "total_sold": 1,
            "order_count": 1
        },
        {
            "jewelry_id": 2,
            "name": "星耀钻石项链",
            "total_sold": 0,
            "order_count": 0
        }
    ]
}


```

 step1:service

```typescript
// jewelry.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class JewelryService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  // 获取所有分类
  getCategories(): Observable<any> {
    return this.http.get(`${this.apiUrl}/categories`);
  }

  // 获取分类下的商品
  getCategoryJewelries(categoryId: number, page: number = 1, pageSize: number = 10): Observable<any> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('page_size', pageSize.toString());

    return this.http.get(`${this.apiUrl}/categories/${categoryId}/jewelries`, { params });
  }

  // 获取商品详情
  getJewelryDetails(jewelryId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/jewelries/${jewelryId}`);
  }

  // 提交订单
  submitOrder(orderData: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/orders`, orderData);
  }

  // 获取订单详情
  getOrderDetails(orderNumber: string, userId: number): Observable<any> {
    const params = new HttpParams().set('user_id', userId.toString());
    return this.http.get(`${this.apiUrl}/orders/${orderNumber}`, { params });
  }

  // 获取用户收藏
  getUserFavorites(userId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/users/${userId}/favorites`);
  }

  // 管理员商品搜索
  adminSearchJewelries(filters: any): Observable<any> {
    const params = new HttpParams({ fromObject: filters });
    return this.http.get(`${this.apiUrl}/admin/jewelries/search`, { params });
  }

  // 提交商品评价
  submitReview(reviewData: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/reviews`, reviewData);
  }

  // 获取商品评价
  getJewelryReviews(jewelryId: number, page: number = 1, pageSize: number = 10): Observable<any> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('page_size', pageSize.toString());

    return this.http.get(`${this.apiUrl}/jewelries/${jewelryId}/reviews`, { params });
  }

  // 获取热销统计
  getHotSalesAnalytics(): Observable<any> {
    return this.http.get(`${this.apiUrl}/analytics/hot-sales`);
  }

  // 添加收藏（示例中没有但可能需要）
  addToFavorites(userId: number, jewelryId: number): Observable<any> {
    return this.http.post(`${this.apiUrl}/users/${userId}/favorites`, { jewelry_id: jewelryId });
  }

  // 更新库存（示例中没有但可能需要）
  updateStock(jewelryId: number, stock: number): Observable<any> {
    return this.http.patch(`${this.apiUrl}/admin/jewelries/${jewelryId}`, { stock });
  }
}
```

step1:ts

```typescript
// 更新后的组件代码
import { Component } from '@angular/core';
import { JewelryService } from '../service/jewelry.service';
import {FormsModule} from '@angular/forms';
import {NgIf} from '@angular/common';

@Component({
  selector: 'app-jewelries-list',
  templateUrl: './jewelries-list.component.html',
  imports: [
    FormsModule,
    NgIf
  ],
  styleUrls: ['./jewelries-list.component.css']
})
export class JewelriesListComponent {
  isLoading = false;
  notification: { type: string, message: string } | null = null;

  // 订单相关参数
  orderUser = 1;
  paymentMethod = 'alipay';

  // 搜索参数
  searchCategory?: number;
  searchMerchant?: number;

   constructor(private jewelryService: JewelryService) {}


  // 获取分类
  loadCategories() {
    this.jewelryService.getCategories().subscribe(data => {
      console.log('分类数据:', data);
    });
  }

  // 获取黄金项链商品
  loadGoldNecklaces() {
    this.jewelryService.getCategoryJewelries(5).subscribe(data => {
      console.log('黄金项链商品:', data);
    });
  }

  // 提交订单
  createOrder() {
    const orderData = {
      user_id: 1,
      payment_method: "alipay",
      items: [
        { jewelry_id: 1, quantity: 2 },
        { jewelry_id: 3, quantity: 1 }
      ]
    };

    this.jewelryService.submitOrder(orderData).subscribe(response => {
      console.log('订单提交成功:', response);
    });
  }

  // 获取热销统计
  loadAnalytics() {
    this.jewelryService.getHotSalesAnalytics().subscribe(data => {
      console.log('热销商品:', data);
    });
  }

  // 管理员搜索示例
  adminSearch() {
    const filters = {
      category_id: 5,
      merchant_id: 1
    };

    this.jewelryService.adminSearchJewelries(filters).subscribe(results => {
      console.log('搜索结果:', results);
    });
  }

}
```

step1:html

```xml
<!-- jewelries-list.component.html -->
<div class="container">
  <!-- 操作通知 -->
  <div *ngIf="notification" class="notification {{notification.type}}">
    {{notification.message}}
  </div>

  <!-- 加载状态 -->
  <div *ngIf="isLoading" class="loading-overlay">
    <div class="loader"></div>
  </div>

  <!-- 分类加载区块 -->
  <section class="card">
    <h2>分类操作</h2>
    <button class="btn primary" (click)="loadCategories()">加载所有分类</button>
  </section>

  <!-- 商品加载区块 -->
  <section class="card">
    <h2>商品操作</h2>
    <div class="input-group">
      <input type="number" #categoryId placeholder="分类ID" value="5">
      <button class="btn success" (click)="loadGoldNecklaces()">加载黄金项链</button>
    </div>
  </section>

  <!-- 订单提交区块 -->
  <section class="card">
    <h2>订单操作</h2>
    <div class="form-group">
      <input type="number" [(ngModel)]="orderUser" placeholder="用户ID">
      <select [(ngModel)]="paymentMethod">
        <option value="alipay">支付宝</option>
        <option value="wechat">微信支付</option>
      </select>
      <button class="btn warning" (click)="createOrder()">提交订单</button>
    </div>
  </section>

  <!-- 管理员搜索区块 -->
  <section class="card">
    <h2>管理员搜索</h2>
    <div class="search-form">
      <input type="number" [(ngModel)]="searchCategory" placeholder="分类ID">
      <input type="number" [(ngModel)]="searchMerchant" placeholder="商家ID">
      <button class="btn danger" (click)="adminSearch()">执行搜索</button>
    </div>
  </section>

  <!-- 数据分析区块 -->
  <section class="card">
    <h2>数据分析</h2>
    <button class="btn info" (click)="loadAnalytics()">获取热销统计</button>
  </section>
</div>
```

step1:css

```css
/* jewelries-list.component.css */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  display: grid;
  gap: 1.5rem;
}

.card {
  background: white;
  border-radius: 10px;
  padding: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.card h2 {
  color: #333;
  margin-bottom: 1rem;
  font-size: 1.25rem;
  border-bottom: 2px solid #eee;
  padding-bottom: 0.5rem;
}

.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 14px;
}

.btn.primary { background: #007bff; color: white; }
.btn.success { background: #28a745; color: white; }
.btn.warning { background: #ffc107; color: black; }
.btn.danger { background: #dc3545; color: white; }
.btn.info { background: #17a2b8; color: white; }

.btn:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

.input-group {
  display: flex;
  gap: 10px;
}

input, select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 5px;
  flex: 1;
  min-width: 120px;
}

.form-group {
  display: grid;
  gap: 10px;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
}

.search-form {
  display: grid;
  gap: 10px;
  grid-template-columns: repeat(2, 1fr);
}

.notification {
  padding: 15px;
  border-radius: 5px;
  margin-bottom: 1rem;
  position: fixed;
  top: 20px;
  right: 20px;
  color: white;
  z-index: 1000;
}

.notification.success { background: #28a745; }
.notification.error { background: #dc3545; }

.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255,255,255,0.8);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 999;
}

.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
```

step:service C:\Users\wangrusheng\WebstormProjects\untitled5\src\app\service\jewelry.service.ts

```typescript
// jewelry.service.ts
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class JewelryService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  // 获取所有分类
  getCategories(): Observable<any> {
    return this.http.get(`${this.apiUrl}/categories`);
  }

  // 获取所有的珠宝列表
  getAllCategoryJewelries(): Observable<any> {
    return this.http.get(`${this.apiUrl}/categories/jewelries`);
  }

  // 获取分类下的商品
  getCategoryJewelries(categoryId: number, page: number = 1, pageSize: number = 10): Observable<any> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('page_size', pageSize.toString());

    return this.http.get(`${this.apiUrl}/categories/${categoryId}/jewelries`, { params });
  }

  // 获取商品详情
  getJewelryDetails(jewelryId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/jewelries/${jewelryId}`);
  }

  // 提交订单
  submitOrder(orderData: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/orders`, orderData);
  }

  // 获取订单详情
  getOrderDetails(orderNumber: string, userId: number): Observable<any> {
    const params = new HttpParams().set('user_id', userId.toString());
    return this.http.get(`${this.apiUrl}/orders/${orderNumber}`, { params });
  }

  // 获取用户收藏
  getUserFavorites(userId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/users/${userId}/favorites`);
  }

  // 管理员商品搜索
  adminSearchJewelries(filters: any): Observable<any> {
    const params = new HttpParams({ fromObject: filters });
    return this.http.get(`${this.apiUrl}/admin/jewelries/search`, { params });
  }

  // 提交商品评价
  submitReview(reviewData: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/reviews`, reviewData);
  }

  // 获取商品评价
  getJewelryReviews(jewelryId: number, page: number = 1, pageSize: number = 10): Observable<any> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('page_size', pageSize.toString());

    return this.http.get(`${this.apiUrl}/jewelries/${jewelryId}/reviews`, { params });
  }

  // 获取热销统计
  getHotSalesAnalytics(): Observable<any> {
    return this.http.get(`${this.apiUrl}/analytics/hot-sales`);
  }

  // 添加收藏（示例中没有但可能需要）
  addToFavorites(userId: number, jewelryId: number): Observable<any> {
    return this.http.post(`${this.apiUrl}/users/${userId}/favorites`, { jewelry_id: jewelryId });
  }

  // 更新库存（示例中没有但可能需要）
  updateStock(jewelryId: number, stock: number): Observable<any> {
    return this.http.patch(`${this.apiUrl}/admin/jewelries/${jewelryId}`, { stock });
  }

  getJewelries() {
    /*这种写法保留  以后碰到复杂json 可以写成本地模拟数据，不需要从后端拿*/
    return {
      data: [
        {
          jewelry_id: 1,
          name: '大飞',
          merchant_name: '周大福珠宝',
          original_price: 2999.0,
          discounted_price: 2799.0,
          images: ['https://example.com/j1_1.jpg', 'https://example.com/j1_2.jpg'],
          favorite_count: 2
        },
        {
          jewelry_id: 9,
          name: '古法黄金项链',
          merchant_name: '钻石世家',
          original_price: 3599.0,
          discounted_price: 3299.0,
          images: ['https://example.com/j9_1.jpg'],
          favorite_count: 1
        }
      ],
      pagination: {
        page: 1,
        page_size: 10
      }
    };
  }
}

```

step:model C:\Users\wangrusheng\WebstormProjects\untitled5\src\app\model\jewelry.model.ts

```typescript
export interface Category {
  category_id: number;
  category_name: string;
  children?: Category[];
}

export interface Jewelry {
  jewelry_id: number;
  name: string;
  merchant_name: string;
  original_price: number;
  discounted_price: number;
  images: string[];
  favorite_count: number;
}

export interface JewelryListResponse {
  data: Jewelry[];
  pagination: {
    page: number;
    page_size: number;
  };
}


export interface Pagination {
  page: number;
  page_size: number;
}

```

step:teee

```typescript
import {Component, Input, Output, EventEmitter, ViewEncapsulation, OnInit} from '@angular/core';
import { Category } from '../model/jewelry.model';
import {NgForOf, NgIf} from '@angular/common';


// Internal node component
@Component({
  selector: 'tree-node',
  template: `
    <li class="tree-node" [class.leaf]="!node.children">
      <div class="node-content">
        <span class="toggle-icon"
              *ngIf="hasChildren"
              (click)="toggle($event)">
          {{ isOpen ? '−' : '+' }}
        </span>

        <span class="category-name"
              (click)="selectNode($event)">
          {{ node.category_name }}
        </span>
      </div>



      <ul class="children" *ngIf="hasChildren && isOpen">
        <tree-node
          *ngFor="let child of node.children"
          [node]="child"
          [depth]="depth + 1"
          (nodeSelected)="onChildSelect($event)">
        </tree-node>
      </ul>
    </li>
  `,
  imports: [
    NgForOf,
    NgIf
  ],
  styles: [`
    .tree-node {
      margin: 4px 0;
      transition: all 0.2s ease;
    }

    .node-content {
      display: flex;
      align-items: center;
      padding: 8px 12px;
      border-radius: 6px;
      cursor: pointer;
      transition: background-color 0.2s;
    }

    .node-content:hover {
      background-color: #f5f7fa;
    }

    .tree-node:not(.leaf) .node-content:hover {
      background-color: #ebf4ff;
    }

    .toggle-icon {
      display: inline-flex;
      justify-content: center;
      width: 24px;
      height: 24px;
      margin-right: 8px;
      font-size: 1.2em;
      color: #409eff;
      transition: transform 0.2s;
    }

    .category-name {
      font-size: 15px;
      color: #2c3e50;
      font-weight: 500;
      transition: color 0.2s;
    }

    .category-name:hover {
      color: #409eff;
    }

    .children {
      list-style: none;
      padding-left: 28px;
      margin: 4px 0;
      border-left: 2px solid #e0e6ed;
      transition: border-color 0.2s;
      animation: slideDown 0.3s ease;
    }

    .tree-node.leaf .node-content {
      padding-left: 36px;
      opacity: 0.9;
    }

    .tree-node.leaf .category-name {
      font-weight: 400;
      color: #5e6d82;
    }

    .tree-node.leaf .category-name:hover {
      color: #67c23a;
    }

    @keyframes slideDown {
      from { opacity: 0; transform: translateY(-10px); }
      to { opacity: 1; transform: translateY(0); }
    }
  `]
})
export class TreeNodeComponent {
  @Input() node!: Category;
  @Input() depth = 0;
  @Output() nodeSelected = new EventEmitter<number>();

  isOpen = false;

  get hasChildren(): boolean {
    return !!this.node.children?.length;
  }

  toggle(event: MouseEvent) {
    if (this.hasChildren) {
      this.isOpen = !this.isOpen;
    }

    // 阻止事件冒泡到父元素
    event.stopPropagation();
  }

  selectNode(event: MouseEvent) {
    this.nodeSelected.emit(this.node.category_id);
    console.log('Selected Category ID:', this.node.category_id);

    // 添加阻止事件冒泡
    event.stopPropagation();
  }

  onChildSelect(categoryId: number) {
    this.nodeSelected.emit(categoryId);
  }
}


@Component({
  selector: 'app-category-tree',
  imports: [
    NgForOf,
    TreeNodeComponent
  ],
  template: `
    <ul class="category-tree">
      <tree-node
        *ngFor="let node of data"
        [node]="node"
        [depth]="0"
        (nodeSelected)="handleNodeSelect($event)">
      </tree-node>
    </ul>
  `,
  styles: [`
    .category-tree {
      list-style: none;
      padding: 12px;
      background: #ffffff;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
      font-family: 'Segoe UI', system-ui, sans-serif;
    }

    @media (max-width: 768px) {
      .category-tree {
        padding: 8px;
      }
    }
  `]
})
export class CategoryTreeComponent implements OnInit{
  @Input() data: Category[] = [];
  @Output() categorySelected = new EventEmitter<number>();

  // 默认数据配置
  private defaultData: Category[] = [
    {
      category_id: 1,
      category_name: 'Necklaces',
      children: [
        { category_id: 5, category_name: 'Gold Necklaces' },
        { category_id: 6, category_name: 'Diamond Necklaces' }
      ]
    },
    {
      category_id: 2,
      category_name: 'Rings',
      children: [
        { category_id: 7, category_name: 'Jade Rings' },
        { category_id: 8, category_name: 'Platinum Rings' }
      ]
    },
    {
      category_id: 3,
      category_name: 'Earrings',
      children: [
        { category_id: 9, category_name: 'Pearl Earrings' },
        { category_id: 10, category_name: 'Silver Earrings' }
      ]
    },
    {
      category_id: 4,
      category_name: 'Bracelets',
      children: [
        { category_id: 11, category_name: "Children's Bracelets" },
        { category_id: 12, category_name: 'Couple Bracelets' }
      ]
    }
  ];
  ngOnInit(): void {
    console.log('Received data:', this.data); // 调试输入数据
    if (!this.data || this.data.length === 0) {
      console.log('Using default data');
      this.data = [...this.defaultData];
    }
  }

  handleNodeSelect(categoryId: number) {
    this.categorySelected.emit(categoryId);

    // 添加控制台输出
    console.log('Tree Node Selected:', categoryId);
  }
}

```

step:parent 

```typescript
 import {Component, OnInit} from '@angular/core';
import {Category, Jewelry, Pagination} from '../model/jewelry.model';
import { JewelryService } from '../service/jewelry.service';
import { CategoryTreeComponent } from '../category-tree/category-tree.component';
import {DecimalPipe, NgForOf} from "@angular/common";


@Component({
  selector: 'app-parent',
  standalone: true,
  imports: [CategoryTreeComponent, DecimalPipe, NgForOf], // 导入子组件
  templateUrl: './jewelry-list.component.html',
  styleUrl: './jewelry-list.component.css'
})
export class JewelryListComponent implements OnInit {
  categories: Category[] = []; // 初始化空数组
  jewelryData: Jewelry[] = [];
  pagination: Pagination = {
    page: 1,
    page_size: 10
  };

  constructor(private jewelryService:JewelryService) {
  }

  // 2. 动态数据加载示例
  loadCategoriesFromService() {
    this.jewelryService.getCategories().subscribe(res => {
      console.log("珠宝分类结果",res)
      this.categories = res.data;
      // 确保数据结构匹配
      if (res.data && Array.isArray(res.data)) {
        this.categories = res.data;
      }
    });
  }

  // 3. 事件处理
  onCategorySelect(categoryId: number) {
    console.log('父组件收到分类ID:', categoryId);
    // 执行相关业务逻辑，如：
    this.loadProducts(categoryId);
  }

  // 4. 方法示例
  private loadProducts(categoryId: number) {
    console.log("珠宝id",categoryId)
    this.jewelryService.getCategoryJewelries(categoryId).subscribe(res => {
      console.log("根据id查询珠宝",res)
      this.jewelryData = res.data;
      this.pagination = res.pagination;
    });
  }

  ngOnInit(): void {
    /*默认加载 请求数据*/
    this.loadCategoriesFromService();  //加载珠宝分类
   // this.loadData();
   this.loadAllJewelries();
  }


  loadAllJewelries() {
    this.jewelryService.getAllCategoryJewelries().subscribe({
      next: (res) => {
        console.log("获取所有的珠宝列表", res);
        this.jewelryData = res.data;

        // 合并分页参数，保留可能存在的默认值
        this.pagination = {
          ...this.pagination, // 保留已有默认值
          ...res.pagination   // 覆盖服务返回的新值
        };
      },
      error: (err) => {
        console.error("加载珠宝列表失败", err);
        // 可以在此重置分页数据
        this.pagination = {
          page: 1,
          page_size: 10
        };
      }
    });
  }
}


<!-- jewelry-list.component.html -->
<div class="main-container">
    <!-- 左侧分类树 -->
    <div class="category-panel">
        <button class="all-products-btn" (click)="loadAllJewelries()">
            <span class="icon">🏷️</span>
            全部商品
        </button>
        <app-category-tree
                [data]="categories"
                (categorySelected)="onCategorySelect($event)">
        </app-category-tree>
    </div>

    <!-- 右侧内容区域 -->
    <div class="content-panel">
        <div class="jewelry-grid">
            <div *ngFor="let item of jewelryData" class="jewelry-card">
                <!-- 原有卡片内容保持不变 -->
                <div class="image-gallery">
                    <img *ngFor="let img of item.images" [src]="img" alt="{{item.name}}" class="product-image">
                </div>
                <div class="product-info">
                    <h2 class="product-title">{{ item.name }}</h2>
                    <p class="merchant">商家：{{ item.merchant_name }}</p>

                    <div class="pricing">
                        <span class="original-price">¥{{ item.original_price | number }}</span>
                        <span class="discounted-price">¥{{ item.discounted_price | number }}</span>
                    </div>

                    <div class="favorite-section">
                        <button class="favorite-btn">
                            ♥ {{ item.favorite_count }}
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <div class="pagination-info">
            当前页码：{{ pagination.page }}，每页数量：{{ pagination.page_size }}
        </div>
    </div>
</div>


.container {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.jewelry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 2rem;
}

.jewelry-card {
  background: white;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  overflow: hidden;
}

.image-gallery {
  position: relative;
  height: 250px;

  .product-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-bottom: 1px solid #eee;
  }
}

.product-info {
  padding: 1.5rem;

  .product-title {
    font-size: 1.2rem;
    margin: 0 0 0.5rem;
    color: #333;
  }

  .merchant {
    color: #666;
    font-size: 0.9rem;
    margin: 0 0 1rem;
  }
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
  margin-left: 1px;
  border-radius: 4px;
  cursor: pointer;
  transition: opacity 0.3s;
}

.pricing {
  margin: 1rem 0;

  .original-price {
    text-decoration: line-through;
    color: #999;
    margin-right: 1rem;
  }

  .discounted-price {
    color: #e53935;
    font-weight: bold;
    font-size: 1.2rem;
  }
}

.favorite-btn {
  background: none;
  border: 1px solid #ff4081;
  color: #ff4081;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s;

  &:hover {
    background: #ff4081;
    color: white;
  }
}

.pagination-info {
  text-align: center;
  margin-top: 2rem;
  color: #666;
}
   .main-container {
     display: flex;
     min-height: 100vh;
     background: #f5f5f5;
   }

.category-panel {
  width: 280px;
  background: white;
  border-right: 1px solid #e0e0e0;
  padding: 20px;
  box-shadow: 2px 0 8px rgba(0,0,0,0.05);
  overflow-y: auto;
}

.content-panel {
  flex: 1;
  padding: 24px;
  max-width: calc(100% - 280px);
}

.jewelry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

/* 保持原有卡片样式 */
.jewelry-card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: transform 0.2s;

  &:hover {
    transform: translateY(-4px);
  }
}

/* 响应式处理 */
@media (max-width: 768px) {
  .main-container {
    flex-direction: column;
  }

  .category-panel {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #e0e0e0;
  }

  .content-panel {
    max-width: 100%;
    padding: 16px;
  }
}

/* 其他原有样式保持不变... */


```


end