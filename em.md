说明：
我计划用fastapi+angular，做一款在线点餐系统，
比如兰州拉面
1.有各种食物列表，
   1.1包括食品名称
   1.2食品对应价格，
   1.3食品分类，比如拉面，拌面，炒菜，饮料等等
2.管理员可以添加和修改 删除食物列表
3.用户可以查看食物列表，并且可以下单购买
4.用户成功购买拉面后，需要有一条订单记录
5.管理员可以查看订单列表
6.订单列表 有下单时间，食品名称，食品类型，用户名，餐桌号，价格，堂食还是外卖，订单状态，待接受，已经付款，未付款，已经完成      
7.管理员可以查看统计 每天的销售额，每月的销售额 ，查看不同食物种类的销售情况，
8.用户吃完后，可以点评食物，其他用户，订餐时，可以点击评论，查看该食物的用户评价
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/48fcab85d04346a78fd4fb838d5ff9e9.png#pic_center)

step1:先设计sql，包括建表，添加模拟数据，多表查询

```sql


-- 用户表（顾客）
CREATE TABLE db_school.users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    phone VARCHAR(20) NOT NULL,
    avatar_url VARCHAR(255),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 管理员表
CREATE TABLE db_school.admins (
    admin_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role ENUM('super','normal') NOT NULL DEFAULT 'normal',
    last_login DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 食品分类表
CREATE TABLE db_school.food_categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(50) NOT NULL,
    display_order INT DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 食品表
CREATE TABLE db_school.foods (
    food_id INT PRIMARY KEY AUTO_INCREMENT,
    food_name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    category_id INT NOT NULL,
    image_url VARCHAR(255),
    is_available BOOLEAN DEFAULT TRUE,
    sales_count INT DEFAULT 0,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES food_categories(category_id)
);

-- 订单表
CREATE TABLE db_school.orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    table_number VARCHAR(20) COMMENT '堂食桌号',
    order_type ENUM('堂食','外卖') NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    status ENUM('待接受','已接单','制作中','已完成','已取消') DEFAULT '待接受',
    payment_status ENUM('未付款','已付款') DEFAULT '未付款',
    delivery_address TEXT,
    order_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_time DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- 订单明细表
CREATE TABLE db_school.order_details (
    detail_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    food_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (food_id) REFERENCES foods(food_id)
);

-- 食品评价表
CREATE TABLE db_school.food_reviews (
    review_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    food_id INT NOT NULL,
    order_id INT NOT NULL,
    rating TINYINT CHECK (rating BETWEEN 1 AND 5),
    comment TEXT,
    review_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (food_id) REFERENCES foods(food_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);



-- 插入用户表数据
INSERT INTO users (username, password_hash, phone, avatar_url) VALUES
('user1', MD5('pass123'), '13800001111', 'avatar1.jpg'),
('user2', MD5('pass123'), '13800002222', 'avatar2.jpg'),
('user3', MD5('pass123'), '13800003333', 'avatar3.jpg'),
('user4', MD5('pass123'), '13800004444', 'avatar4.jpg'),
('user5', MD5('pass123'), '13800005555', 'avatar5.jpg'),
('user6', MD5('pass123'), '13800006666', 'avatar6.jpg'),
('user7', MD5('pass123'), '13800007777', 'avatar7.jpg'),
('user8', MD5('pass123'), '13800008888', 'avatar8.jpg'),
('user9', MD5('pass123'), '13800009999', 'avatar9.jpg'),
('user10', MD5('pass123'), '13800001010', 'avatar10.jpg');

-- 插入管理员表数据
INSERT INTO admins (username, password_hash, role, last_login) VALUES
('admin1', MD5('admin123'), 'super', '2023-08-01 09:00:00'),
('admin2', MD5('admin123'), 'normal', '2023-08-01 10:00:00'),
('admin3', MD5('admin123'), 'normal', '2023-08-01 11:00:00'),
('admin4', MD5('admin123'), 'normal', '2023-08-01 12:00:00'),
('admin5', MD5('admin123'), 'super', '2023-08-01 13:00:00'),
('admin6', MD5('admin123'), 'normal', '2023-08-01 14:00:00'),
('admin7', MD5('admin123'), 'normal', '2023-08-01 15:00:00'),
('admin8', MD5('admin123'), 'normal', '2023-08-01 16:00:00'),
('admin9', MD5('admin123'), 'super', '2023-08-01 17:00:00'),
('admin10', MD5('admin123'), 'normal', '2023-08-01 18:00:00');

-- 插入食品分类表数据
INSERT INTO food_categories (category_name, display_order) VALUES
('拉面类', 1),
('拌面类', 2),
('炒菜类', 3),
('凉菜类', 4),
('汤品类', 5),
('小吃类', 6),
('套餐类', 7),
('饮料类', 8),
('加料区', 9),
('儿童套餐', 10);

-- 插入食品表数据
INSERT INTO foods (food_name, description, price, category_id, image_url) VALUES
('牛肉拉面', '传统兰州牛肉拉面', 18.00, 1, 'noodle1.jpg'),
('羊肉拌面', '秘制羊肉拌面', 20.00, 2, 'noodle2.jpg'),
('大盘鸡', '新疆风味大盘鸡', 68.00, 3, 'dish1.jpg'),
('凉拌黄瓜', '爽口凉菜', 8.00, 4, 'cold1.jpg'),
('番茄蛋汤', '家常汤品', 6.00, 5, 'soup1.jpg'),
('煎饺', '脆皮煎饺6只', 12.00, 6, 'snack1.jpg'),
('双人套餐', '牛肉面+大盘鸡+饮料', 88.00, 7, 'combo1.jpg'),
('酸梅汤', '冰镇酸梅汤', 5.00, 8, 'drink1.jpg'),
('加牛肉', '额外牛肉份量', 6.00, 9, 'extra1.jpg'),
('儿童拉面', '小份无辣拉面', 12.00, 10, 'kid1.jpg');



INSERT INTO foods (food_name, description, price, category_id, image_url) VALUES
('牛肉拉面', '经典牛肉拉面', 12.99, 1, 'beef_noodles.jpg'),
('鸡肉拌面', '香辣鸡肉拌面', 10.99, 2, 'chicken_noodles.jpg'),
('宫保鸡丁', '经典川菜', 15.99, 3, 'kung_pao_chicken.jpg'),
('可乐', '冰镇可乐', 2.99, 4, 'cola.jpg'),
('炸鸡翅', '香脆炸鸡翅', 8.99, 5, 'chicken_wings.jpg'),
('番茄蛋汤', '家常汤品', 6.99, 6, 'tomato_egg_soup.jpg'),
('芒果布丁', '甜品', 5.99, 7, 'mango_pudding.jpg'),
('家庭套餐', '适合全家享用', 25.99, 8, 'family_meal.jpg'),
('素炒时蔬', '健康素食', 9.99, 9, 'vegetable_stir_fry.jpg'),
('儿童餐', '适合小朋友', 7.99, 10, 'kids_meal.jpg');



-- 插入订单表数据
INSERT INTO orders (user_id, table_number, order_type, total_amount, status, payment_status, delivery_address) VALUES
(1, 'A01', '堂食', 36.00, '已完成', '已付款', NULL),
(2, NULL, '外卖', 88.00, '已接单', '已付款', '北区XX街道'),
(3, 'B02', '堂食', 24.00, '制作中', '已付款', NULL),
(4, NULL, '外卖', 32.00, '待接受', '未付款', '海区XX路'),
(5, 'C03', '堂食', 68.00, '已完成', '已付款', NULL),
(6, NULL, '外卖', 45.00, '已取消', '未付款', '广区XX大厦'),
(7, 'D04', '堂食', 18.00, '已完成', '已付款', NULL),
(8, NULL, '外卖', 26.00, '已完成', '已付款', '深区XX小区'),
(9, 'E05', '堂食', 94.00, '制作中', '已付款', NULL),
(10, NULL, '外卖', 15.00, '待接受', '未付款', '洛区XX宾馆');

INSERT INTO orders (user_id, table_number, order_type, total_amount, status, payment_status, delivery_address) VALUES
(1, 'A01', '堂食', 25.98, '已完成', '已付款', NULL),
(2, 'B02', '外卖', 15.99, '已接单', '已付款', '123 Main St'),
(3, 'C03', '堂食', 12.99, '制作中', '已付款', NULL),
(4, 'D04', '外卖', 8.99, '待接受', '未付款', '456 Elm St'),
(5, 'E05', '堂食', 30.97, '已完成', '已付款', NULL),
(6, 'F06', '外卖', 10.99, '已接单', '已付款', '789 Oak St'),
(7, 'G07', '堂食', 15.99, '制作中', '已付款', NULL),
(8, 'H08', '外卖', 12.99, '待接受', '未付款', '101 Pine St'),
(9, 'I09', '堂食', 25.98, '已完成', '已付款', NULL),
(10, 'J10', '外卖', 15.99, '已接单', '已付款', '202 Maple St');

-- 插入订单明细表数据
INSERT INTO order_details (order_id, food_id, quantity, unit_price) VALUES
(1, 1, 2, 18.00),
(2, 7, 1, 88.00),
(3, 6, 2, 12.00),
(4, 1, 1, 18.00),
(4, 9, 1, 6.00),
(4, 8, 2, 5.00),
(5, 3, 1, 68.00),
(6, 1, 1, 18.00),
(6, 3, 1, 68.00),
(7, 1, 1, 18.00),
(8, 2, 1, 20.00),
(8, 8, 1, 5.00),
(9, 3, 1, 68.00),
(9, 7, 1, 88.00),
(10, 10, 1, 12.00);

INSERT INTO order_details (order_id, food_id, quantity, unit_price) VALUES
(1, 1, 2, 12.99),
(2, 3, 1, 15.99),
(3, 2, 1, 10.99),
(4, 5, 1, 8.99),
(5, 8, 1, 25.99),
(6, 4, 1, 2.99),
(7, 7, 1, 5.99),
(8, 9, 1, 9.99),
(9, 10, 1, 7.99),
(10, 6, 1, 6.99);

-- 插入食品评价表数据
INSERT INTO food_reviews (user_id, food_id, order_id, rating, comment) VALUES
(1, 1, 1, 5, '牛肉量足汤头鲜美！'),
(5, 3, 5, 4, '鸡肉很入味，稍微有点辣'),
(7, 1, 7, 5, '每次必点的经典款'),
(8, 2, 8, 3, '羊肉有点膻味'),
(8, 8, 8, 4, '酸梅汤解腻不错'),
(9, 3, 9, 5, '大盘鸡分量超级足'),
(9, 7, 9, 4, '套餐搭配很实惠'),
(10, 10, 10, 2, '儿童面太咸了');


INSERT INTO food_reviews (user_id, food_id, order_id, rating, comment) VALUES
(1, 1, 1, 5, '非常好吃！'),
(2, 3, 2, 4, '味道不错'),
(3, 2, 3, 3, '一般般'),
(4, 5, 4, 5, '很香脆'),
(5, 8, 5, 4, '分量足'),
(6, 4, 6, 5, '冰镇可乐很爽'),
(7, 7, 7, 3, '甜品有点甜'),
(8, 9, 8, 4, '健康又美味'),
(9, 10, 9, 5, '小朋友很喜欢'),
(10, 6, 10, 4, '汤很鲜');

# 1.菜单展示
SELECT f.food_name, f.description, f.price, c.category_name
FROM db_school.foods f
JOIN db_school.food_categories c
ON f.category_id = c.category_id
ORDER BY c.display_order;



# 2.订单详情
SELECT o.order_id, f.food_name, od.quantity, od.unit_price, od.subtotal
FROM db_school.orders o
JOIN db_school.order_details od ON o.order_id = od.order_id
JOIN db_school.foods f ON od.food_id = f.food_id
WHERE o.order_id = 2;  -- 指定订单ID


# 3.销售统计（按分类）

SELECT c.category_name, SUM(od.subtotal) AS total_sales
FROM db_school.order_details od
JOIN db_school.foods f ON od.food_id = f.food_id
JOIN db_school.food_categories c ON f.category_id = c.category_id
GROUP BY c.category_name
ORDER BY total_sales DESC;


# 4.用户评价


SELECT u.username, f.food_name, r.rating, r.comment, r.review_time
FROM db_school.food_reviews r
JOIN db_school.users u ON r.user_id = u.user_id
JOIN db_school.foods f ON r.food_id = f.food_id;




#5.热销商品排名

SELECT f.food_name, SUM(od.quantity) AS total_sold
FROM db_school.order_details od
JOIN db_school.foods f ON od.food_id = f.food_id
GROUP BY f.food_name
ORDER BY total_sold DESC
LIMIT 10;



# 6.查看订单状态及用户信息
SELECT o.order_id, o.status, u.username, u.phone
FROM db_school.orders o
JOIN db_school.users u ON o.user_id = u.user_id
WHERE o.order_id = 2;  -- 指定订单ID

# 7.获取某个食品的所有评价
SELECT f.food_name, u.username, r.rating, r.comment
FROM db_school.food_reviews r
JOIN db_school.users u ON r.user_id = u.user_id
JOIN db_school.foods f ON r.food_id = f.food_id
WHERE f.food_id = 2;  -- 指定食品ID

# 8.用户个人资料及近期订单
SELECT u.*, o.order_id, o.total_amount, o.order_time
FROM db_school.users u
LEFT JOIN db_school.orders o ON u.user_id = o.user_id
WHERE u.user_id = 2   -- 指定用户ID
AND o.order_time > DATE_SUB(NOW(), INTERVAL 30 DAY)  -- 最近30天
ORDER BY o.order_time DESC;

```

step2:到这里，数据库部分已经写完了，接下来写fastapi后端，写8个查询接口
C:\Users\wangrusheng\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import pymysql.cursors

app = FastAPI()

# 添加CORS配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],  # 允许访问的前端域名
    allow_credentials=True,
    allow_methods=["*"],  # 允许所有HTTP方法
    allow_headers=["*"],  # 允许所有请求头
)

# 数据库连接配置（请根据实际情况修改）
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',  # 修改为实际的数据库名称
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

# 1. 菜单展示
@app.get("/menu")
async def get_menu():
    """获取所有食品及其分类信息"""
    query = """
    SELECT f.food_name, f.description, f.price, c.category_name
    FROM foods f
    JOIN food_categories c ON f.category_id = c.category_id
    ORDER BY c.display_order;
    """
    data = query_database(query)
    return {"data": data}

# 2. 订单详情
@app.get("/order_details/{order_id}")
async def get_order_details(order_id: int):
    """获取指定订单的详细信息"""
    query = """
    SELECT o.order_id, f.food_name, od.quantity, od.unit_price, od.subtotal
    FROM orders o
    JOIN order_details od ON o.order_id = od.order_id
    JOIN foods f ON od.food_id = f.food_id
    WHERE o.order_id = %s;
    """
    data = query_database(query, (order_id,))
    return {"data": data}

# 3. 销售统计（按分类）
@app.get("/sales_by_category")
async def get_sales_by_category():
    """获取按食品分类的销售统计"""
    query = """
    SELECT c.category_name, SUM(od.subtotal) AS total_sales
    FROM order_details od
    JOIN foods f ON od.food_id = f.food_id
    JOIN food_categories c ON f.category_id = c.category_id
    GROUP BY c.category_name
    ORDER BY total_sales DESC;
    """
    data = query_database(query)
    return {"data": data}

# 4. 用户评价
@app.get("/reviews")
async def get_reviews():
    """获取所有用户评价"""
    query = """
    SELECT u.username, f.food_name, r.rating, r.comment, r.review_time
    FROM food_reviews r
    JOIN users u ON r.user_id = u.user_id
    JOIN foods f ON r.food_id = f.food_id;
    """
    data = query_database(query)
    return {"data": data}

# 5. 热销商品排名
@app.get("/top_selling_foods")
async def get_top_selling_foods():
    """获取热销商品排名"""
    query = """
    SELECT f.food_name, SUM(od.quantity) AS total_sold
    FROM order_details od
    JOIN foods f ON od.food_id = f.food_id
    GROUP BY f.food_name
    ORDER BY total_sold DESC
    LIMIT 10;
    """
    data = query_database(query)
    return {"data": data}

# 6. 查看订单状态及用户信息
@app.get("/order_status/{order_id}")
async def get_order_status(order_id: int):
    """获取指定订单的状态及用户信息"""
    query = """
    SELECT o.order_id, o.status, u.username, u.phone
    FROM orders o
    JOIN users u ON o.user_id = u.user_id
    WHERE o.order_id = %s;
    """
    data = query_database(query, (order_id,))
    return {"data": data}

# 7. 获取某个食品的所有评价
@app.get("/food_reviews/{food_id}")
async def get_food_reviews(food_id: int):
    """获取指定食品的所有评价"""
    query = """
    SELECT f.food_name, u.username, r.rating, r.comment
    FROM food_reviews r
    JOIN users u ON r.user_id = u.user_id
    JOIN foods f ON r.food_id = f.food_id
    WHERE f.food_id = %s;
    """
    data = query_database(query, (food_id,))
    return {"data": data}

# 8. 用户个人资料及近期订单
@app.get("/user_profile/{user_id}")
async def get_user_profile(user_id: int):
    """获取指定用户的个人资料及近期订单"""
    query = """
    SELECT u.*, o.order_id, o.total_amount, o.order_time
    FROM users u
    LEFT JOIN orders o ON u.user_id = o.user_id
    WHERE u.user_id = %s
    AND o.order_time > DATE_SUB(NOW(), INTERVAL 30 DAY)
    ORDER BY o.order_time DESC;
    """
    data = query_database(query, (user_id,))
    return {"data": data}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:到这里，fastapi的后端部分写完了，接下来去postman里面测试

```bash
 # 1.菜单展示 http://127.0.0.1:8000/menu

{
    "data": [
        {
            "food_name": "牛肉拉面",
            "description": "传统兰州牛肉拉面",
            "price": 18.0,
            "category_name": "拉面类"
        },
        {
            "food_name": "牛肉拉面",
            "description": "经典牛肉拉面",
            "price": 12.99,
            "category_name": "拉面类"
        },
        {
            "food_name": "羊肉拌面",
            "description": "秘制羊肉拌面",
            "price": 20.0,
            "category_name": "拌面类"
        },
        {
            "food_name": "鸡肉拌面",
            "description": "香辣鸡肉拌面",
            "price": 10.99,
            "category_name": "拌面类"
        },
        {
            "food_name": "大盘鸡",
            "description": "新疆风味大盘鸡",
            "price": 68.0,
            "category_name": "炒菜类"
        },
        {
            "food_name": "宫保鸡丁",
            "description": "经典川菜",
            "price": 15.99,
            "category_name": "炒菜类"
        },
        {
            "food_name": "凉拌黄瓜",
            "description": "爽口凉菜",
            "price": 8.0,
            "category_name": "凉菜类"
        },
        {
            "food_name": "可乐",
            "description": "冰镇可乐",
            "price": 2.99,
            "category_name": "凉菜类"
        },
        {
            "food_name": "番茄蛋汤",
            "description": "家常汤品",
            "price": 6.0,
            "category_name": "汤品类"
        },
        {
            "food_name": "炸鸡翅",
            "description": "香脆炸鸡翅",
            "price": 8.99,
            "category_name": "汤品类"
        },
        {
            "food_name": "煎饺",
            "description": "脆皮煎饺6只",
            "price": 12.0,
            "category_name": "小吃类"
        },
        {
            "food_name": "番茄蛋汤",
            "description": "家常汤品",
            "price": 6.99,
            "category_name": "小吃类"
        },
        {
            "food_name": "双人套餐",
            "description": "牛肉面+大盘鸡+饮料",
            "price": 88.0,
            "category_name": "套餐类"
        },
        {
            "food_name": "芒果布丁",
            "description": "甜品",
            "price": 5.99,
            "category_name": "套餐类"
        },
        {
            "food_name": "酸梅汤",
            "description": "冰镇酸梅汤",
            "price": 5.0,
            "category_name": "饮料类"
        },
        {
            "food_name": "家庭套餐",
            "description": "适合全家享用",
            "price": 25.99,
            "category_name": "饮料类"
        },
        {
            "food_name": "加牛肉",
            "description": "额外牛肉份量",
            "price": 6.0,
            "category_name": "加料区"
        },
        {
            "food_name": "素炒时蔬",
            "description": "健康素食",
            "price": 9.99,
            "category_name": "加料区"
        },
        {
            "food_name": "儿童拉面",
            "description": "小份无辣拉面",
            "price": 12.0,
            "category_name": "儿童套餐"
        },
        {
            "food_name": "儿童餐",
            "description": "适合小朋友",
            "price": 7.99,
            "category_name": "儿童套餐"
        }
    ]
}


# 2.订单详情 http://127.0.0.1:8000/order_details/2

{
    "data": [
        {
            "order_id": 2,
            "food_name": "双人套餐",
            "quantity": 1,
            "unit_price": 88.0,
            "subtotal": 88.0
        },
        {
            "order_id": 2,
            "food_name": "大盘鸡",
            "quantity": 1,
            "unit_price": 15.99,
            "subtotal": 15.99
        }
    ]
}

# 3.销售统计（按分类） http://127.0.0.1:8000/sales_by_category

{
    "data": [
        {
            "category_name": "炒菜类",
            "total_sales": 219.99
        },
        {
            "category_name": "套餐类",
            "total_sales": 181.99
        },
        {
            "category_name": "拉面类",
            "total_sales": 115.98
        },
        {
            "category_name": "饮料类",
            "total_sales": 40.99
        },
        {
            "category_name": "小吃类",
            "total_sales": 30.99
        },
        {
            "category_name": "拌面类",
            "total_sales": 30.99
        },
        {
            "category_name": "儿童套餐",
            "total_sales": 19.99
        },
        {
            "category_name": "加料区",
            "total_sales": 15.99
        },
        {
            "category_name": "汤品类",
            "total_sales": 8.99
        },
        {
            "category_name": "凉菜类",
            "total_sales": 2.99
        }
    ]
}

# 4.用户评价 http://127.0.0.1:8000/reviews

{
    "data": [
        {
            "username": "user1",
            "food_name": "牛肉拉面",
            "rating": 5,
            "comment": "牛肉量足汤头鲜美！",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user1",
            "food_name": "牛肉拉面",
            "rating": 5,
            "comment": "非常好吃！",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user10",
            "food_name": "儿童拉面",
            "rating": 2,
            "comment": "儿童面太咸了",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user10",
            "food_name": "煎饺",
            "rating": 4,
            "comment": "汤很鲜",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user2",
            "food_name": "大盘鸡",
            "rating": 4,
            "comment": "味道不错",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user3",
            "food_name": "羊肉拌面",
            "rating": 3,
            "comment": "一般般",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user4",
            "food_name": "番茄蛋汤",
            "rating": 5,
            "comment": "很香脆",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user5",
            "food_name": "大盘鸡",
            "rating": 4,
            "comment": "鸡肉很入味，稍微有点辣",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user5",
            "food_name": "酸梅汤",
            "rating": 4,
            "comment": "分量足",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user6",
            "food_name": "凉拌黄瓜",
            "rating": 5,
            "comment": "冰镇可乐很爽",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user7",
            "food_name": "牛肉拉面",
            "rating": 5,
            "comment": "每次必点的经典款",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user7",
            "food_name": "双人套餐",
            "rating": 3,
            "comment": "甜品有点甜",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user8",
            "food_name": "羊肉拌面",
            "rating": 3,
            "comment": "羊肉有点膻味",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user8",
            "food_name": "酸梅汤",
            "rating": 4,
            "comment": "酸梅汤解腻不错",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user8",
            "food_name": "加牛肉",
            "rating": 4,
            "comment": "健康又美味",
            "review_time": "2025-03-14T06:24:45"
        },
        {
            "username": "user9",
            "food_name": "大盘鸡",
            "rating": 5,
            "comment": "大盘鸡分量超级足",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user9",
            "food_name": "双人套餐",
            "rating": 4,
            "comment": "套餐搭配很实惠",
            "review_time": "2025-03-14T06:21:59"
        },
        {
            "username": "user9",
            "food_name": "儿童拉面",
            "rating": 5,
            "comment": "小朋友很喜欢",
            "review_time": "2025-03-14T06:24:45"
        }
    ]
}

#5.热销商品排名 http://127.0.0.1:8000/top_selling_foods

{
    "data": [
        {
            "food_name": "牛肉拉面",
            "total_sold": 7
        },
        {
            "food_name": "酸梅汤",
            "total_sold": 4
        },
        {
            "food_name": "大盘鸡",
            "total_sold": 4
        },
        {
            "food_name": "双人套餐",
            "total_sold": 3
        },
        {
            "food_name": "煎饺",
            "total_sold": 3
        },
        {
            "food_name": "加牛肉",
            "total_sold": 2
        },
        {
            "food_name": "羊肉拌面",
            "total_sold": 2
        },
        {
            "food_name": "儿童拉面",
            "total_sold": 2
        },
        {
            "food_name": "番茄蛋汤",
            "total_sold": 1
        },
        {
            "food_name": "凉拌黄瓜",
            "total_sold": 1
        }
    ]
}


# 6.查看订单状态及用户信息 http://127.0.0.1:8000/order_status/2

{
    "data": [
        {
            "order_id": 2,
            "status": "已接单",
            "username": "user2",
            "phone": "13800002222"
        }
    ]
}


# 7.获取某个食品的所有评价 http://127.0.0.1:8000/food_reviews/2

{
    "data": [
        {
            "food_name": "羊肉拌面",
            "username": "user8",
            "rating": 3,
            "comment": "羊肉有点膻味"
        },
        {
            "food_name": "羊肉拌面",
            "username": "user3",
            "rating": 3,
            "comment": "一般般"
        }
    ]
}

# 8.用户个人资料及近期订单 http://127.0.0.1:8000/user_profile/2

{
    "data": [
        {
            "user_id": 2,
            "username": "user2",
            "password_hash": "32250170a0dca92d53ec9624f336ca24",
            "phone": "13800002222",
            "avatar_url": "avatar2.jpg",
            "created_at": "2025-03-14T06:21:35",
            "order_id": 12,
            "total_amount": 15.99,
            "order_time": "2025-03-14T06:24:00"
        },
        {
            "user_id": 2,
            "username": "user2",
            "password_hash": "32250170a0dca92d53ec9624f336ca24",
            "phone": "13800002222",
            "avatar_url": "avatar2.jpg",
            "created_at": "2025-03-14T06:21:35",
            "order_id": 2,
            "total_amount": 88.0,
            "order_time": "2025-03-14T06:21:51"
        }
    ]
}
```

 

step4:fastapi接口测试完毕，说明接口都正确，然后写angular前端代码
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\food\food.service.ts

```typescript

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class FoodService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  getMenu(): Observable<any> {
    return this.http.get(`${this.apiUrl}/menu`);
  }

  getOrderDetails(orderId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/order_details/${orderId}`);
  }

  getSalesByCategory(): Observable<any> {
    return this.http.get(`${this.apiUrl}/sales_by_category`);
  }

  getReviews(): Observable<any> {
    return this.http.get(`${this.apiUrl}/reviews`);
  }

  getTopSellingFoods(): Observable<any> {
    return this.http.get(`${this.apiUrl}/top_selling_foods`);
  }

  getOrderStatus(orderId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/order_status/${orderId}`);
  }

  getFoodReviews(foodId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/food_reviews/${foodId}`);
  }

  getUserProfile(userId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/user_profile/${userId}`);
  }
}

```

step5:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\food\food.component.ts

```typescript

import { Component,OnInit } from '@angular/core';

import { FoodService } from './food.service';
import { DatePipe, CurrencyPipe, NgForOf } from '@angular/common';



@Component({
  selector: 'app-food',
  imports: [DatePipe, CurrencyPipe, NgForOf],
  templateUrl: './food.component.html',
  styleUrl: './food.component.css'
})
export class FoodComponent  implements OnInit {
  menu: any[] = [];
  orderDetails: any[] = [];
  salesByCategory: any[] = [];
  reviews: any[] = [];
  topSellingFoods: any[] = [];
  orderStatus: any = {};
  foodReviews: any[] = [];
  userProfile: any = {};

  constructor(private foodService: FoodService) {}

  ngOnInit() {
    this.loadAllData();
  }

  loadAllData() {
    this.loadMenu();
    this.loadOrderDetails(2);
    this.loadSalesByCategory();
    this.loadReviews();
    this.loadTopSellingFoods();
    this.loadOrderStatus(2);
    this.loadFoodReviews(2);
    this.loadUserProfile(2);
  }

  loadMenu() {
    this.foodService.getMenu()
      .subscribe((res: any) => this.menu = res.data);
  }

  loadOrderDetails(orderId: number) {
    this.foodService.getOrderDetails(orderId)
      .subscribe((res: any) => this.orderDetails = res.data);
  }

  loadSalesByCategory() {
    this.foodService.getSalesByCategory()
      .subscribe((res: any) => this.salesByCategory = res.data);
  }

  loadReviews() {
    this.foodService.getReviews()
      .subscribe((res: any) => this.reviews = res.data);
  }

  loadTopSellingFoods() {
    this.foodService.getTopSellingFoods()
      .subscribe((res: any) => this.topSellingFoods = res.data);
  }

  loadOrderStatus(orderId: number) {
    this.foodService.getOrderStatus(orderId)
      .subscribe((res: any) => this.orderStatus = res.data);
  }

  loadFoodReviews(foodId: number) {
    this.foodService.getFoodReviews(foodId)
      .subscribe((res: any) => this.foodReviews = res.data);
  }

  loadUserProfile(userId: number) {
    this.foodService.getUserProfile(userId)
      .subscribe((res: any) => this.userProfile = res.data);
  }
}

```

step6:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\food\food.component.html

```xml
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>餐饮管理系统数据展示</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
<div class="data-container">
  <!-- 菜单展示 -->
  <h2 class="section-title">菜单展示</h2>
  <table class="data-table">
    <thead>
    <tr>
      <th>食品名称</th>
      <th>描述</th>
      <th>价格</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let item of menu">
      <td>{{ item.food_name }}</td>
      <td>{{ item.description }}</td>
      <td class="number highlight">{{ item.price | currency }}</td>
    </tr>
    </tbody>
  </table>

  <!-- 订单详情 -->
  <h2 class="section-title">订单详情</h2>
  <table class="data-table">
    <thead>
    <tr>
      <th>食品名称</th>
      <th>数量</th>
      <th>单价</th>
      <th>小计</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let item of orderDetails">
      <td>{{ item.food_name }}</td>
      <td class="number">{{ item.quantity }}</td>
      <td class="number">{{ item.unit_price | currency }}</td>
      <td class="number highlight">{{ item.subtotal | currency }}</td>
    </tr>
    </tbody>
  </table>

  <!-- 销售统计 -->
  <h2 class="section-title">销售统计（按分类）</h2>
  <table class="data-table">
    <thead>
    <tr>
      <th>分类名称</th>
      <th>销售额</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let item of salesByCategory">
      <td>{{ item.category_name }}</td>
      <td class="number highlight">{{ item.total_sales | currency }}</td>
    </tr>
    </tbody>
  </table>

  <!-- 用户评价 -->
  <h2 class="section-title">用户评价</h2>
  <table class="data-table">
    <thead>
    <tr>
      <th>用户</th>
      <th>食品</th>
      <th>评分</th>
      <th>评价内容</th>
      <th>时间</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let item of reviews">
      <td>{{ item.username }}</td>
      <td>{{ item.food_name }}</td>
      <td class="number">{{ item.rating }}/5</td>
      <td>{{ item.comment }}</td>
      <td>{{ item.review_time | date }}</td>
    </tr>
    </tbody>
  </table>

  <!-- 热销商品 -->
  <h2 class="section-title">热销商品排名</h2>
  <table class="data-table">
    <thead>
    <tr>
      <th>商品名称</th>
      <th>销量</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let item of topSellingFoods">
      <td>{{ item.food_name }}</td>
      <td class="number">{{ item.total_sold }}</td>
    </tr>
    </tbody>
  </table>

  <!-- 订单状态 -->
  <h2 class="section-title">订单状态及用户信息</h2>
  <table class="data-table">
    <thead>
    <tr>
      <th>订单号</th>
      <th>状态</th>
      <th>用户</th>
      <th>电话</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let item of orderStatus">
      <td>#{{ item.order_id }}</td>
      <td>{{ item.status }}</td>
      <td>{{ item.username }}</td>
      <td>{{ item.phone }}</td>
    </tr>
    </tbody>
  </table>

  <!-- 用户资料 -->
  <h2 class="section-title">用户个人资料及近期订单</h2>
  <table class="data-table">
    <thead>
    <tr>
      <th>用户</th>
      <th>电话</th>
      <th>订单号</th>
      <th>金额</th>
      <th>时间</th>
    </tr>
    </thead>
    <tbody>
    <tr *ngFor="let item of userProfile">
      <td>{{ item.username }}</td>
      <td>{{ item.phone }}</td>
      <td>#{{ item.order_id }}</td>
      <td class="number highlight">{{ item.total_amount | currency }}</td>
      <td>{{ item.order_time | date }}</td>
    </tr>
    </tbody>
  </table>
</div>
</body>
</html>

```

step7:
C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\food\food.component.css

```css
/* styles.css */
/* 基础样式 */
body {
  font-family: 'Microsoft YaHei', 'Segoe UI', sans-serif;
  line-height: 1.6;
  background-color: #f8f9fa;
}

.data-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

/* 模块标题样式 */
.section-title {
  color: #2c3e50;
  border-bottom: 3px solid #3498db;
  padding: 10px 0;
  margin: 40px 0 20px;
  font-size: 1.4em;
}

/* 数据表格样式 */
.data-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 30px;
  background: white;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.data-table th {
  background-color: #3498db;
  color: white;
  padding: 15px 20px;
  text-align: left;
  font-weight: 500;
}

.data-table td {
  padding: 12px 20px;
  border-bottom: 1px solid #ecf0f1;
  color: #34495e;
}

.data-table tr:hover {
  background-color: #f4f9ff;
}

/* 数字列样式 */
.number {
  text-align: right;
  font-family: 'Courier New', monospace;
}

.highlight {
  color: #e74c3c;
  font-weight: 600;
}

/* 响应式表格 */
@media (max-width: 768px) {
  .data-container {
    padding: 10px;
  }

  .data-table {
    display: block;
    overflow-x: auto;
  }

  .section-title {
    font-size: 1.2em;
    margin: 30px 0 15px;
  }
}

/* 斑马条纹 */
.data-table tbody tr:nth-child(even) {
  background-color: #f8fbff;
}

/* 表头固定 */
@media (min-width: 1024px) {
  .data-table thead {
    position: sticky;
    top: 0;
    z-index: 1;
  }
}

```

 


end