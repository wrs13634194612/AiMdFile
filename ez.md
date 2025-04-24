说明：
我计划用nodejs+mysql，做一个学生信息管理系统，顺便写几个接口，调用sql表的数据，查询下面的信息，然后在angular前端页面展示这些信息
 1.‌查询学生信息及所属班级‌
 2.‌查询学生成绩详情（含课程名）‌
 3.统计学生平均分及总学分‌
 效果图：
 ![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7294a85ed3bd40f58cdbab4467e26cd0.png#pic_center)

ps：关于如何连接mysql，添加npm依赖，请参考我上一篇博客

step1:先创建sql表，插入数据，然后用sql做查询，先把数据库部分弄好，然后再写接口

step2:创建4张表learners，class_enrollments，academic_subjects，learner_assessments

```sql

-- 学员信息表（原students表）
CREATE TABLE db_spring.learners (
                                    learner_id INT AUTO_INCREMENT PRIMARY KEY,
                                    name VARCHAR(50) NOT NULL,
                                    gender ENUM('M', 'F') NOT NULL,
                                    age TINYINT UNSIGNED,
                                    class_enrollment_id INT NOT NULL,
                                    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                                    FOREIGN KEY (class_enrollment_id) REFERENCES db_spring.class_enrollments(class_enrollment_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 班级注册表（原classes表）
CREATE TABLE db_spring.class_enrollments (
                                             class_enrollment_id INT AUTO_INCREMENT PRIMARY KEY,
                                             className VARCHAR(50) NOT NULL UNIQUE,
                                             instructor_id INT NOT NULL,
                                             max_capacity SMALLINT UNSIGNED DEFAULT 40
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 学科课程表（原courses表）
CREATE TABLE db_spring.academic_subjects (
                                             subject_id INT AUTO_INCREMENT PRIMARY KEY,
                                             subject_name VARCHAR(50) NOT NULL UNIQUE,
                                             credit_hours TINYINT UNSIGNED DEFAULT 2
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 学习评估表（原scores表）
CREATE TABLE db_spring.learner_assessments (
                                               assessment_id INT AUTO_INCREMENT PRIMARY KEY,
                                               learner_id INT NOT NULL,
                                               subject_id INT NOT NULL,
                                               score DECIMAL(4,1) CHECK (score BETWEEN 0 AND 100),
                                               assessment_date DATE,
                                               FOREIGN KEY (learner_id) REFERENCES db_spring.learners(learner_id) ON DELETE CASCADE,
                                               FOREIGN KEY (subject_id) REFERENCES db_spring.academic_subjects(subject_id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

step3:添加测试数据

```sql

-- 插入班级注册数据
INSERT INTO db_spring.class_enrollments (className, instructor_id) VALUES
                                                                       ('Computer Science Program 1', 1001),
                                                                       ('Software Engineering Program 2', 1002);

-- 插入学员数据
INSERT INTO db_spring.learners (name, gender, age, class_enrollment_id) VALUES
                                                                            ('Zhang San', 'M', 20, 1),
                                                                            ('Li Si', 'F', 19, 1),
                                                                            ('Wang Wu', 'M', 21, 2);

-- 插入课程数据
INSERT INTO db_spring.academic_subjects (subject_name, credit_hours) VALUES
                                                                         ('Database Systems', 3),
                                                                         ('Data Structures and Algorithms', 4),
                                                                         ('Operating Systems', 3);

-- 插入评估数据
INSERT INTO db_spring.learner_assessments (learner_id, subject_id, score, assessment_date) VALUES
                                                                                               (1, 1, 88.5, '2025-06-15'),
                                                                                               (1, 2, 92.0, '2025-06-16'),
                                                                                               (2, 1, 75.0, '2025-06-15');
-- 示例：给王五添加数学成绩
INSERT INTO db_spring.learner_assessments (
    learner_id,
    subject_id,
    score,
    assessment_date
) VALUES
    (3, 1, 85.0, '2025-06-20');
```

step4:查询

```sql

# 1.‌查询学生信息及所属班级‌
SELECT
    l.learner_id AS 学员ID,
    l.name AS 姓名,
    ce.className AS 班级名称,
    l.gender AS 性别,
    l.age AS 年龄,
    l.created_at AS 注册时间
FROM
    db_spring.learners l
        INNER JOIN
    db_spring.class_enrollments ce
    ON
        l.class_enrollment_id = ce.class_enrollment_id;

# 2.‌查询学生成绩详情（含课程名）‌
SELECT
    la.assessment_id AS 评估ID,
    l.name AS 学生姓名,
    asub.subject_name AS 课程名称,
    la.score AS 分数,
    la.assessment_date AS 考试日期
FROM
    db_spring.learner_assessments la
        INNER JOIN
    db_spring.learners l ON la.learner_id = l.learner_id
        INNER JOIN
    db_spring.academic_subjects asub ON la.subject_id = asub.subject_id;

# 3.统计学生平均分及总学分‌
SELECT
    l.learner_id AS 学员ID,
    l.name AS 姓名,
    AVG(la.score) AS 平均分,
    SUM(asub.credit_hours) AS 总学分
FROM
    db_spring.learners l
        LEFT JOIN
    db_spring.learner_assessments la ON l.learner_id = la.learner_id
        LEFT JOIN
    db_spring.academic_subjects asub ON la.subject_id = asub.subject_id
GROUP BY
    l.learner_id, l.name
ORDER BY
    l.learner_id ASC;

```

step5:在js里面写service，用于连接查询sql表，获取数据 C:\Users\Administrator\WebstormProjects\untitled4\score-crud.js

```javascript
const config = require('./db'); // 数据库配置

class CrudService {
  // 查询学生信息及所属班级
  static async getStudentWithClass(learnerId) {
    const [rows] = await config.query(`
      SELECT
        l.learner_id,
        l.name AS student_name,
        ce.className AS class_name
      FROM
        db_spring.learners l
      INNER JOIN
        db_spring.class_enrollments ce ON l.class_enrollment_id = ce.class_enrollment_id
      WHERE
        l.learner_id = ?
    `, [learnerId]);
    return rows[0] || null;
  }

  // 查询学生信息及所属班级
  static async getAllStudentWithClass() {
    const [rows] = await config.query(`
      SELECT
        l.learner_id,
        l.name AS student_name,
        ce.className AS class_name,
        l.gender,
        l.age,
        l.created_at
      FROM
        db_spring.learners l
          INNER JOIN
        db_spring.class_enrollments ce ON l.class_enrollment_id = ce.class_enrollment_id
    `);
    return rows;
  }

  // 查询学生成绩详情（含课程名）
  static async getAllStudentScores() {
    const [rows] = await config.query(`
      SELECT
        la.assessment_id,
        l.name AS student_name,
        asub.subject_name AS course_name,
        la.score,
        la.assessment_date
      FROM
        db_spring.learner_assessments la
      INNER JOIN
        db_spring.learners l ON la.learner_id = l.learner_id
      INNER JOIN
        db_spring.academic_subjects asub ON la.subject_id = asub.subject_id
    `,);
    return rows;
  }
  // student-crud.js
  static async getAllStudentStatistics() {
    const [rows] = await config.query(`
    SELECT
      l.learner_id,
      l.name AS student_name,
      AVG(la.score) AS average_score,
      SUM(asub.credit_hours) AS total_credits
    FROM
      db_spring.learners l
    LEFT JOIN
      db_spring.learner_assessments la ON l.learner_id = la.learner_id
    LEFT JOIN
      db_spring.academic_subjects asub ON la.subject_id = asub.subject_id
    GROUP BY
      l.learner_id, l.name
    ORDER BY
      l.learner_id ASC
  `);
    return rows;
  }

  // 查询学生成绩详情（含课程名）
  static async getStudentScores(learnerId) {
    const [rows] = await config.query(`
      SELECT
        la.assessment_id,
        l.name AS student_name,
        asub.subject_name AS course_name,
        la.score,
        la.assessment_date
      FROM
        db_spring.learner_assessments la
      INNER JOIN
        db_spring.learners l ON la.learner_id = l.learner_id
      INNER JOIN
        db_spring.academic_subjects asub ON la.subject_id = asub.subject_id
      WHERE
        l.learner_id = ?
    `, [learnerId]);
    return rows;
  }

  // 统计学生平均分及总学分
  static async getStudentStatistics(learnerId) {
    const [rows] = await config.query(`
      SELECT
        l.learner_id,
        l.name AS student_name,
        AVG(la.score) AS average_score,
        SUM(asub.credit_hours) AS total_credits
      FROM
        db_spring.learners l
      LEFT JOIN
        db_spring.learner_assessments la ON l.learner_id = la.learner_id
      LEFT JOIN
        db_spring.academic_subjects asub ON la.subject_id = asub.subject_id
      WHERE
        l.learner_id = ?
      GROUP BY
        l.learner_id, l.name
    `, [learnerId]);
    return rows[0] || { average_score: null, total_credits: 0 };
  }

}

module.exports = CrudService;

```

step6:写controller接口路由，C:\Users\Administrator\WebstormProjects\untitled4\score-spring.js

```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const crud = require('./score-crud');

const app = express();
app.use(bodyParser.json());
app.use(cors());

// API 路由
app.get('/api/learners/:id', async (req, res) => {
  const learnerId = parseInt(req.params.id);
  try {
    const student = await crud.getStudentWithClass(learnerId);
    if (student) {
      res.status(200).json(student);
    } else {
      res.status(404).json({ message: '学生不存在' });
    }
  } catch (err) {
    console.error('查询学生错误:', err);
    res.status(500).json({ error: '服务器内部错误' });
  }
});

app.get('/api/learners/scores/:id', async (req, res) => {
  const learnerId = parseInt(req.params.id);
  try {
    const scores = await crud.getStudentScores(learnerId);
    res.json(scores);
  } catch (err) {
    console.error('查询成绩错误:', err);
    res.status(500).json({ error: '服务器内部错误' });
  }
});

app.get('/api/learners/statistics/:id', async (req, res) => {
  const learnerId = parseInt(req.params.id);
  try {
    const stats = await crud.getStudentStatistics(learnerId);
    res.json(stats);
  } catch (err) {
    console.error('统计错误:', err);
    res.status(500).json({ error: '服务器内部错误' });
  }
});

// student-spring.js
app.get('/all/learners', async (req, res) => {
  try {
    const students = await crud.getAllStudentWithClass();
    res.json(students);
  } catch (err) {
    console.error('查询所有学生错误:', err);
    res.status(500).json({ error: '服务器内部错误' });
  }
});

app.get('/all/learners/scores', async (req, res) => {
  try {
    const scores = await crud.getAllStudentScores();
    res.json(scores);
  } catch (err) {
    console.error('查询成绩错误:', err);
    res.status(500).json({ error: '服务器内部错误' });
  }
});
app.get('/all/learners/statistics', async (req, res) => {
  try {
    const stats = await crud.getAllStudentStatistics();
    res.json(stats);
  } catch (err) {
    console.error('统计错误:', err);
    res.status(500).json({ error: '服务器内部错误' });
  }
});


// 启动服务器
const PORT = process.env.PORT || 5400;
app.listen(PORT, () => {
  console.log(`🚀 Student Management System listening on port ${PORT}`);
});

```

step7:运行

```bash
PS C:\Users\Administrator\WebstormProjects\untitled4> node score-spring.js
🚀 Student Management System listening on port 5400

```

step8:在postman里面测试数据

step9:查询单个学生的平均分及总学分‌

```bash
get请求 http://localhost:5400/api/learners/statistics/1
{
    "learner_id": 1,
    "student_name": "Zhang San",
    "average_score": "90.25000",
    "total_credits": "7"
}
```

step10:查询所有学生的平均分及总学分‌

```bash
http://localhost:5400/all/learners/statistics
[
    {
        "learner_id": 1,
        "student_name": "Zhang San",
        "average_score": "90.25000",
        "total_credits": "7"
    },
    {
        "learner_id": 2,
        "student_name": "Li Si",
        "average_score": "75.00000",
        "total_credits": "3"
    },
    {
        "learner_id": 3,
        "student_name": "Wang Wu",
        "average_score": "85.00000",
        "total_credits": "6"
    }
]
```
step11:‌查询所有学生成绩详情（含课程名）‌

```bash
get请求 http://localhost:5400/all/learners/scores
[
    {
        "assessment_id": 1,
        "student_name": "Zhang San",
        "course_name": "Database Systems",
        "score": "88.5",
        "assessment_date": "2025-06-14T16:00:00.000Z"
    },
    {
        "assessment_id": 2,
        "student_name": "Zhang San",
        "course_name": "Data Structures and Algorithms",
        "score": "92.0",
        "assessment_date": "2025-06-15T16:00:00.000Z"
    },
    {
        "assessment_id": 3,
        "student_name": "Li Si",
        "course_name": "Database Systems",
        "score": "75.0",
        "assessment_date": "2025-06-14T16:00:00.000Z"
    },
    {
        "assessment_id": 4,
        "student_name": "Wang Wu",
        "course_name": "Database Systems",
        "score": "85.0",
        "assessment_date": "2025-06-19T16:00:00.000Z"
    },
    {
        "assessment_id": 5,
        "student_name": "Wang Wu",
        "course_name": "Database Systems",
        "score": "85.0",
        "assessment_date": "2025-06-19T16:00:00.000Z"
    }
]
```

step12: 查询所有学生信息及所属班级‌

```bash
get请求 http://localhost:5400/all/learners
[
    {
        "learner_id": 1,
        "student_name": "Zhang San",
        "class_name": "Computer Science Program 1",
        "gender": "M",
        "age": 20,
        "created_at": "2025-03-05T03:43:08.000Z"
    },
    {
        "learner_id": 2,
        "student_name": "Li Si",
        "class_name": "Computer Science Program 1",
        "gender": "F",
        "age": 19,
        "created_at": "2025-03-05T03:43:08.000Z"
    },
    {
        "learner_id": 3,
        "student_name": "Wang Wu",
        "class_name": "Software Engineering Program 2",
        "gender": "M",
        "age": 21,
        "created_at": "2025-03-05T03:43:08.000Z"
    }
]
```
step13:到这里，mysql和后端接口就处理完成了，接下来处理angular前端显示部分 
step14: 定义三个接口类C:\Users\Administrator\WebstormProjects\untitled4\src\app\score\learner.model.ts

```javascript
export interface Learner {
  learner_id: number;
  student_name: string;
  class_name: string;
  gender: string;
  age: number;
  created_at: string;
}

```

step15: C:\Users\Administrator\WebstormProjects\untitled4\src\app\score\score.model.ts

```javascript
export interface Score {
  assessment_id: number;
  student_name: string;
  course_name: string;
  score: string;
  assessment_date: string;
}

```

step16: C:\Users\Administrator\WebstormProjects\untitled4\src\app\score\statistic.model.ts

```javascript
export interface Statistic {
  learner_id: number;
  student_name: string;
  average_score: string;
  total_credits: string;
}

```

step17: 网络请求，接口服务类

```javascript
// api.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

import {Learner} from './learner.model';
import {Score} from './score.model';
import {Statistic} from './statistic.model';


@Injectable({
  providedIn: 'root'
})
export class ApiService {
  private baseUrl = 'http://localhost:5400';
  constructor(private http: HttpClient) { }
  getLearners(): Observable<Learner[]> {
    return this.http.get<Learner[]>(`${this.baseUrl}/all/learners`);
  }
  getScores(): Observable<Score[]> {
    return this.http.get<Score[]>(`${this.baseUrl}/all/learners/scores`);
  }
  getStatistics(): Observable<Statistic[]> {
    return this.http.get<Statistic[]>(`${this.baseUrl}/all/learners/statistics`);
  }
}

```

step18:C:\Users\Administrator\WebstormProjects\untitled4\src\app\score\score.component.ts

```javascript
import { Component, OnInit } from '@angular/core';
import { ApiService } from './api.service';
import { DatePipe, DecimalPipe, NgForOf } from '@angular/common';
import { Learner } from './learner.model';
import { Score } from './score.model';
import { Statistic } from './statistic.model';
@Component({
  selector: 'app-score',
  imports: [
    NgForOf,
    DatePipe,
    DecimalPipe
  ],
  templateUrl: './score.component.html',
  styleUrl: './score.component.css'
})
export class ScoreComponent implements OnInit {
  learners: Learner[] = [];
  scores: Score[] = [];
  statistics: Statistic[] = [];
  constructor(private apiService: ApiService) {}
  ngOnInit() {
    this.loadData();
  }
  loadData() {
    this.apiService.getLearners().subscribe(data => this.learners = data);
    this.apiService.getScores().subscribe(data => this.scores = data);
    this.apiService.getStatistics().subscribe(data => {
      // 移除类型转换，保持原始字符串类型
      this.statistics = data;
    });
  }
}

```


step19: C:\Users\Administrator\WebstormProjects\untitled4\src\app\score\score.component.html

```xml
<!-- app.component.html -->
<div class="container">
  <!-- Learners Table -->
  <section>
    <h2>Students Information</h2>
    <table>
      <thead>
      <tr>
        <th>ID</th>
        <th>Name</th>
        <th>Class</th>
        <th>Gender</th>
        <th>Age</th>
        <th>Enrollment Date</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let learner of learners">
        <td>{{ learner.learner_id }}</td>
        <td>{{ learner.student_name }}</td>
        <td>{{ learner.class_name }}</td>
        <td>{{ learner.gender === 'M' ? 'Male' : 'Female' }}</td>
        <td>{{ learner.age }}</td>
        <td>{{ learner.created_at | date: 'mediumDate' }}</td>
      </tr>
      </tbody>
    </table>
  </section>
  <!-- Scores Table -->
  <section>
    <h2>Assessment Scores</h2>
    <table>
      <thead>
      <tr>
        <th>Course</th>
        <th>Student</th>
        <th>Score</th>
        <th>Date</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let score of scores">
        <td>{{ score.course_name }}</td>
        <td>{{ score.student_name }}</td>
        <td>{{ score.score }}</td>
        <td>{{ score.assessment_date | date: 'mediumDate' }}</td>
      </tr>
      </tbody>
    </table>
  </section>
  <!-- Statistics Table -->
  <section>
    <h2>Academic Statistics</h2>
    <table>
      <thead>
      <tr>
        <th>Student</th>
        <th>Average</th>
        <th>Credits</th>
      </tr>
      </thead>
      <tbody>
      <tr *ngFor="let stat of statistics">
        <td>{{ stat.student_name }}</td>
        <td>{{ stat.average_score | number: '1.2-2' }}</td>
        <td>{{ stat.total_credits }}</td>
      </tr>
      </tbody>
    </table>
  </section>
</div>

```

step20: C:\Users\Administrator\WebstormProjects\untitled4\src\app\score\score.component.css

```css
/* app.component.css */
.container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}
section {
  margin-bottom: 40px;
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
tr:hover {
  background-color: #f9f9f9;
}
h2 {
  color: #333;
  border-bottom: 2px solid #eee;
  padding-bottom: 10px;
}

```

end