说明：
fastapi+angular商品分类和展示
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f09d0f3a531748d8af91870e95eea8a8.png#pic_center)

step1:sql

```sql

-- 分类表（支持多级分类）
CREATE TABLE category (
  category_id INT AUTO_INCREMENT PRIMARY KEY,
  category_name VARCHAR(255) NOT NULL,
  parent_id INT DEFAULT NULL,
  thumbnail VARCHAR(255),
  description TEXT,
  FOREIGN KEY (parent_id) REFERENCES category(category_id)
);

-- 商品表
CREATE TABLE product (
  product_id INT AUTO_INCREMENT PRIMARY KEY,
  product_name VARCHAR(255) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  stock INT NOT NULL,
  thumbnail VARCHAR(255),
  description TEXT,
  tags JSON NOT NULL,
  listed_time DATETIME NOT NULL,
  sales_volume INT DEFAULT 0,
  rating JSON NOT NULL,
  is_hot BOOLEAN DEFAULT FALSE,
  is_discount BOOLEAN DEFAULT FALSE,
  is_new BOOLEAN DEFAULT FALSE
);

-- 商品分类关联表（多对多关系）
CREATE TABLE _category (
  product_id INT NOT NULL,
  category_id INT NOT NULL,
  PRIMARY KEY (product_id, category_id),
  FOREIGN KEY (product_id) REFERENCES product(product_id),
  FOREIGN KEY (category_id) REFERENCES category(category_id)
);
-- 插入分类数据
INSERT INTO category (category_id, category_name, parent_id, thumbnail, description)
VALUES
  (1, '手机', NULL, 'https://randomuser.me/api/portraits/men/1.jpg', '智能设备与配件'),
  (2, '笔记本电脑', NULL, 'https://randomuser.me/api/portraits/men/2.jpg', '高性能PC终端'),
  (3, '男装', NULL, 'https://randomuser.me/api/portraits/men/3.jpg', '男士服饰与配件'),
  (4, '女装', NULL, 'https://randomuser.me/api/portraits/men/4.jpg', '女士时尚服饰');

-- 插入商品数据
INSERT INTO product (
  product_id, product_name, price, stock, thumbnail,
  description, tags, listed_time, sales_volume, rating,
  is_hot, is_discount, is_new
) VALUES
  (1, 'iPhone 15 Pro', 8999.0, 150, 'https://randomuser.me/api/portraits/men/5.jpg',
   'A17芯片 钛金属边框', '["5G", "旗舰机型"]', '2024-09-20 10:00:00', 9800,
   '[4.9, 4.9, 4.8]', TRUE, FALSE, FALSE),

  (2, '三星Galaxy S24 Ultra', 9999.0, 90, 'https://randomuser.me/api/portraits/men/6.jpg',
   '2亿像素 钛灰商务版', '["S Pen", "8K摄像"]', '2024-02-01 09:30:00', 4500,
   '[4.8, 4.7, 4.9]', FALSE, FALSE, FALSE),

  (3, '惠普暗影精灵4', 6499.0, 300, 'https://randomuser.me/api/portraits/men/1.jpg',
   '性能笔记本，游戏无忧', '["旗舰产品", "i9处理器"]', '2024-03-15 14:20:00', 12000,
   '[4.9, 5.0, 4.9]', FALSE, TRUE, FALSE),

  (4, 'macbook pro', 8999.0, 50, 'https://randomuser.me/api/portraits/men/2.jpg',
   '最新m2芯片，液晶大屏幕', '["mac OS", "防水"]', '2023-12-25 08:00:00', 28000,
   '[5.0, 5.0, 4.9]', TRUE, FALSE, FALSE),

  (5, '纯棉商务衬衫', 399.0, 200, 'https://randomuser.me/api/portraits/men/3.jpg',
   '抗皱免烫 修身剪裁', '["正装", "免熨烫"]', '2024-03-01 08:00:00', 4500,
   '[4.7, 4.8, 4.6]', FALSE, FALSE, TRUE),

  (6, '潮流印花T恤', 159.0, 500, 'https://randomuser.me/api/portraits/men/2.jpg',
   '街头风oversize设计', '["潮牌", "情侣款"]', '2024-05-20 10:30:00', 12800,
   '[4.9, 4.8, 4.9]', TRUE, FALSE, FALSE),

  (7, '法式碎花连衣裙', 699.0, 150, 'https://randomuser.me/api/portraits/men/4.jpg',
   '雪纺面料 A字裙摆', '["度假风", "收腰设计"]', '2024-04-10 09:15:00', 6800,
   '[4.8, 4.9, 4.7]', FALSE, TRUE, FALSE);

-- 插入商品分类关联数据
INSERT INTO _category (product_id, category_id)
VALUES
  (1, 1), (2, 1),
  (3, 2), (4, 2),
  (5, 3), (6, 3),
  (7, 4);
```

step2:python 测试类

```python
from typing import List, Dict
import json
from collections import defaultdict
import pymysql.cursors
from datetime import datetime
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


def generate_category_product_json() -> List[Dict]:
    # 获取所有分类
    categories = execute_query("SELECT * FROM category ORDER BY category_id")

    # 获取商品及其分类关系
    products = execute_query("""
        SELECT p.*, c.category_id 
        FROM product p
        JOIN _category c ON p.product_id = c.product_id
    """)

    # 按分类分组商品
    category_map = defaultdict(list)
    for product in products:
        # 处理JSON字段
        product['tags'] = json.loads(product['tags'])
        product['rating'] = json.loads(product['rating'])

        # 处理时间格式
        listed_time: datetime = product['listed_time']
        product['listed_time'] = listed_time.isoformat() + 'Z'

        # 构建商品结构（仅包含True的布尔字段）
        product_entry = {
            k: v for k, v in product.items()
            if k in {
                'product_id', 'product_name', 'price', 'stock', 'thumbnail',
                'description', 'tags', 'listed_time', 'sales_volume', 'rating'
            }
        }

        # 添加布尔标记
        for flag in ['is_hot', 'is_discount', 'is_new']:
            if product.get(flag):
                product_entry[flag] = True

        category_map[product['category_id']].append(product_entry)

    # 构建最终结构
    return [{
        "category_id": cat['category_id'],
        "category_name": cat['category_name'],
        "parent_id": cat['parent_id'] if cat['parent_id'] else None,
        "thumbnail": cat['thumbnail'],
        "description": cat['description'],
        "products": category_map.get(cat['category_id'], [])
    } for cat in categories]

def add_women_clothing_product(product_data: dict) -> int:
    """
    新增女装商品并关联到女装分类
    :param product_data: 商品数据字典，需要包含以下键：
        - product_name: 商品名称（必须）
        - price: 价格（必须）
        - stock: 库存（必须）
        - thumbnail: 缩略图URL（可选，默认为空字符串）
        - description: 商品描述（可选，默认为空字符串）
        - tags: 标签列表（可选，默认为空列表）
        - listed_time: 上架时间（可选，默认当前时间）
        - sales_volume: 销量（可选，默认为0）
        - rating: 评分列表（可选，默认为空列表）
        - is_hot: 是否热卖（可选，默认False）
        - is_discount: 是否折扣（可选，默认False）
        - is_new: 是否新品（可选，默认False）
    :return: 新创建的商品ID
    """
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 1. 获取女装分类ID
            cursor.execute("SELECT category_id FROM category WHERE category_name = '女装'")
            category = cursor.fetchone()
            if not category:
                raise ValueError("女装分类不存在，请先创建分类")

            category_id = category['category_id']

            # 2. 处理商品数据默认值
            product_data.setdefault('thumbnail', '')
            product_data.setdefault('description', '')
            product_data.setdefault('tags', [])
            product_data.setdefault('listed_time', datetime.now())
            product_data.setdefault('sales_volume', 0)
            product_data.setdefault('rating', [])
            product_data.setdefault('is_hot', False)
            product_data.setdefault('is_discount', False)
            product_data.setdefault('is_new', False)

            # 3. 插入商品数据
            product_sql = """
                INSERT INTO product (
                    product_name, price, stock, thumbnail, description, 
                    tags, listed_time, sales_volume, rating,
                    is_hot, is_discount, is_new
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            product_params = (
                product_data['product_name'],
                product_data['price'],
                product_data['stock'],
                product_data['thumbnail'],
                product_data['description'],
                json.dumps(product_data['tags']),
                product_data['listed_time'],
                product_data['sales_volume'],
                json.dumps(product_data['rating']),
                product_data['is_hot'],
                product_data['is_discount'],
                product_data['is_new']
            )
            cursor.execute(product_sql, product_params)
            product_id = cursor.lastrowid

            # 4. 插入分类关联
            category_sql = "INSERT INTO _category (product_id, category_id) VALUES (%s, %s)"
            cursor.execute(category_sql, (product_id, category_id))

            # 提交事务
            connection.commit()
            return product_id

    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"创建商品失败: {str(e)}")
    finally:
        connection.close()

def custom_serializer(obj):
    """自定义JSON序列化处理函数"""
    if isinstance(obj, Decimal):
        return float(obj)  # 将Decimal转换为float
    raise TypeError(f"Type {type(obj)} not serializable")


if __name__ == "__main__":
    try:
        # result = generate_category_product_json()
        # 使用自定义序列化函数处理Decimal类型
        # print(json.dumps(result, indent=2, ensure_ascii=False, default=custom_serializer))
        # 示例调用
        new_product = {
            'product_name': '雪纺碎花连衣裙',
            'price': 599.0,
            'stock': 100,
            'thumbnail': 'https://randomuser.me/api/portraits/men/4.jpg',
            'description': '夏季新款雪纺面料连衣裙',
            'tags': ['夏季', '碎花', '新款'],
            'listed_time': datetime(2024, 5, 25, 10, 0),  # 可选，不传则使用当前时间
            'rating': [4.8, 4.9, 5.0],
            'is_new': True
        }

        try:
            new_id = add_women_clothing_product(new_product)
            print(f"成功创建商品，ID：{new_id}")
        except Exception as e:
            print(f"错误：{str(e)}")
    except Exception as e:
        print(f"生成JSON失败: {str(e)}")
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
from datetime import datetime
from typing import List, Optional
import uvicorn
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
# 数据库配置（根据实际情况修改）
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


class ProductCreate(BaseModel):
    product_name: str
    price: float
    stock: int
    category_ids: List[int]  # 新增分类ID列表字段
    thumbnail: Optional[str] = ""
    description: Optional[str] = ""
    tags: Optional[List[str]] = []
    listed_time: Optional[datetime] = None
    sales_volume: Optional[int] = 0
    rating: Optional[List[float]] = []
    is_hot: Optional[bool] = False
    is_discount: Optional[bool] = False
    is_new: Optional[bool] = False

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

def generate_category_product_json() -> List[Dict]:
    # 获取所有分类
    categories = execute_query("SELECT * FROM category ORDER BY category_id")

    # 获取商品及其分类关系
    products = execute_query("""
        SELECT p.*, c.category_id 
        FROM product p
        JOIN _category c ON p.product_id = c.product_id
    """)

    category_map = defaultdict(list)
    for product in products:
        # 处理JSON字段
        product['tags'] = json.loads(product['tags'])
        product['rating'] = json.loads(product['rating'])

        # 处理时间格式
        product['listed_time'] = product['listed_time'].isoformat() + 'Z'

        # 构建商品结构
        product_entry = {
            k: v for k, v in product.items()
            if k in {
                'product_id', 'product_name', 'price', 'stock', 'thumbnail',
                'description', 'tags', 'listed_time', 'sales_volume', 'rating'
            }
        }

        # 添加布尔标记
        for flag in ['is_hot', 'is_discount', 'is_new']:
            if product.get(flag):
                product_entry[flag] = True

        category_map[product['category_id']].append(product_entry)

    # 构建最终结构
    return [{
        "category_id": cat['category_id'],
        "category_name": cat['category_name'],
        "parent_id": cat['parent_id'] or None,
        "thumbnail": cat['thumbnail'],
        "description": cat['description'],
        "products": category_map.get(cat['category_id'], [])
    } for cat in categories]

def custom_serializer(obj):
    """处理Decimal类型的序列化"""
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError(f"Type {type(obj)} not serializable")


@app.post("/products")  # 修改路由路径
async def create_product(product: ProductCreate):  # 修改方法名
    try:
        # 提取分类ID并移除请求体中的该字段
        product_data = product.dict()
        category_ids = product_data.pop('category_ids')

        # 处理时间字段
        if not product_data['listed_time']:
            product_data['listed_time'] = datetime.now()

        # 调用商品创建方法并传递分类ID
        product_id = add_product(product_data, category_ids)
        return {"product_id": product_id}

    except ValueError as e:
        raise HTTPException(status_code=404, detail=str(e))
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


def add_product(product_data: dict, category_ids: List[int]) -> int:
    """通用商品创建方法"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            # 验证分类ID有效性
            if not category_ids:
                raise ValueError("至少需要指定一个分类ID")

            # 查询存在的分类ID
            placeholders = ','.join(['%s'] * len(category_ids))
            cursor.execute(
                f"SELECT category_id FROM category WHERE category_id IN ({placeholders})",
                tuple(category_ids)
            )
            valid_ids = {row['category_id'] for row in cursor.fetchall()}
            invalid_ids = set(category_ids) - valid_ids

            if invalid_ids:
                raise ValueError(f"无效的分类ID: {invalid_ids}")

            # 插入商品数据
            product_sql = """
                INSERT INTO product (
                    product_name, price, stock, thumbnail, description,
                    tags, listed_time, sales_volume, rating,
                    is_hot, is_discount, is_new
                ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
            """
            product_params = (
                product_data['product_name'],
                product_data['price'],
                product_data['stock'],
                product_data['thumbnail'],
                product_data['description'],
                json.dumps(product_data.get('tags', [])),
                product_data['listed_time'],
                product_data.get('sales_volume', 0),
                json.dumps(product_data.get('rating', [])),
                product_data.get('is_hot', False),
                product_data.get('is_discount', False),
                product_data.get('is_new', False)
            )
            cursor.execute(product_sql, product_params)
            product_id = cursor.lastrowid

            # 批量插入分类关系
            if valid_ids:
                category_sql = "INSERT INTO _category (product_id, category_id) VALUES (%s, %s)"
                category_params = [(product_id, cid) for cid in valid_ids]
                cursor.executemany(category_sql, category_params)

            connection.commit()
            return product_id

    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"商品创建失败: {str(e)}")
    finally:
        connection.close()

@app.get("/category-products")
async def get_category_products():
    try:
        result = generate_category_product_json()
        json_str = json.dumps(
            result,
            ensure_ascii=False,
            indent=2,
            default=custom_serializer
        )
        return Response(content=json_str, media_type="application/json; charset=utf-8")
    except Exception as e:
        return {"error": str(e)}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step4:postman

```bash


请求方法： get

http://localhost:8000/category-products
[
    {
        "category_id": 1,
        "category_name": "手机",
        "parent_id": null,
        "thumbnail": "https://randomuser.me/api/portraits/men/1.jpg",
        "description": "智能设备与配件",
        "products": [
            {
                "product_id": 1,
                "product_name": "iPhone 15 Pro",
                "price": 8999.0,
                "stock": 150,
                "thumbnail": "https://randomuser.me/api/portraits/men/5.jpg",
                "description": "A17芯片 钛金属边框",
                "tags": [
                    "5G",
                    "旗舰机型"
                ],
                "listed_time": "2024-09-20T10:00:00Z",
                "sales_volume": 9800,
                "rating": [
                    4.9,
                    4.9,
                    4.8
                ],
                "is_hot": true
            },
            {
                "product_id": 2,
                "product_name": "三星Galaxy S24 Ultra",
                "price": 9999.0,
                "stock": 90,
                "thumbnail": "https://randomuser.me/api/portraits/men/6.jpg",
                "description": "2亿像素 钛灰商务版",
                "tags": [
                    "S Pen",
                    "8K摄像"
                ],
                "listed_time": "2024-02-01T09:30:00Z",
                "sales_volume": 4500,
                "rating": [
                    4.8,
                    4.7,
                    4.9
                ]
            },
            {
                "product_id": 8,
                "product_name": "oppo A5",
                "price": 899.9,
                "stock": 123,
                "thumbnail": "https://randomuser.me/api/portraits/men/2.jpg",
                "description": "",
                "tags": [
                    "科技",
                    "数码",
                    "爆款"
                ],
                "listed_time": "2025-03-25T01:21:33Z",
                "sales_volume": 0,
                "rating": [],
                "is_hot": true
            },
            {
                "product_id": 9,
                "product_name": "iphone 15 ",
                "price": 6799.9,
                "stock": 100,
                "thumbnail": "https://randomuser.me/api/portraits/men/6.jpg",
                "description": "",
                "tags": [
                    "最新",
                    "最热",
                    "促销"
                ],
                "listed_time": "2025-03-25T01:22:38Z",
                "sales_volume": 0,
                "rating": [],
                "is_hot": true
            }
        ]
    },
    {
        "category_id": 2,
        "category_name": "笔记本电脑",
        "parent_id": null,
        "thumbnail": "https://randomuser.me/api/portraits/men/2.jpg",
        "description": "高性能PC终端",
        "products": [
            {
                "product_id": 3,
                "product_name": "惠普暗影精灵4",
                "price": 6499.0,
                "stock": 300,
                "thumbnail": "https://randomuser.me/api/portraits/men/1.jpg",
                "description": "性能笔记本，游戏无忧",
                "tags": [
                    "旗舰产品",
                    "i9处理器"
                ],
                "listed_time": "2024-03-15T14:20:00Z",
                "sales_volume": 12000,
                "rating": [
                    4.9,
                    5.0,
                    4.9
                ],
                "is_discount": true
            },
            {
                "product_id": 4,
                "product_name": "macbook pro",
                "price": 8999.0,
                "stock": 50,
                "thumbnail": "https://randomuser.me/api/portraits/men/2.jpg",
                "description": "最新m2芯片，液晶大屏幕",
                "tags": [
                    "mac OS",
                    "防水"
                ],
                "listed_time": "2023-12-25T08:00:00Z",
                "sales_volume": 28000,
                "rating": [
                    5.0,
                    5.0,
                    4.9
                ],
                "is_hot": true
            }
        ]
    },
    {
        "category_id": 3,
        "category_name": "男装",
        "parent_id": null,
        "thumbnail": "https://randomuser.me/api/portraits/men/3.jpg",
        "description": "男士服饰与配件",
        "products": [
            {
                "product_id": 5,
                "product_name": "纯棉商务衬衫",
                "price": 399.0,
                "stock": 200,
                "thumbnail": "https://randomuser.me/api/portraits/men/3.jpg",
                "description": "抗皱免烫 修身剪裁",
                "tags": [
                    "正装",
                    "免熨烫"
                ],
                "listed_time": "2024-03-01T08:00:00Z",
                "sales_volume": 4500,
                "rating": [
                    4.7,
                    4.8,
                    4.6
                ],
                "is_new": true
            },
            {
                "product_id": 6,
                "product_name": "潮流印花T恤",
                "price": 159.0,
                "stock": 500,
                "thumbnail": "https://randomuser.me/api/portraits/men/2.jpg",
                "description": "街头风oversize设计",
                "tags": [
                    "潮牌",
                    "情侣款"
                ],
                "listed_time": "2024-05-20T10:30:00Z",
                "sales_volume": 12800,
                "rating": [
                    4.9,
                    4.8,
                    4.9
                ],
                "is_hot": true
            }
        ]
    },
    {
        "category_id": 4,
        "category_name": "女装",
        "parent_id": null,
        "thumbnail": "https://randomuser.me/api/portraits/men/4.jpg",
        "description": "女士时尚服饰",
        "products": [
            {
                "product_id": 7,
                "product_name": "法式碎花连衣裙",
                "price": 699.0,
                "stock": 150,
                "thumbnail": "https://randomuser.me/api/portraits/men/4.jpg",
                "description": "雪纺面料 A字裙摆",
                "tags": [
                    "度假风",
                    "收腰设计"
                ],
                "listed_time": "2024-04-10T09:15:00Z",
                "sales_volume": 6800,
                "rating": [
                    4.8,
                    4.9,
                    4.7
                ],
                "is_discount": true
            }
        ]
    }
]


请求方法： POST

请求URL： http://localhost:8000/women-clothing-products

body:

{
    "product_name": "火箭少女101裙子",
    "price": 899.0,
    "stock": 100,
    "thumbnail": "https://randomuser.me/api/portraits/men/3.jpg",
    "description": "和平精英限定款",
    "tags": ["夏季", "碎花", "新款"],
    "listed_time": "2025-03-25T10:00:00",
    "rating": [4.8, 4.9, 5.0],
    "is_new": true
}


{
    "product_id": 10
}


```

step5:angular.json

```bash
     "styles": [
              "node_modules/bootstrap/dist/css/bootstrap.min.css",
              "@angular/material/prebuilt-themes/azure-blue.css",
              "src/styles.css"
            ],
            "scripts": [
              "node_modules/@popperjs/core/dist/umd/popper.min.js",
              "node_modules/bootstrap/dist/js/bootstrap.min.js"
            ]
```

step6:package.json

```bash
  "dependencies": {
    "@angular/cdk": "^19.2.6",
    "@angular/common": "^19.2.0",
    "@angular/compiler": "^19.2.0",
    "@angular/core": "^19.2.0",
    "@angular/forms": "^19.2.0",
    "@angular/material": "^19.2.6",
    "@angular/platform-browser": "^19.2.0",
    "@angular/platform-browser-dynamic": "^19.2.0",
    "@angular/router": "^19.2.0",
    "@ng-bootstrap/ng-bootstrap": "^18.0.0",
    "@popperjs/core": "^2.11.8",
    "bootstrap": "^5.3.3",
    "rxjs": "~7.8.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.15.0"
  },
```

step7:ts

```typescript
// combined-product.component.ts
import {Component, OnInit} from '@angular/core';
import { Category } from './category.model';
import {DatePipe, DecimalPipe, NgForOf, NgIf} from '@angular/common'; // 定义的类型接口
import { HttpClient } from '@angular/common/http';
import {FormsModule, NgForm,ReactiveFormsModule} from '@angular/forms';
import {MatFormField, MatInput, MatLabel} from '@angular/material/input';
import {NgbModal, NgbModule} from '@ng-bootstrap/ng-bootstrap';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatIconModule } from '@angular/material/icon';

@Component({
  selector: 'app-product-list',
  templateUrl: './combined-product.component.html',
  imports: [
    NgForOf,
    NgIf,
    DecimalPipe,
    DatePipe,
    FormsModule,
    MatFormField,
    MatInput,
    MatLabel,
    ReactiveFormsModule,
    FormsModule,
    MatInputModule // 添加Material输入模块

  ],
  styleUrls: ['./combined-product.component.css']
})
export class CombinedProductComponent implements OnInit{
  // 新增代码
  selectedCategoryId: number | null = null;
  categories: Category[] = [];
  loading = true;
  error: string | null = null;

  product = {
    product_name: '',
    price: 99.9,
    stock: 100,
    category_ids: '1',
    thumbnail: 'https://randomuser.me/api/portraits/men/1.jpg',
    tags: '夏季,新品',
    is_hot: true
  };

  constructor(
    private http: HttpClient,
    private modalService: NgbModal // 注入Bootstrap模态服务
  ) {}

   // 新增方法
  selectCategory(categoryId: number): void {
    this.selectedCategoryId = categoryId;
  }

  // 新增getter
  get selectedCategory(): Category | undefined {
    return this.categories.find(c => c.category_id === this.selectedCategoryId);
  }

  // 初始化时自动选择第一个分类
  ngOnInit() {
    this.fetchData();
  }

    private fetchData() {
      const apiUrl = 'http://localhost:8000/category-products';

      this.http.get<Category[]>(apiUrl).subscribe({
        next: (data) => {
          console.log('完整响应数据:', data);
          this.categories = data;
          if (this.categories.length > 0) {
            this.selectedCategoryId = this.categories[0].category_id;
          }
          this.loading = false;
        },
        error: (err) => {
          console.error('请求失败:', err);
          this.error = '数据加载失败，请稍后重试';
          this.loading = false;
        }
      });
    }

      // 使用Bootstrap模态框控制
  openModal(content: any) {
    this.modalService.open(content, {
      size: 'lg',
      centered: true,
      backdrop: 'static'
    });
  }

  submitProduct(form: NgForm) {
    if (form.invalid) return;

    const payload = {
      ...this.product,
      category_ids: this.product.category_ids.split(',').map(id => parseInt(id.trim())),
      tags: this.product.tags.split(',').map(tag => tag.trim())
    };

    this.http.post('http://localhost:8000/products', payload)
      .subscribe({
        next: (response: any) => {
          console.log('创建成功，产品ID:', response.product_id);
          this.modalService.dismissAll();
          window.location.reload();
        },
        error: (error) => {
          console.error('创建失败:', error);
        }
      });
  }
}

<div>
    <button
      class="btn btn-primary"
      (click)="openModal(productForm)">
      <i class="material-icons">add</i> 添加新产品
    </button>
<div class="container mt-4">

  <div class="row">

    <!-- 左侧分类列表 -->
    <div class="col-md-3">
      <div class="list-group">
        <a *ngFor="let category of categories"
           class="list-group-item list-group-item-action d-flex align-items-center"
           [class.active]="category.category_id === selectedCategoryId"
           (click)="selectCategory(category.category_id)">
          <img [src]="category.thumbnail"
               alt="分类缩略图"
               class="category-thumbnail-sm rounded-circle me-2">
          {{ category.category_name }}
        </a>
      </div>
    </div>

    <!-- 右侧商品列表 -->
    <div class="col-md-9">
      <div *ngIf="selectedCategory">
        <!-- 分类标题区 -->
        <div class="d-flex align-items-center mb-3 p-3 category-header bg-light rounded">
          <img [src]="selectedCategory.thumbnail"
               alt="分类缩略图"
               class="category-thumbnail rounded-circle me-3">
          <div>
            <h2 class="mb-1">{{ selectedCategory.category_name }}</h2>
            <p class="text-muted">{{ selectedCategory.description }}</p>
          </div>
        </div>

        <!-- 商品网格 -->
        <div class="row g-4">
          <div *ngFor="let product of selectedCategory.products"
               class="col-12 col-md-6 col-lg-4">


        <!-- 商品卡片 -->
        <div class="card h-100 product-card">
          <!-- 商品状态角标 -->
          <div class="product-badges">
            <span *ngIf="product.is_hot" class="badge bg-danger">热卖</span>
            <span *ngIf="product.is_discount" class="badge bg-success">折扣</span>
            <span *ngIf="product.is_new" class="badge bg-info">新品</span>
          </div>

          <!-- 商品图片 -->
          <img [src]="product.thumbnail"
               class="card-img-top product-thumbnail"
               alt="商品图片">

          <div class="card-body">
            <!-- 标题区 -->
            <h5 class="card-title">
              {{ product.product_name }}
              <span class="text-danger ms-2">¥{{ product.price | number:'1.2-2' }}</span>
            </h5>

            <!-- 商品信息 -->
            <div class="product-meta mb-2">
              <span class="text-success me-3">
                <i class="bi bi-check2-circle"></i>
                库存 {{ product.stock }}件
              </span>
              <span class="text-muted">
                <i class="bi bi-graph-up"></i>
                已售 {{ product.sales_volume | number }}
              </span>
            </div>



            <!-- 商品标签 -->
            <div class="tags">
              <span *ngFor="let tag of product.tags"
                    class="badge bg-light text-dark me-1 mb-1">
                {{ tag }}
              </span>
            </div>
          </div>

          <!-- 上架时间 -->
          <div class="card-footer bg-transparent">
            <small class="text-muted">
              上架时间：{{ product.listed_time | date:'yyyy-MM-dd HH:mm' }}
            </small>
          </div>
        </div>



          </div>
        </div>
      </div>
    </div>
  </div>


  <!-- 使用Bootstrap模态结构 -->
  <ng-template #productForm let-modal>
    <div class="modal-header">
      <h4 class="modal-title">新增产品</h4>
      <button type="button" class="btn-close" (click)="modal.dismiss()"></button>
    </div>

    <div class="modal-body">
      <form #form="ngForm" (ngSubmit)="submitProduct(form)">
        <!-- 使用Material输入框 -->
        <div class="mb-3">
          <mat-form-field appearance="outline" class="w-100">
            <mat-label>产品名称</mat-label>
            <input
              matInput
              type="text"
              [(ngModel)]="product.product_name"
              name="product_name"
              required>
          </mat-form-field>
        </div>

        <!-- 价格输入 -->
        <div class="row g-3">
          <div class="col-md-6">
            <mat-form-field appearance="outline" class="w-100">
              <mat-label>价格</mat-label>
              <input
                matInput
                type="number"
                [(ngModel)]="product.price"
                name="price"
                step="0.1"
                required>
            </mat-form-field>
          </div>

          <!-- 库存输入 -->
          <div class="col-md-6">
            <mat-form-field appearance="outline" class="w-100">
              <mat-label>库存</mat-label>
              <input
                matInput
                type="number"
                [(ngModel)]="product.stock"
                name="stock"
                required>
            </mat-form-field>
          </div>
        </div>

        <!-- 分类ID -->
        <div class="mb-3">
          <mat-form-field appearance="outline" class="w-100">
            <mat-label>分类ID（逗号分隔）</mat-label>
            <input
              matInput
              type="text"
              [(ngModel)]="product.category_ids"
              name="category_ids"
              required>
          </mat-form-field>
        </div>

        <!-- 缩略图URL -->
        <div class="mb-3">
          <mat-form-field appearance="outline" class="w-100">
            <mat-label>缩略图URL</mat-label>
            <input
              matInput
              type="url"
              [(ngModel)]="product.thumbnail"
              name="thumbnail"
              required>
          </mat-form-field>
        </div>

        <!-- 标签 -->
        <div class="mb-3">
          <mat-form-field appearance="outline" class="w-100">
            <mat-label>标签（逗号分隔）</mat-label>
            <input
              matInput
              type="text"
              [(ngModel)]="product.tags"
              name="tags"
              required>
          </mat-form-field>
        </div>

        <!-- 是否热门 -->
        <div class="form-check mb-4">
          <input
            class="form-check-input"
            type="checkbox"
            [(ngModel)]="product.is_hot"
            name="is_hot"
            id="isHot">
          <label class="form-check-label" for="isHot">
            是否热门
          </label>
        </div>

        <!-- 按钮组 -->
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-secondary"
            (click)="modal.dismiss()">
            取消
          </button>
          <button
            type="submit"
            class="btn btn-primary"
            [disabled]="form.invalid">
            提交
          </button>
        </div>
      </form>
    </div>
  </ng-template>



</div>
</div>
/* combined-product.component.css */

/* 容器整体布局 */
.container {
  max-width: 1200px;
  padding: 0 15px;
}

/* 分类列表样式 */
.category-list {
  border-right: 1px solid #eee;
  min-height: 100vh;
  padding-right: 1.5rem;
}

.list-group-item {
  transition: all 0.3s ease;
  border: none;
  border-radius: 8px;
  margin-bottom: 8px;
  padding: 1rem 1.25rem;
}

.list-group-item:hover {
  background-color: #f8f9fa;
  transform: translateX(4px);
}

.list-group-item.active {
  background-color: #007bff;
  border-color: #007bff;
  box-shadow: 0 2px 8px rgba(0,123,255,.2);
}

/* 分类缩略图 */
.category-thumbnail-sm {
  width: 36px;
  height: 36px;
  object-fit: cover;
}

/* 商品列表区 */
.product-grid {
  padding-left: 2rem;
}

/* 分类标题区 */
.category-header {
  box-shadow: 0 2px 8px rgba(0,0,0,.05);
  background: linear-gradient(135deg, #ffffff, #f8f9fa);
}

.category-thumbnail {
  width: 64px;
  height: 64px;
  object-fit: cover;
  box-shadow: 0 2px 6px rgba(0,0,0,.1);
}

/* 商品卡片 */
.product-card {
  border: 1px solid rgba(0,0,0,.075);
  transition: transform 0.3s ease;
  overflow: hidden;
  position: relative;
}

.product-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 15px rgba(0,0,0,.1);
}

/* 商品图片 */
.product-thumbnail {
  height: 200px;
  object-fit: contain;
  padding: 1rem;
  background-color: #f8f9fa;
}

/* 商品角标 */
.product-badges {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 2;
}

.product-badges .badge {
  font-size: 0.75rem;
  padding: 0.35em 0.65em;
  margin-right: 4px;
  text-shadow: 0 1px 1px rgba(0,0,0,.1);
}

/* 评分样式 */
.rating {
  font-size: 0.9rem;
}

.bi-star-fill {
  color: #ffc107;
  margin-right: 2px;
}

.bi-star-half {
  color: #ffc107;
  margin-right: 2px;
}

/* 商品标签 */
.tags .badge {
  border: 1px solid #dee2e6;
  font-weight: 400;
  font-size: 0.8rem;
  padding: 0.3em 0.6em;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .category-list {
    border-right: none;
    padding-right: 0;
    margin-bottom: 2rem;
  }

  .product-grid {
    padding-left: 0;
  }

  .product-card {
    margin-bottom: 1.5rem;
  }
}
/* 混合布局适配 */
.modal-content {
  border: none;
  box-shadow: 0 8px 32px rgba(0,0,0,0.12);
}

.mat-form-field {
  margin-bottom: 1.5rem;
}

/* 调整Material输入框与Bootstrap的兼容 */
.mat-mdc-form-field:not(.mat-form-field-no-animations) .mdc-text-field__input {
  padding: 12px 16px !important;
}

/* 复选框对齐 */
.form-check {
  display: flex;
  align-items: center;
  gap: 8px;
  padding-left: 0;
}

.form-check-input {
  margin: 0;
  width: 20px;
  height: 20px;
}

/* 响应式优化 */
@media (max-width: 768px) {
  .modal-dialog {
    margin: 8px;
  }

  .mat-form-field {
    margin-bottom: 1rem;
  }
}
export interface Product {
  product_id: number;
  product_name: string;
  price: number;
  stock: number;
  thumbnail: string;
  description: string;
  tags?: string[];
  listed_time: string;
  sales_volume: number;
  rating: number[];
  is_hot?: boolean;
  is_discount?: boolean;
  is_new?: boolean;
}

export interface Category {
  category_id: number;
  category_name: string;
  parent_id: number | null;
  thumbnail: string;
  description: string;
  products: Product[];
}

```

end