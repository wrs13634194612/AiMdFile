说明：fastap+mysql闲鱼系统

1.用户表
2.商品分类  （数码，家居，服装，母婴，食品，厨具，运动器材）
3.闲置商品列表(商品名称 描述 图片url，商品标签，发货方式：包邮 or 自提，收藏数，点击可以收藏)
4.发布闲置商品（弹窗填写 商品名称，商品标签，描述，图片url，用户名，用户头像url ，价格，发布人地址，发货方式：包邮 or 自提）
5.点击商品，可以查看详情，商品介绍，评论列表，可以查看评论和写评论
6.详情页，添加按钮，立即购买
7. 购买页（商品信息展示，商品名，图片url，价格，填写地址，联系人姓名和联系电话，运费价，总价格）
8.提交订单
9.查看订单详情或者记录 （商品基本信息，用户名和收货地址，付款金额，订单号，支付方式，商品快照，物流公司，快递单号，下单时间，发货时间，成交时间）
10.评论表 
11.收藏表
step1:sql

```sql


show databases;

DROP TABLE users;


SHOW CREATE TABLE db_school.users;

show tables;

use db_school;

SELECT * FROM db_school.jewelry_categories;

CREATE DATABASE db_school;

# 1. 用户表 (user)

CREATE TABLE `user` (
  `user_id` INT AUTO_INCREMENT PRIMARY KEY,
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `avatar_url` VARCHAR(255) COMMENT '头像URL',
  `address` VARCHAR(255) COMMENT '用户地址',
  `phone` VARCHAR(20) COMMENT '联系电话',
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间'
);

# 2. 商品分类表 (category)


CREATE TABLE `category` (
  `category_id` INT AUTO_INCREMENT PRIMARY KEY,
  `name` VARCHAR(50) NOT NULL UNIQUE COMMENT '分类名称'
);

-- 初始化数据
INSERT INTO `category` (`name`) VALUES
('数码'), ('家居'), ('服装'), ('母婴'), ('食品'), ('厨具'), ('运动器材');

# 3. 商品表 (product)


CREATE TABLE `product` (
  `product_id` INT AUTO_INCREMENT PRIMARY KEY,
  `user_id` INT NOT NULL COMMENT '发布人ID',
  `category_id` INT NOT NULL COMMENT '分类ID',
  `title` VARCHAR(100) NOT NULL COMMENT '商品名称',
  `description` TEXT COMMENT '商品描述',
  `shipping_method` ENUM('包邮', '自提') NOT NULL COMMENT '发货方式',
  `price` DECIMAL(10,2) NOT NULL COMMENT '价格',
  `publish_address` VARCHAR(255) COMMENT '发布人地址',
  `collect_count` INT DEFAULT 0 COMMENT '收藏数',
  `status` ENUM('在售', '已售', '下架') DEFAULT '在售' COMMENT '商品状态',
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '发布时间',
  `updated_at` DATETIME ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`category_id`) REFERENCES `category`(`category_id`)
);

-- 索引优化
CREATE INDEX `idx_category` ON `product` (`category_id`);
CREATE INDEX `idx_user` ON `product` (`user_id`);

# 4. 商品图片表 (product_image)


CREATE TABLE `product_image` (
  `image_id` INT AUTO_INCREMENT PRIMARY KEY,
  `product_id` INT NOT NULL COMMENT '商品ID',
  `image_url` VARCHAR(255) NOT NULL COMMENT '图片URL',
  `sort_order` INT DEFAULT 0 COMMENT '排序字段',
  FOREIGN KEY (`product_id`) REFERENCES `product`(`product_id`) ON DELETE CASCADE
);

# 5. 标签表 (tag) 与商品标签关联表 (product_tag)


CREATE TABLE `tag` (
  `tag_id` INT AUTO_INCREMENT PRIMARY KEY,
  `name` VARCHAR(50) NOT NULL UNIQUE COMMENT '标签名称'
);

CREATE TABLE `product_tag` (
  `product_id` INT NOT NULL,
  `tag_id` INT NOT NULL,
  PRIMARY KEY (`product_id`, `tag_id`),
  FOREIGN KEY (`product_id`) REFERENCES `product`(`product_id`) ON DELETE CASCADE,
  FOREIGN KEY (`tag_id`) REFERENCES `tag`(`tag_id`) ON DELETE CASCADE
);

# 6. 收藏表 (favorite)


CREATE TABLE `favorite` (
  `favorite_id` INT AUTO_INCREMENT PRIMARY KEY,
  `user_id` INT NOT NULL,
  `product_id` INT NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '收藏时间',
  UNIQUE KEY `uniq_user_product` (`user_id`, `product_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`product_id`) REFERENCES `product`(`product_id`)
);

# 7. 评论表 (comment)

CREATE TABLE `comment` (
  `comment_id` INT AUTO_INCREMENT PRIMARY KEY,
  `user_id` INT NOT NULL,
  `product_id` INT NOT NULL,
  `content` TEXT NOT NULL COMMENT '评论内容',
  `parent_comment_id` INT COMMENT '父评论ID（支持回复）',
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '评论时间',
  FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`product_id`) REFERENCES `product`(`product_id`),
  FOREIGN KEY (`parent_comment_id`) REFERENCES `comment`(`comment_id`) ON DELETE CASCADE
);

-- 索引优化
CREATE INDEX `idx_product` ON `comment` (`product_id`);

# 8. 订单表 (order)


CREATE TABLE `order` (
  `order_id` INT AUTO_INCREMENT PRIMARY KEY,
  `user_id` INT NOT NULL COMMENT '买家ID',
  `product_title` VARCHAR(100) NOT NULL COMMENT '商品名称快照',
  `product_price` DECIMAL(10,2) NOT NULL COMMENT '商品价格快照',
  `product_image_url` VARCHAR(255) COMMENT '商品主图快照',
  `quantity` INT DEFAULT 1 COMMENT '购买数量',
  `total_price` DECIMAL(10,2) NOT NULL COMMENT '总价（含运费）',
  `shipping_fee` DECIMAL(10,2) DEFAULT 0 COMMENT '运费',
  `payment_method` ENUM('支付宝', '微信支付', '银行卡') NOT NULL COMMENT '支付方式',
  `order_status` ENUM('待付款', '已付款', '已发货', '已完成', '已取消') DEFAULT '待付款' COMMENT '订单状态',
  `consignee_name` VARCHAR(50) NOT NULL COMMENT '收货人姓名',
  `consignee_phone` VARCHAR(20) NOT NULL COMMENT '收货人电话',
  `shipping_address` VARCHAR(255) NOT NULL COMMENT '收货地址',
  `logistics_company` VARCHAR(50) COMMENT '物流公司',
  `tracking_number` VARCHAR(50) COMMENT '快递单号',
  `order_time` DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '下单时间',
  `pay_time` DATETIME COMMENT '支付时间',
  `deliver_time` DATETIME COMMENT '发货时间',
  `complete_time` DATETIME COMMENT '完成时间',
  FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`)
);

-- 索引优化
CREATE INDEX `idx_order_user` ON `order` (`user_id`);
CREATE INDEX `idx_order_status` ON `order` (`order_status`);


SELECT
  p.product_id,
  p.title AS '商品名称',
  p.description AS '描述',
  (SELECT image_url FROM product_image WHERE product_id = p.product_id ORDER BY sort_order LIMIT 1) AS '封面图',
  GROUP_CONCAT(t.name SEPARATOR ',') AS '商品标签',
  p.shipping_method AS '发货方式',
  p.collect_count AS '收藏数'
FROM product p
LEFT JOIN product_tag pt ON p.product_id = pt.product_id
LEFT JOIN tag t ON pt.tag_id = t.tag_id
WHERE p.status = '在售'
GROUP BY p.product_id
ORDER BY p.created_at DESC
LIMIT 10; -- 分页示例

-- 用户表插入10条数据
INSERT INTO `user` (`username`, `avatar_url`, `address`, `phone`, `created_at`) VALUES
('张三', 'https://avatar.example.com/1.jpg', '北京市海淀区', '13800010001', '2023-01-01 08:00:00'),
('李四', 'https://avatar.example.com/2.jpg', '上海市浦东新区', '13800010002', '2023-01-02 09:00:00'),
('王五', 'https://avatar.example.com/3.jpg', '广州市天河区', '13800010003', '2023-01-03 10:00:00'),
('赵六', 'https://avatar.example.com/4.jpg', '深圳市南山区', '13800010004', '2023-01-04 11:00:00'),
('陈七', 'https://avatar.example.com/5.jpg', '杭州市西湖区', '13800010005', '2023-01-05 12:00:00'),
('林八', 'https://avatar.example.com/6.jpg', '成都市锦江区', '13800010006', '2023-01-06 13:00:00'),
('黄九', 'https://avatar.example.com/7.jpg', '武汉市江汉区', '13800010007', '2023-01-07 14:00:00'),
('周十', 'https://avatar.example.com/8.jpg', '南京市玄武区', '13800010008', '2023-01-08 15:00:00'),
('吴十一', 'https://avatar.example.com/9.jpg', '西安市雁塔区', '13800010009', '2023-01-09 16:00:00'),
('郑十二', 'https://avatar.example.com/10.jpg', '重庆市渝中区', '13800010010', '2023-01-10 17:00:00');

-- 补充商品分类表至10条（原7条基础上新增3条）
INSERT INTO `category` (`name`) VALUES
('美妆'), ('图书'), ('家电');

-- 商品表插入10条数据
INSERT INTO `product` (`user_id`, `category_id`, `title`, `description`, `shipping_method`, `price`, `publish_address`, `status`, `created_at`) VALUES
(1, 1, 'iPhone 13 Pro', '99新，国行版，带发票', '包邮', 6999.00, '北京市海淀区', '在售', '2023-02-01 10:00:00'),
(2, 2, '真皮沙发', '头层牛皮，舒适耐用', '自提', 3200.00, '上海市浦东新区', '已售', '2023-02-02 11:00:00'),
(3, 3, '男士休闲西服', '春秋款，尺码齐全', '包邮', 450.00, '广州市天河区', '在售', '2023-02-03 12:00:00'),
(4, 8, '口红套装', '品牌正品，热门色号', '包邮', 299.00, '深圳市南山区', '在售', '2023-02-04 13:00:00'),
(5, 5, '进口巧克力', '礼盒装，适合送礼', '自提', 150.00, '杭州市西湖区', '下架', '2023-02-05 14:00:00'),
(6, 6, '不粘锅套装', '德国工艺，三层复合底', '包邮', 199.00, '成都市锦江区', '在售', '2023-02-06 15:00:00'),
(7, 9, '科幻小说合集', '经典科幻作品全集', '包邮', 180.00, '武汉市江汉区', '已售', '2023-02-07 16:00:00'),
(8, 10, '滚筒洗衣机', '9公斤大容量，一级能效', '自提', 2899.00, '南京市玄武区', '在售', '2023-02-08 17:00:00'),
(9, 4, '婴儿推车', '可折叠，带遮阳棚', '包邮', 899.00, '西安市雁塔区', '在售', '2023-02-09 18:00:00'),
(10, 7, '瑜伽垫', '加厚防滑，环保材质', '包邮', 68.00, '重庆市渝中区', '在售', '2023-02-10 19:00:00');

-- 商品图片表插入数据（每个商品至少2张图）
INSERT INTO `product_image` (`product_id`, `image_url`, `sort_order`) VALUES
(1, 'https://product.example.com/1-1.jpg', 1),
(1, 'https://product.example.com/1-2.jpg', 2),
(2, 'https://product.example.com/2-1.jpg', 1),
(2, 'https://product.example.com/2-2.jpg', 2),
(3, 'https://product.example.com/3-1.jpg', 1),
(3, 'https://product.example.com/3-2.jpg', 2),
(4, 'https://product.example.com/4-1.jpg', 1),
(4, 'https://product.example.com/4-2.jpg', 2),
(5, 'https://product.example.com/5-1.jpg', 1),
(5, 'https://product.example.com/5-2.jpg', 2);

-- 标签表插入10条数据
INSERT INTO `tag` (`name`) VALUES
('新品'), ('二手'), ('包邮'), ('自提'), ('热门'), ('高销量'), ('推荐'), ('限时折扣'), ('精品'), ('特价');

-- 商品标签关联表插入数据（每个商品关联2个标签）
INSERT INTO `product_tag` (`product_id`, `tag_id`) VALUES
(1, 1), (1, 3),
(2, 2), (2, 4),
(3, 5), (3, 7),
(4, 1), (4, 9),
(5, 8), (5, 10),
(6, 3), (6, 6),
(7, 2), (7, 7),
(8, 4), (8, 9),
(9, 1), (9, 5),
(10, 3), (10, 10);

-- 收藏表插入10条数据
INSERT INTO `favorite` (`user_id`, `product_id`) VALUES
(1, 2), (2, 3), (3, 4), (4, 5), (5, 6),
(6, 7), (7, 8), (8, 9), (9, 10), (10, 1);

-- 评论表插入10条数据（含2条回复）
INSERT INTO `comment` (`user_id`, `product_id`, `content`, `parent_comment_id`) VALUES
(2, 1, '手机成色很好，感谢！', NULL),
(3, 1, '电池续航怎么样？', 1),
(1, 2, '沙发质量很棒！', NULL),
(4, 3, '西服尺码准吗？', NULL),
(5, 4, '口红颜色很正！', NULL),
(6, 5, '巧克力好吃吗？', NULL),
(7, 6, '锅确实不粘', NULL),
(8, 7, '书是正版吗？', NULL),
(9, 8, '洗衣机噪音大吗？', NULL),
(10, 9, '推车很轻便', NULL);

-- 订单表插入10条数据
INSERT INTO `order` (
  `user_id`, `product_title`, `product_price`, `product_image_url`,
  `quantity`, `total_price`, `shipping_fee`, `payment_method`,
  `order_status`, `consignee_name`, `consignee_phone`, `shipping_address`, `order_time`
) VALUES
(1, 'iPhone 13 Pro', 6999.00, 'https://product.example.com/1-1.jpg', 1, 6999.00, 0.00, '支付宝', '已完成', '张三', '13800010001', '北京市海淀区', '2023-03-01 10:00:00'),
(2, '真皮沙发', 3200.00, 'https://product.example.com/2-1.jpg', 1, 3200.00, 0.00, '微信支付', '已付款', '李四', '13800010002', '上海市浦东新区', '2023-03-02 11:00:00'),
(3, '男士休闲西服', 450.00, 'https://product.example.com/3-1.jpg', 2, 900.00, 0.00, '银行卡', '已发货', '王五', '13800010003', '广州市天河区', '2023-03-03 12:00:00'),
(4, '口红套装', 299.00, 'https://product.example.com/4-1.jpg', 1, 299.00, 0.00, '支付宝', '已完成', '赵六', '13800010004', '深圳市南山区', '2023-03-04 13:00:00'),
(5, '进口巧克力', 150.00, 'https://product.example.com/5-1.jpg', 3, 450.00, 0.00, '微信支付', '已取消', '陈七', '13800010005', '杭州市西湖区', '2023-03-05 14:00:00'),
(6, '不粘锅套装', 199.00, 'https://product.example.com/6-1.jpg', 1, 199.00, 0.00, '银行卡', '待付款', '林八', '13800010006', '成都市锦江区', '2023-03-06 15:00:00'),
(7, '科幻小说合集', 180.00, 'https://product.example.com/7-1.jpg', 1, 180.00, 0.00, '支付宝', '已付款', '黄九', '13800010007', '武汉市江汉区', '2023-03-07 16:00:00'),
(8, '滚筒洗衣机', 2899.00, 'https://product.example.com/8-1.jpg', 1, 2899.00, 0.00, '微信支付', '已发货', '周十', '13800010008', '南京市玄武区', '2023-03-08 17:00:00'),
(9, '婴儿推车', 899.00, 'https://product.example.com/9-1.jpg', 1, 899.00, 0.00, '银行卡', '已完成', '吴十一', '13800010009', '西安市雁塔区', '2023-03-09 18:00:00'),
(10, '瑜伽垫', 68.00, 'https://product.example.com/10-1.jpg', 2, 136.00, 0.00, '支付宝', '已完成', '郑十二', '13800010010', '重庆市渝中区', '2023-03-10 19:00:00');

select *from db_school.order;


# 分割线



# 1. 获取闲置商品列表


SELECT
  p.product_id,
  p.title AS '商品名称',
  p.description AS '描述',
  (SELECT image_url FROM product_image WHERE product_id = p.product_id ORDER BY sort_order LIMIT 1) AS '封面图',
  GROUP_CONCAT(t.name SEPARATOR ',') AS '商品标签',
  p.shipping_method AS '发货方式',
  p.collect_count AS '收藏数'
FROM product p
LEFT JOIN product_tag pt ON p.product_id = pt.product_id
LEFT JOIN tag t ON pt.tag_id = t.tag_id
WHERE p.status = '在售'
GROUP BY p.product_id
ORDER BY p.created_at DESC
LIMIT 10; -- 分页示例


# 2. 发布闲置商品

-- 1. 插入商品表
INSERT INTO product (
  user_id, category_id, title, description,
  shipping_method, price, publish_address
) VALUES (
  (SELECT user_id FROM user WHERE username = '用户名'), -- 假设通过用户名获取user_id
  3, -- 假设分类ID为3（服装）
  '二手iPhone 12',
  '99新，无划痕，原装充电器',
  '包邮',
  2999.00,
  '北京市海淀区'
);

-- 2. 插入图片（假设刚插入的product_id为LAST_INSERT_ID()）
INSERT INTO product_image (product_id, image_url, sort_order)
VALUES
  (LAST_INSERT_ID(), 'http://image.com/1.jpg', 1),
  (LAST_INSERT_ID(), 'http://image.com/2.jpg', 2);

-- 3. 处理标签（假设已有标签'手机'和'数码'）
INSERT IGNORE INTO tag (name) VALUES ('手机'), ('数码'); -- 忽略重复标签
INSERT INTO product_tag (product_id, tag_id)
SELECT LAST_INSERT_ID(), tag_id FROM tag WHERE name IN ('手机', '数码');

# 3. 查看商品详情


-- 1. 商品基础信息
SELECT
  p.*,
  u.username AS '发布人',
  u.avatar_url AS '发布人头像',
  c.name AS '分类名称'
FROM product p
JOIN user u ON p.user_id = u.user_id
JOIN category c ON p.category_id = c.category_id
WHERE p.product_id = 1;

-- 2. 商品图片列表
SELECT image_url FROM product_image
WHERE product_id = 1
ORDER BY sort_order;

-- 3. 商品标签
SELECT t.name FROM tag t
JOIN product_tag pt ON t.tag_id = pt.tag_id
WHERE pt.product_id = 1;

-- 4. 评论列表（分页）
SELECT
  cm.content,
  cm.created_at,
  u.username AS '评论用户',
  u.avatar_url AS '用户头像'
FROM comment cm
JOIN user u ON cm.user_id = u.user_id
WHERE cm.product_id = 1
ORDER BY cm.created_at DESC
LIMIT 10;


# 4. 收藏商品


-- 收藏（需判断是否已收藏）
INSERT IGNORE INTO favorite (user_id, product_id)
VALUES (1, 123); -- user_id=1收藏product_id=123
UPDATE product SET collect_count = collect_count + 1 WHERE product_id = 123;

-- 取消收藏
DELETE FROM favorite WHERE user_id = 1 AND product_id = 123;
UPDATE product SET collect_count = collect_count - 1 WHERE product_id = 123;

# 5. 提交订单


-- 1. 插入订单（需事务保证一致性）
INSERT INTO `order` (
  user_id, product_title, product_price, product_image_url,
  total_price, shipping_fee, payment_method,
  consignee_name, consignee_phone, shipping_address
) VALUES (
  1, -- 买家user_id
  '二手iPhone 12',
  2999.00,
  'http://image.com/1.jpg',
  2999.00 + 10.00, -- 总价=商品价格+运费
  10.00,
  '支付宝',
  '张三',
  '13800138000',
  '上海市浦东新区'
);

-- 2. 更新商品状态为已售
UPDATE product SET status = '已售' WHERE product_id = 123;

# 6. 查看订单记录


SELECT
  order_id,
  product_title,
  product_price,
  total_price,
  order_status,
  order_time
FROM `order`
WHERE user_id = 1
ORDER BY order_time DESC;

# 7. 评论商品


INSERT INTO comment (user_id, product_id, content)
VALUES (1, 1, '商品描述一致，非常满意！');

select *from db_school.comment;
```

step2:fastapi

```python
from fastapi import FastAPI, HTTPException, Depends
from fastapi.middleware.cors import CORSMiddleware
import pymysql.cursors
from pymysql import MySQLError
from typing import List, Optional
from pydantic import BaseModel
from datetime import datetime
import logging

app = FastAPI()

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# CORS配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
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


# --- 数据模型 ---
class UserCreate(BaseModel):
    username: str
    avatar_url: Optional[str] = None
    address: Optional[str] = None
    phone: Optional[str] = None


class ProductCreate(BaseModel):
    title: str
    description: str
    category_id: int
    shipping_method: str
    price: float
    publish_address: str
    image_urls: List[str]
    tags: List[str]


class CommentCreate(BaseModel):
    content: str
    parent_comment_id: Optional[int] = None


class OrderCreate(BaseModel):
    product_id: int  # 新增商品ID字段
    consignee_name: str
    consignee_phone: str
    shipping_address: str
    payment_method: str
    quantity: int = 1


# --- 新增数据模型 ---
class OrderDetail(BaseModel):
    order_id: int
    product_title: str
    product_price: float
    product_image_url: Optional[str]
    quantity: int
    total_price: float
    payment_method: str
    consignee_name: str
    consignee_phone: str
    shipping_address: str
    logistics_company: Optional[str]
    tracking_number: Optional[str]
    order_time: datetime
    deliver_time: Optional[datetime]
    complete_time: Optional[datetime]
    username: str
    avatar_url: Optional[str]

class CommentResponse(BaseModel):
    comment_id: int
    content: str
    created_at: datetime
    username: str
    avatar_url: Optional[str]
    product_title: str

class FavoriteResponse(BaseModel):
    favorite_id: int
    product_id: int
    title: str
    price: float
    cover_image: Optional[str]
    created_at: datetime

# --- 数据库工具 ---
def get_db():
    connection = pymysql.connect(**DB_CONFIG)
    try:
        yield connection
    finally:
        connection.close()

# --- 新增数据模型 ---
class CategoryResponse(BaseModel):
    category_id: int
    name: str

# --- 用户相关接口 ---
@app.post("/users/", status_code=201)
async def create_user(user: UserCreate, conn=Depends(get_db)):
    try:
        with conn.cursor() as cursor:
            sql = """INSERT INTO user 
                   (username, avatar_url, address, phone)
                   VALUES (%s, %s, %s, %s)"""
            cursor.execute(sql, (
                user.username,
                user.avatar_url,
                user.address,
                user.phone
            ))
            conn.commit()
            return {"message": "User created successfully"}
    except MySQLError as e:
        logger.error(f"Error creating user: {e}")
        raise HTTPException(500, "Database error")


# --- 商品相关接口 ---
@app.get("/products/")
async def get_products(
        category_id: Optional[int] = None,
        page: int = 1,
        page_size: int = 10,
        conn=Depends(get_db)
):
    try:
        offset = (page - 1) * page_size
        query = """
            SELECT
              p.product_id, p.title, p.description,
              (SELECT image_url FROM product_image 
               WHERE product_id = p.product_id 
               ORDER BY sort_order LIMIT 1) AS cover_image,
              GROUP_CONCAT(t.name) AS tags,
              p.shipping_method, p.collect_count, p.price,
              u.username, u.avatar_url
            FROM product p
            LEFT JOIN product_tag pt ON p.product_id = pt.product_id
            LEFT JOIN tag t ON pt.tag_id = t.tag_id
            JOIN user u ON p.user_id = u.user_id
            WHERE p.status = '在售'
        """
        params = []
        if category_id:
            query += " AND p.category_id = %s"
            params.append(category_id)

        query += " GROUP BY p.product_id ORDER BY p.created_at DESC LIMIT %s OFFSET %s"
        params.extend([page_size, offset])

        with conn.cursor() as cursor:
            cursor.execute(query, params)
            return {"data": cursor.fetchall()}
    except MySQLError as e:
        logger.error(f"Error fetching products: {e}")
        raise HTTPException(500, "Database error")


@app.post("/products/", status_code=201)
async def create_product(product: ProductCreate, conn=Depends(get_db)):
    try:
        with conn.cursor() as cursor:
            # 获取示例用户（实际应来自认证）
            cursor.execute("SELECT user_id FROM user LIMIT 1")
            user = cursor.fetchone()
            if not user:
                raise HTTPException(400, "User not found")

            # 插入商品
            product_sql = """
                INSERT INTO product (
                    user_id, category_id, title, description,
                    shipping_method, price, publish_address
                ) VALUES (%s, %s, %s, %s, %s, %s, %s)
            """
            cursor.execute(product_sql, (
                user['user_id'],
                product.category_id,
                product.title,
                product.description,
                product.shipping_method,
                product.price,
                product.publish_address
            ))
            product_id = cursor.lastrowid

            # 插入图片
            for idx, url in enumerate(product.image_urls):
                cursor.execute(
                    "INSERT INTO product_image (product_id, image_url, sort_order) "
                    "VALUES (%s, %s, %s)",
                    (product_id, url, idx)
                )

            # 处理标签
            for tag_name in product.tags:
                # 插入标签（如果不存在）
                cursor.execute(
                    "INSERT IGNORE INTO tag (name) VALUES (%s)",
                    (tag_name,)
                )
                # 获取标签ID
                cursor.execute(
                    "SELECT tag_id FROM tag WHERE name = %s",
                    (tag_name,)
                )
                tag = cursor.fetchone()
                if tag:
                    cursor.execute(
                        "INSERT INTO product_tag (product_id, tag_id) "
                        "VALUES (%s, %s)",
                        (product_id, tag['tag_id'])
                    )

            conn.commit()
            return {"product_id": product_id}
    except MySQLError as e:
        conn.rollback()
        logger.error(f"Error creating product: {e}")
        raise HTTPException(500, "Database error")


# 订单接口实现
@app.post("/orders/", status_code=201)
async def create_order(order: OrderCreate, conn=Depends(get_db)):
    try:
        with conn.cursor() as cursor:
            # 1. 获取商品信息并锁定行
            cursor.execute(
                "SELECT product_id, title, price, status "
                "FROM product WHERE product_id = %s FOR UPDATE",
                (order.product_id,)
            )
            product = cursor.fetchone()

            # 商品校验
            if not product:
                raise HTTPException(404, "商品不存在")
            if product['status'] != '在售':
                raise HTTPException(400, "商品已下架或售出")

            # 2. 获取商品首图
            cursor.execute(
                "SELECT image_url FROM product_image "
                "WHERE product_id = %s ORDER BY sort_order LIMIT 1",
                (order.product_id,)
            )
            image = cursor.fetchone()
            main_image = image['image_url'] if image else None

            # 3. 获取当前用户（示例逻辑，实际应从认证获取）
            cursor.execute("SELECT user_id FROM user LIMIT 1")
            user = cursor.fetchone()
            if not user:
                raise HTTPException(400, "用户不存在")

            # 4. 计算总金额
            total_price = product['price'] * order.quantity
            shipping_fee = 0.00  # 简化运费计算

            # 5. 插入订单
            order_sql = """
                INSERT INTO `order` (
                    user_id, product_title, product_price, product_image_url,
                    quantity, total_price, shipping_fee, payment_method,
                    consignee_name, consignee_phone, shipping_address
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            cursor.execute(order_sql, (
                user['user_id'],
                product['title'],
                product['price'],
                main_image,
                order.quantity,
                total_price,
                shipping_fee,
                order.payment_method,
                order.consignee_name,
                order.consignee_phone,
                order.shipping_address
            ))

            # 6. 更新商品状态
            cursor.execute(
                "UPDATE product SET status = '已售' "
                "WHERE product_id = %s",
                (order.product_id,)
            )

            conn.commit()
            return {"order_id": cursor.lastrowid}

    except MySQLError as e:
        conn.rollback()
        logger.error(f"订单创建失败: {e}")
        raise HTTPException(500, "数据库操作失败")
    except HTTPException as he:
        conn.rollback()
        raise he
    except Exception as e:
        conn.rollback()
        logger.error(f"系统错误: {e}")
        raise HTTPException(500, "服务器内部错误")


# --- 接口实现 ---
# 1. 查看订单详情
@app.get("/orders/{order_id}", response_model=OrderDetail)
async def get_order_detail(
        order_id: int,
        conn=Depends(get_db)
):
    try:
        with conn.cursor() as cursor:
            # 联查订单和用户信息
            query = """
                SELECT o.*, u.username, u.avatar_url 
                FROM `order` o
                JOIN user u ON o.user_id = u.user_id
                WHERE o.order_id = %s
            """
            cursor.execute(query, (order_id,))
            order = cursor.fetchone()

            if not order:
                raise HTTPException(404, "订单不存在")

            return order
    except MySQLError as e:
        logger.error(f"获取订单详情失败: {e}")
        raise HTTPException(500, "数据库错误")


# 2. 查看评论
@app.get("/comments/", response_model=List[CommentResponse])
async def get_comments(
        product_id: Optional[int] = None,
        user_id: Optional[int] = None,
        page: int = 1,
        page_size: int = 10,
        conn=Depends(get_db)
):
    try:
        offset = (page - 1) * page_size
        params = []

        base_query = """
            SELECT 
                c.comment_id, c.content, c.created_at,
                u.username, u.avatar_url,
                p.title AS product_title
            FROM comment c
            JOIN user u ON c.user_id = u.user_id
            JOIN product p ON c.product_id = p.product_id
            WHERE 1=1
        """

        if product_id:
            base_query += " AND c.product_id = %s"
            params.append(product_id)
        if user_id:
            base_query += " AND c.user_id = %s"
            params.append(user_id)

        base_query += " ORDER BY c.created_at DESC LIMIT %s OFFSET %s"
        params.extend([page_size, offset])

        with conn.cursor() as cursor:
            cursor.execute(base_query, params)
            return cursor.fetchall()

    except MySQLError as e:
        logger.error(f"获取评论失败: {e}")
        raise HTTPException(500, "数据库错误")


# 3. 查看收藏
@app.get("/favorites/", response_model=List[FavoriteResponse])
async def get_favorites(
        user_id: int,  # 实际应通过认证获取
        page: int = 1,
        page_size: int = 10,
        conn=Depends(get_db)
):
    try:
        offset = (page - 1) * page_size
        query = """
            SELECT 
                f.favorite_id, f.created_at,
                p.product_id, p.title, p.price,
                (SELECT image_url FROM product_image 
                 WHERE product_id = p.product_id 
                 ORDER BY sort_order LIMIT 1) AS cover_image
            FROM favorite f
            JOIN product p ON f.product_id = p.product_id
            WHERE f.user_id = %s
            ORDER BY f.created_at DESC
            LIMIT %s OFFSET %s
        """
        with conn.cursor() as cursor:
            cursor.execute(query, (user_id, page_size, offset))
            return cursor.fetchall()

    except MySQLError as e:
        logger.error(f"获取收藏失败: {e}")
        raise HTTPException(500, "数据库错误")


# --- 新增接口实现 ---
# 1. 获取商品分类
@app.get("/categories/", response_model=List[CategoryResponse])
async def get_categories(conn=Depends(get_db)):
    try:
        with conn.cursor() as cursor:
            cursor.execute(
                "SELECT category_id, name FROM category ORDER BY category_id"
            )
            return cursor.fetchall()
    except MySQLError as e:
        logger.error(f"获取分类失败: {e}")
        raise HTTPException(500, "数据库错误")


# 2. 获取所有商品列表（增强版）
@app.get("/products/list/")
async def get_all_products(
        sort_by: Optional[str] = "new",  # 排序方式：new（最新）/hot（热门）
        page: int = 1,
        page_size: int = 10,
        conn=Depends(get_db)
):
    try:
        offset = (page - 1) * page_size

        # 构建排序条件
        order_clause = "p.created_at DESC"
        if sort_by == "hot":
            order_clause = "p.collect_count DESC, p.created_at DESC"

        query = f"""
            SELECT
              p.product_id, p.title, p.description,
              (SELECT image_url FROM product_image 
               WHERE product_id = p.product_id 
               ORDER BY sort_order LIMIT 1) AS cover_image,
              p.shipping_method, 
              p.collect_count,
              p.price,
              p.status,
              u.username,
              u.avatar_url,
              c.name AS category_name,
              GROUP_CONCAT(t.name) AS tags
            FROM product p
            LEFT JOIN product_tag pt ON p.product_id = pt.product_id
            LEFT JOIN tag t ON pt.tag_id = t.tag_id
            JOIN user u ON p.user_id = u.user_id
            JOIN category c ON p.category_id = c.category_id
            GROUP BY p.product_id
            ORDER BY {order_clause}
            LIMIT %s OFFSET %s
        """

        with conn.cursor() as cursor:
            cursor.execute(query, (page_size, offset))
            products = cursor.fetchall()

            # 格式化标签字段
            for product in products:
                if product["tags"]:
                    product["tags"] = product["tags"].split(",")
                else:
                    product["tags"] = []

            return {
                "data": products,
                "pagination": {
                    "page": page,
                    "page_size": page_size,
                    "total": len(products)
                }
            }
    except MySQLError as e:
        logger.error(f"获取商品失败: {e}")
        raise HTTPException(500, "数据库错误")
# --- 其他接口（收藏、评论等）实现方式类似 ---

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:postman

```bash
 1. 创建用户
post  http://localhost:8000/users/


{
    "username": "test_user",
    "avatar_url": "https://example.com/avatar.jpg",
    "address": "北京市朝阳区",
    "phone": "13800138000"
}


{
    "message": "User created successfully"
}

2. 获取商品列表
请求方法: GET
URL: http://localhost:8000/products/?category_id=1&page=1&page_size=2

{
    "data": [
        {
            "product_id": 1,
            "title": "iPhone 13 Pro",
            "description": "99新，国行版，带发票",
            "cover_image": "https://product.example.com/1-1.jpg",
            "tags": "新品,包邮",
            "shipping_method": "包邮",
            "collect_count": 0,
            "price": 6999.0,
            "username": "张三",
            "avatar_url": "https://avatar.example.com/1.jpg"
        }
    ]
}

3. 发布商品
请求方法: POST
URL: http://localhost:8000/products/
Headers:
Content-Type: application/json

{
    "title": "二手单反相机",
    "description": "Canon EOS 5D Mark IV，95新",
    "category_id": 1,
    "shipping_method": "包邮",
    "price": 12000.00,
    "publish_address": "上海市浦东新区",
    "image_urls": [
        "http://img.example.com/camera1.jpg",
        "http://img.example.com/camera2.jpg"
    ],
    "tags": ["数码", "摄影器材"]
}

{
    "product_id": 11
}

4. 创建订单
请求方法: POST
http://localhost:8000/orders

{
  "product_id": 1,       // 对应示例数据中的iPhone 13 Pro
  "consignee_name": "张三",
  "consignee_phone": "13800010001",
  "shipping_address": "北京市海淀区中关村大街1号",
  "payment_method": "支付宝",
  "quantity": 2
}

{
    "order_id": 0
}


5. 查看订单详情
请求：

复制
GET http://localhost:8000/orders/1

 {
    "order_id": 1,
    "product_title": "iPhone 13 Pro",
    "product_price": 6999.0,
    "product_image_url": "https://product.example.com/1-1.jpg",
    "quantity": 1,
    "total_price": 6999.0,
    "payment_method": "支付宝",
    "consignee_name": "张三",
    "consignee_phone": "13800010001",
    "shipping_address": "北京市海淀区",
    "logistics_company": null,
    "tracking_number": null,
    "order_time": "2023-03-01T10:00:00",
    "deliver_time": null,
    "complete_time": null,
    "username": "张三",
    "avatar_url": "https://avatar.example.com/1.jpg"
}

6. 查看评论

请求：
 
GET http://localhost:8000/comments/?product_id=1&page=1&page_size=5

[
    {
        "comment_id": 12,
        "content": "商品描述一致，非常满意！",
        "created_at": "2025-03-21T03:23:47",
        "username": "张三",
        "avatar_url": "https://avatar.example.com/1.jpg",
        "product_title": "iPhone 13 Pro"
    },
    {
        "comment_id": 1,
        "content": "手机成色很好，感谢！",
        "created_at": "2025-03-21T03:18:29",
        "username": "李四",
        "avatar_url": "https://avatar.example.com/2.jpg",
        "product_title": "iPhone 13 Pro"
    },
    {
        "comment_id": 2,
        "content": "电池续航怎么样？",
        "created_at": "2025-03-21T03:18:29",
        "username": "王五",
        "avatar_url": "https://avatar.example.com/3.jpg",
        "product_title": "iPhone 13 Pro"
    }
]

7. 查看收藏

GET http://localhost:8000/favorites/?user_id=1&page=1&page_size=3

[
    {
        "favorite_id": 1,
        "product_id": 2,
        "title": "真皮沙发",
        "price": 3200.0,
        "cover_image": "https://product.example.com/2-1.jpg",
        "created_at": "2025-03-21T03:18:26"
    }
]



 
8. 获取商品分类
请求：

复制
GET http://localhost:8000/categories/

[
    {
        "category_id": 1,
        "name": "数码"
    },
    {
        "category_id": 2,
        "name": "家居"
    },
    {
        "category_id": 3,
        "name": "服装"
    },
    {
        "category_id": 4,
        "name": "母婴"
    },
    {
        "category_id": 5,
        "name": "食品"
    },
    {
        "category_id": 6,
        "name": "厨具"
    },
    {
        "category_id": 7,
        "name": "运动器材"
    },
    {
        "category_id": 8,
        "name": "美妆"
    },
    {
        "category_id": 9,
        "name": "图书"
    },
    {
        "category_id": 10,
        "name": "家电"
    }
]


9. 获取所有商品列表
请求：

复制
GET http://localhost:8000/products/list/?sort_by=hot&page=1&page_size=3


{
    "data": [
        {
            "product_id": 11,
            "title": "二手单反相机",
            "description": "Canon EOS 5D Mark IV，95新",
            "cover_image": "http://img.example.com/camera1.jpg",
            "shipping_method": "包邮",
            "collect_count": 0,
            "price": 12000.0,
            "status": "在售",
            "username": "张三",
            "avatar_url": "https://avatar.example.com/1.jpg",
            "category_name": "数码",
            "tags": [
                "数码",
                "摄影器材"
            ]
        },
        {
            "product_id": 10,
            "title": "瑜伽垫",
            "description": "加厚防滑，环保材质",
            "cover_image": null,
            "shipping_method": "包邮",
            "collect_count": 0,
            "price": 68.0,
            "status": "在售",
            "username": "郑十二",
            "avatar_url": "https://avatar.example.com/10.jpg",
            "category_name": "运动器材",
            "tags": [
                "包邮",
                "特价"
            ]
        },
        {
            "product_id": 9,
            "title": "婴儿推车",
            "description": "可折叠，带遮阳棚",
            "cover_image": null,
            "shipping_method": "包邮",
            "collect_count": 0,
            "price": 899.0,
            "status": "在售",
            "username": "吴十一",
            "avatar_url": "https://avatar.example.com/9.jpg",
            "category_name": "母婴",
            "tags": [
                "新品",
                "热门"
            ]
        }
    ],
    "pagination": {
        "page": 1,
        "page_size": 3,
        "total": 3
    }
}


```

step4:

step5:

step6:

end