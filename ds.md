说明：
律师系统

1.用户表（包含 委托人和委托律师，用户姓名，联系电话，头像url，用户标签，所属公司）

3.委托分类表 婚姻家庭 财务纠纷   劳资问题，商业合同，刑事辩护，知识产权，金融证券，拆迁补偿等等

3.用户发布法律委托信息表，（委托分类，标题，描述，价格，委托人信息，受委托律师信息）

4.咨询订单表，（单号，价格，时间，）

5.咨询订单记录

6.评价表（用户下单完成服务后，可以对服务进行评价）

step1:mysql

```sql


show databases;

DROP TABLE users;


SHOW CREATE TABLE db_school.users;

show tables;

use db_school;

SELECT * FROM db_school.jewelry_categories;

CREATE DATABASE db_school;


-- 用户表（委托人/律师共用）
CREATE TABLE `user` (
  `user_id` INT PRIMARY KEY AUTO_INCREMENT,
  `name` VARCHAR(255) NOT NULL,
  `phone` VARCHAR(20) NOT NULL UNIQUE,
  `avatar_url` VARCHAR(255),
  `company` VARCHAR(255),
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
  `updated_at` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 用户标签多对多关系表
CREATE TABLE `tag` (
  `tag_id` INT PRIMARY KEY AUTO_INCREMENT,
  `tag_name` VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE `user_tag` (
  `user_id` INT NOT NULL,
  `tag_id` INT NOT NULL,
  PRIMARY KEY (`user_id`, `tag_id`),
  FOREIGN KEY (`user_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`tag_id`) REFERENCES `tag`(`tag_id`)
);

-- 委托分类表
CREATE TABLE `case_category` (
  `category_id` INT PRIMARY KEY AUTO_INCREMENT,
  `category_name` VARCHAR(50) UNIQUE NOT NULL
);

-- 法律委托信息表
CREATE TABLE `legal_case` (
  `case_id` INT PRIMARY KEY AUTO_INCREMENT,
  `category_id` INT NOT NULL,
  `title` VARCHAR(255) NOT NULL,
  `description` TEXT,
  `price` DECIMAL(10,2) NOT NULL,
  `client_id` INT NOT NULL,
  `lawyer_id` INT,
  `status` ENUM('pending', 'in_progress', 'completed', 'closed') DEFAULT 'pending',
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
  `updated_at` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (`category_id`) REFERENCES `case_category`(`category_id`),
  FOREIGN KEY (`client_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`lawyer_id`) REFERENCES `user`(`user_id`)
);


select *from consult_order;

-- 咨询订单表
CREATE TABLE `consult_order` (
  `order_id` INT PRIMARY KEY AUTO_INCREMENT,
  `order_number` VARCHAR(50) UNIQUE NOT NULL,
  `price` DECIMAL(10,2) NOT NULL,
  `client_id` INT NOT NULL,
  `lawyer_id` INT NOT NULL,
  `case_id` INT,
  `status` ENUM('pending_payment', 'paid', 'in_service', 'completed', 'cancelled') DEFAULT 'pending_payment',
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
  `updated_at` DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (`client_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`lawyer_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`case_id`) REFERENCES `legal_case`(`case_id`)
);

-- 咨询记录表
CREATE TABLE `consult_record` (
  `record_id` INT PRIMARY KEY AUTO_INCREMENT,
  `order_id` INT NOT NULL,
  `consult_time` DATETIME NOT NULL,
  `notes` TEXT,
  `duration` INT,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (`order_id`) REFERENCES `consult_order`(`order_id`)
);


select *from review;

-- 评价表
CREATE TABLE `review` (
  `review_id` INT PRIMARY KEY AUTO_INCREMENT,
  `order_id` INT UNIQUE NOT NULL,
  `rating` TINYINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
  `comment` TEXT,
  `reviewer_id` INT NOT NULL,
  `reviewee_id` INT NOT NULL,
  `created_at` DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (`order_id`) REFERENCES `consult_order`(`order_id`),
  FOREIGN KEY (`reviewer_id`) REFERENCES `user`(`user_id`),
  FOREIGN KEY (`reviewee_id`) REFERENCES `user`(`user_id`)
);

CREATE INDEX idx_legal_case_status ON legal_case(status);
CREATE INDEX idx_consult_order_status ON consult_order(status);
CREATE INDEX idx_user_company ON user(company);


select *from user;

-- 插入用户数据（混合委托人和律师）
INSERT INTO `user` (`name`, `phone`, `avatar_url`, `company`, `created_at`) VALUES
('张三', '13800010001', 'https://avatar.com/1.jpg', NULL, '2024-01-01 09:00:00'),
('李四律师', '13800020002', 'https://avatar.com/2.jpg', '正义律师事务所', '2024-01-02 10:00:00'),
('王五', '13800030003', 'https://avatar.com/3.jpg', NULL, '2024-01-03 11:00:00'),
('赵六律师', '13800040004', 'https://avatar.com/4.jpg', '天平律师事务所', '2024-01-04 12:00:00'),
('陈七', '13800050005', 'https://avatar.com/5.jpg', NULL, '2024-01-05 13:00:00'),
('林八律师', '13800060006', 'https://avatar.com/6.jpg', '法理律师事务所', '2024-01-06 14:00:00'),
('黄九', '13800070007', 'https://avatar.com/7.jpg', NULL, '2024-01-07 15:00:00'),
('周十律师', '13800080008', 'https://avatar.com/8.jpg', '维法律师事务所', '2024-01-08 16:00:00'),
('吴十一', '13800090009', 'https://avatar.com/9.jpg', NULL, '2024-01-09 17:00:00'),
('郑十二律师', '13800100010', 'https://avatar.com/10.jpg', '明理律师事务所', '2024-01-10 18:00:00');

-- 插入标签数据
INSERT INTO `tag` (`tag_name`) VALUES
('婚姻法'), ('合同法'), ('劳动法'), ('刑事辩护'), ('知识产权'),
('房产纠纷'), ('交通事故'), ('公司法律'), ('遗产继承'), ('涉外法律');

-- 插入用户标签关联数据
INSERT INTO `user_tag` (`user_id`, `tag_id`) VALUES
(2,1), (2,4), (2,9),  -- 李四律师
(4,2), (4,5), (4,8),  -- 赵六律师
(6,3), (6,6), (6,7),  -- 林八律师
(8,1), (8,4), (8,10), -- 周十律师
(10,5), (10,8), (10,9); -- 郑十二律师

-- 插入委托分类数据
INSERT INTO `case_category` (`category_name`) VALUES
('民事诉讼'), ('刑事辩护'), ('婚姻家事'), ('合同纠纷'),
('劳动纠纷'), ('知识产权'), ('交通事故'), ('房产纠纷'),
('公司法律'), ('涉外案件');


INSERT INTO db_school.legal_case (case_id, category_id, title, description, price, client_id, lawyer_id, status, created_at, updated_at)
VALUES (12, 3, '我要举报垃圾食品', '食品有毒', 8000.00, 7, null, 'pending', '2025-03-21 06:38:32', '2025-03-21 06:38:32');


select *from legal_case  ;

-- 第一步：更新案件信息
UPDATE db_school.legal_case
SET
    lawyer_id = 2,
    status = 'completed',
    updated_at = CURRENT_TIMESTAMP
WHERE case_id = 12;

-- 第二步：验证更新结果
SELECT * FROM db_school.consult_order;


-- 插入法律委托数据
INSERT INTO `legal_case` (`category_id`, `title`, `description`, `price`, `client_id`, `lawyer_id`, `status`) VALUES
(3, '离婚财产分割案', '涉及房产和存款的离婚财产分割', 5000.00, 1, 2, 'completed'),
(1, '民间借贷纠纷案', '20万元借款追讨诉讼', 3000.00, 3, 4, 'in_progress'),
(4, '房屋买卖合同违约', '开发商延期交房索赔', 4500.00, 5, 6, 'pending'),
(2, '故意伤害罪辩护', '轻伤二级刑事辩护', 8000.00, 7, 8, 'in_progress'),
(5, '劳动仲裁案件', '违法解除劳动合同赔偿', 2000.00, 9, 10, 'pending'),
(6, '商标侵权诉讼', '电商平台商标侵权维权', 12000.00, 1, 2, 'completed'),
(7, '交通事故赔偿案', '人伤事故保险理赔纠纷', 3500.00, 3, 4, 'closed'),
(8, '房产继承纠纷', '多继承人房产分配纠纷', 6000.00, 5, 6, 'in_progress'),
(9, '公司股权纠纷', '股东权益争议调解', 9000.00, 7, 8, 'pending'),
(10, '涉外婚姻咨询', '中美跨国离婚法律咨询', 2500.00, 9, 10, 'completed');

-- 插入咨询订单数据
INSERT INTO `consult_order` (`order_number`, `price`, `client_id`, `lawyer_id`, `case_id`, `status`) VALUES
('CO2024010001', 500.00, 1, 2, 1, 'completed'),
('CO2024010002', 300.00, 3, 4, 2, 'in_service'),
('CO2024010003', 200.00, 5, 6, 3, 'paid'),
('CO2024010004', 800.00, 7, 8, 4, 'pending_payment'),
('CO2024010005', 150.00, 9, 10, 5, 'completed'),
('CO2024010006', 600.00, 1, 2, 6, 'cancelled'),
('CO2024010007', 400.00, 3, 4, 7, 'completed'),
('CO2024010008', 350.00, 5, 6, 8, 'in_service'),
('CO2024010009', 250.00, 7, 8, 9, 'paid'),
('CO2024010010', 700.00, 9, 10, 10, 'completed');

INSERT INTO `consult_order` (`order_number`, `price`, `client_id`, `lawyer_id`, `case_id`, `status`) VALUES

('CO2024010011', 700.00, 9, 10, 10, 'completed');

select *from consult_order;

 SELECT * FROM consult_order;
-- 插入咨询记录数据
INSERT INTO `consult_record` (`order_id`, `consult_time`, `notes`, `duration`) VALUES
(1, '2024-01-05 14:00:00', '初步了解案情，收集证据材料', 60),
(1, '2024-01-07 10:30:00', '讨论诉讼策略', 45),
(2, '2024-01-08 15:00:00', '审核借款合同有效性', 30),
(3, '2024-01-09 11:00:00', '分析开发商违约证据', 50),
(4, '2024-01-10 09:00:00', '刑事辩护方案沟通', 90),
(5, '2024-01-11 16:00:00', '劳动仲裁流程讲解', 40),
(7, '2024-01-12 14:30:00', '保险理赔方案确认', 35),
(8, '2024-01-13 10:00:00', '房产分配方案讨论', 60),
(9, '2024-01-14 13:00:00', '股权纠纷调解建议', 55),
(10, '2024-01-15 15:30:00', '跨国法律差异讲解', 75);

ALTER TABLE review ADD UNIQUE (order_id);

-- 插入评价数据
INSERT INTO `review` (`order_id`, `rating`, `comment`, `reviewer_id`, `reviewee_id`) VALUES
(1, 5, '律师专业，解决问题效率高', 1, 2),
(5, 4, '服务态度很好，解答详细', 9, 10),
(7, 5, '成功帮助获得合理赔偿', 3, 4),
(10, 5, '对涉外法律非常熟悉', 9, 10),
(2, 3, '响应速度可以再提高些', 3, 4),
(3, 4, '专业度不错，方案可行', 5, 6),
(4, 2, '沟通不够及时', 7, 8),
(6, 1, '未能及时处理取消请求', 1, 2),
(8, 4, '给出了合理的分配方案', 5, 6),
(9, 5, '股权纠纷处理经验丰富', 7, 8);

-- 清空原有数据（如果需要）
TRUNCATE TABLE db_school.review;

select *from review;

# 1.查询婚姻家庭分类下的所有委托案件


SELECT lc.*, cc.category_name
FROM legal_case lc
JOIN case_category cc ON lc.category_id = cc.category_id
WHERE cc.category_id =2;

# 2.查询指定委托人发布的所有委托



SELECT lc.*, u.name AS client_name
FROM legal_case lc
JOIN user u ON lc.client_id = u.user_id
WHERE lc.client_id = 1;  -- 替换具体委托人ID

# 3.查询律师正在处理的案件



SELECT lc.*, u.name AS lawyer_name
FROM legal_case lc
JOIN user u ON lc.lawyer_id = u.user_id
WHERE lc.status = 'in_progress'
AND lc.lawyer_id = 2;  -- 替换具体律师ID


# 4.统计律师的订单数量和平均评分


SELECT
  u.user_id,
  u.name,
  COUNT(co.order_id) AS total_orders,
  AVG(r.rating) AS avg_rating
FROM user u
LEFT JOIN consult_order co ON u.user_id = co.lawyer_id
LEFT JOIN review r ON co.order_id = r.order_id
WHERE u.user_id = 2  -- 替换具体律师ID
GROUP BY u.user_id, u.name;


# 5.获取订单详情（含咨询记录和评价）


SELECT
  co.*,
  cr.consult_time,
  cr.notes,
  r.rating,
  r.comment
FROM consult_order co
LEFT JOIN consult_record cr ON co.order_id = cr.order_id
LEFT JOIN review r ON co.order_id = r.order_id
WHERE co.order_number = 'CO2024010001';  -- 替换具体订单号

# 6.查询某公司律师信息及其标签

SELECT
  u.user_id,
  u.name,
  u.phone,
  GROUP_CONCAT(t.tag_name) AS tags
FROM user u
LEFT JOIN user_tag ut ON u.user_id = ut.user_id
LEFT JOIN tag t ON ut.tag_id = t.tag_id
WHERE u.user_id = 2
GROUP BY u.user_id;

# 7.查询最近一个月完成的订单


SELECT
  co.*,
  r.rating,
  r.comment
FROM consult_order co
JOIN review r ON co.order_id = r.order_id
WHERE co.status = 'completed'
AND co.created_at >= DATE_SUB(NOW(), INTERVAL 1 MONTH);

# 8.统计各分类委托数量及平均价格
SELECT
  cc.category_name,
  COUNT(lc.case_id) AS case_count,
  AVG(lc.price) AS avg_price
FROM case_category cc
LEFT JOIN legal_case lc ON cc.category_id = lc.category_id
GROUP BY cc.category_id
ORDER BY case_count DESC;

# 9.查询用户的评价记录


SELECT
  r.*,
  reviewee.name AS reviewee_name,
  reviewer.name AS reviewer_name
FROM review r
JOIN user reviewee ON r.reviewee_id = reviewee.user_id
JOIN user reviewer ON r.reviewer_id = reviewer.user_id
WHERE r.reviewer_id = 1 OR r.reviewee_id = 1;  -- 替换具体用户ID

# 10.查询律师待处理委托


SELECT
  lc.*,
  u.name AS client_name
FROM legal_case lc
JOIN user u ON lc.client_id = u.user_id
WHERE lc.lawyer_id = 6  -- 替换具体律师ID
AND lc.status = 'pending';



```

step2:fastapi

```python

from fastapi import FastAPI, HTTPException, Query
from fastapi.middleware.cors import CORSMiddleware
import pymysql.cursors
from datetime import datetime, timedelta
from pydantic import BaseModel
from typing import Optional, List

app = FastAPI()

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

def db_query(query: str, params=None, fetch_one=False):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params or ())
            result = cursor.fetchone() if fetch_one else cursor.fetchall()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# Pydantic模型
class CaseCreate(BaseModel):
    category_id: int
    title: str
    description: str
    price: float
    client_id: int

class ReviewCreate(BaseModel):
    order_id: int
    rating: int
    comment: Optional[str] = None
    reviewer_id: int


# 在已有的Pydantic模型后添加新的OrderCreate模型
class OrderCreate(BaseModel):
    price: float
    client_id: int
    lawyer_id: int
    case_id: Optional[int] = None


# 生成订单号的辅助函数
def generate_order_number():
    today = datetime.now().strftime("CO%Y%m")
    query = """
    SELECT MAX(order_number) AS max_num 
    FROM consult_order 
    WHERE order_number LIKE %s
    """
    params = (f"{today}%",)
    result = db_query(query, params, fetch_one=True)

    if result and result["max_num"]:
        last_num = int(result["max_num"][-4:])
        new_num = last_num + 1
    else:
        new_num = 1
    return f"{today}{new_num:04d}"
# 1. 获取分类委托案件
@app.get("/cases/category/{category_id}")
async def get_cases_by_category(category_id: int):
    query = """
    SELECT lc.*, cc.category_name, u.name as client_name 
    FROM legal_case lc
    JOIN case_category cc ON lc.category_id = cc.category_id
    JOIN user u ON lc.client_id = u.user_id
    WHERE lc.category_id = %s  -- 改为使用ID进行查询
    """
    return {"data": db_query(query, (category_id,))}

# 2. 获取委托人发布的委托
@app.get("/users/{user_id}/cases")
async def get_user_cases(user_id: int):
    query = """
    SELECT lc.*, cc.category_name 
    FROM legal_case lc
    JOIN case_category cc ON lc.category_id = cc.category_id
    WHERE lc.client_id = %s
    """
    return {"data": db_query(query, (user_id,))}

# 3. 律师进行中的案件
@app.get("/lawyers/{lawyer_id}/active-cases")
async def get_lawyer_active_cases(lawyer_id: int):
    query = """
    SELECT lc.*, cc.category_name, u.name as client_name 
    FROM legal_case lc
    JOIN case_category cc ON lc.category_id = cc.category_id
    JOIN user u ON lc.client_id = u.user_id
    WHERE lc.lawyer_id = %s AND lc.status = 'in_progress'
    """
    return {"data": db_query(query, (lawyer_id,))}

# 4. 律师统计信息
@app.get("/lawyers/{lawyer_id}/stats")
async def get_lawyer_stats(lawyer_id: int):
    query = """
    SELECT 
        u.user_id,
        u.name,
        COUNT(co.order_id) AS total_orders,
        AVG(r.rating) AS avg_rating
    FROM user u
    LEFT JOIN consult_order co ON u.user_id = co.lawyer_id
    LEFT JOIN review r ON co.order_id = r.order_id
    WHERE u.user_id = %s
    GROUP BY u.user_id
    """
    return {"data": db_query(query, (lawyer_id,), fetch_one=True)}


# 创建订单接口
@app.post("/orders")
async def create_order(order: OrderCreate):
    # 验证用户信息
    client = db_query(
        "SELECT * FROM user WHERE user_id = %s",
        (order.client_id,),
        fetch_one=True
    )
    if not client:
        raise HTTPException(status_code=400, detail="委托人不存在")

    lawyer = db_query(
        "SELECT * FROM user WHERE user_id = %s",
        (order.lawyer_id,),
        fetch_one=True
    )
    if not lawyer:
        raise HTTPException(status_code=400, detail="律师不存在")

    # 验证案件信息（如果提供case_id）
    if order.case_id:
        case = db_query(
            "SELECT * FROM legal_case WHERE case_id = %s",
            (order.case_id,),
            fetch_one=True
        )
        if not case:
            raise HTTPException(status_code=400, detail="案件不存在")

    # 生成订单号
    order_number = generate_order_number()

    # 执行数据库插入
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            query = """
            INSERT INTO consult_order 
                (order_number, price, client_id, lawyer_id, case_id, status)
            VALUES (%s, %s, %s, %s, %s, 'pending_payment')
            """
            cursor.execute(query, (
                order_number,
                order.price,
                order.client_id,
                order.lawyer_id,
                order.case_id
            ))
            order_id = cursor.lastrowid
            connection.commit()
            return {
                "order_id": order_id,
                "order_number": order_number,
                "status": "pending_payment"
            }
    except pymysql.err.IntegrityError as e:
        connection.rollback()
        if e.args[0] == 1062:
            raise HTTPException(status_code=400, detail="订单号冲突，请重试")
        else:
            raise HTTPException(status_code=500, detail="数据库错误")
    except Exception as e:
        connection.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        connection.close()

# 5. 订单详情
@app.get("/orders/{order_number}")
async def get_order_details(order_number: str):
    query = """
    SELECT 
        co.*, 
        GROUP_CONCAT(cr.consult_time) as consult_times,
        GROUP_CONCAT(cr.notes) as consult_notes,
        r.rating,
        r.comment
    FROM consult_order co
    LEFT JOIN consult_record cr ON co.order_id = cr.order_id
    LEFT JOIN review r ON co.order_id = r.order_id
    WHERE co.order_number = %s
    GROUP BY co.order_id
    """
    return {"data": db_query(query, (order_number,), fetch_one=True)}





#6. 分类统计
@app.get("/cases/category-stats")
async def get_category_stats():
    query = """
    SELECT 
        cc.category_name,
        COUNT(lc.case_id) AS case_count,
        AVG(lc.price) AS avg_price
    FROM case_category cc
    LEFT JOIN legal_case lc ON cc.category_id = lc.category_id
    GROUP BY cc.category_id
    """
    return {"data": db_query(query)}

#7. 用户评价记录
@app.get("/users/{user_id}/reviews")
async def get_user_reviews(user_id: int):
    query = """
    SELECT 
        r.*,
        reviewee.name AS reviewee_name
    FROM review r
    JOIN user reviewee ON r.reviewee_id = reviewee.user_id
    WHERE r.reviewer_id = %s OR r.reviewee_id = %s
    """
    return {"data": db_query(query, (user_id, user_id))}

#8. 创建法律委托
@app.post("/cases")
async def create_legal_case(case: CaseCreate):
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            query = """
            INSERT INTO legal_case 
                (category_id, title, description, price, client_id, status)
            VALUES (%s, %s, %s, %s, %s, 'pending')
            """
            cursor.execute(query, (
                case.category_id,
                case.title,
                case.description,
                case.price,
                case.client_id
            ))
            case_id = cursor.lastrowid
            connection.commit()
            return {"case_id": case_id}
    except Exception as e:
        connection.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        connection.close()

# 9. 创建评价  待修改 新增接口
@app.post("/reviews")
async def create_review(review: ReviewCreate):
    # 获取订单信息并验证
    order = db_query(
        "SELECT * FROM consult_order WHERE order_id = %s",
        (review.order_id,),
        fetch_one=True
    )
    if not order:
        raise HTTPException(status_code=404, detail="订单不存在")

    # 验证订单状态是否为已完成（不区分大小写）
    if order["status"].lower() != "completed":
        raise HTTPException(status_code=400, detail="订单未完成，不可评价")

    # 检查是否已存在评价
    existing_review = db_query(
        "SELECT * FROM review WHERE order_id = %s",
        (review.order_id,),
        fetch_one=True
    )
    if existing_review:
        raise HTTPException(status_code=400, detail="该订单已存在评价")

    # 验证评价人身份
    client_id = order["client_id"]
    lawyer_id = order["lawyer_id"]
    if review.reviewer_id not in [client_id, lawyer_id]:
        raise HTTPException(status_code=403, detail="无权限评价此订单")

    # 确定被评价人
    reviewee_id = lawyer_id if review.reviewer_id == client_id else client_id

    # 执行数据库插入
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            query = """
             INSERT INTO review 
                 (order_id, rating, comment, reviewer_id, reviewee_id)
             VALUES (%s, %s, %s, %s, %s)
             """
            cursor.execute(query, (
                review.order_id,
                review.rating,
                review.comment,
                review.reviewer_id,
                reviewee_id
            ))
            review_id = cursor.lastrowid
            connection.commit()
            return {"review_id": review_id}
    except pymysql.err.IntegrityError as e:
        connection.rollback()
        if e.args[0] == 1062:  # MySQL重复条目错误码
            raise HTTPException(status_code=400, detail="该订单已存在评价")
        else:
            raise HTTPException(status_code=500, detail="数据库错误")
    except Exception as e:
        connection.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        connection.close()

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)





```

step3:postman

```bash
 1. 按分类获取案件
GET

http://localhost:8000/cases/category/2

{
    "data": [
        {
            "case_id": 4,
            "category_id": 2,
            "title": "故意伤害罪辩护",
            "description": "轻伤二级刑事辩护",
            "price": 8000.0,
            "client_id": 7,
            "lawyer_id": 8,
            "status": "in_progress",
            "created_at": "2025-03-21T06:11:59",
            "updated_at": "2025-03-21T06:11:59",
            "category_name": "刑事辩护",
            "client_name": "黄九"
        }
    ]
}



2. 获取用户发布的委托
GET
http://localhost:8000/users/5/cases

{
    "data": [
        {
            "case_id": 3,
            "category_id": 4,
            "title": "房屋买卖合同违约",
            "description": "开发商延期交房索赔",
            "price": 4500.0,
            "client_id": 5,
            "lawyer_id": 6,
            "status": "pending",
            "created_at": "2025-03-21T06:11:59",
            "updated_at": "2025-03-21T06:11:59",
            "category_name": "合同纠纷"
        },
        {
            "case_id": 8,
            "category_id": 8,
            "title": "房产继承纠纷",
            "description": "多继承人房产分配纠纷",
            "price": 6000.0,
            "client_id": 5,
            "lawyer_id": 6,
            "status": "in_progress",
            "created_at": "2025-03-21T06:11:59",
            "updated_at": "2025-03-21T06:11:59",
            "category_name": "房产纠纷"
        }
    ]
}

3. 律师进行中案件
GET
http://localhost:8000/lawyers/8/active-cases

{
    "data": [
        {
            "case_id": 4,
            "category_id": 2,
            "title": "故意伤害罪辩护",
            "description": "轻伤二级刑事辩护",
            "price": 8000.0,
            "client_id": 7,
            "lawyer_id": 8,
            "status": "in_progress",
            "created_at": "2025-03-21T06:11:59",
            "updated_at": "2025-03-21T06:11:59",
            "category_name": "刑事辩护",
            "client_name": "黄九"
        }
    ]
}



4. 律师统计信息
GET
http://localhost:8000/lawyers/10/stats

{
    "data": {
        "user_id": 10,
        "name": "郑十二律师",
        "total_orders": 2,
        "avg_rating": 4.5
    }
}



5. 订单详情查询
GET
http://localhost:8000/orders/CO2024010007

{
    "data": {
        "order_id": 7,
        "order_number": "CO2024010007",
        "price": 400.0,
        "client_id": 3,
        "lawyer_id": 4,
        "case_id": 7,
        "status": "completed",
        "created_at": "2025-03-21T06:12:02",
        "updated_at": "2025-03-21T06:12:02",
        "consult_times": "2024-01-12 14:30:00",
        "consult_notes": "保险理赔方案确认",
        "rating": 5,
        "comment": "成功帮助获得合理赔偿"
    }
}

 

 

6. 分类统计
GET
http://localhost:8000/cases/category-stats

{
    "data": [
        {
            "category_name": "民事诉讼",
            "case_count": 1,
            "avg_price": 3000.0
        },
        {
            "category_name": "刑事辩护",
            "case_count": 1,
            "avg_price": 8000.0
        },
        {
            "category_name": "婚姻家事",
            "case_count": 1,
            "avg_price": 5000.0
        },
        {
            "category_name": "合同纠纷",
            "case_count": 1,
            "avg_price": 4500.0
        },
        {
            "category_name": "劳动纠纷",
            "case_count": 1,
            "avg_price": 2000.0
        },
        {
            "category_name": "知识产权",
            "case_count": 1,
            "avg_price": 12000.0
        },
        {
            "category_name": "交通事故",
            "case_count": 1,
            "avg_price": 3500.0
        },
        {
            "category_name": "房产纠纷",
            "case_count": 1,
            "avg_price": 6000.0
        },
        {
            "category_name": "公司法律",
            "case_count": 1,
            "avg_price": 9000.0
        },
        {
            "category_name": "涉外案件",
            "case_count": 1,
            "avg_price": 2500.0
        }
    ]
}

7. 用户评价记录
GET
http://localhost:8000/users/9/reviews
{
    "data": [
        {
            "review_id": 2,
            "order_id": 5,
            "rating": 4,
            "comment": "服务态度很好，解答详细",
            "reviewer_id": 9,
            "reviewee_id": 10,
            "created_at": "2025-03-21T06:12:08",
            "reviewee_name": "郑十二律师"
        },
        {
            "review_id": 4,
            "order_id": 10,
            "rating": 5,
            "comment": "对涉外法律非常熟悉",
            "reviewer_id": 9,
            "reviewee_id": 10,
            "created_at": "2025-03-21T06:12:08",
            "reviewee_name": "郑十二律师"
        }
    ]
}


8. 创建法律委托
POST
http://localhost:8000/cases
Headers: Content-Type: application/json
Body:

{
  "category_id": 3,
  "title": "离婚子女抚养权纠纷",
  "description": "争取3岁女儿的抚养权和抚养费",
  "price": 8000.00,
  "client_id": 7
}

{
    "case_id": 11
}

9.添加评论

http://localhost:8000/reviews


{
    "order_id": 1,
    "rating": 5,
    "comment": "服务很，麻辣烫好吃好",
    "reviewer_id": 1
}

10. 创建订单接口的详细示例：

1. 基本配置
请求方法: POST

请求地址: http://localhost:8000/orders

Headers:

复制
Content-Type: application/json


http://localhost:8000/orders
{
    "price": 999.00,
    "client_id": 1,
    "lawyer_id": 2,
    "case_id": 3
}


{
    "order_id": 12,
    "order_number": "CO2025030001",
    "status": "pending_payment"
}


```

end