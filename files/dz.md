说明：我计划用fastapi+angular实现菜鸟驿站系统
userid  和stationid  暂时先写死 全部写成1   也就是用户1  驿站1   这样就可以简化流程


1.新增包裹入库   增加一个添加入库的按钮 然后填写信息  然后入库

2.新增包裹取件按钮  post     请求，弹窗填写取件码，  取件成功  需要刷新包裹状态


3.获取超时列表   比如有些包裹 严重超时  我需要查看超时包裹的信息 和位置


4.还需要取件记录表  用弹窗  每次用户取件 都必须有一条取件记录

5.还需要 新增包裹   比如入库  有一个新增按钮  可以插入包裹列表
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7d391e478edc47e7bd7966fecf3d3ccf.png#pic_center)

step1:sql

```sql
-- Users Table (Stores Recipient Information)
CREATE TABLE db_school.users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    phone VARCHAR(11) NOT NULL UNIQUE,   -- Recipient's phone number
    full_name VARCHAR(50) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Stations Table (Multiple Service Points)
CREATE TABLE db_school.stations (
    station_id INT PRIMARY KEY AUTO_INCREMENT,
    station_name VARCHAR(100) NOT NULL,  -- Station name
    address VARCHAR(255) NOT NULL,       -- Physical address
    manager_id INT,                      -- Manager ID (foreign key)
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (manager_id) REFERENCES staff(staff_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Staff Table (Station Operators)
CREATE TABLE db_school.staff (
    staff_id INT PRIMARY KEY AUTO_INCREMENT,
    station_id INT NOT NULL,             -- Associated station
    full_name VARCHAR(50) NOT NULL,
    role ENUM('admin', 'operator') NOT NULL DEFAULT 'operator',
    phone VARCHAR(11) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL, -- Password hash
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (station_id) REFERENCES stations(station_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Packages Table (Core Business Data)
CREATE TABLE db_school.packages (
    tracking_no VARCHAR(20) PRIMARY KEY,  -- Tracking number (PK)
    user_id INT NOT NULL,                 -- Recipient ID
    station_id INT NOT NULL,             -- Current station
    carrier VARCHAR(50) NOT NULL,        -- Logistics company
    status ENUM('pending', 'picked_up', 'returned') DEFAULT 'pending',
    check_in_time DATETIME NOT NULL,     -- Storage timestamp
    pickup_code CHAR(6) NOT NULL,        -- Pickup verification code
    expire_time DATETIME,                 -- Latest pickup deadline
    shelf_location VARCHAR(20),          -- Storage position (optional)
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (station_id) REFERENCES stations(station_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Pickup Records Table
CREATE TABLE db_school.pickup_records (
    record_id INT PRIMARY KEY AUTO_INCREMENT,
    tracking_no VARCHAR(20) NOT NULL,
    user_id INT NOT NULL,
    pickup_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    operator_id INT NOT NULL,            -- Operator ID
    FOREIGN KEY (tracking_no) REFERENCES packages(tracking_no),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (operator_id) REFERENCES staff(staff_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Notifications Table
CREATE TABLE db_school.notifications (
    notification_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    tracking_no VARCHAR(20) NOT NULL,
    content TEXT NOT NULL,               -- Notification message
    sent_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    is_read TINYINT DEFAULT 0,          -- 0=unread, 1=read
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (tracking_no) REFERENCES packages(tracking_no)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Sample Data Insertions
INSERT INTO users (phone, full_name) VALUES
('13800138001', 'Zhang San'),
('13800138002', 'Li Si'),
('13800138003', 'Wang Wu');

INSERT INTO stations (station_name, address) VALUES
('South Campus Station', '1 Keji Road'),
('North Campus Station', '88 Chuangxin Road');

INSERT INTO staff (station_id, full_name, role, phone, password_hash) VALUES
(1, 'Station Admin A', 'admin', '13900139001', '$2a$10$N7h/Zq'),
(1, 'Operator A1', 'operator', '13900139002', '$2a$10$N7h/Zq');

UPDATE stations SET manager_id = 1 WHERE station_id = 1;

INSERT INTO packages VALUES
('YT001', 1, 1, 'YTO', 'pending', NOW(), 'A1B2C3', NOW() + INTERVAL 7 DAY, 'A3'),
('STO002', 2, 1, 'STO', 'picked_up', NOW(), 'D4E5F6', NOW() + INTERVAL 7 DAY, 'B2');

INSERT INTO pickup_records (tracking_no, user_id, operator_id) VALUES
('STO002', 2, 2);

INSERT INTO notifications (user_id, tracking_no, content) VALUES
(1, 'YT001', 'Your package has arrived at South Campus Station. Pickup code: A1B2C3');
```

step2:fastapi

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pymysql import MySQLError
import pymysql.cursors
from datetime import datetime, timedelta
import secrets
from typing import Optional
from pydantic import BaseModel
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


# 数据库工具函数
def db_operation(query: str, params=None, fetch: bool = False):
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params or ())
            if fetch:
                result = cursor.fetchall()
            else:
                result = cursor.lastrowid
            connection.commit()
            return result
    except MySQLError as e:
        connection.rollback()
        raise HTTPException(status_code=400, detail=f"Database error: {e}")
    finally:
        connection.close()

# 新增请求体模型
class PackageCreate(BaseModel):
    tracking_no: str
    user_id: int
    station_id: int
    carrier: str
    shelf_location: Optional[str] = None

class PickupRequest(BaseModel):
    tracking_no: str
    pickup_code: str
    operator_id: int

# ---------- 基础接口 ----------
@app.post("/packages/")
async def create_package(package: PackageCreate):
    """包裹入库接口"""
    # 生成6位取件码
    pickup_code = f"{secrets.randbelow(999999):06d}"
    expire_time = datetime.now() + timedelta(days=3)

    query = """
    INSERT INTO packages 
    (tracking_no, user_id, station_id, carrier, check_in_time, 
     pickup_code, expire_time, shelf_location)
    VALUES (%s, %s, %s, %s, NOW(), %s, %s, %s)
    """
    params = (
        package.tracking_no,
        package.user_id,
        package.station_id,
        package.carrier,
        pickup_code,
        expire_time,
        package.shelf_location
    )

    try:
        db_operation(query, params)
        # 创建通知记录
        noti_query = """
        INSERT INTO notifications 
        (user_id, tracking_no, content)
        VALUES (%s, %s, %s)
        """
        content = f"您的包裹已到达驿站，取件码：{pickup_code}"
        db_operation(noti_query, (package.user_id, package.tracking_no, content))
        return {"code": pickup_code, "expire_time": expire_time}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))


@app.get("/users/{user_id}/packages")
async def get_user_packages(user_id: int, status: Optional[str] = None):
    """查询用户包裹"""
    base_query = """
    SELECT p.*, s.station_name, u.phone 
    FROM packages p
    JOIN stations s ON p.station_id = s.station_id
    JOIN users u ON p.user_id = u.user_id
    WHERE p.user_id = %s
    """
    params = [user_id]

    if status:
        base_query += " AND p.status = %s"
        params.append(status)

    data = db_operation(base_query, params, fetch=True)
    return {"data": data}


@app.post("/packages/pickup")
async def pickup_package(request: PickupRequest):
    """包裹取件接口"""
    # 验证取件码
    check_query = """
    SELECT user_id, station_id 
    FROM packages 
    WHERE tracking_no = %s AND pickup_code = %s AND status = 'pending'
    """
    package = db_operation(check_query, (request.tracking_no, request.pickup_code), fetch=True)
    if not package:
        raise HTTPException(status_code=404, detail="取件码无效或包裹已取走")

    # 更新包裹状态
    update_query = """
    UPDATE packages 
    SET status = 'picked_up' 
    WHERE tracking_no = %s
    """
    db_operation(update_query, (request.tracking_no,))

    # 创建取件记录
    record_query = """
    INSERT INTO pickup_records 
    (tracking_no, user_id, operator_id)
    VALUES (%s, %s, %s)
    """
    db_operation(record_query,
                 (request.tracking_no, package[0]['user_id'], request.operator_id))

    return {"message": "取件成功"}


# ---------- 管理接口 ----------
@app.get("/stations/{station_id}/daily-stats")
async def get_daily_stats(station_id: int):
    """驿站当日统计"""
    query = """
    SELECT 
        COUNT(CASE WHEN status = 'pending' THEN 1 END) AS pending_count,
        COUNT(CASE WHEN status = 'picked_up' THEN 1 END) AS picked_count,
        COUNT(CASE WHEN status = 'returned' THEN 1 END) AS returned_count
    FROM packages
    WHERE station_id = %s 
      AND DATE(check_in_time) = CURDATE()
    """
    data = db_operation(query, (station_id,), fetch=True)
    return data[0] if data else {}


@app.get("/stations/{station_id}/expired-packages")
async def get_expired_packages(station_id: int):
    """获取超期未取包裹"""
    query = """
    SELECT tracking_no, user_id, check_in_time 
    FROM packages
    WHERE station_id = %s 
      AND status = 'pending' 
      AND expire_time < NOW()
    """
    return {"data": db_operation(query, (station_id,), fetch=True)}


# ---------- 通知接口 ----------
@app.get("/users/{user_id}/notifications")
async def get_unread_notifications(user_id: int):
    """获取用户未读通知"""
    query = """
    SELECT * FROM notifications 
    WHERE user_id = %s AND is_read = 0
    ORDER BY sent_time DESC
    """
    return {"data": db_operation(query, (user_id,), fetch=True)}


@app.post("/notifications/{notification_id}/read")
async def mark_as_read(notification_id: int):
    """标记通知已读"""
    db_operation("""
    UPDATE notifications 
    SET is_read = 1 
    WHERE notification_id = %s
    """, (notification_id,))
    return {"message": "标记成功"}


# ---------- 取件记录接口 ----------
from typing import Optional


@app.get("/stations/{station_id}/detailed-pickup-records")
async def get_detailed_pickup_records(
        station_id: int,
        page: int = 1,
        size: int = 20
):
    """获取驿站详细取件记录（分页版）"""
    offset = (page - 1) * size

    query = """
    SELECT
        pr.tracking_no,
        pr.operator_id,
        pr.pickup_time,
        u.phone AS user_phone,
        p.carrier,
        p.station_id,
        p.shelf_location
    FROM pickup_records pr
    JOIN packages p ON pr.tracking_no = p.tracking_no
    JOIN users u ON pr.user_id = u.user_id
    WHERE p.station_id = %s
    ORDER BY pr.pickup_time DESC
    LIMIT %s OFFSET %s
    """

    try:
        # 获取分页数据
        records = db_operation(query, (station_id, size, offset), fetch=True)

        # 获取总数
        count_query = """
        SELECT COUNT(*) AS total 
        FROM pickup_records pr
        JOIN packages p ON pr.tracking_no = p.tracking_no
        WHERE p.station_id = %s
        """
        total = db_operation(count_query, (station_id,), fetch=True)[0]['total']

        return {
            "data": records,
            "pagination": {
                "total": total,
                "page": page,
                "size": size
            }
        }

    except MySQLError as e:
        raise HTTPException(
            status_code=500,
            detail=f"数据库查询失败: {e.args[1]}"
        )

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\services\station.service.ts

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class StationService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  getUserPackages(userId: number, status?: string): Observable<any> {
    const params: any = { user_id: userId };
    if (status) params.status = status;
    return this.http.get(`${this.apiUrl}/users/${userId}/packages`, { params });
  }

  createPackage(
    tracking_no: string,       // ✅ 统一使用下划线
    user_id: number,            // ✅
    station_id: number,        // ✅
    carrier: string,
    shelf_location?: string
  ) {
    return this.http.post(`${this.apiUrl}/packages/`, {
      tracking_no,             // ✅ ES6 简写语法
      user_id,
      station_id,
      carrier,
      shelf_location
    });
  }


  pickupPackage(tracking_no: string, pickup_code: string, operator_id: number) {
    console.log("tracking_no",tracking_no);
    console.log("pickup_code",pickup_code);
    console.log("operator_id",operator_id);
    return this.http.post(`${this.apiUrl}/packages/pickup`, {
      tracking_no,
      pickup_code,
      operator_id
    }, {
      headers: { 'Content-Type': 'application/json' }  // ✅ 明确指定请求头
    });
  }

  // 新增方法
  getExpiredPackages(stationId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/stations/${stationId}/expired-packages`);
  }


  getDetailedPickupRecords(stationId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/stations/${stationId}/detailed-pickup-records`);
  }

}

```

step4:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\station-list\status-translate.pipe.ts

```typescript
// status-translate.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'statusTranslate'
})
export class StatusTranslatePipe implements PipeTransform {
  transform(value: string): string {
    const statusMap: { [key: string]: string } = {
      pending: '待取件',
      picked_up: '已取件',
      returned: '已退回'
    };
    return statusMap[value] || value;
  }
}

```

step5:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\station-list\station.model.ts

```typescript
// package.model.ts
export interface Station {
  tracking_no: string;
  carrier: string;
  status: 'pending' | 'picked_up' | 'returned';
  pickup_code: string;
  shelf_location: string;
  check_in_time: string;
  station_name: string;
  phone: string;
}

```

step6:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\station-list\station-list.component.ts

```typescript
import { Component, OnInit } from '@angular/core';
import { Station } from './station.model';
import { StationService } from '../services/station.service';
import {DatePipe, NgClass, NgForOf, NgIf} from '@angular/common';
import {FormsModule} from '@angular/forms';
import {StatusTranslatePipe} from './status-translate.pipe';


@Component({
  selector: 'app-package',
  templateUrl: './station-list.component.html',
  imports: [
    NgForOf,
    NgClass,
    FormsModule,
    StatusTranslatePipe,
    DatePipe,
    NgIf
  ],
  styleUrls: ['./station-list.component.css']
})
export class StationComponent implements OnInit {
  packages: Station[] = [];
  selectedStatus = '';
  currentUserId = 1; // 从认证服务获取实际用户ID

  // 新增表单相关属性
  showAddForm = false;
  newPackage = {
    tracking_no: '',
    user_id: 1, // 默认当前用户
    station_id: 1, // 默认驿站ID
    carrier: '',
    shelf_location: ''
  };


  // 新增取件相关状态
  showPickupModal = false;
  currentPickupTrackingNo = '';
  pickupCodeInput = '';
  operatorId = 1; // 假设当前操作员ID为1

  // 新增状态
  showExpiredModal = false;
  expiredPackages: any[] = [];
  currentStationId = 1; // 默认当前驿站ID

  // 新增状态
  showPickupRecordsModal = false;
  pickupRecords: any[] = [];


  constructor(private stationService: StationService) {}

  // 新增表单方法
  openAddForm() {
    this.showAddForm = true;
    this.resetForm();
  }

  ngOnInit() {
    this.loadPackages();
  }

  loadPackages() {
    this.stationService.getUserPackages(this.currentUserId, this.selectedStatus)
      .subscribe((res: any) => {
        this.packages = res.data;
      });
  }

  onStatusChange() {
    this.loadPackages();
  }

  // 新增关闭弹窗方法
  closeAddForm() {
    this.showAddForm = false;
    this.resetForm();
  }

 submitPackage() {
    this.stationService.createPackage(
      this.newPackage.tracking_no,
      this.newPackage.user_id,
      this.newPackage.station_id,
      this.newPackage.carrier,
      this.newPackage.shelf_location
    ).subscribe({
      next: (res: any) => {
        alert(`入库成功！\n取件码：${res.code}\n过期时间：${res.expire_time}`);
        this.closeAddForm();
        this.loadPackages();
      },
      error: (err) => {
        alert(`入库失败：${err.error.detail}`);
      }
    });
  }


  private resetForm() {
    this.newPackage = {
      tracking_no: '',
      user_id: 1,
      station_id: 1,
      carrier: '',
      shelf_location: ''
    };
  }

  // 打开取件弹窗
  openPickupModal(trackingNo: string) {
    this.currentPickupTrackingNo = trackingNo;
    this.showPickupModal = true;
    this.pickupCodeInput = ''; // 清空上次输入
  }

  // 提交取件
  confirmPickup() {
    this.stationService.pickupPackage(
      this.currentPickupTrackingNo,
      this.pickupCodeInput,
      this.operatorId
    ).subscribe({
      next: () => {
        alert('取件成功！');
        this.showPickupModal = false;
        this.loadPackages(); // 刷新包裹列表
        this.loadPickupRecords(); // 刷新取件记录
      },
      error: (err) => {
        const msg = err.status === 404 ? '取件码错误或包裹已取走' : '取件失败';
        alert(msg);
      }
    });
  }

  // 打开超期包裹弹窗
  openExpiredModal() {
    this.loadExpiredPackages();
    this.showExpiredModal = true;
  }

  // 加载超期包裹
  loadExpiredPackages() {
    this.stationService.getExpiredPackages(this.currentStationId)
      .subscribe((res: any) => {
        this.expiredPackages = res.data;
      });
  }

  // 关闭弹窗
  closeExpiredModal() {
    this.showExpiredModal = false;
  }

  // 打开取件记录弹窗
  openPickupRecords() {
    this.loadPickupRecords();
    this.showPickupRecordsModal = true;
  }

  // station-list.component.ts
  loadPickupRecords() {
    this.stationService.getDetailedPickupRecords(1).subscribe({
      next: (res) => this.pickupRecords = res.data,
      error: (err) => console.error('获取记录失败', err)
    });
  }



}

```

step7:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\station-list\station-list.component.html

```xml
<div class="container">
  <!-- 新增入库按钮 -->
  <div class="header-section">
    <h2>包裹管理</h2>
    <button class="btn-info" (click)="openAddForm()">+ 新增入库</button>
    <button class="btn-info" (click)="openExpiredModal()">查看超期包裹</button>
    <button class="btn-info" (click)="openPickupRecords()">取件记录</button>
  </div>
  <div class="filter-section">
    <select [(ngModel)]="selectedStatus" (change)="onStatusChange()">
      <option value="">全部状态</option>
      <option value="pending">待取件</option>
      <option value="picked_up">已取件</option>
      <option value="returned">已退回</option>
    </select>
  </div>

  <div class="package-list">
    <h2>我的包裹</h2>
    <table>
      <thead>
      <tr>
        <th>快递单号</th>
        <th>物流公司</th>
        <th>状态</th>
        <th>取件码</th>
        <th>货架位置</th>
        <th>入库时间</th>
        <th>操作</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let pkg of packages">
        <td>{{ pkg.tracking_no }}</td>
        <td>{{ pkg.carrier }}</td>

        <td [ngClass]="{
            'status-pending': pkg.status === 'pending',
            'status-picked': pkg.status === 'picked_up',
            'status-returned': pkg.status === 'returned'
          }">{{ pkg.status | statusTranslate }}</td>
        <td>{{ pkg.pickup_code }}</td>
        <td>{{ pkg.shelf_location }}</td>
        <td>{{ pkg.check_in_time | date: 'yyyy-MM-dd HH:mm' }}</td>

        <td>
          <button *ngIf="pkg.status === 'pending'"
                  (click)="openPickupModal(pkg.tracking_no)">
            取件
          </button>
        </td>
      </tr>
      </tbody>
    </table>
  </div>

  <!-- 弹窗背景层 -->
  <div *ngIf="showAddForm" class="modal-backdrop">
    <div class="comment-modal">
      <h3>包裹入库 <span class="required-tip">（* 为必填项）</span></h3>

      <!-- 表单内容 -->
      <div class="form-content">
        <div class="form-row">
          <label>快递单号 *</label>
          <input type="text" [(ngModel)]="newPackage.tracking_no" name="trackingNo" required>
        </div>

        <div class="form-row">
          <label>物流公司 *</label>
          <input type="text" [(ngModel)]="newPackage.carrier" name="carrier" required>
        </div>

        <div class="form-row">
          <label>驿站编号 *</label>
          <input type="number" [(ngModel)]="newPackage.station_id" name="stationId" required>
        </div>

        <div class="form-row">
          <label>货架位置</label>
          <input type="text" [(ngModel)]="newPackage.shelf_location" name="shelfLocation">
        </div>
      </div>

      <!-- 操作按钮 -->
      <div class="modal-actions">
        <button class="btn-cancel" (click)="closeAddForm()">取消</button>
        <button class="btn-submit" (click)="submitPackage()">提交入库</button>
      </div>
    </div>
  </div>


  <!-- 取件弹窗 -->
  <div *ngIf="showPickupModal" class="modal-backdrop">
    <div class="comment-modal">
      <h3>包裹取件 (单号: {{ currentPickupTrackingNo }})</h3>

      <div class="form-content">
        <div class="form-row">
          <label>取件码 *</label>
          <input type="text" [(ngModel)]="pickupCodeInput" required>
        </div>
      </div>

      <div class="modal-actions">
        <button class="btn-cancel" (click)="showPickupModal = false">取消</button>
        <button class="btn-submit"
                [disabled]="!pickupCodeInput"
                (click)="confirmPickup()">
          确认取件
        </button>
      </div>
    </div>
  </div>

  <!-- 超期包裹弹窗 -->
  <div *ngIf="showExpiredModal" class="modal-backdrop">
    <div class="comment-modal">
      <h3>超期未取包裹 <span class="tip">(驿站ID: {{ currentStationId }})</span></h3>

      <table class="expired-table">
        <thead>
        <tr>
          <th>快递单号</th>
          <th>物流公司</th>
          <th>货架位置</th>
          <th>入库时间</th>
          <th>过期时间</th>
        </tr>
        </thead>
        <tbody>
        <tr *ngFor="let pkg of expiredPackages">
          <td>{{ pkg.tracking_no }}</td>
          <td>{{ pkg.carrier }}</td>
          <td class="warning">{{ pkg.shelf_location || '未指定' }}</td>
          <td>{{ pkg.check_in_time | date: 'yyyy-MM-dd HH:mm' }}</td>
          <td class="warning">{{ pkg.expire_time | date: 'yyyy-MM-dd HH:mm' }}</td>
        </tr>
        </tbody>
      </table>

      <div class="modal-actions">
        <button class="btn-cancel" (click)="closeExpiredModal()">关闭</button>
      </div>
    </div>
  </div>

  <!-- 取件记录弹窗 -->
  <div *ngIf="showPickupRecordsModal" class="modal-backdrop">
    <div class="comment-modals" style="max-width: 1000px;">
      <h3>取件记录</h3>

      <table class="record-table">
        <thead>
        <tr>
          <th>快递单号</th>
          <th>操作员ID</th>
          <th>取件时间</th>
          <!-- 新增字段 -->
          <th>用户电话</th>
          <th>物流公司</th>
          <th>驿站ID</th>
          <th>货架位置</th>
        </tr>
        </thead>
        <tbody>
        <tr *ngFor="let record of pickupRecords">
          <td>{{ record.tracking_no }}</td>
          <td>{{ record.operator_id }}</td>
          <td>{{ record.pickup_time | date: 'yyyy-MM-dd HH:mm' }}</td>
          <!-- 新增数据绑定 -->
          <td>{{ record.user_phone }}</td>
          <td>{{ record.carrier }}</td>
          <td>{{ record.station_id }}</td>
          <td>{{ record.shelf_location || '未指定' }}</td>
        </tr>
        </tbody>
      </table>

      <div class="modal-actions">
        <button class="btn-cancel" (click)="showPickupRecordsModal = false">关闭</button>
      </div>
    </div>
  </div>
</div>




```

step8:C:\Users\wangrusheng\WebstormProjects\untitled4\src\app\station-list\station-list.component.css

```css
.container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

.filter-section {
  margin-bottom: 20px;
}

select {
  padding: 8px 12px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.package-list table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #f5f5f5;
}

.status-pending {
  color: #ff9800;
  font-weight: 500;
}

.status-picked {
  color: #4caf50;
}

.status-returned {
  color: #9e9e9e;
}


/* 弹窗样式 */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.comment-modal {
  background: white;
  border-radius: 8px;
  padding: 20px;
  width: 500px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

/* 修改弹窗容器样式 */
.comment-modals {
  background: white;
  padding: 20px;
  width: 95vw;          /* 占据视口宽度的95% */
  max-width: 1200px;     /* 最大宽度限制 */
  min-width: 800px;     /* 最小宽度保证 */
  overflow-x: auto;     /* 允许横向滚动 */
}

/* 表格容器添加滚动 */
.record-table-container {
  max-height: 70vh;     /* 限制高度 */
  overflow-x: auto;     /* 横向滚动 */
}


.comment-modal h3 {
  margin: 0 0 20px 0;
  color: #333;
  border-bottom: 1px solid #eee;
  padding-bottom: 10px;
}

.required-tip {
  color: #666;
  font-size: 0.9em;
}

.form-content {
  margin-bottom: 20px;
}

.form-row {
  margin-bottom: 15px;
}

.form-row label {
  display: block;
  margin-bottom: 5px;
  color: #444;
}

.form-row input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.btn-submit, .btn-cancel {
  padding: 8px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-submit {
  background: #1890ff;
  color: white;
}

.btn-cancel {
  background: #f5222d;
  color: white;
}

.btn-submit:hover {
  background: #40a9ff;
}

.btn-cancel:hover {
  background: #ff4d4f;
}

/* 操作按钮容器 */
.action-buttons {
  display: flex;
  gap: 10px;
}

/* 警告按钮样式 */
.btn-warning {
  background: #ff4d4f;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 3px;
}

/* 超期表格样式 */
.expired-table {
  width: 100%;
  margin: 15px 0;
  border-collapse: collapse;
}

.expired-table th,
.expired-table td {
  padding: 8px;
  border: 1px solid #f0f0f0;
}

.expired-table th {
  background: #fafafa;
}

.warning {
  color: #ff4d4f;
  font-weight: 500;
}


/* 信息按钮样式 */
.btn-info {
  background: #1890ff;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
}

/* 记录表格样式 */
.record-table {
  width: 100%;
  margin: 15px 0;
  border-collapse: collapse;
}

.record-table th,
.record-table td {
  padding: 8px;
  border: 1px solid #f0f0f0;
  text-align: left;
}

.record-table th {
  background: #fafafa;
}

/* 调整表格列宽和文本对齐 */
.record-table {
  width: 100%;
  table-layout: auto; /* 自动列宽 */
}

.record-table th,
.record-table td {
  padding: 8px 12px;
  border-bottom: 1px solid #f0f0f0;
  text-align: left; /* 左对齐更易阅读 */
}

.record-table th:nth-child(1) { width: 15%; }  /* 快递单号 */
.record-table th:nth-child(2) { width: 10%; }  /* 操作员ID */
.record-table th:nth-child(3) { width: 15%; }  /* 取件时间 */
.record-table th:nth-child(4) { width: 15%; }  /* 用户电话 */
.record-table th:nth-child(5) { width: 15%; }  /* 物流公司 */
.record-table th:nth-child(6) { width: 10%; }  /* 驿站ID */
.record-table th:nth-child(7) { width: 10%; }  /* 货架位置 */

```

 

end