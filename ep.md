说明：
我计划用fastapi+angular做一款停车管理系统，支持跨域
1.设计mysql数据库表，
2.建表，添加测试数据，多表查询，
3.在fastapi写接口查询数据，
4.用postman测试，
5.在angular前端展示

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4bcfae3f59e34c76815aeab1dfb2d0ad.png#pic_center)

用fastapi写4个查询接口 
1.查询具体用户的所有停车记录 
2.查询当前正在停车的记录，显示车牌，车主姓名，进入时间，车位位置
3.查询所有已结束停车的费用明细表，详细列举车型和费率
4.统计停车场车位的利用率和空闲率

step1:sql

```sql


-- 1. 建表语句（6张表）
-- --------------------------------------------------------
-- 用户表
CREATE TABLE users (
                       user_id INT AUTO_INCREMENT PRIMARY KEY,
                       name VARCHAR(50) NOT NULL,
                       phone VARCHAR(20) UNIQUE,
                       email VARCHAR(100) UNIQUE,
                       reg_date DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 车辆表
CREATE TABLE vehicles (
                          vehicle_id INT AUTO_INCREMENT PRIMARY KEY,
                          user_id INT NOT NULL,
                          license_plate VARCHAR(20) UNIQUE NOT NULL,
                          type ENUM('轿车','SUV','卡车','电动车','摩托车'),
                          FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- 车位表
CREATE TABLE parking_spaces (
                                space_id INT AUTO_INCREMENT PRIMARY KEY,
                                zone VARCHAR(10) NOT NULL,  -- 区域（A区/B区等）
                                number VARCHAR(10) NOT NULL, -- 车位编号
                                status ENUM('AVAILABLE','OCCUPIED','MAINTENANCE') DEFAULT 'AVAILABLE',
                                UNIQUE(zone, number)
);

-- 收费规则表
CREATE TABLE fee_rules (
                           rule_id INT AUTO_INCREMENT PRIMARY KEY,
                           vehicle_type ENUM('轿车','SUV','卡车','电动车','摩托车'),
                           hourly_rate DECIMAL(8,2) NOT NULL,
                           effective_date DATE NOT NULL
);

-- 停车记录表
CREATE TABLE parking_records (
                                 record_id INT AUTO_INCREMENT PRIMARY KEY,
                                 vehicle_id INT NOT NULL,
                                 space_id INT NOT NULL,
                                 entry_time DATETIME NOT NULL,
                                 exit_time DATETIME,
                                 fee DECIMAL(10,2),
                                 FOREIGN KEY (vehicle_id) REFERENCES vehicles(vehicle_id),
                                 FOREIGN KEY (space_id) REFERENCES parking_spaces(space_id)
);

-- 支付记录表（新增）
CREATE TABLE payment_records (
                                 payment_id INT AUTO_INCREMENT PRIMARY KEY,
                                 record_id INT NOT NULL,
                                 amount DECIMAL(10,2) NOT NULL,
                                 payment_time DATETIME DEFAULT CURRENT_TIMESTAMP,
                                 payment_method ENUM('支付宝','微信','现金','信用卡'),
                                 status ENUM('PAID','UNPAID'),
                                 FOREIGN KEY (record_id) REFERENCES parking_records(record_id)
);


-- 2. 模拟数据（每表至少10条）
-- --------------------------------------------------------
-- 用户表数据
INSERT INTO users (name, phone, email) VALUES
                                           ('张三', '13800010001', 'zhangsan@park.com'),
                                           ('李四', '13800010002', 'lisi@park.com'),
                                           ('王五', '13800010003', 'wangwu@park.com'),
                                           ('赵六', '13800010004', 'zhaoliu@park.com'),
                                           ('陈七', '13800010005', 'chenqi@park.com'),
                                           ('刘八', '13800010006', 'liuba@park.com'),
                                           ('周九', '13800010007', 'zhoujiu@park.com'),
                                           ('吴十', '13800010008', 'wushi@park.com'),
                                           ('郑十一', '13800010009', 'zheng11@park.com'),
                                           ('孙十二', '13800010010', 'sun12@park.com');

-- 车辆表数据
INSERT INTO vehicles (user_id, license_plate, type) VALUES
                                                        (1, '川A12345', '轿车'),
                                                        (1, '川A67890', 'SUV'),
                                                        (2, '赣B12345', '卡车'),
                                                        (3, '赣C12345', '电动车'),
                                                        (4, '浙D12345', '摩托车'),
                                                        (5, '苏E12345', '轿车'),
                                                        (6, '川F12345', 'SUV'),
                                                        (7, '湘G12345', '卡车'),
                                                        (8, '鄂H12345', '电动车'),
                                                        (9, '鲁J12345', '摩托车'),
                                                        (10, '晋K12345', '轿车');

-- 车位表数据
INSERT INTO parking_spaces (zone, number, status) VALUES
                                                      ('A', '001', 'AVAILABLE'),
                                                      ('A', '002', 'OCCUPIED'),
                                                      ('A', '003', 'MAINTENANCE'),
                                                      ('B', '101', 'AVAILABLE'),
                                                      ('B', '102', 'OCCUPIED'),
                                                      ('C', '201', 'AVAILABLE'),
                                                      ('C', '202', 'OCCUPIED'),
                                                      ('D', '301', 'AVAILABLE'),
                                                      ('D', '302', 'OCCUPIED'),
                                                      ('E', '401', 'MAINTENANCE');


INSERT INTO parking_spaces (zone, number, status) VALUES    ('F', '402', 'AVAILABLE');

-- 收费规则表数据
INSERT INTO fee_rules (vehicle_type, hourly_rate, effective_date) VALUES
                                                                      ('轿车', 5.00, '2024-01-01'),
                                                                      ('SUV', 7.50, '2024-01-01'),
                                                                      ('卡车', 10.00, '2024-01-01'),
                                                                      ('电动车', 3.00, '2024-01-01'),
                                                                      ('摩托车', 2.50, '2024-01-01'),
                                                                      ('轿车', 6.00, '2024-06-01'),  -- 费率调整
                                                                      ('SUV', 8.50, '2024-06-01'),
                                                                      ('卡车', 12.00, '2024-06-01'),
                                                                      ('电动车', 3.50, '2024-06-01'),
                                                                      ('摩托车', 3.00, '2024-06-01');

-- 停车记录表数据
INSERT INTO parking_records (vehicle_id, space_id, entry_time, exit_time, fee) VALUES
                                                                                   (1, 2, '2024-03-01 08:00:00', '2024-03-01 10:00:00', 10.00),
                                                                                   (2, 4, '2024-03-01 09:00:00', '2024-03-01 12:00:00', 22.50),
                                                                                   (3, 5, '2024-03-01 10:00:00', '2024-03-01 15:00:00', 50.00),
                                                                                   (4, 6, NOW() - INTERVAL 2 HOUR, NULL, NULL),
                                                                                   (5, 7, '2024-03-01 11:00:00', '2024-03-01 13:00:00', 5.00),
                                                                                   (6, 8, NOW() - INTERVAL 1 HOUR, NULL, NULL),
                                                                                   (7, 9, '2024-03-01 14:00:00', '2024-03-01 17:00:00', 25.50),
                                                                                   (8, 10, '2024-03-01 15:00:00', '2024-03-01 18:00:00', 36.00),
                                                                                   (9, 1, NOW() - INTERVAL 3 HOUR, NULL, NULL),
                                                                                   (10, 3, '2024-03-01 16:00:00', '2024-03-01 19:00:00', 18.00);

-- 支付记录表数据
INSERT INTO payment_records (record_id, amount, payment_method, status) VALUES
                                                                            (1, 10.00, '支付宝', 'PAID'),
                                                                            (2, 22.50, '微信', 'PAID'),
                                                                            (3, 50.00, '现金', 'PAID'),
                                                                            (5, 5.00, '信用卡', 'PAID'),
                                                                            (7, 25.50, '支付宝', 'PAID'),
                                                                            (8, 36.00, '微信', 'PAID'),
                                                                            (10, 18.00, '信用卡', 'PAID');

-- 3. 查询语句（4个核心查询）
-- --------------------------------------------------------
-- 1. 查询具体用户的所有停车记录（例如：张三）
SELECT
    u.name AS 车主,
    v.license_plate AS 车牌,
    CONCAT(p.zone, '-', p.number) AS 车位,
    pr.entry_time AS 入场时间,
    pr.exit_time AS 离场时间,
    pr.fee AS 费用
FROM users u
         JOIN vehicles v ON u.user_id = v.user_id
         JOIN parking_records pr ON v.vehicle_id = pr.vehicle_id
         JOIN parking_spaces p ON pr.space_id = p.space_id
WHERE u.name = '张三';

-- 2. 查询当前正在停车的记录
SELECT
    v.license_plate AS 车牌,
    u.name AS 车主姓名,
    pr.entry_time AS 进入时间,
    CONCAT(p.zone, '-', p.number) AS 车位位置
FROM parking_records pr
         JOIN vehicles v ON pr.vehicle_id = v.vehicle_id
         JOIN users u ON v.user_id = u.user_id
         JOIN parking_spaces p ON pr.space_id = p.space_id
WHERE pr.exit_time IS NULL;

-- 3. 已结束停车的费用明细表
SELECT
    v.type AS 车型,
    fr.hourly_rate AS 费率,
    TIMESTAMPDIFF(HOUR, pr.entry_time, pr.exit_time) AS 停车小时,
    pr.fee AS 总费用,
    py.payment_method AS 支付方式
FROM parking_records pr
         JOIN vehicles v ON pr.vehicle_id = v.vehicle_id
         JOIN fee_rules fr ON v.type = fr.vehicle_type
    AND pr.entry_time >= fr.effective_date
         JOIN payment_records py ON pr.record_id = py.record_id
WHERE pr.exit_time IS NOT NULL;

-- 4. 统计车位利用率（每日动态计算）
SELECT
    COUNT(*) AS 总车位数,
    SUM(CASE WHEN status = 'OCCUPIED' THEN 1 ELSE 0 END) AS 已用车位,
    SUM(CASE WHEN status = 'AVAILABLE' THEN 1 ELSE 0 END) AS 空闲车位,
    CONCAT(ROUND(SUM(CASE WHEN status = 'OCCUPIED' THEN 1 ELSE 0 END)/COUNT(*)*100,2), '%') AS 利用率,
    CONCAT(ROUND(SUM(CASE WHEN status = 'AVAILABLE' THEN 1 ELSE 0 END)/COUNT(*)*100,2), '%') AS 空闲率
FROM parking_spaces;
```

step2: 写路由接口，支持跨域

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
    'db': 'school_db',  # 修改为实际的数据库名称
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

# 1. 查询具体用户的所有停车记录
@app.get("/user_parking_records/{user_id}")
async def get_user_parking_records(user_id: int):
    """
    获取指定用户的所有停车记录
    - user_id: 用户ID
    """
    query = """
    SELECT 
        u.name AS owner,
        v.license_plate AS license_plate,
        CONCAT(p.zone, '-', p.number) AS space_location,
        pr.entry_time AS entry_time,
        pr.exit_time AS exit_time,
        pr.fee AS fee
    FROM users u
    JOIN vehicles v ON u.user_id = v.user_id
    JOIN parking_records pr ON v.vehicle_id = pr.vehicle_id
    JOIN parking_spaces p ON pr.space_id = p.space_id
    WHERE u.user_id = %s
    """
    data = query_database(query, (user_id,))
    return {"data": data}

# 2. 查询当前正在停车的记录
@app.get("/active_parking_records")
async def get_active_parking_records():
    """获取所有正在进行的停车记录"""
    query = """
    SELECT 
        v.license_plate AS license_plate,
        u.name AS owner_name,
        pr.entry_time AS entry_time,
        CONCAT(p.zone, '-', p.number) AS space_location
    FROM parking_records pr
    JOIN vehicles v ON pr.vehicle_id = v.vehicle_id
    JOIN users u ON v.user_id = u.user_id
    JOIN parking_spaces p ON pr.space_id = p.space_id
    WHERE pr.exit_time IS NULL
    """
    data = query_database(query)
    return {"data": data}

# 3. 查询已结束停车的费用明细
@app.get("/completed_parking_records")
async def get_completed_payments():
    """获取所有已完成停车的费用明细"""
    query = """
    SELECT 
        v.type AS vehicle_type,
        fr.hourly_rate AS rate,
        TIMESTAMPDIFF(HOUR, pr.entry_time, pr.exit_time) AS hours,
        pr.fee AS total_fee,
        py.payment_method AS payment_method
    FROM parking_records pr
    JOIN vehicles v ON pr.vehicle_id = v.vehicle_id
    JOIN fee_rules fr ON v.type = fr.vehicle_type 
        AND pr.entry_time >= fr.effective_date
    JOIN payment_records py ON pr.record_id = py.record_id
    WHERE pr.exit_time IS NOT NULL
    """
    data = query_database(query)
    return {"data": data}

# 4. 统计车位利用率
@app.get("/parking_space_utilization")
async def get_parking_space_stats():
    """获取车位利用率统计"""
    query = """
    SELECT 
        COUNT(*) AS total_spaces,
        SUM(CASE WHEN status = 'OCCUPIED' THEN 1 ELSE 0 END) AS occupied,
        SUM(CASE WHEN status = 'AVAILABLE' THEN 1 ELSE 0 END) AS available,
        ROUND(SUM(CASE WHEN status = 'OCCUPIED' THEN 1 ELSE 0 END)/COUNT(*)*100, 2) AS usage_rate,
        ROUND(SUM(CASE WHEN status = 'AVAILABLE' THEN 1 ELSE 0 END)/COUNT(*)*100, 2) AS free_rate
    FROM parking_spaces
    """
    data = query_database(query)
    return {"data": data[0] if data else {}}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:后端+mysql部分弄完了，接下来写angular前端部分，下面是postman测试

```bash
 

查询具体用户的所有停车记录 
查询当前正在停车的记录，显示车牌，车主姓名，进入时间，车位位置
查询所有已结束停车的费用明细表，详细列举车型和费率
统计停车场车位的利用率和空闲率


http://localhost:8000/user_parking_records/1

{
    "data": [
        {
            "owner": "张三",
            "license_plate": "A12345",
            "space_location": "A-002",
            "entry_time": "2024-03-01T08:00:00",
            "exit_time": "2024-03-01T10:00:00",
            "fee": 10.0
        },
        {
            "owner": "张三",
            "license_plate": "A67890",
            "space_location": "B-101",
            "entry_time": "2024-03-01T09:00:00",
            "exit_time": "2024-03-01T12:00:00",
            "fee": 22.5
        }
    ]
}


http://localhost:8000/active_parking_records

{
    "data": [
        {
            "license_plate": "粤C12345",
            "owner_name": "王五",
            "entry_time": "2025-03-07T13:15:43",
            "space_location": "C-201"
        },
        {
            "license_plate": "苏E12345",
            "owner_name": "陈七",
            "entry_time": "2025-03-07T14:15:43",
            "space_location": "D-301"
        },
        {
            "license_plate": "鄂H12345",
            "owner_name": "吴十",
            "entry_time": "2025-03-07T12:15:43",
            "space_location": "A-001"
        }
    ]
}




http://localhost:8000/completed_parking_records
{
    "data": [
        {
            "vehicle_type": "轿车",
            "rate": 5.0,
            "hours": 2,
            "total_fee": 10.0,
            "payment_method": "支付宝"
        },
        {
            "vehicle_type": "SUV",
            "rate": 7.5,
            "hours": 3,
            "total_fee": 25.5,
            "payment_method": "支付宝"
        },
        {
            "vehicle_type": "SUV",
            "rate": 7.5,
            "hours": 3,
            "total_fee": 22.5,
            "payment_method": "微信"
        },
        {
            "vehicle_type": "卡车",
            "rate": 10.0,
            "hours": 3,
            "total_fee": 36.0,
            "payment_method": "微信"
        },
        {
            "vehicle_type": "卡车",
            "rate": 10.0,
            "hours": 5,
            "total_fee": 50.0,
            "payment_method": "现金"
        },
        {
            "vehicle_type": "摩托车",
            "rate": 2.5,
            "hours": 3,
            "total_fee": 18.0,
            "payment_method": "信用卡"
        },
        {
            "vehicle_type": "摩托车",
            "rate": 2.5,
            "hours": 2,
            "total_fee": 5.0,
            "payment_method": "信用卡"
        }
    ]
}




 http://localhost:8000/parking_space_utilization

{
    "data": {
        "total_spaces": 11,
        "occupied": 4,
        "available": 5,
        "usage_rate": 36.36,
        "free_rate": 45.45
    }
}


 
```

step4:C:\Users\Administrator\WebstormProjects\untitled4\src\app\park\parking.service.ts

```typescript
// parking.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ParkingService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  getUserRecords(userId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/user_parking_records/${userId}`);
  }

  getActiveRecords(): Observable<any> {
    return this.http.get(`${this.apiUrl}/active_parking_records`);
  }

  getCompletedRecords(): Observable<any> {
    return this.http.get(`${this.apiUrl}/completed_parking_records`);
  }

  getParkingStats(): Observable<any> {
    return this.http.get(`${this.apiUrl}/parking_space_utilization`);
  }
}

```

step5:C:\Users\Administrator\WebstormProjects\untitled4\src\app\park\park.component.ts

```typescript
import { Component ,OnInit} from '@angular/core';
import { ParkingService } from './parking.service';
import {FormsModule} from '@angular/forms';
import {CurrencyPipe, DatePipe, NgForOf, NgIf} from '@angular/common';

@Component({
  selector: 'app-park',
  imports: [
    FormsModule,
    DatePipe,
    CurrencyPipe,
    NgForOf,
    NgIf
  ],
  templateUrl: './park.component.html',
  styleUrl: './park.component.css'
})
export class ParkComponent implements OnInit {
  userRecords: any[] = [];
  activeRecords: any[] = [];
  completedRecords: any[] = [];
  parkingStats: any = {};
  selectedUserId: number = 1;

  constructor(private parkingService: ParkingService) {}

  ngOnInit() {
    this.loadAllData();
  }

  loadAllData() {
    this.loadUserRecords();
    this.loadActiveRecords();
    this.loadCompletedRecords();
    this.loadParkingStats();
  }

  loadUserRecords() {
    this.parkingService.getUserRecords(this.selectedUserId)
      .subscribe((res: any) => this.userRecords = res.data);
  }

  loadActiveRecords() {
    this.parkingService.getActiveRecords()
      .subscribe((res: any) => this.activeRecords = res.data);
  }

  loadCompletedRecords() {
    this.parkingService.getCompletedRecords()
      .subscribe((res: any) => this.completedRecords = res.data);
  }

  loadParkingStats() {
    this.parkingService.getParkingStats()
      .subscribe((res: any) => this.parkingStats = res.data);
  }
}

```

step6:C:\Users\Administrator\WebstormProjects\untitled4\src\app\park\park.component.html

```xml
<!-- app.component.html -->
<div class="container">
  <!-- 用户选择 -->
  <div class="user-select">
    <label>选择用户ID: </label>
    <input type="number" [(ngModel)]="selectedUserId" (change)="loadUserRecords()" min="1" max="10">
  </div>

  <!-- 用户停车记录 -->
  <section>
    <h2>用户停车记录 (ID: {{selectedUserId}})</h2>
    <table *ngIf="userRecords.length > 0">
      <thead>
      <tr>
        <th>车主</th>
        <th>车牌号</th>
        <th>车位位置</th>
        <th>入场时间</th>
        <th>离场时间</th>
        <th>费用</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let record of userRecords">
        <td>{{ record.owner }}</td>
        <td>{{ record.license_plate }}</td>
        <td>{{ record.space_location }}</td>
        <td>{{ record.entry_time | date:'medium' }}</td>
        <td>{{ record.exit_time | date:'medium' }}</td>
        <td>{{ record.fee | currency:'CNY':'symbol' }}</td>
      </tr>
      </tbody>
    </table>
    <p *ngIf="userRecords.length === 0">暂无停车记录</p>
  </section>

  <!-- 当前停车记录 -->
  <section>
    <h2>当前停车记录</h2>
    <table *ngIf="activeRecords.length > 0">
      <thead>
      <tr>
        <th>车牌号</th>
        <th>车主姓名</th>
        <th>进入时间</th>
        <th>车位位置</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let record of activeRecords">
        <td>{{ record.license_plate }}</td>
        <td>{{ record.owner_name }}</td>
        <td>{{ record.entry_time | date: 'yyyy-MM-dd HH:mm' }}</td>
        <td>{{ record.space_location }}</td>
      </tr>
      </tbody>
    </table>
    <p *ngIf="activeRecords.length === 0">当前没有车辆停放</p>
  </section>

  <!-- 完成停车费用 -->
  <section>
    <h2>历史费用明细</h2>
    <table *ngIf="completedRecords.length > 0">
      <thead>
      <tr>
        <th>车型</th>
        <th>费率 (小时)</th>
        <th>停车时长</th>
        <th>总费用</th>
        <th>支付方式</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let record of completedRecords">
        <td>{{ record.vehicle_type }}</td>
        <td>{{ record.rate | currency:'CNY':'symbol' }}</td>
        <td>{{ record.hours }} 小时</td>
        <td>{{ record.total_fee | currency:'CNY':'symbol' }}</td>
        <td>{{ record.payment_method }}</td>
      </tr>
      </tbody>
    </table>
    <p *ngIf="completedRecords.length === 0">暂无历史费用记录</p>
  </section>

  <!-- 车位统计 -->
  <section>
    <h2>车位利用率统计</h2>
    <div class="stats-container" *ngIf="parkingStats.total_spaces">
      <div class="stat-item">
        <span class="label">总车位数:</span>
        <span class="value">{{ parkingStats.total_spaces }}</span>
      </div>
      <div class="stat-item">
        <span class="label">已用车位:</span>
        <span class="value">{{ parkingStats.occupied }}</span>
      </div>
      <div class="stat-item">
        <span class="label">空闲车位:</span>
        <span class="value">{{ parkingStats.available }}</span>
      </div>
      <div class="stat-item highlight">
        <span class="label">利用率:</span>
        <span class="value">{{ parkingStats.usage_rate }}%</span>
      </div>
      <div class="stat-item highlight">
        <span class="label">空闲率:</span>
        <span class="value">{{ parkingStats.free_rate }}%</span>
      </div>
    </div>
    <p *ngIf="!parkingStats.total_spaces">加载统计数据中...</p>
  </section>
</div>

```

step7:C:\Users\Administrator\WebstormProjects\untitled4\src\app\park\park.component.css

```css
/* app.component.css */
.container {
  max-width: 1200px;
  margin: 20px auto;
  padding: 20px;
}

section {
  margin-bottom: 40px;
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

h2 {
  color: #2c3e50;
  border-bottom: 2px solid #3498db;
  padding-bottom: 10px;
  margin-bottom: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ecf0f1;
}

th {
  background-color: #3498db;
  color: white;
}

tr:hover {
  background-color: #f8f9fa;
}

.user-select {
  margin-bottom: 30px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 6px;
}

.user-select input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.stats-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 20px;
  margin-top: 20px;
}

.stat-item {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 6px;
  text-align: center;
}

.stat-item .label {
  display: block;
  color: #7f8c8d;
  margin-bottom: 5px;
}

.stat-item .value {
  font-size: 1.4em;
  color: #2c3e50;
  font-weight: 500;
}

.highlight .value {
  color: #3498db;
}

```

end