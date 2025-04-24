说明：
我希望用fastapi+vue实现租房系统
可以新增房源和展示房源
效果图:
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/d49c0c879ac744c4ac96a94495a0f1c0.png#pic_center)

step1:sql

```sql


show databases;

DROP TABLE users;


SHOW CREATE TABLE product_category;

show tables;

use db_school;

SELECT * FROM db_school.jewelry_categories;
SHOW TABLES FROM db_school;

CREATE DATABASE db_school;

-- 使用utf8mb4字符集和InnoDB存储引擎
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- 1. 用户表
-- ----------------------------
CREATE TABLE `users` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(50) NOT NULL COMMENT '用户姓名',
  `email` VARCHAR(100) NOT NULL UNIQUE COMMENT '唯一邮箱',
  `password` VARCHAR(255) NOT NULL COMMENT '加密密码',
  `phone` VARCHAR(20) NOT NULL COMMENT '手机号码',
  `avatar` VARCHAR(255) DEFAULT NULL COMMENT '头像URL',
  `role` ENUM('landlord', 'tenant') NOT NULL COMMENT '用户角色',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  INDEX `idx_email` (`email`),
  INDEX `idx_phone` (`phone`)
) ENGINE=InnoDB COMMENT='用户信息表';

select *from users;

-- ----------------------------
-- 2. 分类表
-- ----------------------------
CREATE TABLE `categories` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` VARCHAR(50) NOT NULL UNIQUE COMMENT '分类名称',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE INDEX `uniq_name` (`name`)
) ENGINE=InnoDB COMMENT='房源分类表';

-- ----------------------------
-- 3. 房源分类关联表
-- ----------------------------
CREATE TABLE `property_categories` (
  `property_id` INT UNSIGNED NOT NULL,
  `category_id` INT UNSIGNED NOT NULL,
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`property_id`, `category_id`),
  CONSTRAINT `fk_pc_property` FOREIGN KEY (`property_id`) REFERENCES `properties` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_pc_category` FOREIGN KEY (`category_id`) REFERENCES `categories` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB COMMENT='房源分类多对多关联表';

-- ----------------------------
-- 4. 房源表
-- ----------------------------
CREATE TABLE `properties` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `landlord_id` INT UNSIGNED NOT NULL COMMENT '房东ID',
  `title` VARCHAR(100) NOT NULL COMMENT '房源标题',
  `description` TEXT NOT NULL COMMENT '详细描述',
  `price` DECIMAL(10,2) UNSIGNED NOT NULL COMMENT '月租金',
  `address` VARCHAR(255) NOT NULL COMMENT '详细地址',
  `area` INT UNSIGNED NOT NULL COMMENT '面积(平方米)',
  `bedrooms` TINYINT UNSIGNED NOT NULL COMMENT '卧室数量',
  `bathrooms` TINYINT UNSIGNED NOT NULL COMMENT '卫生间数量',
  `status` ENUM('available', 'rented', 'maintenance') NOT NULL DEFAULT 'available' COMMENT '房源状态',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  INDEX `idx_status` (`status`),
  INDEX `idx_price` (`price`),
  CONSTRAINT `fk_property_landlord` FOREIGN KEY (`landlord_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB COMMENT='房源信息表';

-- ----------------------------
-- 5. 房源图片表
-- ----------------------------
CREATE TABLE `property_images` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `property_id` INT UNSIGNED NOT NULL,
  `image_url` VARCHAR(255) NOT NULL COMMENT '图片URL',
  `sort_order` TINYINT UNSIGNED NOT NULL DEFAULT 0 COMMENT '排序序号',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  CONSTRAINT `fk_images_property` FOREIGN KEY (`property_id`) REFERENCES `properties` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB COMMENT='房源图片表';

-- ----------------------------
-- 6. 预约表
-- ----------------------------
CREATE TABLE `viewing_appointments` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `property_id` INT UNSIGNED NOT NULL,
  `tenant_id` INT UNSIGNED NOT NULL COMMENT '租客ID',
  `appointment_time` DATETIME NOT NULL COMMENT '预约时间',
  `status` ENUM('pending', 'confirmed', 'canceled') NOT NULL DEFAULT 'pending' COMMENT '预约状态',
  `notes` TEXT COMMENT '附加说明',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  CONSTRAINT `fk_va_property` FOREIGN KEY (`property_id`) REFERENCES `properties` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_va_tenant` FOREIGN KEY (`tenant_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB COMMENT='看房预约表';

-- ----------------------------
-- 7. 租赁合同表
-- ----------------------------
CREATE TABLE `contracts` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `property_id` INT UNSIGNED NOT NULL,
  `landlord_id` INT UNSIGNED NOT NULL,
  `tenant_id` INT UNSIGNED NOT NULL,
  `start_date` DATE NOT NULL COMMENT '租赁开始日期',
  `end_date` DATE NOT NULL COMMENT '租赁结束日期',
  `monthly_rent` DECIMAL(10,2) UNSIGNED NOT NULL COMMENT '月租金',
  `deposit` DECIMAL(10,2) UNSIGNED NOT NULL COMMENT '押金金额',
  `status` ENUM('active', 'expired', 'terminated') NOT NULL DEFAULT 'active' COMMENT '合同状态',
  `contract_file` VARCHAR(255) COMMENT '电子合同存储路径',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  CONSTRAINT `fk_contract_property` FOREIGN KEY (`property_id`) REFERENCES `properties` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_contract_landlord` FOREIGN KEY (`landlord_id`) REFERENCES `users` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_contract_tenant` FOREIGN KEY (`tenant_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB COMMENT='租赁合同表';

-- ----------------------------
-- 8. 支付记录表
-- ----------------------------
CREATE TABLE `payments` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `contract_id` INT UNSIGNED NOT NULL,
  `tenant_id` INT UNSIGNED NOT NULL,
  `amount` DECIMAL(10,2) UNSIGNED NOT NULL COMMENT '支付金额',
  `payment_method` ENUM('alipay', 'wechat', 'bank_transfer') NOT NULL COMMENT '支付方式',
  `payment_date` DATE NOT NULL COMMENT '支付日期',
  `status` ENUM('pending', 'success', 'failed') NOT NULL DEFAULT 'pending' COMMENT '支付状态',
  `transaction_id` VARCHAR(100) COMMENT '第三方支付流水号',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  CONSTRAINT `fk_payment_contract` FOREIGN KEY (`contract_id`) REFERENCES `contracts` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_payment_tenant` FOREIGN KEY (`tenant_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB COMMENT='支付记录表';

-- ----------------------------
-- 9. 评价表
-- ----------------------------
CREATE TABLE `reviews` (
  `id` INT UNSIGNED NOT NULL AUTO_INCREMENT,
  `contract_id` INT UNSIGNED NOT NULL UNIQUE COMMENT '一个合同对应一个评价',
  `tenant_id` INT UNSIGNED NOT NULL,
  `rating` TINYINT UNSIGNED NOT NULL COMMENT '评分(1-5)',
  `comment` TEXT COMMENT '评价内容',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  CONSTRAINT `chk_rating` CHECK (`rating` BETWEEN 1 AND 5),
  CONSTRAINT `fk_review_contract` FOREIGN KEY (`contract_id`) REFERENCES `contracts` (`id`) ON DELETE CASCADE,
  CONSTRAINT `fk_review_tenant` FOREIGN KEY (`tenant_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
) ENGINE=InnoDB COMMENT='用户评价表';

SET FOREIGN_KEY_CHECKS = 1;

-- 1. 用户表数据（房东和租客）
INSERT INTO `users` (`name`, `email`, `password`, `phone`, `avatar`, `role`) VALUES
('张三', 'zhangsan@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', '13800138000', 'avatar1.jpg', 'landlord'),
('李四', 'lisi@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', '13900139000', NULL, 'tenant'),
('王五', 'wangwu@example.com', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', '13600136000', 'landlord_avatar.jpg', 'landlord');

-- 2. 分类表数据
INSERT INTO `categories` (`name`) VALUES
('公寓'),
('别墅'),
('合租房'),
('学区房');

-- 3. 房源表数据（注意landlord_id关联房东用户）
INSERT INTO `properties` (
  `landlord_id`, `title`, `description`,
  `price`, `address`, `area`,
  `bedrooms`, `bathrooms`, `status`
) VALUES
(1, '朝阳区精装公寓', '朝阳公园旁豪华公寓，交通便利', 6800.00, '北京市朝阳区朝阳公园路1号', 85, 2, 1, 'available'),
(3, '海淀区学区房', '中关村二小学区 三居室', 12000.00, '北京市海淀区中关村大街5号', 120, 3, 2, 'available');

-- 4. 房源分类关联表
INSERT INTO `property_categories` (`property_id`, `category_id`) VALUES
(1, 1),
(1, 4),
(2, 1),
(2, 4);

-- 5. 房源图片表数据


INSERT INTO `property_images` (`property_id`, `image_url`, `sort_order`) VALUES
(1, 'https://randomuser.me/api/portraits/men/1.jpg', 1),
(1, 'https://randomuser.me/api/portraits/men/2.jpg', 2),
(2, 'https://randomuser.me/api/portraits/men/3.jpg', 1);

-- 6. 预约表数据（tenant_id需为租客）
INSERT INTO `viewing_appointments` (
  `property_id`, `tenant_id`,
  `appointment_time`, `status`, `notes`
) VALUES
(1, 2, '2023-08-20 14:00:00', 'confirmed', '周末看房'),
(2, 2, '2023-08-21 10:30:00', 'pending', '工作日看房');

-- 7. 租赁合同表数据（需关联三方用户）
INSERT INTO `contracts` (
  `property_id`, `landlord_id`, `tenant_id`,
  `start_date`, `end_date`, `monthly_rent`,
  `deposit`, `status`, `contract_file`
) VALUES
(1, 1, 2, '2023-09-01', '2024-08-31', 6500.00, 13000.00, 'active', 'contracts/1.pdf');

-- 8. 支付记录表数据（关联有效合同）
INSERT INTO `payments` (
  `contract_id`, `tenant_id`, `amount`,
  `payment_method`, `payment_date`, `status`,
  `transaction_id`
) VALUES
(1, 2, 6500.00, 'alipay', '2023-09-01', 'success', '20230901123456'),
(1, 2, 6500.00, 'wechat', '2023-10-01', 'success', '20231001987654');

-- 9. 评价表数据（一个合同一个评价）
INSERT INTO `reviews` (`contract_id`, `tenant_id`, `rating`, `comment`) VALUES
(1, 2, 5, '房屋整洁，房东服务很好！');
```

step2:python test


```python
from typing import List, Dict
import json
import pymysql
from datetime import datetime
from collections import defaultdict

DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


def execute_query(query: str, params=None) -> List[Dict]:
    """执行SQL查询返回字典结果"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            return cursor.fetchall()
    finally:
        connection.close()

def add_property(property_data: Dict, category_ids: List[int] = None, images: List[str] = None) -> int:
    """
    添加新房源并关联分类与图片
    :param property_data: 必须包含字段：landlord_id, title, description, price, address, area, bedrooms, bathrooms
    :param category_ids: 需要关联的分类ID列表
    :param images: 房源图片URL列表
    :return: 新创建房源的ID
    """
    if category_ids is None:
        category_ids = []
    if images is None:
        images = []

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证房东存在且角色正确
            cursor.execute(
                "SELECT id, role FROM users WHERE id = %s",
                (property_data['landlord_id'],)
            )
            landlord = cursor.fetchone()
            if not landlord:
                raise ValueError("用户不存在")
            if landlord['role'] != 'landlord':
                raise ValueError("用户不是房东")

            # 插入房源数据
            sql = """
                INSERT INTO properties (
                    landlord_id, title, description, price,
                    address, area, bedrooms, bathrooms, status
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            params = (
                property_data['landlord_id'],
                property_data['title'],
                property_data.get('description', ''),
                property_data['price'],
                property_data['address'],
                property_data['area'],
                property_data['bedrooms'],
                property_data['bathrooms'],
                property_data.get('status', 'available')
            )
            cursor.execute(sql, params)
            property_id = cursor.lastrowid

            # 验证并关联分类
            for cat_id in category_ids:
                cursor.execute(
                    "SELECT id FROM categories WHERE id = %s",
                    (cat_id,)
                )
                if not cursor.fetchone():
                    raise ValueError(f"分类ID {cat_id} 不存在")
                cursor.execute(
                    "INSERT INTO property_categories (property_id, category_id) VALUES (%s, %s)",
                    (property_id, cat_id)
                )

            # 插入图片数据
            for index, image_url in enumerate(images):
                cursor.execute(
                    "INSERT INTO property_images (property_id, image_url, sort_order) VALUES (%s, %s, %s)",
                    (property_id, image_url, index + 1)
                )

            connection.commit()
            return property_id

    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加房源失败: {str(e)}")
    finally:
        connection.close()

def generate_properties_json() -> List[Dict]:
    """生成分类与房源信息的嵌套JSON结构"""
    # 基础数据查询
    categories = execute_query("SELECT id AS category_id, name FROM categories")
    properties_data = execute_query("""
        SELECT p.id, p.title, p.description, p.price, p.address, p.area,
               p.bedrooms, p.bathrooms, p.status, p.created_at, p.updated_at,
               u.id AS landlord_id, u.name AS landlord_name
        FROM properties p
        JOIN users u ON p.landlord_id = u.id
    """)

    # 关联数据查询
    category_relations = execute_query("SELECT property_id, category_id FROM property_categories")
    property_images = execute_query("SELECT property_id, image_url FROM property_images ORDER BY sort_order")

    # 构建数据映射关系
    image_map = defaultdict(list)
    for img in property_images:
        image_map[img['property_id']].append(img['image_url'])

    category_map = defaultdict(list)
    for rel in category_relations:
        category_map[rel['category_id']].append(rel['property_id'])

    # 状态映射
    status_mapping = {
        'available': '可租',
        'rented': '已出租',
        'maintenance': '维护中'
    }

    # 构建最终数据结构
    result = []
    for category in categories:
        category_id = category['category_id']
        properties = []

        # 获取当前分类下的房源ID
        property_ids = category_map.get(category_id, [])

        # 构建每个房源信息
        for prop in properties_data:
            if prop['id'] not in property_ids:
                continue

            # 处理图片数据
            images = image_map.get(prop['id'], [])

            # 构造房源详情
            property_info = {
                "property_id": prop['id'],
                "title": prop['title'],
                "cover": images[0] if images else None,
                "landlord": {
                    "user_id": prop['landlord_id'],
                    "username": prop['landlord_name']
                },
                "description": prop['description'],
                "status": status_mapping.get(prop['status'], prop['status']),
                "price": float(prop['price']),
                "address": prop['address'],
                "created_at": prop['created_at'].isoformat(),
                "updated_at": prop['updated_at'].isoformat(),
                "details": {
                    "area": prop['area'],
                    "bedrooms": prop['bedrooms'],
                    "bathrooms": prop['bathrooms'],
                    "tags": ["智能门锁", "独立卫浴"]  # 根据实际情况修改
                },
                "images": images
            }
            properties.append(property_info)

        # 添加到分类结构
        result.append({
            "category_id": category_id,
            "name": category['name'],
            "description": f"{category['name']}描述",  # 根据实际字段修改
            "properties": properties
        })

    return result


def custom_serializer(obj):
    """自定义JSON序列化"""
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")


if __name__ == "__main__":
    try:
        # 添加新房源示例
        new_property = {
            "landlord_id": 1,
            "title": "CBD精装别墅",
            "description": "国贸商圈/24小时安保/智能家居",
            "price": 4399.00,
            "address": "华阳地铁松柏花园701号",
            "area": 65,
            "bedrooms": 1,
            "bathrooms": 1
        }
        prop_id = add_property(
            property_data=new_property,
            category_ids=[1, 4],  # 公寓和学区房分类
            images=[
                "https://randomuser.me/api/portraits/men/4.jpg",
                "https://randomuser.me/api/portraits/men/5.jpg"
            ]
        )
        print(f"成功添加房源，ID：{prop_id}")
        data = generate_properties_json()
        print(json.dumps(data, indent=2, ensure_ascii=False, default=custom_serializer))
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


# ======== Pydantic 请求/响应模型 ========
class PropertyCreate(BaseModel):
    landlord_id: int
    title: str
    description: str
    price: float
    address: str
    area: int
    bedrooms: int
    bathrooms: int
    status: Optional[str] = "available"
    category_ids: Optional[List[int]] = None
    images: Optional[List[str]] = None

class LandlordInfo(BaseModel):
    user_id: int
    username: str

class PropertyDetail(BaseModel):
    property_id: int
    title: str
    cover: Optional[str]
    landlord: LandlordInfo
    description: str
    status: str
    price: float
    address: str
    created_at: str
    updated_at: str
    details: dict
    images: List[str]

class CategoryProperties(BaseModel):
    category_id: int
    name: str
    description: str
    properties: List[PropertyDetail]



def execute_query(query: str, params=None) -> List[Dict]:
    """执行SQL查询返回字典结果"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            return cursor.fetchall()
    finally:
        connection.close()

def add_property(property_data: Dict, category_ids: List[int] = None, images: List[str] = None) -> int:
    """
    添加新房源并关联分类与图片
    :param property_data: 必须包含字段：landlord_id, title, description, price, address, area, bedrooms, bathrooms
    :param category_ids: 需要关联的分类ID列表
    :param images: 房源图片URL列表
    :return: 新创建房源的ID
    """
    if category_ids is None:
        category_ids = []
    if images is None:
        images = []

    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证房东存在且角色正确
            cursor.execute(
                "SELECT id, role FROM users WHERE id = %s",
                (property_data['landlord_id'],)
            )
            landlord = cursor.fetchone()
            if not landlord:
                raise ValueError("用户不存在")
            if landlord['role'] != 'landlord':
                raise ValueError("用户不是房东")

            # 插入房源数据
            sql = """
                INSERT INTO properties (
                    landlord_id, title, description, price,
                    address, area, bedrooms, bathrooms, status
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            params = (
                property_data['landlord_id'],
                property_data['title'],
                property_data.get('description', ''),
                property_data['price'],
                property_data['address'],
                property_data['area'],
                property_data['bedrooms'],
                property_data['bathrooms'],
                property_data.get('status', 'available')
            )
            cursor.execute(sql, params)
            property_id = cursor.lastrowid

            # 验证并关联分类
            for cat_id in category_ids:
                cursor.execute(
                    "SELECT id FROM categories WHERE id = %s",
                    (cat_id,)
                )
                if not cursor.fetchone():
                    raise ValueError(f"分类ID {cat_id} 不存在")
                cursor.execute(
                    "INSERT INTO property_categories (property_id, category_id) VALUES (%s, %s)",
                    (property_id, cat_id)
                )

            # 插入图片数据
            for index, image_url in enumerate(images):
                cursor.execute(
                    "INSERT INTO property_images (property_id, image_url, sort_order) VALUES (%s, %s, %s)",
                    (property_id, image_url, index + 1)
                )

            connection.commit()
            return property_id

    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"添加房源失败: {str(e)}")
    finally:
        connection.close()

def generate_properties_json() -> List[Dict]:
    """生成分类与房源信息的嵌套JSON结构"""
    # 基础数据查询
    categories = execute_query("SELECT id AS category_id, name FROM categories")
    properties_data = execute_query("""
        SELECT p.id, p.title, p.description, p.price, p.address, p.area,
               p.bedrooms, p.bathrooms, p.status, p.created_at, p.updated_at,
               u.id AS landlord_id, u.name AS landlord_name
        FROM properties p
        JOIN users u ON p.landlord_id = u.id
    """)

    # 关联数据查询
    category_relations = execute_query("SELECT property_id, category_id FROM property_categories")
    property_images = execute_query("SELECT property_id, image_url FROM property_images ORDER BY sort_order")

    # 构建数据映射关系
    image_map = defaultdict(list)
    for img in property_images:
        image_map[img['property_id']].append(img['image_url'])

    category_map = defaultdict(list)
    for rel in category_relations:
        category_map[rel['category_id']].append(rel['property_id'])

    # 状态映射
    status_mapping = {
        'available': '可租',
        'rented': '已出租',
        'maintenance': '维护中'
    }

    # 构建最终数据结构
    result = []
    for category in categories:
        category_id = category['category_id']
        properties = []

        # 获取当前分类下的房源ID
        property_ids = category_map.get(category_id, [])

        # 构建每个房源信息
        for prop in properties_data:
            if prop['id'] not in property_ids:
                continue

            # 处理图片数据
            images = image_map.get(prop['id'], [])

            # 构造房源详情
            property_info = {
                "property_id": prop['id'],
                "title": prop['title'],
                "cover": images[0] if images else None,
                "landlord": {
                    "user_id": prop['landlord_id'],
                    "username": prop['landlord_name']
                },
                "description": prop['description'],
                "status": status_mapping.get(prop['status'], prop['status']),
                "price": float(prop['price']),
                "address": prop['address'],
                "created_at": prop['created_at'].isoformat(),
                "updated_at": prop['updated_at'].isoformat(),
                "details": {
                    "area": prop['area'],
                    "bedrooms": prop['bedrooms'],
                    "bathrooms": prop['bathrooms'],
                    "tags": ["智能门锁", "独立卫浴"]  # 根据实际情况修改
                },
                "images": images
            }
            properties.append(property_info)

        # 添加到分类结构
        result.append({
            "category_id": category_id,
            "name": category['name'],
            "description": f"{category['name']}描述",  # 根据实际字段修改
            "properties": properties
        })

    return result


def custom_serializer(obj):
    """自定义JSON序列化"""
    if isinstance(obj, datetime):
        return obj.isoformat()
    raise TypeError(f"Type {type(obj)} not serializable")



@app.get("/properties", response_model=List[CategoryProperties])
def get_properties():
    """获取所有分类及房源信息"""
    try:
        return generate_properties_json()
    except Exception as e:
        raise HTTPException(
            status_code=500,
            detail=f"获取数据失败: {str(e)}"
        )


@app.post("/properties", status_code=status.HTTP_201_CREATED)
def create_property(property_data: PropertyCreate):
    """创建新房源"""
    try:
        # 转换数据格式
        data_dict = property_data.dict()
        category_ids = data_dict.pop("category_ids", [])
        images = data_dict.pop("images", [])

        property_id = add_property(
            property_data=data_dict,
            category_ids=category_ids,
            images=images
        )
        return {"property_id": property_id}
    except ValueError as e:
        raise HTTPException(
            status_code=400,
            detail=str(e))
    except RuntimeError as e:
        raise HTTPException(
            status_code=500,
            detail=str(e))

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step4: postman

```bash
 1. 获取所有房源（GET）
请求配置：

URL: GET http://localhost:8000/properties
 

[
    {
        "category_id": 1,
        "name": "公寓",
        "description": "公寓描述",
        "properties": [
            {
                "property_id": 1,
                "title": "朝阳区精装公寓",
                "cover": "https://randomuser.me/api/portraits/men/1.jpg",
                "landlord": {
                    "user_id": 1,
                    "username": "张三"
                },
                "description": "朝阳公园旁豪华公寓，交通便利",
                "status": "可租",
                "price": 6800.0,
                "address": "北京市朝阳区朝阳公园路1号",
                "created_at": "2025-04-03T03:07:37",
                "updated_at": "2025-04-03T03:07:37",
                "details": {
                    "area": 85,
                    "bedrooms": 2,
                    "bathrooms": 1,
                    "tags": [
                        "智能门锁",
                        "独立卫浴"
                    ]
                },
                "images": [
                    "https://randomuser.me/api/portraits/men/1.jpg",
                    "https://randomuser.me/api/portraits/men/2.jpg"
                ]
            },
            {
                "property_id": 2,
                "title": "海淀区学区房",
                "cover": "https://randomuser.me/api/portraits/men/3.jpg",
                "landlord": {
                    "user_id": 3,
                    "username": "王五"
                },
                "description": "中关村二小学区 三居室",
                "status": "可租",
                "price": 12000.0,
                "address": "北京市海淀区中关村大街5号",
                "created_at": "2025-04-03T03:07:37",
                "updated_at": "2025-04-03T03:07:37",
                "details": {
                    "area": 120,
                    "bedrooms": 3,
                    "bathrooms": 2,
                    "tags": [
                        "智能门锁",
                        "独立卫浴"
                    ]
                },
                "images": [
                    "https://randomuser.me/api/portraits/men/3.jpg"
                ]
            },
            {
                "property_id": 3,
                "title": "CBD精装别墅",
                "cover": "https://randomuser.me/api/portraits/men/4.jpg",
                "landlord": {
                    "user_id": 1,
                    "username": "张三"
                },
                "description": "国贸商圈/24小时安保/智能家居",
                "status": "可租",
                "price": 4399.0,
                "address": "华阳地铁松柏花园701号",
                "created_at": "2025-04-03T03:23:39",
                "updated_at": "2025-04-03T03:23:39",
                "details": {
                    "area": 65,
                    "bedrooms": 1,
                    "bathrooms": 1,
                    "tags": [
                        "智能门锁",
                        "独立卫浴"
                    ]
                },
                "images": [
                    "https://randomuser.me/api/portraits/men/4.jpg",
                    "https://randomuser.me/api/portraits/men/5.jpg"
                ]
            }
        ]
    },
    {
        "category_id": 2,
        "name": "别墅",
        "description": "别墅描述",
        "properties": []
    },
    {
        "category_id": 3,
        "name": "合租房",
        "description": "合租房描述",
        "properties": []
    },
    {
        "category_id": 4,
        "name": "学区房",
        "description": "学区房描述",
        "properties": [
            {
                "property_id": 1,
                "title": "朝阳区精装公寓",
                "cover": "https://randomuser.me/api/portraits/men/1.jpg",
                "landlord": {
                    "user_id": 1,
                    "username": "张三"
                },
                "description": "朝阳公园旁豪华公寓，交通便利",
                "status": "可租",
                "price": 6800.0,
                "address": "北京市朝阳区朝阳公园路1号",
                "created_at": "2025-04-03T03:07:37",
                "updated_at": "2025-04-03T03:07:37",
                "details": {
                    "area": 85,
                    "bedrooms": 2,
                    "bathrooms": 1,
                    "tags": [
                        "智能门锁",
                        "独立卫浴"
                    ]
                },
                "images": [
                    "https://randomuser.me/api/portraits/men/1.jpg",
                    "https://randomuser.me/api/portraits/men/2.jpg"
                ]
            },
            {
                "property_id": 2,
                "title": "海淀区学区房",
                "cover": "https://randomuser.me/api/portraits/men/3.jpg",
                "landlord": {
                    "user_id": 3,
                    "username": "王五"
                },
                "description": "中关村二小学区 三居室",
                "status": "可租",
                "price": 12000.0,
                "address": "北京市海淀区中关村大街5号",
                "created_at": "2025-04-03T03:07:37",
                "updated_at": "2025-04-03T03:07:37",
                "details": {
                    "area": 120,
                    "bedrooms": 3,
                    "bathrooms": 2,
                    "tags": [
                        "智能门锁",
                        "独立卫浴"
                    ]
                },
                "images": [
                    "https://randomuser.me/api/portraits/men/3.jpg"
                ]
            },
            {
                "property_id": 3,
                "title": "CBD精装别墅",
                "cover": "https://randomuser.me/api/portraits/men/4.jpg",
                "landlord": {
                    "user_id": 1,
                    "username": "张三"
                },
                "description": "国贸商圈/24小时安保/智能家居",
                "status": "可租",
                "price": 4399.0,
                "address": "华阳地铁松柏花园701号",
                "created_at": "2025-04-03T03:23:39",
                "updated_at": "2025-04-03T03:23:39",
                "details": {
                    "area": 65,
                    "bedrooms": 1,
                    "bathrooms": 1,
                    "tags": [
                        "智能门锁",
                        "独立卫浴"
                    ]
                },
                "images": [
                    "https://randomuser.me/api/portraits/men/4.jpg",
                    "https://randomuser.me/api/portraits/men/5.jpg"
                ]
            }
        ]
    }
]




2. 创建新房源（POST）
请求配置：

URL: POST http://localhost:8000/properties

 

Body (raw/JSON):

{
  "landlord_id": 3,
  "title": "世纪城合租房",
  "description": "世纪城商圈/12小时安保/智能家居",
  "price": 360.00,
  "address": "武侯区建国路88号",
  "area": 65,
  "bedrooms": 1,
  "bathrooms": 1,
  "category_ids": [2, 3],
  "images": [
    "https://randomuser.me/api/portraits/men/6.jpg",
    "https://randomuser.me/api/portraits/men/5.jpg"
  ]
}

返回结果

{
    "property_id": 4
}
```

step5:房源列表C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Properties.vue

```typescript
<template>
  <div class="container">
    <div v-if="loading" class="loading">加载中...</div>
    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="!loading && !error" class="main-layout">
      <!-- 左侧分类列表 -->
      <div class="category-list">
        <div class="button-container">
          <button @click="goToAddProperties" class="add-button">+ 发布新房源</button>
        </div>
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

      <!-- 右侧房源列表 -->
      <div class="property-list">
        <div
          v-for="property in currentProperties"
          :key="property.property_id"
          class="property-card"
        >
          <img :src="property.cover" alt="房源封面" class="cover">
          <div class="content">
            <h2>{{ property.title }}</h2>
            <div class="meta">
              <span class="price">¥{{ property.price }} / 月</span>
              <span class="status">{{ property.status }}</span>
              <span class="area">{{ property.details.area }}㎡</span>
            </div>
            <p class="address">{{ property.address }}</p>
            <div class="details">
              <span>{{ property.details.bedrooms }}室</span>
              <span>{{ property.details.bathrooms }}卫</span>
              <span v-for="(tag, index) in property.details.tags"
                     :key="index"
                     class="tag">
                {{ tag }}
              </span>
            </div>
            <div class="landlord">
              <img :src="property.landlord.avatar"
                   alt="房东头像"
                   class="avatar">
              <span>{{ property.landlord.username }}</span>
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
    currentProperties() {
      const category = this.categories.find(c => c.category_id === this.currentCategoryId);
      return category ? category.properties : [];
    }
  },
  async created() {
    try {
      const response = await axios.get('http://localhost:8000/properties');
      this.categories = response.data;
      if (this.categories.length > 0) {
        this.currentCategoryId = this.categories[0].category_id;
      }
    } catch (error) {
      this.error = '数据加载失败，请稍后重试';
      console.error('API请求失败:', error);
    } finally {
      this.loading = false;
    }
  },
  methods: {
    selectCategory(categoryId) {
      this.currentCategoryId = categoryId;
    },
    goToAddProperties() {
      this.$router.push('/add-properties');
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

.category-item.active {
  background: #007bff;
  color: white;
}

.property-list {
  flex: 1;
}

.property-card {
  display: flex;
  gap: 20px;
  padding: 20px;
  margin-bottom: 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.cover {
  width: 240px;
  height: 180px;
  object-fit: cover;
  border-radius: 4px;
}

.meta {
  display: flex;
  gap: 15px;
  margin: 10px 0;
  color: #666;
}

.price {
  color: #ff4d4f;
  font-weight: bold;
}

.details {
  margin: 10px 0;
  display: flex;
  gap: 10px;
}

.tag {
  padding: 4px 8px;
  background: #f0f2f5;
  border-radius: 4px;
}

.landlord {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 10px;
}

.avatar {
  width: 24px;
  height: 24px;
  border-radius: 50%;
}
</style>
```

step6:新增房源 C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Add-Properties.vue

```typescript
<template>
  <div class="form-container">
    <form @submit.prevent="createProperty">
      <!-- 标题 -->
      <div class="form-item">
        <label>房源标题*</label>
        <input
          v-model="formData.title"
          type="text"
          placeholder="请输入房源标题"
          required
          maxlength="50"
        >
        <div class="tip">当前已输入：{{ formData.title.length }}字</div>
      </div>

      <!-- 价格 -->
      <div class="form-item">
        <label>价格（元/月）*</label>
        <input
          v-model.number="formData.price"
          type="number"
          step="0.01"
          placeholder="请输入月租金"
          required
        >
      </div>

      <!-- 地址 -->
      <div class="form-item">
        <label>详细地址*</label>
        <input
          v-model="formData.address"
          type="text"
          placeholder="请输入完整地址"
          required
        >
      </div>

      <!-- 房型信息 -->
      <div class="form-group">
        <div class="form-item">
          <label>面积（㎡）</label>
          <input
            v-model.number="formData.area"
            type="number"
            placeholder="请输入房屋面积"
          >
        </div>
        <div class="form-item">
          <label>卧室数量</label>
          <input
            v-model.number="formData.bedrooms"
            type="number"
            min="0"
            placeholder="请输入卧室数量"
          >
        </div>
        <div class="form-item">
          <label>卫生间数量</label>
          <input
            v-model.number="formData.bathrooms"
            type="number"
            min="0"
            placeholder="请输入卫生间数量"
          >
        </div>
      </div>

      <!-- 房源描述 -->
      <div class="form-item">
        <label>房源描述</label>
        <textarea
          v-model="formData.description"
          placeholder="请输入房源详细描述"
          maxlength="500"
        ></textarea>
        <div class="tip">已输入 {{ formData.description.length }}/500 字</div>
      </div>

      <!-- 房源分类 -->
      <div class="form-item">
        <label>房源分类</label>
        <div class="category-group">
          <label v-for="cat in categories" :key="cat.id">
            <input
              type="checkbox"
              v-model="formData.category_ids"
              :value="cat.id"
            >
            {{ cat.name }}
          </label>
        </div>
      </div>

      <!-- 图片URL -->
      <div class="form-item">
        <label>图片URL</label>
        <div v-for="(img, index) in formData.images" :key="index" class="image-input">
          <input
            v-model="formData.images[index]"
            type="url"
            placeholder="请输入图片URL"
          >
          <button type="button" @click="removeImage(index)">删除</button>
        </div>
        <button type="button" @click="addImage" class="add-btn">+ 添加图片</button>
      </div>

      <button type="submit" class="submit-btn">提交房源</button>
    </form>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 表单初始数据
const formData = ref({
  title: '',
  description: '',
  price: null,
  address: '',
  area: null,
  bedrooms: null,
  bathrooms: null,
  category_ids: [],
  images: []
})

// 房源分类选项
const categories = ref([
  { id: 1, name: '公寓' },
  { id: 2, name: '别墅' },
  { id: 3, name: '合租房' },
  { id: 4, name: '学区房' }
])

// 添加图片URL输入框
const addImage = () => {
  formData.value.images.push('')
}

// 删除图片URL
const removeImage = (index) => {
  formData.value.images.splice(index, 1)
}

// 提交表单
const createProperty = async () => {
  try {
    // 构造请求数据
    const payload = {
      ...formData.value,
      landlord_id: 3, // 根据实际情况设置
      images: formData.value.images.filter(url => url.trim() !== '')
    }

    // 发送请求
    const response = await fetch('http://localhost:8000/properties', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(payload)
    })

    if (!response.ok) {
      const error = await response.json()
      throw new Error(error.message || '提交失败')
    }

    const result = await response.json()
    alert(`创建成功! 房源ID: ${result.property_id}`)
    // 重置表单
    formData.value = {
      title: '',
      description: '',
      price: null,
      address: '',
      area: null,
      bedrooms: null,
      bathrooms: null,
      category_ids: [],
      images: []
    }
  } catch (error) {
    console.error('提交失败:', error)
    alert(`提交失败: ${error.message}`)
  }
}
</script>

<style scoped>
.form-container {
  max-width: 800px;
  margin: 20px auto;
  padding: 30px;
  background: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.form-group {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  margin-bottom: 20px;
}

.form-item {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2c3e50;
}

input, select, textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

input[type="number"] {
  appearance: textfield;
}

input[type="checkbox"] {
  width: auto;
  margin-right: 8px;
}

.category-group {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
}

.image-input {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.image-input input {
  flex: 1;
}

.add-btn {
  background: none;
  color: #007bff;
  border: 1px dashed #007bff;
  padding: 8px 15px;
  margin-top: 10px;
}

.add-btn:hover {
  background: rgba(0,123,255,0.05);
}

.submit-btn {
  width: 100%;
  padding: 12px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.3s;
}

.submit-btn:hover {
  background: #0056b3;
}

.tip {
  color: #6c757d;
  font-size: 12px;
  margin-top: 4px;
}
</style>
```

step7:路由C:\Users\wangrusheng\PycharmProjects\untitled3\src\router\index.js

```typescript
// src/router/index.js
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import User from '../views/User.vue'
import Game from '../views/Game.vue'
import Chapters from "../views/Chapters.vue";
import Links from "../views/Links.vue";
import Properties from "../views/Properties.vue";
import AddProperties from "../views/Add-Properties.vue";



const routes = [
  { path: '/', redirect: '/home' },
  { path: '/home', name: 'Home', component: Home },
  { path: '/user', name: 'User', component: User },
  { path: '/game', name: 'game', component: Game },
  { path: '/links', name: 'links', component: Links },
  { path: '/properties', name: 'properties', component: Properties },
  { path: '/add-properties', name: 'add-properties', component: AddProperties },
  { path: '/chapters', name: 'chapters', component: Chapters }

]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```

end