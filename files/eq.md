  说明：
我希望用fastapi写几个接口，查询房产交易系统的几条数据，然后在postman里面测试
1. 查询客户所有预约记录（含房源信息）需要对应销售经理 
2. 查询客户所有订单（含房源信息）
3. 统计销售经理名下所有房源销售情况和销售金额
4. 查看房源详情及评价列表

step1:sql数据库，建表，添加数据，写查询sql

```sql


-- 用户表（管理员/客户/销售经理）
CREATE TABLE `user` (
                        `user_id` INT PRIMARY KEY AUTO_INCREMENT,
                        `username` VARCHAR(50) UNIQUE NOT NULL,
                        `password` VARCHAR(100) NOT NULL,
                        `role` ENUM('admin', 'customer', 'sales') NOT NULL DEFAULT 'customer',
                        `real_name` VARCHAR(50),
                        `phone` VARCHAR(20),
                        `email` VARCHAR(50),
                        `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP
);



-- 房源信息表
CREATE TABLE `house` (
                         `house_id` INT PRIMARY KEY AUTO_INCREMENT,
                         `title` VARCHAR(100) NOT NULL,
                         `price` DECIMAL(12,2) NOT NULL,
                         `area` DECIMAL(6,2) COMMENT '面积（平方米）',
                         `room_type` VARCHAR(20) COMMENT '户型（如3室2厅）',
                         `address` VARCHAR(200),
                         `status` ENUM('pending', 'listed', 'sold') DEFAULT 'pending',
                         `sales_id` INT COMMENT '负责的销售经理ID',
                         `description` TEXT,
                         `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP,
                         FOREIGN KEY (`sales_id`) REFERENCES `user`(`user_id`)
);



-- 预约看房表
CREATE TABLE `appointment` (
                               `appoint_id` INT PRIMARY KEY AUTO_INCREMENT,
                               `user_id` INT NOT NULL,
                               `house_id` INT NOT NULL,
                               `appoint_time` DATETIME NOT NULL,
                               `status` ENUM('pending', 'confirmed', 'canceled') DEFAULT 'pending',
                               `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP,
                               FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`),
                               FOREIGN KEY (`house_id`) REFERENCES `house`(`house_id`)
);



-- 订单表
CREATE TABLE `order` (
                         `order_id` VARCHAR(32) PRIMARY KEY COMMENT '订单号（如UUID）',
                         `user_id` INT NOT NULL,
                         `house_id` INT NOT NULL,
                         `total_price` DECIMAL(12,2) NOT NULL,
                         `payment_status` ENUM('unpaid', 'paid') DEFAULT 'unpaid',
                         `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP,
                         FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`),
                         FOREIGN KEY (`house_id`) REFERENCES `house`(`house_id`)
);



-- 房源评价表
CREATE TABLE `comment` (
                           `comment_id` INT PRIMARY KEY AUTO_INCREMENT,
                           `user_id` INT NOT NULL,
                           `house_id` INT NOT NULL,
                           `content` TEXT NOT NULL,
                           `rating` TINYINT CHECK (rating BETWEEN 1 AND 5),
                           `create_time` DATETIME DEFAULT CURRENT_TIMESTAMP,
                           FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`),
                           FOREIGN KEY (`house_id`) REFERENCES `house`(`house_id`)
);



-- 用户表（包含10条记录）
INSERT INTO `user` (username, password, role, real_name, phone, email, create_time) VALUES
                                                                                        ('admin', '123456', 'admin', '系统管理员', '13800000000', 'admin@example.com', '2025-03-01 09:00:00'),
                                                                                        ('sales_01', '123456', 'sales', '张伟', '13911111111', 'sales_01@example.com', '2025-03-01 09:00:00'),
                                                                                        ('sales_02', '123456', 'sales', '李娜', '13922222222', 'sales_02@example.com', '2025-03-01 09:00:00'),
                                                                                        ('sales_03', '123456', 'sales', '王刚', '13933333333', 'sales_03@example.com', '2025-03-01 09:00:00'),
                                                                                        ('customer_01', '123456', 'customer', '陈浩', '13844444444', 'customer_01@example.com', '2025-03-01 09:00:00'),
                                                                                        ('customer_02', '123456', 'customer', '刘芳', '13855555555', 'customer_02@example.com', '2025-03-01 09:00:00'),
                                                                                        ('customer_03', '123456', 'customer', '张婷', '13866666666', 'customer_03@example.com', '2025-03-01 09:00:00'),
                                                                                        ('customer_04', '123456', 'customer', '廖学彬', '13877777777', 'customer_04@example.com', '2025-03-01 09:00:00'),
                                                                                        ('customer_05', '123456', 'customer', '王芳', '13888888888', 'customer_05@example.com', '2025-03-01 09:00:00'),
                                                                                        ('customer_06', '123456', 'customer', '赵敏', '13899999999', 'customer_06@example.com', '2025-03-01 09:00:00');

-- 房源信息表（10条记录）
INSERT INTO `house` (title, price, area, room_type, address, sales_id, status, description, create_time) VALUES
                                                                                                             ('SOHO', 12000000.00, 150.0, '3室2厅', '北街道', 2, 'listed', '高端商务公寓，带健身房和游泳池', '2025-03-01 09:00:00'),
                                                                                                             ('村学区房', 18000000.00, 120.0, '2室1厅', '北大街', 3, 'sold', '重点小学学区房，南北通透', '2025-03-01 09:00:00'),
                                                                                                             ('陆景房', 25000000.00, 200.0, '4室3厅', '上环路', 4, 'pending', '高层江景住宅，视野开阔', '2025-03-01 09:00:00'),
                                                                                                             ('金街铺', 30000000.00, 80.0, '临街商铺', '北融街', 2, 'listed', '黄金地段商铺，租金回报率高', '2025-03-01 09:00:00'),
                                                                                                             ('东环别墅', 60000000.00, 300.0, '独栋别墅', '北北路', 3, 'sold', '豪华独栋别墅，带私人花园', '2025-03-01 09:00:00'),
                                                                                                             ('科技园写字楼', 80000000.00, 150.0, '甲级写字楼', '深技园', 4, 'pending', '现代化办公大楼，配套完善', '2025-03-01 09:00:00'),
                                                                                                             ('老洋房', 45000000.00, 100.0, '2室1厅', '上路', 2, 'listed', '历史建筑改造，复古风格', '2025-03-01 09:00:00'),
                                                                                                             ('广高端公寓', 10000000.00, 130.0, '3室2厅', '广江新城', 3, 'sold', 'CBD核心区高端公寓', '2025-03-01 09:00:00'),
                                                                                                             ('州湖区景观房', 28000000.00, 180.0, '4室2厅', '州杨公堤', 4, 'pending', '湖滨景观住宅，环境优美', '2025-03-01 09:00:00'),
                                                                                                             ('都府新科技园', 15000000.00, 120.0, '3室2厅', '都府大道', 2, 'listed', '新兴科技产业园区，交通便利', '2025-03-01 09:00:00');

-- 预约看房表（10条记录）
INSERT INTO `appointment` (user_id, house_id, appoint_time, status, create_time) VALUES
                                                                                     (6, 1, '2025-04-15 10:00:00', 'confirmed', '2025-04-10 09:00:00'),
                                                                                     (7, 2, '2025-04-16 14:30:00', 'pending', '2025-04-10 09:00:00'),
                                                                                     (8, 3, '2025-04-17 09:00:00', 'canceled', '2025-04-10 09:00:00'),
                                                                                     (9, 4, '2025-04-18 11:00:00', 'confirmed', '2025-04-10 09:00:00'),
                                                                                     (10,5, '2025-04-19 15:00:00', 'pending', '2025-04-10 09:00:00'),
                                                                                     (6, 6, '2025-04-20 10:00:00', 'confirmed', '2025-04-10 09:00:00'),
                                                                                     (7, 7, '2025-04-21 14:00:00', 'canceled', '2025-04-10 09:00:00'),
                                                                                     (8, 8, '2025-04-22 09:00:00', 'confirmed', '2025-04-10 09:00:00'),
                                                                                     (9, 9, '2025-04-23 11:00:00', 'pending', '2025-04-10 09:00:00'),
                                                                                     (10,10, '2025-04-24 15:00:00', 'confirmed', '2025-04-10 09:00:00');

-- 订单表（10条记录）
INSERT INTO `order` (order_id, user_id, house_id, total_price, payment_status, create_time) VALUES
                                                                                                ('202504100001', 6, 1, 12000000.00, 'paid', '2025-04-10 09:00:00'),
                                                                                                ('202504100002', 7, 2, 18000000.00, 'unpaid', '2025-04-10 09:00:00'),
                                                                                                ('202504100003', 8, 3, 25000000.00, 'paid', '2025-04-10 09:00:00'),
                                                                                                ('202504100004', 9, 4, 30000000.00, 'unpaid', '2025-04-10 09:00:00'),
                                                                                                ('202504100005', 10,5, 60000000.00, 'paid', '2025-04-10 09:00:00'),
                                                                                                ('202504100006', 6, 6, 80000000.00, 'unpaid', '2025-04-10 09:00:00'),
                                                                                                ('202504100007', 7, 7, 45000000.00, 'paid', '2025-04-10 09:00:00'),
                                                                                                ('202504100008', 8, 8, 10000000.00, 'unpaid', '2025-04-10 09:00:00'),
                                                                                                ('202504100009', 9, 9, 28000000.00, 'paid', '2025-04-10 09:00:00'),
                                                                                                ('202504100010', 10,10, 15000000.00, 'unpaid', '2025-04-10 09:00:00');

-- 房源评价表（10条记录）
INSERT INTO `comment` (user_id, house_id, content, rating, create_time) VALUES
                                                                            (6, 1, '房屋设施非常完善，交通便利！', 5, '2025-04-10 09:00:00'),
                                                                            (7, 2, '学区房位置优越，但周边配套一般。', 4, '2025-04-10 09:00:00'),
                                                                            (8, 3, '江景房视野开阔，但物业费较高。', 3, '2025-04-10 09:00:00'),
                                                                            (9, 4, '商铺租金回报率高，适合投资。', 5, '2025-04-10 09:00:00'),
                                                                            (10,5, '别墅面积大，适合家庭居住。', 4, '2025-04-10 09:00:00'),
                                                                            (6,6, '写字楼采光好，租赁需求旺。', 5, '2025-04-10 09:00:00'),
                                                                            (7,7, '老洋房设计独特，但维护成本高。', 3, '2025-04-10 09:00:00'),
                                                                            (8,8, '公寓性价比高，适合年轻人。', 4, '2025-04-10 09:00:00'),
                                                                            (9,9, '景观房环境优美，但离市区较远。', 3, '2025-04-10 09:00:00'),
                                                                            (10,10, '科技园区交通便利，未来发展潜力大。', 5, '2025-04-10 09:00:00');



# 1. 查询客户所有预约记录（含房源信息）需要对应销售经理
SELECT
    u.real_name AS customer_name,
    u.user_id AS customer_id,
    a.appoint_time,
    a.status,
    h.title,
    h.address,
    h.price,
    h.room_type,
    s.real_name,
    s.phone
FROM
    appointment a
        INNER JOIN user u ON a.user_id = u.user_id
        LEFT JOIN house h ON a.house_id = h.house_id
        INNER JOIN user s ON h.sales_id = s.user_id
WHERE
    u.role = 'customer'
ORDER BY
    a.create_time DESC;


# 2. 查询客户所有订单（含房源信息）
SELECT o.order_id, u.real_name AS customer_name,  u.user_id AS customer_id,o.total_price,
       o.payment_status, h.title, h.address, h.status AS house_status
FROM `order` o
         JOIN user u ON o.user_id = u.user_id
         JOIN house h ON o.house_id = h.house_id
WHERE u.role = 'customer'
ORDER BY o.create_time DESC;

# 3. 统计销售经理名下所有房源销售情况和销售金额 ,并列出房源位置信息，和交易时间
SELECT u.real_name AS sales_manager,
       COUNT(DISTINCT h.house_id) AS sold_count,
       SUM(o.total_price) AS total_sales
FROM house h
         JOIN user u ON h.sales_id = u.user_id
         JOIN `order` o ON h.house_id = o.house_id
WHERE u.role = 'sales' AND h.status = 'sold'
GROUP BY u.user_id;

# 4. 查看房源详情及评价列表
SELECT h.title, h.price, h.area, h.room_type, h.address,
       c.content AS evaluation, c.rating,c.house_id, c.content,  c.create_time AS eval_time
FROM house h
         LEFT JOIN comment c ON h.house_id = c.house_id
ORDER BY h.house_id, c.create_time DESC;


  

```

step2:fastapi路由和查询 C:\Users\Administrator\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException
import pymysql.cursors
app = FastAPI()
# 数据库连接配置
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'school_db',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}
# 查询数据库的函数
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

@app.get("/customer_appointments/{customer_id}")
async def get_customer_appointments(customer_id: int):
    query = """
    SELECT 
        a.appoint_time, a.status, a.create_time,
        h.title, h.address, h.room_type, h.price,
        s.real_name AS sales_manager, s.phone
    FROM appointment a
    INNER JOIN user u ON a.user_id = u.user_id
    LEFT JOIN house h ON a.house_id = h.house_id
    INNER JOIN user s ON h.sales_id = s.user_id
    WHERE u.role = 'customer' AND a.user_id = %s
    ORDER BY a.create_time DESC
    """
    data = query_database(query, (customer_id,))
    return {"data": data}

@app.get("/customer_orders/{customer_id}")
async def get_customer_orders(customer_id: int):
    query = """
    SELECT 
        o.order_id, o.total_price, o.payment_status, o.create_time,
        h.title, h.address, h.status AS house_status
    FROM `order` o
    JOIN user u ON o.user_id = u.user_id
    JOIN house h ON o.house_id = h.house_id
    WHERE u.role = 'customer' AND o.user_id = %s
    ORDER BY o.create_time DESC
    """
    data = query_database(query, (customer_id,))
    return {"data": data}


@app.get("/sales_stats/{sales_id}")
async def get_sales_stats(sales_id: int):
    query = """
    SELECT 
        COUNT(DISTINCT o.house_id) AS sold_count,
        SUM(o.total_price) AS total_sales,
        AVG(o.total_price) AS avg_price
    FROM `order` o
    JOIN house h ON o.house_id = h.house_id
    WHERE h.sales_id = %s AND h.status = 'sold'
    """
    data = query_database(query, (sales_id,))
    return {"data": data[0] if data else {}}


@app.get("/house_details/{house_id}")
async def get_house_details(house_id: int):
    # 房源基本信息
    house_query = """
    SELECT title, price, area, room_type, address, 
           description, status, create_time
    FROM house WHERE house_id = %s
    """
    house_data = query_database(house_query, (house_id,))

    # 关联评价信息
    comment_query = """
    SELECT c.content, c.rating, c.create_time, u.real_name
    FROM comment c
    JOIN user u ON c.user_id = u.user_id
    WHERE c.house_id = %s
    ORDER BY c.create_time DESC
    """
    comment_data = query_database(comment_query, (house_id,))

    return {
        "house_info": house_data[0] if house_data else {},
        "comments": comment_data
    }
# 启动应用
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:postman验证

```bash
用fastapi帮我写4个查询接口，参考下面的代码格式 
1. 查询客户所有预约记录（含房源信息）需要对应销售经理 
2. 查询客户所有订单（含房源信息）
 3. 统计销售经理名下所有房源销售情况和销售金额
 4. 查看房源详情及评价列表


http://localhost:8000/customer_appointments/6

{
    "data": [
        {
            "appoint_time": "2025-04-15T10:00:00",
            "status": "confirmed",
            "create_time": "2025-04-10T09:00:00",
            "title": "朝",
            "address": "街道",
            "room_type": "3室2厅",
            "price": 12000000.0,
            "sales_manager": "张伟",
            "phone": "13911111111"
        },
        {
            "appoint_time": "2025-04-20T10:00:00",
            "status": "confirmed",
            "create_time": "2025-04-10T09:00:00",
            "title": "字楼",
            "address": "技园",
            "room_type": "楼",
            "price": 80000000.0,
            "sales_manager": "王刚",
            "phone": "13933333333"
        }
    ]
}

http://localhost:8000/customer_orders/6

{
    "data": [
        {
            "order_id": "202504100001",
            "total_price": 12000000.0,
            "payment_status": "paid",
            "create_time": "2025-04-10T09:00:00",
            "title": "",
            "address": "道",
            "house_status": "listed"
        },
        {
            "order_id": "202504100006",
            "total_price": 80000000.0,
            "payment_status": "unpaid",
            "create_time": "2025-04-10T09:00:00",
            "title": "字楼",
            "address": "技园",
            "house_status": "pending"
        }
    ]
}


http://localhost:8000/sales_stats/3

{
    "data": {
        "sold_count": 3,
        "total_sales": 88000000.0,
        "avg_price": 29333333.333333
    }
}

http://localhost:8000/house_details/1

{
    "house_info": {
        "title": "",
        "price": 12000000.0,
        "area": 150.0,
        "room_type": "3室2厅",
        "address": "道",
        "description": "高端商务公寓，带健身房和游泳池",
        "status": "listed",
        "create_time": "2025-03-01T09:00:00"
    },
    "comments": [
        {
            "content": "房屋设施非常完善，交通便利！",
            "rating": 5,
            "create_time": "2025-04-10T09:00:00",
            "real_name": "刘芳"
        }
    ]
}
```


end