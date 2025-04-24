说明：
我计划用fastapi+angular实现毕业生就业管理系统，
1.mysql写数据库表，创建表，插入数据，写查询语句
2.fastapi写后端接口，在postman里面测试
3.angular前端展示

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a506825dc7184f3188ecf0ab338077d3.png#pic_center)

step1:sql

```sql

-- 1. 院系表
CREATE TABLE department (
                            dept_id INT AUTO_INCREMENT PRIMARY KEY,
                            dept_name VARCHAR(50) NOT NULL UNIQUE,
                            dean VARCHAR(20),
                            office_phone VARCHAR(20)
);

-- 2. 专业表
CREATE TABLE major (
                       major_id INT AUTO_INCREMENT PRIMARY KEY,
                       dept_id INT NOT NULL,
                       major_name VARCHAR(50) NOT NULL,
                       years INT,
                       FOREIGN KEY (dept_id) REFERENCES department(dept_id)
);

-- 3. 企业表
CREATE TABLE company (
                         company_id INT AUTO_INCREMENT PRIMARY KEY,
                         name VARCHAR(100) NOT NULL UNIQUE,
                         industry VARCHAR(50),
                         address VARCHAR(200),
                         scale ENUM('小型','中型','大型'),
                         hr_name VARCHAR(20),
                         hr_phone VARCHAR(20)
);

-- 4. 用户表（合并学生表）
CREATE TABLE user (
                      user_id VARCHAR(20) PRIMARY KEY,
                      password VARCHAR(100) NOT NULL,
                      role ENUM('student','company','admin') NOT NULL,
                      name VARCHAR(50) NOT NULL,
                      gender ENUM('男','女'),
                      major_id INT,
                      phone VARCHAR(20),
                      email VARCHAR(50),
                      employment_status ENUM('已就业','待就业','升学','创业'),
                      ref_id INT, -- 企业用户关联公司ID
                      FOREIGN KEY (major_id) REFERENCES major(major_id),
                      FOREIGN KEY (ref_id) REFERENCES company(company_id)
);

-- 5. 招聘信息表
CREATE TABLE job_post (
                          post_id INT AUTO_INCREMENT PRIMARY KEY,
                          company_id INT NOT NULL,
                          job_title VARCHAR(50) NOT NULL,
                          job_type VARCHAR(20),
                          salary_range VARCHAR(20),
                          post_date DATE,
                          end_date DATE,
                          FOREIGN KEY (company_id) REFERENCES company(company_id)
);

-- 6. 投递记录表
CREATE TABLE application (
                             apply_id INT AUTO_INCREMENT PRIMARY KEY,
                             user_id VARCHAR(20) NOT NULL,
                             post_id INT NOT NULL,
                             apply_time DATETIME DEFAULT CURRENT_TIMESTAMP,
                             status ENUM('已投递','已查看','已面试','已录用'),
                             FOREIGN KEY (user_id) REFERENCES user(user_id),
                             FOREIGN KEY (post_id) REFERENCES job_post(post_id)
);

-- 7. 就业记录表
CREATE TABLE employment (
                            employ_id INT AUTO_INCREMENT PRIMARY KEY,
                            user_id VARCHAR(20) NOT NULL,  -- 移除了UNIQUE约束
                            company_id INT NOT NULL,
                            post_id INT,
                            salary DECIMAL(10,2),
                            sign_date DATE,
                            is_current BOOLEAN DEFAULT TRUE, -- 新增字段标识当前任职状态
                            FOREIGN KEY (user_id) REFERENCES user(user_id),
                            FOREIGN KEY (company_id) REFERENCES company(company_id),
                            FOREIGN KEY (post_id) REFERENCES job_post(post_id)
);



-- 院系表
INSERT INTO department (dept_name, dean, office_phone) VALUES
                                                           ('计算机学院', '张老师', '028-12345678'),
                                                           ('经济学院', '李老师', '028-23456789'),
                                                           ('机械学院', '王老师', '028-34567890'),
                                                           ('外语学院', '赵老师', '028-45678901'),
                                                           ('医学院', '孙老师', '028-56789012'),
                                                           ('法学院', '周老师', '028-67890123'),
                                                           ('艺术学院', '吴老师', '028-78901234'),
                                                           ('化工学院', '郑老师', '028-89012345'),
                                                           ('管理学院', '廖老师', '028-90123456'),
                                                           ('建筑学院', '孟老师', '028-01234567');

-- 专业表
INSERT INTO major (dept_id, major_name, years) VALUES
                                                   (1, '软件工程',4),(1,'人工智能',4),(2,'金融学',4),(2,'国际贸易',4),
                                                   (3,'机械工程',4),(4,'英语',4),(5,'临床医学',5),(6,'法学',4),
                                                   (7,'视觉传达',4),(8,'化学工程',4);

-- 企业表
INSERT INTO company (name, industry, address, scale, hr_name, hr_phone) VALUES
                                                                            ('Tencent Technology', '互联网', 'Shenzhen', '大型', '马先生', '0755-12345678'),
                                                                            ('Huawei Technologies', '通信', 'Shenzhen', '大型', '任先生', '0755-23456789'),
                                                                            ('China Merchants Bank', '金融', 'Shenzhen', '大型', '李女士', '0755-34567890'),
                                                                            ('ByteDance', '互联网', 'Beijing', '大型', '张先生', '010-12345678'),
                                                                            ('BYD Auto', '制造业', 'Shenzhen', '大型', '王先生', '0755-45678901'),
                                                                            ('New Oriental Education', '教育', 'Beijing', '大型', '周女士', '010-23456789'),
                                                                            ('Vanke Real Estate', '房地产', 'Shenzhen', '大型', '陈先生', '0755-56789012'),
                                                                            ('Sany Heavy Industry', '制造业', 'Changsha', '大型', '梁先生', '0731-1234567'),
                                                                            ('Hikvision', '安防', 'Hangzhou', '大型', '胡先生', '0571-1234567'),
                                                                            ('Gree Electric', '家电', 'Zhuhai', '大型', '董女士', '0756-1234567');


-- 用户表
INSERT INTO user VALUES
                     ('S2020001', 'pass123', 'student', '王小明', '男', 1, '13800010001', 'wangxm@edu.cn', '已就业', NULL),
                     ('S2020002', 'pass123', 'student', '李红', '女', 1, '13800010002', 'lih@edu.cn', '待就业', NULL),
                     ('S2020003', 'pass123', 'student', '张伟', '男', 3, '13800010003', 'zhangw@edu.cn', '已就业', NULL),
                     ('C1001', 'pass123', 'company', '腾讯科技HR', NULL, NULL, NULL, 'hr@tencent.com', NULL, 1),
                     ('C1002', 'pass123', 'company', '华为HR', NULL, NULL, NULL, 'hr@huawei.com', NULL, 2),
                     ('admin', 'admin123', 'admin', '系统管理员', NULL, NULL, NULL, 'admin@edu.cn', NULL, NULL),
                     ('S2020004', 'pass123', 'student', '陈芳', '女', 6, '13800010004', 'chenf@edu.cn', '升学', NULL),
                     ('S2020005', 'pass123', 'student', '刘洋', '男', 5, '13800010005', 'liuy@edu.cn', '创业', NULL),
                     ('S2020006', 'pass123', 'student', '周杰', '男', 2, '13800010006', 'zhouj@edu.cn', '已就业', NULL),
                     ('S2020007', 'pass123', 'student', '吴敏', '女', 4, '13800010007', 'wum@edu.cn', '待就业', NULL);


-- 补充四个专业的学生数据（临床医学、法学、视觉传达、化学工程）
INSERT INTO user (user_id, password, role, name, gender, major_id, phone, email, employment_status, ref_id) VALUES
-- 临床医学专业（major_id=7）
('S2020008', 'pass123', 'student', '张医生', '男', 7, '13800010011', 'zhangys@edu.cn', '已就业', NULL),
('S2020009', 'pass123', 'student', '李护士', '女', 7, '13800010012', 'lihs@edu.cn', '待就业', NULL),

-- 法学专业（major_id=8）
('S2020010', 'pass123', 'student', '王律师', '男', 8, '13800010013', 'wangls@edu.cn', '已就业', NULL),
('S2020011', 'pass123', 'student', '赵法务', '女', 8, '13800010014', 'zhaofw@edu.cn', '升学', NULL),

-- 视觉传达专业（major_id=9）
('S2020012', 'pass123', 'student', '陈设计', '男', 9, '13800010015', 'chensj@edu.cn', '创业', NULL),
('S2020013', 'pass123', 'student', '刘美工', '女', 9, '13800010016', 'liumg@edu.cn', '已就业', NULL),

-- 化学工程专业（major_id=10）
('S2020014', 'pass123', 'student', '黄化学', '男', 10, '13800010017', 'huanghx@edu.cn', '待就业', NULL),
('S2020015', 'pass123', 'student', '周实验', '女', 10, '13800010018', 'zhousy@edu.cn', '已就业', NULL),
('S2020016', 'pass123', 'student', '吴化工', '男', 10, '13800010019', 'wuhg@edu.cn', '待就业', NULL),
('S2020017', 'pass123', 'student', '郑材料', '女', 10, '13800010020', 'zhengcl@edu.cn', '升学', NULL);

-- 招聘信息表
INSERT INTO job_post (company_id, job_title, job_type, salary_range, post_date, end_date) VALUES
                                                                                              (1,'Java开发','技术岗','15-25K','2024-03-01','2024-06-30'),
                                                                                              (1,'产品经理','产品岗','20-35K','2024-03-05','2024-06-30'),
                                                                                              (2,'5G工程师','技术岗','18-30K','2024-03-10','2024-06-30'),
                                                                                              (3,'金融分析','金融岗','25-40K','2024-03-15','2024-06-30'),
                                                                                              (4,'前端开发','技术岗','16-28K','2024-03-20','2024-06-30'),
                                                                                              (5,'汽车设计','技术岗','20-35K','2024-03-25','2024-06-30'),
                                                                                              (6,'英语教师','教育岗','12-20K','2024-04-01','2024-06-30'),
                                                                                              (7,'建筑设计师','设计岗','18-30K','2024-04-05','2024-06-30'),
                                                                                              (8,'机械工程师','技术岗','15-25K','2024-04-10','2024-06-30'),
                                                                                              (9,'算法工程师','技术岗','30-50K','2024-04-15','2024-06-30');

-- 投递记录表
INSERT INTO application (user_id, post_id, status) VALUES
                                                       ('S2020001',1,'已录用'),
                                                       ('S2020001',3,'已投递'),
                                                       ('S2020002',7,'已查看'),
                                                       ('S2020001',5,'已投递'),
                                                       ('S2020002',2,'已投递'),
                                                       ('S2020001',8,'已查看'),
                                                       ('S2020002',4,'已投递'),
                                                       ('S2020001',6,'已查看'),
                                                       ('S2020002',9,'已投递'),
                                                       ('S2020001',10,'已录用');

-- 就业记录表
-- 添加10条就业记录测试数据
INSERT INTO employment (user_id, company_id, post_id, salary, sign_date, is_current) VALUES
('S2020001', 2, 3, 20000, '2024-05-01', TRUE),
('S2020001', 3, 4, 24000, '2023-10-01', FALSE),
('S2020002', 4, 5, 18000, '2024-05-02', TRUE),
('S2020002', 2, 3, 21000, '2024-04-28', TRUE),
('S2020003', 5, 6, 22000, '2024-04-20', FALSE),
('S2020003', 7, 8, 23000, '2024-05-03', TRUE),
('S2020004', 6, 7, 15000, '2024-03-15', FALSE),
('S2020005', 8, 9, 25000, '2024-05-05', TRUE),
('S2020006', 9, 10, 28000, '2024-04-25', TRUE),
('S2020007', 10, NULL, 26000, '2024-05-10', TRUE);

# 1. 根据学生ID查就业情况
SELECT
    u.name AS student_name,
    c.name AS company_name,
    j.job_title,
    e.salary,
    e.sign_date
FROM user u
         LEFT JOIN employment e ON u.user_id = e.user_id AND e.is_current = TRUE
         LEFT JOIN company c ON e.company_id = c.company_id
         LEFT JOIN job_post j ON e.post_id = j.post_id
WHERE u.user_id = 'S2020002';

# 2. 各专业就业率
SELECT
    m.major_name,
    COUNT(u.user_id) AS total_students,
    SUM(CASE WHEN u.employment_status = '已就业' THEN 1 ELSE 0 END) AS employed_count,
    ROUND(SUM(CASE WHEN u.employment_status = '已就业' THEN 1 ELSE 0 END) / COUNT(u.user_id) * 100, 2) AS employment_rate
FROM user u
         JOIN major m ON u.major_id = m.major_id
GROUP BY m.major_name;

# 3. 各院系就业率统计
SELECT
    d.dept_name,
    COUNT(u.user_id) AS total_students,
    SUM(CASE WHEN u.employment_status = '已就业' THEN 1 ELSE 0 END) AS employed_count,
    ROUND(SUM(CASE WHEN u.employment_status = '已就业' THEN 1 ELSE 0 END) / COUNT(u.user_id) * 100, 2) AS employment_rate
FROM user u
         JOIN major m ON u.major_id = m.major_id
         JOIN department d ON m.dept_id = d.dept_id
GROUP BY d.dept_name;

# 4. 热门岗位分析
SELECT
    j.job_title,
    COUNT(DISTINCT a.apply_id) AS apply_count,
    COUNT(DISTINCT e.employ_id) AS hire_count
FROM job_post j
         LEFT JOIN application a ON j.post_id = a.post_id
         LEFT JOIN employment e ON j.post_id = e.post_id
GROUP BY j.post_id
ORDER BY apply_count DESC;

# 5. 未就业学生名单
SELECT
    u.user_id AS student_id,
    u.name,
    m.major_name,
    d.dept_name
FROM user u
         JOIN major m ON u.major_id = m.major_id
         JOIN department d ON m.dept_id = d.dept_id
WHERE u.employment_status = '待就业';

# 6. 企业招聘就业分析
SELECT
    c.name AS company_name,
    COUNT(DISTINCT j.post_id) AS job_postings,
    COUNT(DISTINCT a.apply_id) AS applications_received,
    COUNT(DISTINCT e.employ_id) AS hires_made
FROM company c
         LEFT JOIN job_post j ON c.company_id = j.company_id
         LEFT JOIN application a ON j.post_id = a.post_id
         LEFT JOIN employment e ON j.post_id = e.post_id
GROUP BY c.company_id
ORDER BY job_postings DESC;
```

step2:后端fastapi写接口 C:\Users\Administrator\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
import pymysql.cursors

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
    'db': 'school_db',  # 替换为实际数据库名
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}

def query_db(query: str, params=None):
    try:
        connection = pymysql.connect(**DB_CONFIG)
        with connection.cursor() as cursor:
            cursor.execute(query, params or ())
            result = cursor.fetchall()
        connection.close()
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

# 1. 学生就业情况查询
@app.get("/student_employment/{student_id}")
async def get_student_employment(student_id: str):
    query = """
    SELECT 
        u.name AS student_name,
        c.name AS company_name,
        j.job_title,
        e.salary,
        e.sign_date
    FROM user u
    LEFT JOIN employment e ON u.user_id = e.user_id AND e.is_current = TRUE
    LEFT JOIN company c ON e.company_id = c.company_id
    LEFT JOIN job_post j ON e.post_id = j.post_id
    WHERE u.user_id = %s
    """
    return {"data": query_db(query, (student_id,))}

# 2. 专业就业率统计
@app.get("/major_stats")
async def get_major_employment_rates():
    query = """
    SELECT
        m.major_name,
        COUNT(u.user_id) AS total_students,
        SUM(u.employment_status = '已就业') AS employed_count,
        ROUND(SUM(u.employment_status = '已就业') / COUNT(u.user_id) * 100, 2) AS employment_rate
    FROM user u
    JOIN major m ON u.major_id = m.major_id
    GROUP BY m.major_name
    """
    return {"data": query_db(query)}

# 3. 院系就业率统计
@app.get("/dept_stats")
async def get_department_employment_rates():
    query = """
    SELECT
        d.dept_name,
        COUNT(u.user_id) AS total_students,
        SUM(u.employment_status = '已就业') AS employed_count,
        ROUND(SUM(u.employment_status = '已就业') / COUNT(u.user_id) * 100, 2) AS employment_rate
    FROM user u
    JOIN major m ON u.major_id = m.major_id
    JOIN department d ON m.dept_id = d.dept_id
    GROUP BY d.dept_name
    """
    return {"data": query_db(query)}

# 4. 热门岗位分析
@app.get("/jobs/hot")
async def get_hot_jobs():
    query = """
    SELECT
        j.job_title,
        COUNT(DISTINCT a.apply_id) AS apply_count,
        COUNT(DISTINCT e.employ_id) AS hire_count
    FROM job_post j
    LEFT JOIN application a ON j.post_id = a.post_id
    LEFT JOIN employment e ON j.post_id = e.post_id
    GROUP BY j.post_id
    ORDER BY apply_count DESC
    """
    return {"data": query_db(query)}

# 5. 未就业学生名单
@app.get("/unemployed_students")
async def get_unemployed_students():
    query = """
    SELECT
        u.user_id AS student_id,
        u.name,
        m.major_name,
        d.dept_name
    FROM user u
    JOIN major m ON u.major_id = m.major_id
    JOIN department d ON m.dept_id = d.dept_id
    WHERE u.employment_status = '待就业'
    """
    return {"data": query_db(query)}

# 6. 企业招聘分析
@app.get("/company_stats")
async def get_company_recruitment():
    query = """
    SELECT
        c.name AS company_name,
        COUNT(DISTINCT j.post_id) AS job_postings,
        COUNT(DISTINCT a.apply_id) AS applications_received,
        COUNT(DISTINCT e.employ_id) AS hires_made
    FROM company c
    LEFT JOIN job_post j ON c.company_id = j.company_id
    LEFT JOIN application a ON j.post_id = a.post_id
    LEFT JOIN employment e ON j.post_id = e.post_id
    GROUP BY c.company_id
    ORDER BY job_postings DESC
    """
    return {"data": query_db(query)}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:postman测试fastapi写的接口，验证接口是否正确

```bash
 http://localhost:8000/company_stats

 {
    "data": [
        {
            "company_name": "Tencent Technology",
            "job_postings": 2,
            "applications_received": 2,
            "hires_made": 0
        },
        {
            "company_name": "Huawei Technologies",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 2
        },
        {
            "company_name": "China Merchants Bank",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 1
        },
        {
            "company_name": "ByteDance",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 1
        },
        {
            "company_name": "BYD Auto",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 1
        },
        {
            "company_name": "New Oriental Education",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 1
        },
        {
            "company_name": "Vanke Real Estate",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 1
        },
        {
            "company_name": "Sany Heavy Industry",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 1
        },
        {
            "company_name": "Hikvision",
            "job_postings": 1,
            "applications_received": 1,
            "hires_made": 1
        },
        {
            "company_name": "Gree Electric",
            "job_postings": 0,
            "applications_received": 0,
            "hires_made": 0
        }
    ]
}


http://localhost:8000/unemployed_students

{
    "data": [
        {
            "student_id": "S2020002",
            "name": "李红",
            "major_name": "软件工程",
            "dept_name": "计算机学院"
        },
        {
            "student_id": "S2020007",
            "name": "吴敏",
            "major_name": "国际贸易",
            "dept_name": "经济学院"
        },
        {
            "student_id": "S2020009",
            "name": "李护士",
            "major_name": "临床医学",
            "dept_name": "医学院"
        },
        {
            "student_id": "S2020014",
            "name": "黄化学",
            "major_name": "化学工程",
            "dept_name": "化工学院"
        },
        {
            "student_id": "S2020016",
            "name": "吴化工",
            "major_name": "化学工程",
            "dept_name": "化工学院"
        }
    ]
}



http://localhost:8000/jobs/hot

{
    "data": [
        {
            "job_title": "Java开发",
            "apply_count": 1,
            "hire_count": 0
        },
        {
            "job_title": "产品经理",
            "apply_count": 1,
            "hire_count": 0
        },
        {
            "job_title": "5G工程师",
            "apply_count": 1,
            "hire_count": 2
        },
        {
            "job_title": "金融分析",
            "apply_count": 1,
            "hire_count": 1
        },
        {
            "job_title": "前端开发",
            "apply_count": 1,
            "hire_count": 1
        },
        {
            "job_title": "汽车设计",
            "apply_count": 1,
            "hire_count": 1
        },
        {
            "job_title": "英语教师",
            "apply_count": 1,
            "hire_count": 1
        },
        {
            "job_title": "建筑设计师",
            "apply_count": 1,
            "hire_count": 1
        },
        {
            "job_title": "机械工程师",
            "apply_count": 1,
            "hire_count": 1
        },
        {
            "job_title": "算法工程师",
            "apply_count": 1,
            "hire_count": 1
        }
    ]
}


http://localhost:8000/dept_stats

{
    "data": [
        {
            "dept_name": "计算机学院",
            "total_students": 3,
            "employed_count": 2,
            "employment_rate": 66.67
        },
        {
            "dept_name": "经济学院",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "dept_name": "机械学院",
            "total_students": 1,
            "employed_count": 0,
            "employment_rate": 0.0
        },
        {
            "dept_name": "外语学院",
            "total_students": 1,
            "employed_count": 0,
            "employment_rate": 0.0
        },
        {
            "dept_name": "医学院",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "dept_name": "法学院",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "dept_name": "艺术学院",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "dept_name": "化工学院",
            "total_students": 4,
            "employed_count": 1,
            "employment_rate": 25.0
        }
    ]
}


http://localhost:8000/major_stats

{
    "data": [
        {
            "major_name": "软件工程",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "major_name": "人工智能",
            "total_students": 1,
            "employed_count": 1,
            "employment_rate": 100.0
        },
        {
            "major_name": "金融学",
            "total_students": 1,
            "employed_count": 1,
            "employment_rate": 100.0
        },
        {
            "major_name": "国际贸易",
            "total_students": 1,
            "employed_count": 0,
            "employment_rate": 0.0
        },
        {
            "major_name": "机械工程",
            "total_students": 1,
            "employed_count": 0,
            "employment_rate": 0.0
        },
        {
            "major_name": "英语",
            "total_students": 1,
            "employed_count": 0,
            "employment_rate": 0.0
        },
        {
            "major_name": "临床医学",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "major_name": "法学",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "major_name": "视觉传达",
            "total_students": 2,
            "employed_count": 1,
            "employment_rate": 50.0
        },
        {
            "major_name": "化学工程",
            "total_students": 4,
            "employed_count": 1,
            "employment_rate": 25.0
        }
    ]
}

http://localhost:8000/student_employment/S2020002

{
    "data": [
        {
            "student_name": "李红",
            "company_name": "ByteDance",
            "job_title": "前端开发",
            "salary": 18000.0,
            "sign_date": "2024-05-02"
        },
        {
            "student_name": "李红",
            "company_name": "Huawei Technologies",
            "job_title": "5G工程师",
            "salary": 21000.0,
            "sign_date": "2024-04-28"
        }
    ]
}


```

step4:接下来写前端angular

step5:service统一http接口 C:\Users\Administrator\WebstormProjects\untitled4\src\app\job\employment.service.ts

```typescript
// employment.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class EmploymentService {
  private apiUrl = 'http://localhost:8000';

  constructor(private http: HttpClient) { }

  getStudentEmployment(id: string): Observable<any> {
    return this.http.get(`${this.apiUrl}/student_employment/${id}`);
  }

  getMajorStats(): Observable<any> {
    return this.http.get(`${this.apiUrl}/major_stats`);
  }

  getDepartmentStats(): Observable<any> {
    return this.http.get(`${this.apiUrl}/dept_stats`);
  }

  getHotJobs(): Observable<any> {
    return this.http.get(`${this.apiUrl}/jobs/hot`);
  }

  getUnemployed(): Observable<any> {
    return this.http.get(`${this.apiUrl}/unemployed_students`);
  }

  getCompanyStats(): Observable<any> {
    return this.http.get(`${this.apiUrl}/company_stats`);
  }
}

```

step6:主界面 C:\Users\Administrator\WebstormProjects\untitled4\src\app\job\job.component.ts

```typescript
import { Component ,OnInit} from '@angular/core';

import { EmploymentService } from './employment.service';
import {FormsModule} from '@angular/forms';
import {CurrencyPipe, DatePipe, NgForOf, NgIf} from '@angular/common';

@Component({
  selector: 'app-job',
  imports: [
    FormsModule,
    NgIf,
    NgForOf,
    CurrencyPipe,
    DatePipe
  ],
  templateUrl: './job.component.html',
  styleUrl: './job.component.css'
})
export class JobComponent implements OnInit {
  studentEmployment: any[] = [];
  majorStats: any[] = [];
  departmentStats: any[] = [];
  hotJobs: any[] = [];
  unemployedStudents: any[] = [];
  companyStats: any[] = [];
  selectedStudentId = 'S2020002';

  constructor(private employmentService: EmploymentService) {}

  ngOnInit() {
    this.loadAllData();
  }

  loadAllData() {
    this.employmentService.getStudentEmployment(this.selectedStudentId)
      .subscribe(res => this.studentEmployment = res.data);

    this.employmentService.getMajorStats()
      .subscribe(res => this.majorStats = res.data);

    this.employmentService.getDepartmentStats()
      .subscribe(res => this.departmentStats = res.data);

    this.employmentService.getHotJobs()
      .subscribe(res => this.hotJobs = res.data);

    this.employmentService.getUnemployed()
      .subscribe(res => this.unemployedStudents = res.data);

    this.employmentService.getCompanyStats()
      .subscribe(res => this.companyStats = res.data);
  }
}

```

step7:布局 C:\Users\Administrator\WebstormProjects\untitled4\src\app\job\job.component.html

```xml
<!-- app.component.html -->
<div class="dashboard-container">
  <!-- 学生就业查询 -->
  <section class="card">
    <h2>学生就业查询</h2>
    <div class="search-box">
      <input [(ngModel)]="selectedStudentId" placeholder="输入学号">
      <button (click)="loadAllData()">查询</button>
    </div>
    <table *ngIf="studentEmployment.length > 0">
      <thead>
      <tr>
        <th>姓名</th>
        <th>企业</th>
        <th>岗位</th>
        <th>薪资</th>
        <th>入职日期</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let item of studentEmployment">
        <td>{{item.student_name}}</td>
        <td>{{item.company_name}}</td>
        <td>{{item.job_title}}</td>
        <td>{{ item.salary  | currency:'CNY':'symbol' }}</td>
        <td>{{ item.sign_date| date:'medium' }}</td>
      </tr>
      </tbody>
    </table>
  </section>

  <!-- 专业就业率 -->
  <section class="card">
    <h2>专业就业率</h2>
    <table>
      <thead>
      <tr>
        <th>专业</th>
        <th>总人数</th>
        <th>就业率</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let item of majorStats">
        <td>{{item.major_name}}</td>
        <td>{{item.total_students}}</td>
        <td>{{item.employment_rate}}%</td>
      </tr>
      </tbody>
    </table>
  </section>

  <!-- 院系就业统计 -->
  <section class="card">
    <h2>院系就业统计</h2>
    <table>
      <thead>
      <tr>
        <th>院系</th>
        <th>总人数</th>
        <th>就业率</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let item of departmentStats">
        <td>{{item.dept_name}}</td>
        <td>{{item.total_students}}</td>
        <td>{{item.employment_rate}}%</td>
      </tr>
      </tbody>
    </table>
  </section>

  <!-- 热门岗位分析 -->
  <section class="card">
    <h2>热门岗位分析</h2>
    <table>
      <thead>
      <tr>
        <th>岗位名称</th>
        <th>投递次数</th>
        <th>录用人数</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let job of hotJobs">
        <td>{{ job.job_title }}</td>
        <td>{{ job.apply_count }}</td>
        <td>{{ job.hire_count }}</td>
      </tr>
      </tbody>
    </table>
  </section>

  <!-- 未就业名单 -->
  <section class="card">
    <h2>未就业学生名单</h2>
    <table>
      <thead>
      <tr>
        <th>学号</th>
        <th>姓名</th>
        <th>专业</th>
        <th>院系</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let item of unemployedStudents">
        <td>{{item.student_id}}</td>
        <td>{{item.name}}</td>
        <td>{{item.major_name}}</td>
        <td>{{item.dept_name}}</td>
      </tr>
      </tbody>
    </table>
  </section>

  <!-- 企业招聘分析 -->
  <section class="card">
    <h2>企业招聘分析</h2>
    <table>
      <thead>
      <tr>
        <th>企业名称</th>
        <th>岗位数</th>
        <th>收到简历</th>
        <th>实际录用</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let item of companyStats">
        <td>{{item.company_name}}</td>
        <td>{{item.job_postings}}</td>
        <td>{{item.applications_received}}</td>
        <td>{{item.hires_made}}</td>
      </tr>
      </tbody>
    </table>
  </section>
</div>

```

step8:css C:\Users\Administrator\WebstormProjects\untitled4\src\app\job\job.component.css

```css
/* app.component.css */
.dashboard-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 20px;
  padding: 20px;
}

.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  padding: 20px;
}

table {
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

.search-box {
  margin-bottom: 20px;
  display: flex;
  gap: 10px;
}

input {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  flex: 1;
}

button {
  padding: 8px 16px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.chart-section {
  grid-column: 1 / -1;
}

.chart-container {
  height: 400px;
}

```

end