说明：
宠物列表  

宠物的种类、年龄、健康状况、是否已接种疫苗 宠物标签


领养申请


领养记录 

 疫苗记录

健康检查记录


宠物评论表
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2bf6089ce57146a49cd24e3080deb730.png#pic_center)

step0:mysql

```sql

-- 宠物品种表（字典表）
CREATE TABLE pet_breeds (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  species VARCHAR(50) NOT NULL COMMENT '物种（猫/狗等）',
  breed_name VARCHAR(100) NOT NULL COMMENT '品种名称',
  UNIQUE INDEX idx_species_breed (species, breed_name)
);

-- 宠物基本信息表
CREATE TABLE pets (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  breed_id INT UNSIGNED NOT NULL COMMENT '品种ID',
  age_months SMALLINT UNSIGNED COMMENT '年龄（月）',
  gender ENUM('male', 'female', 'unknown') DEFAULT 'unknown',
  health_status ENUM('excellent', 'good', 'fair', 'poor') NOT NULL DEFAULT 'good',
  is_vaccinated TINYINT(1) DEFAULT 0 COMMENT '是否已接种疫苗',
  adoption_status ENUM('available', 'pending', 'adopted') DEFAULT 'available',
  description TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (breed_id) REFERENCES pet_breeds(id)
);

-- 用户表（领养人/评论用户）
CREATE TABLE users (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  phone VARCHAR(20),
  address TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);


select *from adoption_applications;

-- 领养申请表
CREATE TABLE adoption_applications (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  pet_id INT UNSIGNED NOT NULL,
  applicant_id INT UNSIGNED NOT NULL,
  application_date DATETIME DEFAULT CURRENT_TIMESTAMP,
  status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
  notes TEXT,
  FOREIGN KEY (pet_id) REFERENCES pets(id),
  FOREIGN KEY (applicant_id) REFERENCES users(id),
  INDEX idx_status (status)
);

-- 领养记录表
CREATE TABLE adoption_records (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  pet_id INT UNSIGNED UNIQUE NOT NULL COMMENT '确保一宠只能被领养一次',
  adopter_id INT UNSIGNED NOT NULL,
  adoption_date DATETIME NOT NULL,
  contract_number VARCHAR(50),
  FOREIGN KEY (pet_id) REFERENCES pets(id),
  FOREIGN KEY (adopter_id) REFERENCES users(id)
);

-- 疫苗记录表
CREATE TABLE vaccination_records (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  pet_id INT UNSIGNED NOT NULL,
  vaccine_type VARCHAR(100) NOT NULL,
  vaccination_date DATE NOT NULL,
  next_due_date DATE,
  veterinarian VARCHAR(100),
  notes TEXT,
  FOREIGN KEY (pet_id) REFERENCES pets(id),
  INDEX idx_vaccine_date (vaccination_date)
);

-- 健康检查表
CREATE TABLE health_checks (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  pet_id INT UNSIGNED NOT NULL,
  check_date DATETIME DEFAULT CURRENT_TIMESTAMP,
  veterinarian VARCHAR(100),
  weight_kg DECIMAL(5,2),
  checkup_notes TEXT,
  diagnosis VARCHAR(200),
  treatment TEXT,
  FOREIGN KEY (pet_id) REFERENCES pets(id),
  INDEX idx_check_date (check_date)
);

select *from pet_comments;

-- 宠物评论表
CREATE TABLE pet_comments (
  id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  pet_id INT UNSIGNED NOT NULL,
  user_id INT UNSIGNED NOT NULL,
  comment TEXT NOT NULL,
  rating TINYINT UNSIGNED COMMENT '评分（1-5）',
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (pet_id) REFERENCES pets(id),
  FOREIGN KEY (user_id) REFERENCES users(id),
  INDEX idx_rating (rating)
);

-- Insert pet breeds data (12 entries)
INSERT INTO pet_breeds (species, breed_name) VALUES
('Cat', 'Persian Cat'), ('Cat', 'Siamese Cat'), ('Dog', 'Labrador'), ('Dog', 'Siberian Husky'),
('Cat', 'Ragdoll'), ('Dog', 'Golden Retriever'), ('Rabbit', 'Holland Lop'), ('Cat', 'American Shorthair'),
('Dog', 'Poodle'), ('Bird', 'Budgerigar'), ('Hamster', 'Golden Hamster'), ('Dog', 'Border Collie');

-- Insert pets data (10 entries)
INSERT INTO pets (name, breed_id, age_months, gender, health_status, is_vaccinated, adoption_status) VALUES
('Xiaobai', 1, 12, 'male', 'excellent', 1, 'available'),
('Xiaohei', 3, 24, 'female', 'good', 1, 'adopted'),
('Buding', 5, 6, 'female', 'fair', 0, 'pending'),
('Doudou', 7, 3, 'unknown', 'good', 0, 'available'),
('Lele', 9, 18, 'male', 'excellent', 1, 'available'),
('Qiuqiu', 11, 2, 'male', 'poor', 0, 'available'),
('Niuniu', 2, 36, 'female', 'good', 1, 'adopted'),
('Dahuang', 4, 48, 'male', 'fair', 1, 'available'),
('Xueqiu', 6, 12, 'female', 'excellent', 1, 'pending'),
('Shandian', 12, 9, 'male', 'good', 0, 'available');

-- Insert users data (10 entries)
INSERT INTO users (username, email, phone, address) VALUES
('user1', 'user1@example.com', '13800000001', 'Chaoyang District, Beijing'),
('user2', 'user2@example.com', '13800000002', 'Pudong New Area, Shanghai'),
('user3', 'user3@example.com', '13800000003', 'Tianhe District, Guangzhou'),
('user4', 'user4@example.com', '13800000004', 'Nanshan District, Shenzhen'),
('user5', 'user5@example.com', '13800000005', 'Jinjiang District, Chengdu'),
('user6', 'user6@example.com', '13800000006', 'Xihu District, Hangzhou'),
('user7', 'user7@example.com', '13800000007', 'Jianghan District, Wuhan'),
('user8', 'user8@example.com', '13800000008', 'Gulou District, Nanjing'),
('user9', 'user9@example.com', '13800000009', 'Yanta District, Xi''an'),
('user10', 'user10@example.com', '13800000010', 'Yuzhong District, Chongqing');

-- Insert adoption applications data (10 entries)
INSERT INTO adoption_applications (pet_id, applicant_id, status) VALUES
(1, 3, 'pending'), (3, 5, 'approved'), (5, 2, 'rejected'),
(7, 8, 'approved'), (9, 4, 'pending'), (2, 6, 'approved'),
(4, 1, 'rejected'), (6, 7, 'pending'), (8, 9, 'approved'),
(10, 10, 'pending');

-- Insert adoption records data (10 entries)
INSERT INTO adoption_records (pet_id, adopter_id, adoption_date, contract_number) VALUES
(2, 6, '2023-01-15', 'CN2023011501'),
(7, 8, '2023-02-20', 'CN2023022001'),
(9, 4, '2023-03-10', 'CN2023031001'),
(4, 1, '2023-04-05', 'CN2023040501'),
(6, 7, '2023-05-12', 'CN2023051201'),
(8, 9, '2023-06-18', 'CN2023061801'),
(10, 10, '2023-07-01', 'CN2023070101'),
(1, 3, '2023-08-09', 'CN2023080901'),
(3, 5, '2023-09-22', 'CN2023092201'),
(5, 2, '2023-10-30', 'CN2023103001');

-- Insert vaccination records data (10 entries)
INSERT INTO vaccination_records (pet_id, vaccine_type, vaccination_date, next_due_date) VALUES
(1, 'Rabies Vaccine', '2023-01-05', '2024-01-05'),
(2, 'Quadrivalent Vaccine', '2023-02-10', '2024-02-10'),
(3, 'Feline Triple Vaccine', '2023-03-15', '2024-03-15'),
(4, 'Rabbit Hemorrhagic Disease Vaccine', '2023-04-20', '2024-04-20'),
(5, 'Rabies Vaccine', '2023-05-25', '2024-05-25'),
(6, 'Hamster Coccidiosis Vaccine', '2023-06-30', '2024-06-30'),
(7, 'Canine Hexavalent Vaccine', '2023-07-05', '2024-07-05'),
(8, 'Rabies Vaccine', '2023-08-10', '2024-08-10'),
(9, 'Feline Leukemia Vaccine', '2023-09-15', '2024-09-15'),
(10, 'Avian Pox Vaccine', '2023-10-20', '2024-10-20');

-- Insert health check data (10 entries)
INSERT INTO health_checks (pet_id, weight_kg, checkup_notes) VALUES
(1, 4.2, 'Healthy posture, clean teeth'),
(2, 28.5, 'Good joint development'),
(3, 3.8, 'Requires fur care attention'),
(4, 0.5, 'Normal body temperature'),
(5, 5.1, 'Complete vaccination status'),
(6, 0.3, 'Normal appetite'),
(7, 22.0, 'Needs diet control for weight management'),
(8, 15.5, 'No skin abnormalities found'),
(9, 4.5, 'Clean ear canals'),
(10, 0.2, 'Good feather glossiness');

-- Insert pet comments data (10 entries)
INSERT INTO pet_comments (pet_id, user_id, comment, rating) VALUES
(1, 3, 'A very lively and adorable kitten', 5),
(2, 6, 'Adapted very well after adoption', 4),
(3, 5, 'Gentle and affectionate personality', 5),
(4, 1, 'Requires more patient care', 3),
(5, 2, 'Excellent health condition', 5),
(6, 7, 'Great companion for children', 4),
(7, 8, 'Already part of our family', 5),
(8, 9, 'Loves outdoor activities', 4),
(9, 4, 'Regular grooming is important', 4),
(10, 10, 'Beautiful singing voice', 5);
```

step1:fastapi

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from datetime import date
import pymysql.cursors
from enum import Enum
from fastapi.middleware.cors import CORSMiddleware
app = FastAPI()

# 添加CORS配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:4200"],  # 允许访问的前端域名
    allow_credentials=True,
    allow_methods=["*"],  # 允许所有HTTP方法
    allow_headers=["*"],  # 允许所有请求头
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


# 公共数据库操作方法
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
class AdoptionApplicationCreate(BaseModel):
    pet_id: int
    applicant_id: int
    notes: str = None


class VaccinationRecordCreate(BaseModel):
    pet_id: int
    vaccine_type: str
    vaccination_date: date
    next_due_date: date = None
    veterinarian: str = None
    notes: str = None

class AdoptionStatus(str, Enum):
    available = "available"
    pending = "pending"
    adopted = "adopted"

@app.get("/pets")
async def get_pets(
        species: str = None,
        breed: str = None,
        min_age: int = None,
        max_age: int = None,
        health_status: str = None,
        is_vaccinated: bool = None,
        adoption_status: AdoptionStatus = None  # 使用枚举类型
):
    """获取宠物列表"""
    base_query = """
    SELECT p.*, b.species, b.breed_name 
    FROM pets p
    JOIN pet_breeds b ON p.breed_id = b.id
    WHERE 1=1
    """
    params = []

    # 领养状态筛选（新增）
    if adoption_status:
        base_query += " AND p.adoption_status = %s"
        params.append(adoption_status.value)  # 使用枚举值

    if species:
        base_query += " AND b.species = %s"
        params.append(species)
    if breed:
        base_query += " AND b.breed_name = %s"
        params.append(breed)
    if min_age is not None:
        base_query += " AND p.age_months >= %s"
        params.append(min_age)
    if max_age is not None:
        base_query += " AND p.age_months <= %s"
        params.append(max_age)
    if health_status:
        base_query += " AND p.health_status = %s"
        params.append(health_status)
    if is_vaccinated is not None:
        base_query += " AND p.is_vaccinated = %s"
        params.append(int(is_vaccinated))
    # 调试日志
    print("[DEBUG] Query:", base_query)
    print("[DEBUG] Params:", params)
    return {"data": db_execute(base_query, params)}

@app.post("/adoption-applications")
async def create_adoption_application(app_data: AdoptionApplicationCreate):
    """提交领养申请"""
    # 检查宠物是否存在
    pet_exists = db_execute("SELECT id FROM pets WHERE id = %s", (app_data.pet_id,))
    if not pet_exists:
        raise HTTPException(status_code=404, detail="Pet not found")

    # 检查用户是否存在
    user_exists = db_execute("SELECT id FROM users WHERE id = %s", (app_data.applicant_id,))
    if not user_exists:
        raise HTTPException(status_code=404, detail="User not found")

    query = """
    INSERT INTO adoption_applications 
    (pet_id, applicant_id, application_date, status, notes)
    VALUES (%s, %s, NOW(), 'pending', %s)
    """
    db_execute(query, (app_data.pet_id, app_data.applicant_id, app_data.notes), fetch=False)
    return {"message": "Application submitted successfully"}


@app.get("/vaccination-records/{pet_id}")
async def get_vaccination_records(pet_id: int):
    """获取疫苗记录"""
    pet_exists = db_execute("SELECT id FROM pets WHERE id = %s", (pet_id,))
    if not pet_exists:
        raise HTTPException(status_code=404, detail="Pet not found")

    records = db_execute("SELECT * FROM vaccination_records WHERE pet_id = %s", (pet_id,))
    return {"data": records}


@app.post("/vaccination-records")
async def create_vaccination_record(record: VaccinationRecordCreate):
    """添加疫苗记录"""
    pet_exists = db_execute("SELECT id FROM pets WHERE id = %s", (record.pet_id,))
    if not pet_exists:
        raise HTTPException(status_code=404, detail="Pet not found")

    query = """
    INSERT INTO vaccination_records 
    (pet_id, vaccine_type, vaccination_date, next_due_date, veterinarian, notes)
    VALUES (%s, %s, %s, %s, %s, %s)
    """
    params = (
        record.pet_id, record.vaccine_type, record.vaccination_date,
        record.next_due_date, record.veterinarian, record.notes
    )
    db_execute(query, params, fetch=False)

    # 更新宠物疫苗接种状态
    db_execute("UPDATE pets SET is_vaccinated = 1 WHERE id = %s", (record.pet_id,), fetch=False)
    return {"message": "Vaccination record added successfully"}


@app.get("/adoption-records")
async def get_adoption_records():
    """获取所有领养记录"""
    query = """
    SELECT a.*, p.name as pet_name, u.username as adopter_name 
    FROM adoption_records a
    JOIN pets p ON a.pet_id = p.id
    JOIN users u ON a.adopter_id = u.id
    """
    return {"data": db_execute(query)}


@app.get("/health-checks/{pet_id}")
async def get_health_checks(pet_id: int):
    """获取健康检查记录"""
    pet_exists = db_execute("SELECT id FROM pets WHERE id = %s", (pet_id,))
    if not pet_exists:
        raise HTTPException(status_code=404, detail="Pet not found")

    records = db_execute("SELECT * FROM health_checks WHERE pet_id = %s", (pet_id,))
    return {"data": records}


@app.post("/comments")
async def create_comment(comment: dict):
    """提交宠物评论"""
    required_fields = ["pet_id", "user_id", "comment"]
    if not all(field in comment for field in required_fields):
        raise HTTPException(status_code=400, detail="Missing required fields")

    query = """
    INSERT INTO pet_comments 
    (pet_id, user_id, comment, rating, created_at)
    VALUES (%s, %s, %s, %s, NOW())
    """
    params = (
        comment["pet_id"],
        comment["user_id"],
        comment["comment"],
        comment.get("rating")
    )
    db_execute(query, params, fetch=False)
    return {"message": "Comment added successfully"}


# 在现有代码中添加以下内容

@app.get("/pet-comments/{pet_id}")
async def get_pet_comments(pet_id: int):
    """获取指定宠物的评论"""
    # 检查宠物是否存在
    pet_exists = db_execute("SELECT id FROM pets WHERE id = %s", (pet_id,))
    if not pet_exists:
        raise HTTPException(status_code=404, detail="Pet not found")

    # 获取评论数据并关联用户信息
    query = """
    SELECT 
        pc.id,
        pc.comment,
        pc.rating,
        pc.created_at,
        u.username as commenter
    FROM pet_comments pc
    JOIN users u ON pc.user_id = u.id
    WHERE pc.pet_id = %s
    ORDER BY pc.created_at DESC
    """
    comments = db_execute(query, (pet_id,))

    return {"data": comments}

if __name__ == "__main__":
    import uvicorn

    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step2:postman

```bash
 http://localhost:8000/pets     """获取宠物列表"""

 {
    "data": [
        {
            "id": 1,
            "name": "Xiaobai",
            "breed_id": 1,
            "age_months": 12,
            "gender": "male",
            "health_status": "excellent",
            "is_vaccinated": 1,
            "adoption_status": "available",
            "description": null,
            "created_at": "2025-03-18T00:25:45",
            "updated_at": "2025-03-18T00:25:45",
            "species": "Cat",
            "breed_name": "Persian Cat"
        },
        // ... (other pets follow the same pattern)
    ]
}


  """提交领养申请"""  

  http://localhost:8000/adoption-applications

请求体body：
{
  "pet_id": 1,
  "applicant_id": 1,
  "notes": "Would like to adopt this lovely dog"
}

**Response:**  
 
{
    "message": "Application submitted successfully"
}


 """获取疫苗记录"""

 http://localhost:8000/vaccination-records/1

{
    "data": [
        {
            "id": 1,
            "pet_id": 1,
            "vaccine_type": "Rabies Vaccine",
            "vaccination_date": "2023-01-05",
            "next_due_date": "2024-01-05",
            "veterinarian": null,
            "notes": null
        }
    ]
}


"""添加疫苗记录"""

http://localhost:8000/vaccination-records 

{
  "pet_id": 1,
  "vaccine_type": "FIV Vaccine",
  "vaccination_date": "2023-05-15",
  "next_due_date": "2024-05-15",
  "veterinarian": "Dr. Zhang"
}

**Response:**  
 
{
    "message": "Vaccination record added successfully"
}




    """获取所有领养记录"""


    http://localhost:8000/adoption-records

 {
    "data": [
        {
            "id": 1,
            "pet_id": 2,
            "adopter_id": 6,
            "adoption_date": "2023-01-15T00:00:00",
            "contract_number": "CN2023011501",
            "pet_name": "Xiaohei",
            "adopter_name": "user6"
        },
        // ... (other records follow the same pattern)
    ]
}

    """获取健康检查记录"""


    http://localhost:8000/health-checks/1

  {
    "data": [
        {
            "id": 1,
            "pet_id": 1,
            "check_date": "2025-03-18T00:26:02",
            "veterinarian": null,
            "weight_kg": 4.2,
            "checkup_notes": "Healthy posture, clean teeth",
            "diagnosis": null,
            "treatment": null
        }
    ]
}


    """提交宠物评论"""

http://localhost:8000/comments

{
  "pet_id": 1,
  "user_id": 1,
  "comment": "Very gentle and lovely pet",
  "rating": 5
}

**Response:**  
 
{
    "message": "Comment added successfully"
}

 


获取指定宠物的评论
get请求
http://localhost:8000/pet-comments/1

{
    "data": [
        {
            "id": 11,
            "comment": "Very gentle and lovely pet",
            "rating": 5,
            "created_at": "2025-03-18T00:41:32",
            "commenter": "user1"
        },
        {
            "id": 1,
            "comment": "Very lively and adorable kitten",
            "rating": 5,
            "created_at": "2025-03-18T00:26:05",
            "commenter": "user3"
        }
    ]
}
```

step3:service

```typescript
// pet.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class PetService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }


  // 在PetService中添加/修改getPets方法
  getPets(filters?: any): Observable<any> {
    // 转换参数格式适配Python后端
    const params: any = {};

    if (filters) {
      // 字段名转换映射
      const fieldMapping: { [key: string]: string } = {
        species: 'species',
        breed: 'breed',
        minAge: 'min_age',
        maxAge: 'max_age',
        healthStatus: 'health_status',
        isVaccinated: 'is_vaccinated',
        adoptionStatus: 'adoption_status'  // 新增映射
      };

      Object.entries(filters).forEach(([key, value]) => {
        if (value !== null && value !== '') {
          const paramName = fieldMapping[key] || key;
          params[paramName] = value;
        }
      });
    }

    return this.http.get(`${this.apiUrl}/pets`, { params });
  }


  // 提交领养申请
  submitAdoption(data: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/adoption-applications`, data);
  }

  // 获取疫苗记录
  getVaccinations(petId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/vaccination-records/${petId}`);
  }

  // 添加疫苗记录
  addVaccination(data: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/vaccination-records`, data);
  }

  // 获取领养记录
  getAdoptionRecords(): Observable<any> {
    return this.http.get(`${this.apiUrl}/adoption-records`);
  }

  // 获取健康检查记录
  getHealthChecks(petId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/health-checks/${petId}`);
  }

  // 提交评论
  submitComment(data: any): Observable<any> {
    return this.http.post(`${this.apiUrl}/comments`, data);
  }

  // pet.service.ts
  getPetComments(petId: number): Observable<any> {
    return this.http.get(`${this.apiUrl}/pet-comments/${petId}`);
  }
}

```

step4:list.ts

```typescript
// pet-list.component.ts
import { Component, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { PetService } from '../services/pet.service';
import { Pet } from './pet.model';

@Component({
  selector: 'app-pet-list',
  standalone: true,
  imports: [FormsModule, CommonModule],
  templateUrl: './pet-list.component.html',
  styleUrls: ['./pet-list.component.css']
})
export class PetListComponent implements OnInit {
  pets: Pet[] = [];
  filters = {
    species: '',
    breed: '',
    minAge: null,
    maxAge: null,
    healthStatus: '',
    isVaccinated: null,
    adoptionStatus: '' // 新增领养状态筛选
  };

  // 新增领养相关属性
  showAdoptionModal = false;
  selectedPet: any = null;
  applicationData = {
    applicant_id: null,
    notes: ''
  };
  applicationError = '';
  applicationSuccess = '';

  // 在组件类中添加
  adoptionStatusMap = {
    'available': '可领养',
    'pending': '审核中',
    'adopted': '已领养'
  };

  healthStatusMap = {
    'excellent': '非常健康',
    'good': '良好',
    'fair': '一般',
    'poor': '需要照料'
  };


  // 新增疫苗相关属性
  showVaccinationModal = false;
  selectedPetId?: number;
  vaccinationRecords: any[] = [];
  vaccinationError = '';

  // 新增健康检查相关属性
  showHealthCheckModal = false;
  healthChecks: any[] = [];
  healthCheckError = '';

  // 新增评论相关属性
  showCommentModal = false;
  comments: any[] = [];
  commentError = '';
  selectedPetIdForComments?: number;

  constructor(private petService: PetService) {}

  ngOnInit() {
    this.loadPets();
  }

  loadPets() {
    const params = this.cleanFilters();
    console.log('Request Params:', params); // 新增调试日志
    this.petService.getPets(params).subscribe((res: any) => {
      console.log('API Response:', res);  // 新增响应日志
      this.pets = res.data;
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

  // 显示领养申请弹窗
  showAdoptionForm(pet: any) {
    this.selectedPet = pet;
    this.showAdoptionModal = true;
    this.resetApplicationForm();
  }

  // 提交领养申请
  submitApplication() {
    if (!this.validateApplication()) return;

    const requestBody = {
      pet_id: this.selectedPet.id,
      applicant_id: this.applicationData.applicant_id,
      notes: this.applicationData.notes
    };

    this.petService.submitAdoption(requestBody).subscribe({
      next: () => {
        this.applicationSuccess = '申请提交成功！';
        setTimeout(() => {
          this.closeModal();
          this.loadPets(); // 刷新数据
        }, 1500);
      },
      error: (err) => {
        this.applicationError = '提交失败，请检查信息是否正确';
        console.error('领养申请错误:', err);
      }
    });
  }



  // 验证表单
  private validateApplication(): boolean {
    this.applicationError = '';

    if (!this.applicationData.applicant_id) {
      this.applicationError = '请输入申请人ID';
      return false;
    }

    if (!this.applicationData.notes) {
      this.applicationError = '请输入申请说明';
      return false;
    }

    return true;
  }

  // 关闭弹窗
  closeModal() {
    this.showAdoptionModal = false;
    this.selectedPet = null;
    this.resetApplicationForm();
  }

  // 在构造函数后添加以下方法
  showVaccinationRecords(petId: number) {
    this.selectedPetId = petId;
    this.vaccinationError = '';
    this.petService.getVaccinations(petId).subscribe({
      next: (res) => {
        this.vaccinationRecords = res.data;
        this.showVaccinationModal = true;
      },
      error: (err) => {
        this.vaccinationError = '获取疫苗记录失败';
        console.error('疫苗记录错误:', err);
      }
    });
  }

  closeVaccinationModal() {
    this.showVaccinationModal = false;
    this.selectedPetId = undefined;
    this.vaccinationRecords = [];
  }


  // 添加获取评论方法
  showComments(petId: number) {
    this.selectedPetIdForComments = petId;
    this.petService.getPetComments(petId).subscribe({
      next: (res) => {
        this.comments = res.data;
        this.showCommentModal = true;
      },
      error: (err) => {
        this.commentError = '获取评论失败';
        console.error('评论获取错误:', err);
      }
    });
  }

  closeCommentModal() {
    this.showCommentModal = false;
    this.comments = [];
    this.commentError = '';
  }

  // 添加获取健康检查方法
  showHealthChecks(petId: number) {
    this.petService.getHealthChecks(petId).subscribe({
      next: (res) => {
        this.healthChecks = res.data;
        this.showHealthCheckModal = true;
      },
      error: (err) => {
        this.healthCheckError = '获取健康记录失败';
        console.error('健康记录错误:', err);
      }
    });
  }

  closeHealthCheckModal() {
    this.showHealthCheckModal = false;
    this.healthChecks = [];
    this.healthCheckError = '';
  }

  // 重置表单
  private resetApplicationForm() {
    this.applicationData = {
      applicant_id: null,
      notes: ''
    };
    this.applicationError = '';
    this.applicationSuccess = '';
  }

  // pet-list.component.ts
  generateStars(rating: number): string {
    return '★'.repeat(rating) + '☆'.repeat(5 - rating);
  }

  resetFilters() {
    this.filters = {
      species: '',
      breed: '',
      minAge: null,
      maxAge: null,
      healthStatus: '',
      isVaccinated: null,
      adoptionStatus: '' // 重置时清空领养状态
    };
    this.loadPets();
  }
}

```

step5:html

```xml
<!-- pet-list.component.html -->
<div class="filters">
  <!-- 新增领养状态筛选 -->
  <select [(ngModel)]="filters.adoptionStatus">
    <option value="">所有领养状态</option>
    <option value="available">可领养</option>
    <option value="pending">待审核</option>
    <option value="adopted">已领养</option>
  </select>

  <select [(ngModel)]="filters.species">
    <option value="">所有物种</option>
    <option>猫</option>
    <option>狗</option>
    <option>兔子</option>
    <option>仓鼠</option>
  </select>

  <input type="number" [(ngModel)]="filters.minAge" placeholder="最小年龄">
  <input type="number" [(ngModel)]="filters.maxAge" placeholder="最大年龄">

  <select [(ngModel)]="filters.healthStatus">
    <option value="">所有健康状态</option>
    <option>excellent</option>
    <option>good</option>
    <option>fair</option>
    <option>poor</option>
  </select>

  <button (click)="loadPets()">筛选</button>
  <button (click)="resetFilters()">重置</button>
</div>

<!-- 宠物列表部分保持不变 -->

<div class="pet-grid">
  <div *ngFor="let pet of pets" class="pet-card">
    <h3>{{ pet.name }}</h3>
    <p>品种：{{ pet.species }} - {{ pet.breed_name }}</p>
    <p>年龄：{{ pet.age_months }} 个月</p>
    <p>健康状况：{{ healthStatusMap[pet.health_status] }}</p>
    <p>疫苗状态：{{ pet.is_vaccinated ? '已接种' : '未接种' }}</p>

    <!-- 新增领养状态显示 -->


    <p [ngClass]="'status-' + pet.adoption_status" class="status-indicator">
      <span class="status-label">领养状态：</span>
      <span class="status-value">{{ adoptionStatusMap[pet.adoption_status] }}</span>
    </p>

    <!-- 根据状态控制按钮 -->
    <button
      (click)="showAdoptionForm(pet)"
      [disabled]="pet.adoption_status !== 'available'"
      [class.disabled]="pet.adoption_status !== 'available'">
      {{ pet.adoption_status === 'available' ? '申请领养' : '不可申请' }}
    </button>
    <button (click)="showVaccinationRecords(pet.id)">查看疫苗记录</button>
    <button (click)="showHealthChecks(pet.id)">查看健康记录</button>
    <button (click)="showComments(pet.id)">查看评论</button>
  </div>


  <!-- 领养申请弹窗 -->
  <div *ngIf="showAdoptionModal" class="modal-backdrop">
    <div class="adoption-modal">
      <h3>领养申请 - {{selectedPet?.name}}</h3>

      <div class="form-group">
        <label>申请人ID:</label>
        <input type="number" [(ngModel)]="applicationData.applicant_id">
      </div>

      <div class="form-group">
        <label>申请说明:</label>
        <textarea [(ngModel)]="applicationData.notes" rows="4"></textarea>
      </div>

      <div *ngIf="applicationError" class="error-message">
        {{applicationError}}
      </div>

      <div *ngIf="applicationSuccess" class="success-message">
        {{applicationSuccess}}
      </div>

      <div class="modal-actions">
        <button (click)="submitApplication()">提交申请</button>
        <button (click)="closeModal()">取消</button>
      </div>
    </div>
  </div>

  <!-- 添加疫苗记录弹窗 -->
  <div *ngIf="showVaccinationModal" class="modal-backdrop">
    <div class="vaccination-modal">
      <h3>疫苗记录 <span *ngIf="!vaccinationRecords.length">（无记录）</span></h3>

      <div *ngIf="vaccinationError" class="error-message">
        {{ vaccinationError }}
      </div>

      <table *ngIf="vaccinationRecords.length">
        <thead>
        <tr>
          <th>疫苗类型</th>
          <th>接种日期</th>
          <th>到期日期</th>
          <th>兽医</th>
        </tr>
        </thead>
        <tbody>
        <tr *ngFor="let record of vaccinationRecords">
          <td>{{ record.vaccine_type }}</td>
          <td>{{ record.vaccination_date | date: 'yyyy-MM-dd' }}</td>
          <td>{{ record.next_due_date | date: 'yyyy-MM-dd' }}</td>
          <td>{{ record.veterinarian || '—' }}</td>
        </tr>
        </tbody>
      </table>

      <div class="modal-actions">
        <button (click)="closeVaccinationModal()">关闭</button>
      </div>
    </div>
  </div>

  <!-- 添加健康检查弹窗 -->
  <div *ngIf="showHealthCheckModal" class="modal-backdrop">
    <div class="healthcheck-modal">
      <h3>健康检查记录 <span *ngIf="!healthChecks.length">（无记录）</span></h3>

      <div *ngIf="healthCheckError" class="error-message">
        {{ healthCheckError }}
      </div>

      <table *ngIf="healthChecks.length">
        <thead>
        <tr>
          <th>检查日期</th>
          <th>体重(kg)</th>
          <th>兽医</th>
          <th>检查说明</th>
          <th>诊断结果</th>
          <th>治疗方案</th>
        </tr>
        </thead>
        <tbody>
        <tr *ngFor="let check of healthChecks">
          <td>{{ check.check_date | date: 'yyyy-MM-dd HH:mm' }}</td>
          <td>{{ check.weight_kg }}</td>
          <td>{{ check.veterinarian || '—' }}</td>
          <td>{{ check.checkup_notes }}</td>
          <td>{{ check.diagnosis || '—' }}</td>
          <td>{{ check.treatment || '—' }}</td>
        </tr>
        </tbody>
      </table>

      <div class="modal-actions">
        <button (click)="closeHealthCheckModal()">关闭</button>
      </div>
    </div>
  </div>
  <!-- 添加评论弹窗 -->
  <div *ngIf="showCommentModal" class="modal-backdrop">
    <div class="comment-modal">
      <h3>用户评价 <span *ngIf="!comments.length">（暂无评论）</span></h3>

      <div *ngIf="commentError" class="error-message">
        {{ commentError }}
      </div>

      <div class="comment-list" *ngIf="comments.length">
        <div *ngFor="let comment of comments" class="comment-item">
          <div class="comment-header">
            <span class="commenter">{{ comment.commenter }}</span>
            <span class="rating">
            {{ generateStars(comment.rating) }}
          </span>
            <span class="date">{{ comment.created_at | date: 'yyyy-MM-dd HH:mm' }}</span>
          </div>
          <div class="comment-content">{{ comment.comment }}</div>
        </div>
      </div>

      <div class="modal-actions">
        <button (click)="closeCommentModal()">关闭</button>
      </div>
    </div>
  </div>
</div>

```

 

step6:model

```typescript
export interface Pet {
  id: number;
  name: string;
  breed_id: number;
  age_months: number;
  gender: 'male' | 'female' | 'unknown';
  health_status: 'excellent' | 'good' | 'fair' | 'poor';
  is_vaccinated: number;
  adoption_status: 'available' | 'pending' | 'adopted';
  description: string | null;
  created_at: string;
  updated_at: string;
  species: string;
  breed_name: string;
  vaccination_records?: VaccinationRecord[]; // 可选字段
  health_checks?: HealthCheck[]; // 可选字段
  comments?: PetComment[]; // 可选字段
}


// pet.model.ts
export interface VaccinationRecord {
  id: number;
  pet_id: number;
  vaccine_type: string;
  vaccination_date: string;
  next_due_date?: string;
  veterinarian?: string;
  notes?: string;
}

export interface HealthCheck {
  id: number;
  pet_id: number;
  check_date: string;
  veterinarian?: string;
  weight_kg: number;
  checkup_notes: string;
  diagnosis?: string;
  treatment?: string;
}

export interface PetComment {
  id: number;
  comment: string;
  rating: number;
  created_at: string;
  commenter: string;
}

```
step7:css

```css
/* 弹窗样式 */
.modal-backdrop {
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

.adoption-modal {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 500px;
  max-width: 90%;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.modal-actions {
  margin-top: 20px;
  text-align: right;
}

.modal-actions button {
  margin-left: 10px;
  padding: 8px 15px;
}

.error-message {
  color: #dc3545;
  margin: 10px 0;
}

.success-message {
  color: #28a745;
  margin: 10px 0;
}

/* 状态标签样式 */
.status-available {
  color: #28a745;
  font-weight: 500;
}
.status-pending {
  color: #ffc107;
}
.status-adopted {
  color: #6c757d;
}

/* 禁用按钮样式 */
button.disabled {
  background-color: #e9ecef;
  cursor: not-allowed;
  opacity: 0.65;
}

/* 新增状态指示样式 */
.status-indicator {
  padding: 4px 8px;
  border-radius: 12px;
  display: inline-block;
  font-size: 0.9em;
}

.status-available {
  background-color: #d4edda;
  color: #155724;
}

.status-pending {
  background-color: #fff3cd;
  color: #856404;
}

.status-adopted {
  background-color: #f8d7da;
  color: #721c24;
}

/* 优化筛选栏样式 */
.filters {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 10px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
  margin-bottom: 20px;
}

.filters select, .filters input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  background: white;
}


/*f分割线*/
/* pet-list.component.css */
/* 按钮组样式 */
.button-group {
  margin-top: 15px;
  display: flex;
  gap: 10px;
}

/* 疫苗弹窗样式 */
.vaccination-modal {
  background: white;
  padding: 25px;
  border-radius: 8px;
  width: 600px;
  max-width: 90%;
  max-height: 80vh;
  overflow-y: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin: 15px 0;
}

th, td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

th {
  background-color: #f8f9fa;
}

/* 健康检查弹窗特定样式 */
.healthcheck-modal {
  background: white;
  padding: 25px;
  border-radius: 8px;
  width: 800px;
  max-width: 90%;
  max-height: 80vh;
  overflow-y: auto;
}

.healthcheck-modal table {
  font-size: 0.9em;
}

.healthcheck-modal th:nth-child(1),
.healthcheck-modal td:nth-child(1) {
  width: 140px;
}

.healthcheck-modal th:nth-child(3),
.healthcheck-modal td:nth-child(3) {
  width: 120px;
}

/* 评论弹窗样式 */
.comment-modal {
  background: white;
  padding: 25px;
  border-radius: 8px;
  width: 600px;
  max-width: 90%;
  max-height: 80vh;
  overflow-y: auto;
}

.comment-item {
  margin-bottom: 20px;
  padding: 15px;
  border: 1px solid #eee;
  border-radius: 4px;
}

.comment-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  font-size: 0.9em;
}

.commenter {
  font-weight: bold;
  color: #333;
}

.rating {
  color: #ffd700;
  font-size: 1.1em;
}

.date {
  color: #666;
}

.comment-content {
  line-height: 1.6;
  white-space: pre-wrap;
}

```

end