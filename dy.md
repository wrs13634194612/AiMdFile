说明：fastapi+angular实现汽车销售系统 

1.车辆列表
2.可以立即购买车辆
3.结算页
4.查看订单详情
5.可以对车辆评论
6.新增车辆按钮  
7.查询车辆交易记录表
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/8d0b620a3bc54ab0ab9158adbd32617d.png#pic_center)

step1:sql

```sql
-- Users Table
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100),
    phone VARCHAR(20),
    address VARCHAR(255),
    role ENUM('admin', 'customer') DEFAULT 'customer',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Cars Table
CREATE TABLE cars (
    car_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    status ENUM('available', 'sold_out', 'removed') DEFAULT 'available',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Orders Table
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    status ENUM('pending_payment', 'paid', 'cancelled', 'completed') DEFAULT 'pending_payment',
    address VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Order Details Table
CREATE TABLE order_details (
    order_detail_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    car_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    subtotal DECIMAL(10, 2) GENERATED ALWAYS AS (quantity * price) STORED,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (car_id) REFERENCES cars(car_id) ON DELETE CASCADE
);

-- Reviews Table
CREATE TABLE reviews (
    review_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    car_id INT NOT NULL,
    content TEXT NOT NULL,
    rating INT CHECK (rating BETWEEN 1 AND 5),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (car_id) REFERENCES cars(car_id) ON DELETE CASCADE
);

-- Payments Table
CREATE TABLE payments (
    payment_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    method ENUM('credit_card', 'paypal', 'other'),
    status ENUM('pending', 'success', 'failed') DEFAULT 'pending',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
);

-- Indexes
CREATE INDEX idx_cars_name ON cars(name);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_created ON orders(created_at);
CREATE INDEX idx_reviews_car ON reviews(car_id);

-- Users
INSERT INTO users (username, password, email, phone, address, role) VALUES
('admin1', 'admin123', 'admin1@example.com', '13800000001', 'Chaoyang District, Beijing', 'admin'),
('customer1', 'cust123', 'cust1@example.com', '13800000002', 'Pudong New Area, Shanghai', 'customer');

-- Cars (10 items)
INSERT INTO cars (name, description, price, stock, status) VALUES
('Tesla Model S', 'High-performance electric sedan with 650 km range', 799900.00, 5, 'available'),
('Toyota Camry', 'Classic family sedan with 2.5L engine', 239800.00, 0, 'sold_out');

-- Orders
INSERT INTO orders (user_id, total_amount, status, address) VALUES
(2, 799900.00, 'completed', 'Pudong New Area, Shanghai'),
(3, 239800.00, 'cancelled', 'Tianhe District, Guangzhou');

-- Order Details
INSERT INTO order_details (order_id, car_id, quantity, price) VALUES
(1, 1, 1, 799900.00),
(2, 2, 1, 239800.00);

-- Reviews
INSERT INTO reviews (user_id, car_id, content, rating) VALUES
(2, 1, 'Fast acceleration, full of tech features!', 5),
(3, 2, 'Fuel efficient, perfect for families', 4);

-- Payments
INSERT INTO payments (order_id, amount, method, status) VALUES
(1, 799900.00, 'credit_card', 'success'),
(2, 239800.00, 'paypal', 'failed');
```

step2:fastapi

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import pymysql.cursors
from enum import Enum
from fastapi.middleware.cors import CORSMiddleware
from typing import Optional
from decimal import Decimal
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


# 公共数据库方法
def db_execute(query: str, params=None, fetch=True):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params or ())
            if fetch:
                result = cursor.fetchall()
            else:
                result = cursor.lastrowid
            connection.commit()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Database error: {str(e)}")


# 数据模型
class CarStatus(str, Enum):
    available = "available"
    sold_out = "sold_out"
    removed = "removed"


class OrderStatus(str, Enum):
    pending_payment = "pending_payment"
    paid = "paid"
    cancelled = "cancelled"
    completed = "completed"


class CarCreate(BaseModel):
    name: str
    description: str
    price: float
    stock: int


class OrderCreate(BaseModel):
    user_id: int
    items: list[dict]  # [{car_id: int, quantity: int}]
    address: str


class ReviewCreate(BaseModel):
    user_id: int
    content: str
    rating: int


# 接口实现
@app.get("/cars")
async def get_cars(
        name: Optional[str] = None,
        min_price: Optional[float] = None,
        max_price: Optional[float] = None,
        status: Optional[CarStatus] = None
):
    """获取车辆列表"""
    base_query = "SELECT * FROM cars WHERE 1=1"
    params = []

    if name:
        base_query += " AND name LIKE %s"
        params.append(f"%{name}%")
    if min_price is not None:
        base_query += " AND price >= %s"
        params.append(min_price)
    if max_price is not None:
        base_query += " AND price <= %s"
        params.append(max_price)
    if status:
        base_query += " AND status = %s"
        params.append(status.value)

    return {"data": db_execute(base_query, params)}


@app.post("/cars")
async def create_car(car: CarCreate):
    """新增车辆（管理员）"""
    query = """
    INSERT INTO cars 
    (name, description, price, stock, status)
    VALUES (%s, %s, %s, %s, 'available')
    """
    params = (car.name, car.description, car.price, car.stock)
    db_execute(query, params, fetch=False)
    return {"message": "Car created successfully"}


@app.post("/orders")
async def create_order(order: OrderCreate):
    """创建订单"""
    try:
        # 开始事务
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            # 验证用户
            cursor.execute("SELECT user_id FROM users WHERE user_id = %s", order.user_id)
            if not cursor.fetchone():
                raise HTTPException(status_code=404, detail="User not found")

            # 计算总金额
            total = Decimal('0')
            order_details = []
            for item in order.items:
                cursor.execute(
                    "SELECT price, stock FROM cars WHERE car_id = %s FOR UPDATE",
                    item['car_id']
                )
                car = cursor.fetchone()
                if not car:
                    raise HTTPException(status_code=404, detail=f"Car {item['car_id']} not found")
                if car['stock'] < item['quantity']:
                    raise HTTPException(status_code=400, detail=f"Car {item['car_id']} stock insufficient")

                subtotal = car['price'] * item['quantity']
                total += subtotal
                order_details.append({
                    'car_id': item['car_id'],
                    'quantity': item['quantity'],
                    'price': car['price']
                })

            # 创建订单
            order_query = """
            INSERT INTO orders 
            (user_id, total_amount, status, address)
            VALUES (%s, %s, 'pending_payment', %s)
            """
            cursor.execute(order_query, (order.user_id,  float(total), order.address))
            order_id = cursor.lastrowid

            # 创建订单详情
            for detail in order_details:
                detail_query = """
                INSERT INTO order_details 
                (order_id, car_id, quantity, price)
                VALUES (%s, %s, %s, %s)
                """
                cursor.execute(detail_query,
                               (order_id, detail['car_id'], detail['quantity'], float(detail['price'])))

                # 更新库存
                cursor.execute(
                    "UPDATE cars SET stock = stock - %s WHERE car_id = %s",
                    (detail['quantity'], detail['car_id'])
                )

            connection.commit()
            return {"order_id": order_id, "total_amount": total}
    except HTTPException as he:
        connection.rollback()
        raise he
    except Exception as e:
        connection.rollback()
        raise HTTPException(status_code=500, detail=str(e))
    finally:
        connection.close()


@app.get("/orders/{order_id}")
async def get_order(order_id: int):
    """获取订单详情"""
    order_query = """
    SELECT o.*, u.username 
    FROM orders o
    JOIN users u ON o.user_id = u.user_id
    WHERE o.order_id = %s
    """
    order = db_execute(order_query, (order_id,))
    if not order:
        raise HTTPException(status_code=404, detail="Order not found")

    details_query = """
    SELECT od.*, c.name as car_name 
    FROM order_details od
    JOIN cars c ON od.car_id = c.car_id
    WHERE od.order_id = %s
    """
    details = db_execute(details_query, (order_id,))

    return {
        "order_info": order[0],
        "order_details": details
    }


@app.post("/reviews")
async def create_review(review: ReviewCreate, car_id: int):
    """创建车辆评论"""
    # 验证用户和车辆
    db_execute("SELECT user_id FROM users WHERE user_id = %s", (review.user_id,))
    db_execute("SELECT car_id FROM cars WHERE car_id = %s", (car_id,))

    if not 1 <= review.rating <= 5:
        raise HTTPException(status_code=400, detail="Rating must be between 1-5")

    query = """
    INSERT INTO reviews 
    (user_id, car_id, content, rating)
    VALUES (%s, %s, %s, %s)
    """
    params = (review.user_id, car_id, review.content, review.rating)
    db_execute(query, params, fetch=False)
    return {"message": "Review created successfully"}


@app.get("/cars/{car_id}/reviews")
async def get_car_reviews(car_id: int):
    """获取车辆评论"""
    query = """
    SELECT r.*, u.username 
    FROM reviews r
    JOIN users u ON r.user_id = u.user_id
    WHERE r.car_id = %s
    ORDER BY created_at DESC
    """
    return {"data": db_execute(query, (car_id,))}


@app.get("/cars/{car_id}/transactions")
async def get_transactions(car_id: int):
    """获取车辆交易记录"""
    query = """
    SELECT 
        o.order_id,
        o.created_at,
        od.quantity,
        od.price,
        u.username as buyer
    FROM order_details od
    JOIN orders o ON od.order_id = o.order_id
    JOIN users u ON o.user_id = u.user_id
    WHERE od.car_id = %s
    """
    return {"data": db_execute(query, (car_id,))}


if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\services\car.service.ts

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
    providedIn: 'root'
})
export class CarService {
    private apiUrl = 'http://localhost:8000';

    constructor(private http: HttpClient) { }

    getCars(filters?: any): Observable<any> {
        const params: any = {};

        // 参数映射
        const fieldMapping: { [key: string]: string } = {
            name: 'name',
            minPrice: 'min_price',
            maxPrice: 'max_price',
            status: 'status'
        };

        if (filters) {
            Object.entries(filters).forEach(([key, value]) => {
                if (value !== null && value !== '') {
                    const paramName = fieldMapping[key] || key;
                    params[paramName] = value;
                }
            });
        }

        return this.http.get(`${this.apiUrl}/cars`, { params });
    }


    // car.service.ts
    addCar(carData: any): Observable<any> {
        return this.http.post(`${this.apiUrl}/cars`, carData);
    }

    // car.service.ts
    getCarReviews(carId: number): Observable<any> {
        return this.http.get(`${this.apiUrl}/cars/${carId}/reviews`);
    }

    // 在CarService中添加
    createOrder(orderData: any): Observable<any> {
        console.log("createOrder:",JSON.stringify(orderData))
        return this.http.post(`${this.apiUrl}/orders`, orderData);
    }

    getTransactions(carId: number) {
        return this.http.get(`${this.apiUrl}/cars/${carId}/transactions`);
    }

    getOrderDetails(orderId: number): Observable<any> {
        return this.http.get(`${this.apiUrl}/orders/${orderId}`);
    }

}

```

step4:路由 C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\app.routes.ts

```typescript

  { path: 'car-list', component: CarListComponent },
  {
    path: 'cars/:carId/transactions',
    component: CarRecordComponent
  },

```

step5:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\car-list\car.model.ts

```typescript
export interface Car {
  car_id: number;
  name: string;
  description: string;
  price: number;
  stock: number;
  status: 'available' | 'sold_out' | 'removed';
  created_at: string;
  updated_at: string;
}

```

step6:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\car-list\car-list.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { CarService } from '../services/car.service';
import { Car } from './car.model';
import {Router} from '@angular/router';

@Component({
  selector: 'app-car-list',
  standalone: true,
  imports: [FormsModule, CommonModule],
  templateUrl: './car-list.component.html',
  styleUrls: ['./car-list.component.css']
})
export class CarListComponent implements OnInit {
  cars: Car[] = [];
  isLoading = false;
  errorMessage = '';

  filters = {
    name: '',
    minPrice: null,
    maxPrice: null,
    status: ''
  };

  statusMap = {
    'available': '可购买',
    'sold_out': '已售罄',
    'removed': '已下架'
  };


  // 新增车辆相关属性
  showAddCarModal = false;
  newCar = {
    name: '',
    description: '',
    price: null as number | null,
    stock: null as number | null,
    status: 'available' as 'available' | 'sold_out' | 'removed'
  };
  addCarError = '';
  isSubmitting = false;

  // 新增评论相关属性
  showReviewsModal = false;
  selectedCarReviews: any[] = [];
  reviewsLoading = false;
  reviewsError = '';
  selectedCarId?: number;

  // 新增购买相关属性
  showPurchaseModal = false;
  selectedCar: Car | null = null;
  purchaseFormData = {
    userId: null as number | null,
    address: '',
    quantity: 1
  };
  purchaseError = '';
  isPurchasing = false;
  purchaseSuccess = false;


  constructor(private carService: CarService, private router: Router) {}

  ngOnInit() {
    this.loadCars();
  }

  loadCars() {
    this.isLoading = true;
    this.errorMessage = '';

    const params = this.cleanFilters();
    this.carService.getCars(params).subscribe({
      next: (res: any) => {
        this.cars = res.data;
        this.isLoading = false;
      },
      error: (err) => {
        this.errorMessage = '加载车辆数据失败，请稍后重试';
        this.isLoading = false;
        console.error('加载错误:', err);
      }
    });
  }

  cleanFilters() {
    const cleaned: any = {};
    Object.entries(this.filters).forEach(([key, value]) => {
      if (value !== null && value !== '') {
        cleaned[key] = value;
      }
    });
    return cleaned;
  }

  resetFilters() {
    this.filters = {
      name: '',
      minPrice: null,
      maxPrice: null,
      status: ''
    };
    this.loadCars();
  }

  // 新增显示添加车辆弹窗方法
  showAddCarForm() {
    this.showAddCarModal = true;
    this.addCarError = '';
  }

  // 新增提交车辆方法
  submitCar() {
    if (!this.validateCarForm()) return;

    this.isSubmitting = true;
    this.carService.addCar(this.newCar).subscribe({
      next: () => {
        this.closeAddModal();
        this.loadCars(); // 刷新列表
      },
      error: (err) => {
        this.addCarError = '添加车辆失败，请检查数据格式';
        this.isSubmitting = false;
        console.error('添加错误:', err);
      }
    });
  }

  // 表单验证方法
  private validateCarForm(): boolean {
    this.addCarError = '';

    if (!this.newCar.name.trim()) {
      this.addCarError = '车辆名称不能为空';
      return false;
    }

    if (this.newCar.price === null || this.newCar.price <= 0) {
      this.addCarError = '价格必须大于0';
      return false;
    }

    if (this.newCar.stock === null || this.newCar.stock < 0) {
      this.addCarError = '库存不能为负数';
      return false;
    }

    return true;
  }

  // 关闭弹窗并重置表单
  closeAddModal() {
    this.showAddCarModal = false;
    this.newCar = {
      name: '',
      description: '',
      price: null,
      stock: null,
      status: 'available'
    };
    this.isSubmitting = false;
    this.addCarError = '';
  }

  // 在类中添加以下方法
  // 显示评论弹窗
  showCarReviews(carId: number) {
    this.selectedCarId = carId;
    this.reviewsLoading = true;
    this.reviewsError = '';

    this.carService.getCarReviews(carId).subscribe({
      next: (res: any) => {
        this.selectedCarReviews = res.data;
        this.showReviewsModal = true;
        this.reviewsLoading = false;
      },
      error: (err) => {
        this.reviewsError = '获取评论失败';
        this.reviewsLoading = false;
        console.error('评论加载错误:', err);
      }
    });
  }

  // 关闭评论弹窗
  closeReviewsModal() {
    this.showReviewsModal = false;
    this.selectedCarReviews = [];
    this.selectedCarId = undefined;
    this.reviewsError = '';
  }

  // 添加评分星星生成方法
  generateStars(rating: number): string {
    return '★'.repeat(rating) + '☆'.repeat(5 - rating);
  }




  // 在类中添加以下方法
  // 显示购买弹窗
  showPurchaseForm(car: Car) {
    this.selectedCar = car;
    this.showPurchaseModal = true;
    this.resetPurchaseForm();
  }

  // 提交购买
  submitPurchase() {
    if (!this.validatePurchaseForm()) return;

    this.isPurchasing = true;
    const orderData = {
      user_id: this.purchaseFormData.userId,
      address: this.purchaseFormData.address,
      items: [{
        car_id: this.selectedCar?.car_id,
        quantity: this.purchaseFormData.quantity
      }]
    };

    this.carService.createOrder(orderData).subscribe({
      next: () => {
        this.purchaseSuccess = true;
        this.isPurchasing = false;
        setTimeout(() => {
          this.closePurchaseModal();
          this.loadCars(); // 刷新列表
        }, 1500);
      },
      error: (err) => {
        this.purchaseError = this.getPurchaseErrorMessage(err);
        this.isPurchasing = false;
      }
    });
  }

  // 表单验证
  private validatePurchaseForm(): boolean {
    this.purchaseError = '';

    if (!this.purchaseFormData.userId) {
      this.purchaseError = '请输入用户ID';
      return false;
    }

    if (!this.purchaseFormData.address.trim()) {
      this.purchaseError = '请输入收货地址';
      return false;
    }

    if (this.purchaseFormData.quantity < 1) {
      this.purchaseError = '购买数量至少为1';
      return false;
    }

    return true;
  }

  // 获取错误信息
  private getPurchaseErrorMessage(error: any): string {
    if (error?.error?.detail) {
      return error.error.detail;
    }
    return '购买失败，请稍后重试';
  }

  // 关闭弹窗
  closePurchaseModal() {
    this.showPurchaseModal = false;
    this.selectedCar = null;
    this.resetPurchaseForm();
  }

  // 重置表单
  private resetPurchaseForm() {
    this.purchaseFormData = {
      userId: null,
      address: '',
      quantity: 1
    };
    this.purchaseError = '';
    this.purchaseSuccess = false;
    this.isPurchasing = false;
  }

  // 新增导航方法
  navigateToTransactions(carId: number) {
    this.router.navigate([`/cars/${carId}/transactions`]);
  }


}
<div class="container">
  <!-- 筛选表单 -->
  <div class="filter-section">
    <h2>车辆筛选</h2>
    <div class="filter-controls">
      <div class="form-group">
        <label>名称:</label>
        <input type="text" [(ngModel)]="filters.name">
      </div>

      <div class="form-group">
        <label>价格区间:</label>
        <input type="number" [(ngModel)]="filters.minPrice" placeholder="最低价">
        <span>-</span>
        <input type="number" [(ngModel)]="filters.maxPrice" placeholder="最高价">
      </div>

      <div class="form-group">
        <label>状态:</label>
        <select [(ngModel)]="filters.status">
          <option value="">全部</option>
          <option value="available">可购买</option>
          <option value="sold_out">已售罄</option>
          <option value="removed">已下架</option>
        </select>
      </div>

      <div class="button-group">
        <button class="btn-add" (click)="loadCars()">搜索</button>
        <button class="btn-add" (click)="resetFilters()">重置</button>
        <button class="btn-add" (click)="showAddCarForm()">+ 新增车辆</button>
      </div>
    </div>
  </div>

  <!-- 加载状态 -->
  <div *ngIf="isLoading" class="loading">加载中...</div>

  <!-- 错误提示 -->
  <div *ngIf="errorMessage" class="error">{{ errorMessage }}</div>

  <!-- 车辆列表 -->
  <div *ngIf="!isLoading && !errorMessage">
    <div *ngIf="cars.length === 0" class="no-data">暂无车辆数据</div>

    <div class="car-list">
      <div *ngFor="let car of cars" class="car-card">
        <div class="car-header">
          <h3>{{ car.name }}</h3>
          <span [class]="car.status" class="status">{{ statusMap[car.status] }}</span>
        </div>

        <div class="car-body">
          <p class="description">{{ car.description }}</p>

          <div class="details">
            <div class="price">
              <label>价格:</label>
              <span>¥{{ car.price | number:'1.2-2' }}</span>
            </div>

            <div class="stock">
              <label>库存:</label>
              <span>{{ car.stock }} 辆</span>
            </div>

            <div class="update-time">
              <label>最后更新:</label>
              <span>{{ car.updated_at | date: 'yyyy-MM-dd HH:mm' }}</span>
            </div>
          </div>
          <div class="card-actions">
            <button  class="btn-add" (click)="showCarReviews(car.car_id)">查看评论</button>
            <button  class="btn-add" (click)="showPurchaseForm(car)">立即购买</button>
            <button class="btn-add" (click)="navigateToTransactions(car.car_id)">交易记录</button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 在container末尾添加新增车辆弹窗 -->
  <div *ngIf="showAddCarModal" class="modal-backdrop">
    <div class="add-car-modal">
      <div class="modal-header">
        <h3>新增车辆</h3>
        <button class="btn-close" (click)="closeAddModal()">&times;</button>
      </div>

      <div class="modal-body">
        <div *ngIf="addCarError" class="error-message">{{ addCarError }}</div>

        <form (submit)="submitCar()">
          <div class="form-group">
            <label>车辆名称：</label>
            <input type="text" [(ngModel)]="newCar.name" name="name" required>
          </div>

          <div class="form-group">
            <label>车辆描述：</label>
            <textarea [(ngModel)]="newCar.description" name="description"></textarea>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label>价格（元）：</label>
              <input type="number" [(ngModel)]="newCar.price" name="price" step="0.01" required>
            </div>

            <div class="form-group">
              <label>库存数量：</label>
              <input type="number" [(ngModel)]="newCar.stock" name="stock" required>
            </div>
          </div>

          <div class="form-group">
            <label>状态：</label>
            <select [(ngModel)]="newCar.status" name="status">
              <option value="available">可购买</option>
              <option value="sold_out">已售罄</option>
              <option value="removed">已下架</option>
            </select>
          </div>

          <div class="form-actions">
            <button type="button" (click)="closeAddModal()">取消</button>
            <button type="submit" [disabled]="isSubmitting">
              {{ isSubmitting ? '提交中...' : '确认添加' }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>

  <!-- 添加评论弹窗 -->
  <div *ngIf="showReviewsModal" class="modal-backdrop">
    <div class="reviews-modal">
      <div class="modal-header">
        <h3>用户评价</h3>
      </div>

      <div class="modal-body">
        <div *ngIf="reviewsLoading" class="loading">加载中...</div>
        <div *ngIf="reviewsError" class="error">{{ reviewsError }}</div>

        <div *ngIf="!reviewsLoading && !reviewsError">
          <div *ngIf="selectedCarReviews.length === 0" class="no-reviews">
            暂无用户评价
          </div>

          <div class="review-list">
            <div *ngFor="let review of selectedCarReviews" class="review-item">
              <div class="review-header">
              <span class="user-info">
                {{ review.username }}
                <span class="rating">{{ generateStars(review.rating) }}</span>
              </span>
                <span class="review-time">
                {{ review.created_at | date: 'yyyy-MM-dd HH:mm' }}
              </span>
              </div>
              <div class="review-content">
                {{ review.content }}
              </div>
            </div>
          </div>
        </div>
        <div class="form-actions">
          <button type="button" (click)="closeReviewsModal()">关闭</button>
        </div>
      </div>
    </div>
  </div>


  <!-- 添加购买弹窗 -->
  <div *ngIf="showPurchaseModal" class="modal-backdrop">
    <div class="purchase-modal">
      <div class="modal-header">
        <h3>购买 {{ selectedCar?.name }}</h3>
        <button class="btn-close" (click)="closePurchaseModal()">&times;</button>
      </div>

      <div class="modal-body">
        <div *ngIf="purchaseSuccess" class="success-message">
          ✔️ 购买成功，库存已更新！
        </div>

        <div *ngIf="!purchaseSuccess">
          <div *ngIf="purchaseError" class="error-message">{{ purchaseError }}</div>

          <form (submit)="submitPurchase()">
            <div class="form-group">
              <label>用户ID：</label>
              <input
                type="number"
                [(ngModel)]="purchaseFormData.userId"
                name="userId"
                required
                [disabled]="isPurchasing"
              >
            </div>

            <div class="form-group">
              <label>收货地址：</label>
              <input
                type="text"
                [(ngModel)]="purchaseFormData.address"
                name="address"
                required
                [disabled]="isPurchasing">
            </div>

            <div class="form-group">
              <label>购买数量：</label>
              <input
                type="number"
                [(ngModel)]="purchaseFormData.quantity"
                name="quantity"
                min="1"
                required
                [disabled]="isPurchasing"
              >
              <span class="stock-tip">当前库存：{{ selectedCar?.stock }} 辆</span>
            </div>

            <div class="form-actions">
              <button
                type="button"
                (click)="closePurchaseModal()"
                [disabled]="isPurchasing"
              >
                取消
              </button>
              <button
                type="submit"
                [disabled]="isPurchasing"
                class="btn-confirm"
              >
                {{ isPurchasing ? '购买中...' : '确认购买' }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</div>


.container {
  max-width: 1200px;
  margin: 20px auto;
  padding: 20px;
}

.filter-section {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 30px;
}

.filter-controls {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
  align-items: end;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

input, select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.button-group {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}

button {
  padding: 8px 15px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #0056b3;
}

.car-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.car-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  background: white;
}

.car-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.status {
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 0.9em;
}

.status.available {
  background: #d4edda;
  color: #155724;
}

.status.sold_out {
  background: #f8d7da;
  color: #721c24;
}

.status.removed {
  background: #fff3cd;
  color: #856404;
}

.details {
  display: grid;
  gap: 8px;
  margin-top: 10px;
}

.details div {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.loading, .error, .no-data {
  text-align: center;
  padding: 20px;
  font-size: 1.1em;
}

.error {
  color: #dc3545;
}


/* 新增按钮样式 */
.header-actions {
  margin-bottom: 20px;
  text-align: right;
}

.btn-add {
  background-color: #007bff;
  padding: 8px 16px;
  color: white;
  margin-left: 1px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
}



/* 弹窗样式 */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.add-car-modal {
  background: white;
  width: 500px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.modal-header {
  padding: 16px;
  border-bottom: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.btn-close {
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
}

.modal-body {
  padding: 16px;
}

.form-row {
  display: flex;
  gap: 16px;
}

.form-row .form-group {
  flex: 1;
}

.form-group textarea {
  width: 100%;
  height: 80px;
  padding: 8px;
}

.form-actions {
  margin-top: 20px;
  text-align: right;
  display: flex;
  gap: 8px;
  justify-content: flex-end;
}

.error-message {
  color: #dc3545;
  margin-bottom: 12px;
}

/* 卡片操作按钮 */
.card-actions {
  margin-top: 15px;
  border-top: 1px solid #eee;
  padding-top: 10px;
}

.btn-review {
  background-color: #17a2b8;
  color: white;
  padding: 6px 12px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
}

.btn-review:hover {
  background-color: #138496;
}

/* 评论弹窗 */
.reviews-modal {
  background: white;
  width: 600px;
  max-height: 80vh;
  border-radius: 8px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-body {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}

.review-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.review-item {
  background: #f8f9fa;
  border-radius: 4px;
  padding: 12px;
}

.review-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}

.user-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.rating {
  color: #ffc107;
  font-size: 1.2em;
}

.review-time {
  color: #6c757d;
  font-size: 0.9em;
}

.review-content {
  line-height: 1.6;
}

.no-reviews {
  text-align: center;
  color: #6c757d;
  padding: 20px;
}

/* 购买按钮样式 */
.btn-buy {
  background-color: #ff5722;
  color: white;
  margin-right: 8px;
}

.btn-buy:hover {
  background-color: #e64a19;
}

/* 购买弹窗 */
.purchase-modal {
  background: white;
  width: 400px;
  border-radius: 8px;
}

.stock-tip {
  color: #666;
  font-size: 0.9em;
  margin-left: 8px;
}

.success-message {
  color: #28a745;
  padding: 20px;
  text-align: center;
  font-size: 1.1em;
}

.btn-confirm {
  background-color: #2196f3;
}

.btn-confirm:hover {
  background-color: #1976d2;
}

/* 表单禁用样式 */
input:disabled,
button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

```

step7:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\car-record\car-record.component.ts

 

```typescript
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute, Router } from '@angular/router';
import { CarService } from '../services/car.service';
import { DatePipe, DecimalPipe, NgForOf, NgIf } from '@angular/common';

@Component({
  selector: 'app-car-record',
  templateUrl: './car-record.component.html',
  styleUrls: ['./car-record.component.css'],
  standalone: true,
  imports: [DatePipe, DecimalPipe, NgForOf, NgIf]
})
export class CarRecordComponent implements OnInit {
  carId!: number;
  carName = '';
  transactions: any[] = [];
  isLoading = true;
  error = '';


  // 新增订单详情相关属性
  showOrderDetailModal = false;
  selectedOrder: any = null;
  orderDetails: any[] = [];
  orderDetailLoading = false;
  orderDetailError = '';

  constructor(
    private route: ActivatedRoute,
    private recordService: CarService,
    private router: Router
  ) {}

  ngOnInit() {
    this.carId = +this.route.snapshot.params['carId'];
    this.loadTransactions();
  }

  async loadTransactions() {
    try {
      const res: any = await this.recordService.getTransactions(this.carId).toPromise();

      console.log("loadTransactions:",JSON.stringify(res))
      this.transactions = res.data;
      this.carName = '车辆#' + this.carId;
    } catch (err) {
      this.error = '获取交易记录失败';
      console.error(err);
    } finally {
      this.isLoading = false;
    }
  }

  goBack() {
    this.router.navigate(['/car-list']);
  }

  // 新增查看详情方法
  async showOrderDetail(orderId: number) {
    this.orderDetailLoading = true;
    this.orderDetailError = '';
    this.selectedOrder = null;
    this.orderDetails = [];

    try {
      const res = await this.recordService.getOrderDetails(orderId).toPromise();
      this.selectedOrder = res.order_info;
      this.orderDetails = res.order_details;
      this.showOrderDetailModal = true;
    } catch (error) {
      this.orderDetailError = '获取订单详情失败';
      console.error(error);
    } finally {
      this.orderDetailLoading = false;
    }
  }

  // 关闭详情弹窗
  closeOrderDetailModal() {
    this.showOrderDetailModal = false;
    this.selectedOrder = null;
    this.orderDetails = [];
  }
}


 <div class="container">
  <!-- 返回按钮调整到左上角 -->
  <button  class="btn-detail" (click)="goBack()">← 返回列表</button>

  <h2>{{ carName }} 交易记录</h2>

  <div *ngIf="isLoading" class="loading">加载中...</div>
  <div *ngIf="error" class="error">{{ error }}</div>

  <div *ngIf="!isLoading && !error">
    <div *ngIf="transactions.length === 0" class="no-data">
      暂无交易记录
    </div>

    <div class="transaction-list">
      <div *ngFor="let t of transactions" class="transaction-item">
        <div class="header">
          <span class="order-id">订单号: {{ t.order_id }}</span>
          <span class="date">{{ t.created_at | date }}</span>
        </div>
        <div class="details">
          <div>买家: {{ t.buyer }}</div>
          <div>数量: {{ t.quantity }} 辆</div>
          <div>单价: ¥{{ t.price | number:'1.2-2' }}</div>
          <button class="btn-detail"
                  (click)="showOrderDetail(t.order_id)"
                  [disabled]="orderDetailLoading">
            查看详情
          </button>
        </div>
      </div>
    </div>
  </div>


  <!-- 订单详情弹窗（样式与评论弹窗统一） -->
  <div *ngIf="showOrderDetailModal" class="modal-backdrops">
    <div class="order-modal">
      <div class="modal-header">
        <h3>订单详情 #{{selectedOrder?.order_id}}</h3>
      </div>

      <div class="modal-body">
        <div *ngIf="orderDetailLoading" class="loading">加载中...</div>
        <div *ngIf="orderDetailError" class="error-message">
          {{ orderDetailError }}
        </div>

        <div *ngIf="!orderDetailLoading && !orderDetailError">
          <div class="order-info">
            <div class="info-item">
              <label>订单状态:</label>
              <span class="status">{{ selectedOrder?.status }}</span>
            </div>
            <div class="info-item">
              <label>总金额:</label>
              <span class="amount">¥{{ selectedOrder?.total_amount | number:'1.2-2' }}</span>
            </div>
            <div class="info-item">
              <label>收货地址:</label>
              <span class="address">{{ selectedOrder?.address }}</span>
            </div>
          </div>

          <div class="detail-section">
            <h4>商品明细</h4>
            <div class="detail-list">
              <div *ngFor="let item of orderDetails" class="detail-item">
                <div class="car-info">
                  <span class="name">{{ item.car_name }}</span>
                  <span class="quantity">x{{ item.quantity }}</span>
                </div>
                <div class="price">¥{{ item.price | number:'1.2-2' }}</div>
              </div>
            </div>
          </div>
        </div>

        <div class="modal-actions">
          <button (click)="closeOrderDetailModal()">关闭</button>
        </div>
      </div>
    </div>
  </div>
</div>


/* car-record.component.css */
.transaction-list {
  margin: 20px 0;
}

.transaction-item {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 15px;
  margin-bottom: 10px;
}

.header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.order-id {
  font-weight: bold;
}

.date {
  color: #666;
}

.details {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.btn-back {
  margin-top: 20px;
  padding: 8px 20px;
  background-color: #f0f0f0;
  border: 1px solid #ddd;
}

/* 统一弹窗样式 */
.order-modal {
  background: white;
  width: 600px;
  max-height: 80vh;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 1px solid #eee;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.4em;
  color: #333;
}

.btn-close {
  background: none;
  border: none;
  font-size: 1.5em;
  cursor: pointer;
  color: #666;
  transition: color 0.3s;
}

.btn-close:hover {
  color: #333;
}

.error-message {
  color: #dc3545;
  padding: 10px;
  background: #f8d7da;
  border-radius: 4px;
  margin: 10px 0;
}

.info-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #f5f5f5;
}

.info-item label {
  color: #666;
  min-width: 80px;
}

.detail-section {
  margin-top: 20px;
}

.detail-section h4 {
  color: #444;
  margin-bottom: 15px;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  margin: 8px 0;
  background: #f9f9f9;
  border-radius: 4px;
}

.car-info {
  display: flex;
  align-items: center;
  gap: 15px;
}

.quantity {
  color: #666;
  font-size: 0.9em;
}

.modal-actions {
  text-align: right;
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px solid #eee;
}

.modal-actions button {
  padding: 8px 20px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.modal-actions button:hover {
  background: #0056b3;
}

/* 交易记录列表中的查看详情按钮 */
.btn-detail {
  margin-top: 10px;
  padding: 6px 12px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-detail:hover {
  background: #0056b3;
}

.btn-detail:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

/* 弹窗样式 */
.modal-backdrops {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}



```

end