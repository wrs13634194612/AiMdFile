说明：我计划用angular+mysql+nodejs，做一套问卷调查系统，
1.先设计数据库表，
2.然后添加模拟数据，
3.然后写几个查询方法
4.然后用nodejs写service服务，查询mysql数据
5.然后写contrller路由，指向调用service方法
6.在angular里面写http请求
7.拿到json数据，解析
8.把解析的数据，展示在angular界面上
效果图:
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/322a12ff6e734fe8b56d975aef79af19.png#pic_center)

step1:mysql 建表，添加数据，写查询方法

```sql

-- 1. 问卷表
CREATE TABLE survey (
                        id INT PRIMARY KEY AUTO_INCREMENT,
                        title VARCHAR(255) NOT NULL,
                        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- 2. 问题表（包含题型）
CREATE TABLE question (
                          id INT PRIMARY KEY AUTO_INCREMENT,
                          survey_id INT NOT NULL,
                          content TEXT NOT NULL,
                          type ENUM('single_choice', 'multiple_choice', 'text') NOT NULL,
                          FOREIGN KEY (survey_id) REFERENCES survey(id)
);
-- 3. 选项表（用于单选/多选题）
CREATE TABLE `option` (
                          id INT PRIMARY KEY AUTO_INCREMENT,
                          question_id INT NOT NULL,
                          content VARCHAR(255) NOT NULL,
                          FOREIGN KEY (question_id) REFERENCES question(id)
);
-- 4. 用户表
CREATE TABLE user (
                      id INT PRIMARY KEY AUTO_INCREMENT,
                      username VARCHAR(50) NOT NULL
);
-- 5. 答案表（统一存储所有类型答案）
CREATE TABLE answer (
                        id INT PRIMARY KEY AUTO_INCREMENT,
                        user_id INT NOT NULL,
                        question_id INT NOT NULL,
                        option_id INT DEFAULT NULL,
                        answer_text TEXT DEFAULT NULL,
                        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                        FOREIGN KEY (user_id) REFERENCES user(id),
                        FOREIGN KEY (question_id) REFERENCES question(id),
                        FOREIGN KEY (option_id) REFERENCES `option`(id)
);

-- 插入问卷
INSERT INTO survey (title) VALUES
                               ('消费者满意度调查'),
                               ('产品使用习惯调查');
-- 插入问题（前3题属于问卷1）
INSERT INTO question (survey_id, content, type) VALUES
                                                    (1, '您对产品的满意度如何？', 'single_choice'),
                                                    (1, '您通过哪些渠道了解我们？', 'multiple_choice'),
                                                    (1, '请提出改进建议', 'text'),
                                                    (2, '您每天使用产品的频率？', 'single_choice');
-- 插入选项（问题1有4个选项，问题2有4个选项，问题4有3个选项）
INSERT INTO `option` (question_id, content) VALUES
                                                (1, '非常满意'), (1, '满意'), (1, '一般'), (1, '不满意'),
                                                (2, '互联网广告'), (2, '朋友推荐'), (2, '社交媒体'), (2, '其他'),
                                                (4, '1-3次'), (4, '4-6次'), (4, '6次以上');
-- 插入用户
INSERT INTO user (username) VALUES
                                ('张三'), ('李四'), ('王五');
-- 插入答案（模拟不同用户的回答）
-- 用户1的回答
INSERT INTO answer (user_id, question_id, option_id, answer_text) VALUES
                                                                      (1, 1, 1, NULL),
                                                                      (1, 2, 5, NULL), (1, 2, 6, NULL),
                                                                      (1, 3, NULL, '增加更多功能');

INSERT INTO answer (user_id, question_id, option_id) VALUES
    (1, 4, 5);
-- 用户2的回答
INSERT INTO answer (user_id, question_id, option_id) VALUES
                                                         (2, 1, 2), (2, 2, 5), (2, 2, 7);
-- 用户3的回答
INSERT INTO answer (user_id, question_id, option_id, answer_text) VALUES
                                                                      (3, 1, 1, NULL),
                                                                      (3, 2, 6, NULL), (3, 2, 8, NULL),
                                                                      (3, 3, NULL, '服务态度很好');

# 1. 查询单选问题选项统计
SELECT
    o.content AS '选项内容',
    COUNT(a.id) AS '选择次数'
FROM question q
         JOIN `option` o ON q.id = o.question_id
         LEFT JOIN answer a ON o.id = a.option_id AND q.id = a.question_id
WHERE q.id = 1
GROUP BY o.id
ORDER BY COUNT(a.id) DESC;


# 2. 查询多选题选项统计
SELECT
    o.content AS '选项内容',
    COUNT(a.id) AS '被选次数'
FROM question q
         JOIN `option` o ON q.id = o.question_id
         LEFT JOIN answer a ON o.id = a.option_id AND q.id = a.question_id
WHERE q.id = 2  -- 查询问题ID=2的多选题
GROUP BY o.id
ORDER BY COUNT(a.id) DESC;

# 3. 查询文本题反馈内容
SELECT
    u.username AS '用户',
    a.answer_text AS '反馈内容'
FROM answer a
         JOIN user u ON a.user_id = u.id
WHERE a.question_id = 3;

# 4. 按问卷统计整体回答率


SELECT
    s.title AS '问卷标题',
    COUNT(DISTINCT CONCAT(a.user_id, '-', q.id)) AS '实际回答数',
    COUNT(DISTINCT q.id) * COUNT(DISTINCT u.id) AS '理论最大回答数',
    ROUND(
            COUNT(DISTINCT CONCAT(a.user_id, '-', q.id)) /
            (COUNT(DISTINCT q.id) * COUNT(DISTINCT u.id)) * 100,
            2
    ) AS '回答率(%)'
FROM survey s
         LEFT JOIN question q ON s.id = q.survey_id
         CROSS JOIN user u
         LEFT JOIN answer a ON q.id = a.question_id AND u.id = a.user_id
GROUP BY s.id;





# 5. 按用户统计答题记录
SELECT
    u.username AS '用户名',
    s.title AS '问卷标题',
    COUNT(DISTINCT a.question_id) AS '已答问题数',
    COUNT(DISTINCT q.id) AS '总问题数',
    ROUND(
            COUNT(DISTINCT a.question_id) /
            COUNT(DISTINCT q.id) * 100,
            2
    ) AS '完成率(%)'
FROM user u
         CROSS JOIN survey s
         LEFT JOIN question q ON s.id = q.survey_id
         LEFT JOIN answer a ON u.id = a.user_id AND q.id = a.question_id
GROUP BY u.id, s.id;


# 分割线


-- 1. 单选问题选项统计
SELECT
    `option`.content AS '选项内容',
    COUNT(answer.id) AS '选择次数'
FROM question
         JOIN `option` ON question.id = `option`.question_id
         LEFT JOIN answer
                   ON `option`.id = answer.option_id
                       AND question.id = answer.question_id
WHERE question.id = 1
GROUP BY `option`.id
ORDER BY COUNT(answer.id) DESC;
-- 2. 多选问题选项统计
SELECT
    `option`.content AS '选项内容',
    COUNT(answer.id) AS '被选次数'
FROM question
         JOIN `option` ON question.id = `option`.question_id
         LEFT JOIN answer
                   ON `option`.id = answer.option_id
                       AND question.id = answer.question_id
WHERE question.id = 2
GROUP BY `option`.id
ORDER BY COUNT(answer.id) DESC;
-- 3. 文本题反馈内容
SELECT
    user.username AS '用户',
    answer.answer_text AS '反馈内容'
FROM answer
         JOIN user ON answer.user_id = user.id
WHERE answer.question_id = 3;
-- 4. 按问卷统计回答率
SELECT
    survey.title AS '问卷标题',
    COUNT(DISTINCT CONCAT(answer.user_id, '-', question.id)) AS '实际回答数',
    COUNT(DISTINCT question.id) * COUNT(DISTINCT user.id) AS '理论最大回答数',
    ROUND(
            COUNT(DISTINCT CONCAT(answer.user_id, '-', question.id)) /
            (COUNT(DISTINCT question.id) * COUNT(DISTINCT user.id)) * 100,
            2
    ) AS '回答率(%)'
FROM survey
         LEFT JOIN question ON survey.id = question.survey_id
         CROSS JOIN user
         LEFT JOIN answer
                   ON question.id = answer.question_id
                       AND user.id = answer.user_id
GROUP BY survey.id;
-- 5. 按用户统计答题记录
SELECT
    user.username AS '用户名',
    survey.title AS '问卷标题',
    COUNT(DISTINCT answer.question_id) AS '已答问题数',
    COUNT(DISTINCT question.id) AS '总问题数',
    ROUND(
            COUNT(DISTINCT answer.question_id) /
            COUNT(DISTINCT question.id) * 100,
            2
    ) AS '完成率(%)'
FROM user
         CROSS JOIN survey
         LEFT JOIN question ON survey.id = question.survey_id
         LEFT JOIN answer
                   ON user.id = answer.user_id
                       AND question.id = answer.question_id
GROUP BY user.id, survey.id;
```

step2:service层查询数据库中表的数据
C:\Users\Administrator\WebstormProjects\untitled4\question-crud.js 
```javascript
const config = require('./db');
class SurveyCrudService {
  // [1] 查询单选题统计
  static async getSingleQuestionStats(questionId) {
    const [rows] = await config.query(`
      SELECT
        o.content AS option_content,
        COUNT(a.id) AS selection_count
      FROM question q
      JOIN \`option\` o ON q.id = o.question_id
      LEFT JOIN answer a ON o.id = a.option_id AND q.id = a.question_id
      WHERE q.id = ?
      GROUP BY o.id
      ORDER BY selection_count DESC
    `, [questionId]);
    return rows;
  }
  // [2] 查询多选题统计
  static async getMultipleQuestionStats(questionId) {
    const [rows] = await config.query(`
      SELECT
        o.content AS option_content,
        COUNT(a.id) AS selected_count
      FROM question q
      JOIN \`option\` o ON q.id = o.question_id
      LEFT JOIN answer a ON o.id = a.option_id AND q.id = a.question_id
      WHERE q.id = ?
      GROUP BY o.id
      ORDER BY selected_count DESC
    `, [questionId]);
    return rows;
  }
  // [3] 查询文本题反馈
  static async getTextFeedbacks(questionId) {
    const [rows] = await config.query(`
      SELECT
        u.username AS user,
        a.answer_text AS feedback_content
      FROM answer a
      JOIN user u ON a.user_id = u.id
      JOIN question q ON a.question_id = q.id
      WHERE q.type = 'text' AND q.id = ?
    `, [questionId]);
    return rows;
  }
  // [4] 查询问卷回答率
  static async getSurveyResponseRate(surveyId) {
    const [rows] = await config.query(`
      SELECT
        s.title AS survey_title,
        COUNT(DISTINCT CONCAT(a.user_id, '-', q.id)) AS actual_responses,
        COUNT(DISTINCT q.id) * COUNT(DISTINCT u.id) AS theoretical_max_responses,
        ROUND(
          COUNT(DISTINCT CONCAT(a.user_id, '-', q.id)) /
          (COUNT(DISTINCT q.id) * COUNT(DISTINCT u.id)) * 100,
          2
        ) AS response_rate
      FROM survey s
      LEFT JOIN question q ON s.id = q.survey_id
      CROSS JOIN user u
      LEFT JOIN answer a ON q.id = a.question_id AND u.id = a.user_id
      WHERE s.id = ?
      GROUP BY s.id
    `, [surveyId]);
    return rows[0] || {};
  }

  // [5] 查询用户答题记录
  static async getUserAnswerRecords(userId) {
    const [rows] = await config.query(`
      SELECT
        s.title AS survey_title,
        COUNT(DISTINCT a.question_id) AS answered_questions,
        COUNT(DISTINCT q.id) AS total_questions,
        ROUND(
          COUNT(DISTINCT a.question_id) /
          COUNT(DISTINCT q.id) * 100,
          2
        ) AS completion_rate
      FROM user u
      CROSS JOIN survey s
      LEFT JOIN question q ON s.id = q.survey_id
      LEFT JOIN answer a ON u.id = a.user_id AND q.id = a.question_id
      WHERE u.id = ?
      GROUP BY u.id, s.id
    `, [userId]);
    return rows;
  }


}
module.exports = SurveyCrudService;

```

step3:写controller路由，调用service层的方法 
C:\Users\Administrator\WebstormProjects\untitled4\question-spring.js
```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const crud = require('./question-crud');
const app = express();
app.use(bodyParser.json());
app.use(cors());
// [接口1] 单选题统计
app.get('/api/stats/single/:questionId', async (req, res) => {
  try {
    const data = await crud.getSingleQuestionStats(parseInt(req.params.questionId));
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
// [接口2] 多选题统计
app.get('/api/stats/multiple/:questionId', async (req, res) => {
  try {
    const data = await crud.getMultipleQuestionStats(parseInt(req.params.questionId));
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
// [接口3] 文本题反馈
app.get('/api/feedback/text/:questionId', async (req, res) => {
  try {
    const data = await crud.getTextFeedbacks(parseInt(req.params.questionId));
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
// [接口4] 问卷回答率
app.get('/api/stats/response-rate/:surveyId', async (req, res) => {
  try {
    const data = await crud.getSurveyResponseRate(parseInt(req.params.surveyId));
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// [接口5] 用户答题记录
app.get('/api/user/records/:userId', async (req, res) => {
  try {
    const data = await crud.getUserAnswerRecords(parseInt(req.params.userId));
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// 启动服务器
const PORT = process.env.PORT || 5500;
app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT}`);
});

```

step4:运行服务

```bash
PS C:\Users\Administrator\WebstormProjects\untitled4> node question-spring.js
🚀 Server running on port 5500

```

step5:在postman里面测试，

```bash
-- 1. 单选问题选项统计URL: GET http://localhost:5500/api/stats/single/1


[
    {
        "选项内容": "非常满意",
        "选择次数": 2
    },
    {
        "选项内容": "满意",
        "选择次数": 1
    },
    {
        "选项内容": "一般",
        "选择次数": 0
    },
    {
        "选项内容": "不满意",
        "选择次数": 0
    }
]


-- 2. 多选问题选项统计URL: GET http://localhost:5500/api/stats/multiple/2

[
    {
        "选项内容": "互联网广告",
        "被选次数": 2
    },
    {
        "选项内容": "朋友推荐",
        "被选次数": 2
    },
    {
        "选项内容": "社交媒体",
        "被选次数": 1
    },
    {
        "选项内容": "其他",
        "被选次数": 1
    }
]


-- 3. 文本题反馈内容URL: GET http://localhost:5500/api/feedback/text/3
[
    {
        "用户": "张三",
        "反馈内容": "增加更多功能"
    },
    {
        "用户": "王五",
        "反馈内容": "服务态度很好"
    }
]


-- 4. 按问卷统计回答率URL: GET http://localhost:5500/api/stats/response-rate/1

{
    "问卷标题": "消费者满意度调查",
    "实际回答数": 8,
    "理论最大回答数": 9,
    "回答率": "88.89"
}
 

-- 5. 按用户统计答题记录URL: GET http://localhost:5500/api/user/records/1

[
    {
        "问卷标题": "消费者满意度调查",
        "已答问题数": 3,
        "总问题数": 3,
        "完成率": "100.00"
    },
    {
        "问卷标题": "产品使用习惯调查",
        "已答问题数": 1,
        "总问题数": 1,
        "完成率": "100.00"
    }
]


```

step6:到这里，nodejs后端和mysql就写完了，接下来写angular前端展示

step7:接口返回的数据，用于json解析
C:\Users\Administrator\WebstormProjects\untitled4\src\app\question\survey.model.ts

```javascript
export interface Survey {
  // 单题统计
  singleStats?: Array<{ option_content: string; selection_count: number }>;
  // 多选统计
  multipleStats?: Array<{ option_content: string; selected_count: number }>;
  // 文本反馈
  textFeedback?: Array<{ user: string; feedback_content: string }>;
  // 回答率
  responseRate?: {
    survey_title: string;
    actual_responses: number;
    theoretical_max_responses: number;
    response_rate: string;
  };
  // 用户记录
  userRecords?: Array<{
    survey_title: string;
    answered_questions: number;
    total_questions: number;
    completion_rate: string;
  }>;
}

/*
‌选项内容‌	option_content
‌选择次数‌	selection_count
‌被选次数‌	selected_count
‌用户‌	user
‌反馈内容‌	feedback_content
‌问卷标题‌	survey_title
‌实际回答数‌	actual_responses
‌理论最大回答数‌	theoretical_max_responses
‌回答率‌	response_rate
‌已答问题数‌	answered_questions
‌总问题数‌	total_questions
‌完成率‌	completion_rate
* */

```

step8:http服务，封装的api请求
C:\Users\Administrator\WebstormProjects\untitled4\src\app\question\survey.service.ts
```javascript
// survey.service.ts
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'root'
})
export class SurveyService {
  private baseUrl = 'http://localhost:5500/api';
  constructor(private http: HttpClient) { }
  getSingleStats() {
    return this.http.get(`${this.baseUrl}/stats/single/1`);
  }
  getMultipleStats() {
    return this.http.get(`${this.baseUrl}/stats/multiple/2`);
  }
  getTextFeedback() {
    return this.http.get(`${this.baseUrl}/feedback/text/3`);
  }
  getResponseRate() {
    return this.http.get(`${this.baseUrl}/stats/response-rate/1`);
  }
  getUserRecords() {
    return this.http.get(`${this.baseUrl}/user/records/1`);
  }
}

```

step9:C:\Users\Administrator\WebstormProjects\untitled4\src\app\question\question.component.ts

```javascript
import { Component,OnInit } from '@angular/core';
import { SurveyService } from './survey.service';

import { Survey } from './survey.model';
import {MatCard, MatCardContent, MatCardHeader,MatCardModule} from '@angular/material/card';
import {
  MatCell,
  MatCellDef,
  MatColumnDef,
  MatHeaderCell,
  MatHeaderCellDef,
  MatHeaderRow, MatHeaderRowDef, MatRow, MatRowDef,
  MatTable
} from '@angular/material/table';
import {NgForOf, NgIf} from '@angular/common';
import {MatList, MatListItem} from '@angular/material/list';
import {MatChip, MatChipListbox} from '@angular/material/chips';
import {MatAccordion, MatExpansionPanel, MatExpansionPanelTitle,MatExpansionModule} from '@angular/material/expansion';

@Component({
  selector: 'app-question',
  imports: [
    MatCard,
    MatCardHeader,
    MatCardContent,
    MatTable,
    MatHeaderCell,
    NgIf,
    MatCardModule,
    MatCell,
    MatExpansionModule,
    MatHeaderCellDef,
    MatColumnDef,
    MatCellDef,
    MatHeaderRow,
    MatRow,
    MatList,
    MatListItem,
    MatChipListbox,
    MatChip,
    MatAccordion,
    MatExpansionPanel,
    MatExpansionPanelTitle,
    MatRowDef,
    MatHeaderRowDef,
    NgForOf
  ],
  templateUrl: './question.component.html',
  styleUrl: './question.component.css'
})
export class QuestionComponent implements OnInit {
  data: Survey = {};
  constructor(private surveyService: SurveyService) {}
  ngOnInit() {
    this.loadAllData();
  }
  private loadAllData() {
    this.surveyService.getSingleStats().subscribe(res =>{
      this.data.singleStats = res as any
      console.log('getSingleStats', res)
    });

    this.surveyService.getMultipleStats().subscribe(res =>{
      this.data.multipleStats = res as any
      console.log('getMultipleStats', res)
    });

    this.surveyService.getTextFeedback().subscribe(res =>{
      console.log('getTextFeedback', res)
      this.data.textFeedback = res as any
    });

    this.surveyService.getResponseRate().subscribe(res =>{
      console.log('getResponseRate', res)
      this.data.responseRate = res as any
    });

    this.surveyService.getUserRecords().subscribe(res =>{
      console.log('getUserRecords', res)
      this.data.userRecords = res as any
    });
  }
}

```

step10:C:\Users\Administrator\WebstormProjects\untitled4\src\app\question\question.component.html

```xml

<!-- survey.component.html -->
<div class="dashboard-container">
  <!-- 响应率卡片 -->
  <mat-card *ngIf="data.responseRate" class="response-rate">
    <mat-card-header>
      <mat-card-title>{{ data.responseRate.survey_title }}</mat-card-title>
    </mat-card-header>
    <mat-card-content>
      <p>回答率: {{ data.responseRate.response_rate }}%</p>
      <p>实际回答数 / 最大回答数: {{ data.responseRate.actual_responses }} / {{ data.responseRate.theoretical_max_responses	 }}</p>
    </mat-card-content>
  </mat-card>
  <!-- 用户记录表格 -->
  <mat-table *ngIf="data.userRecords" [dataSource]="data.userRecords" class="user-records">
    <ng-container matColumnDef="问卷标题">
      <mat-header-cell *matHeaderCellDef> 问卷标题 </mat-header-cell>
      <mat-cell *matCellDef="let record"> {{ record.survey_title }} </mat-cell>
    </ng-container>
    <ng-container matColumnDef="完成率">
      <mat-header-cell *matHeaderCellDef> 完成率 </mat-header-cell>
      <mat-cell *matCellDef="let record"> {{ record.completion_rate }}% </mat-cell>
    </ng-container>
    <mat-header-row *matHeaderRowDef="['问卷标题', '完成率']"></mat-header-row>
    <mat-row *matRowDef="let row; columns: ['问卷标题', '完成率'];"></mat-row>
  </mat-table>
  <!-- 单选题统计 -->
  <mat-card *ngIf="data.singleStats">
    <h3>单选题统计</h3>
    <mat-list>
      <mat-list-item *ngFor="let item of data.singleStats">
        {{ item.option_content }}: {{ item.selection_count }}次
      </mat-list-item>
    </mat-list>
  </mat-card>
  <!-- 多选题统计 -->
  <mat-card *ngIf="data.multipleStats">
    <h3>多选题统计</h3>
    <mat-chip-listbox>
      <mat-chip *ngFor="let item of data.multipleStats">
        {{ item.option_content }} ({{ item.selected_count }})
      </mat-chip>
    </mat-chip-listbox>
  </mat-card>
  <!-- 文本反馈 -->
  <mat-card *ngIf="data.textFeedback">
    <h3>用户反馈</h3>
    <mat-accordion>
      <mat-expansion-panel *ngFor="let feedback of data.textFeedback">
        <mat-expansion-panel-header>
          <mat-panel-title>{{ feedback.user }}</mat-panel-title>
        </mat-expansion-panel-header>
        <p>{{ feedback.feedback_content }}</p>
      </mat-expansion-panel>
    </mat-accordion>
  </mat-card>
</div>

```

step11: C:\Users\Administrator\WebstormProjects\untitled4\src\app\question\question.component.css

```css
.dashboard-container {
  padding: 20px;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  mat-card {
    margin-bottom: 20px;
    h3 {
      margin: 0 0 15px;
      color: #3f51b5;
    }
  }
  .response-rate {
    grid-column: 1 / -1;
    background: #f5f5f5;

    mat-card-content {
      display: flex;
      justify-content: space-around;
      font-size: 1.2em;
    }
  }
  .user-records {
    width: 100%;
    margin-top: 20px;
  }
  mat-chip {
    margin: 5px;
  }
}

```


end