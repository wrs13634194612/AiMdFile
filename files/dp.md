说明：python使用pandas+mysql学生成绩排名
我计划用python去处理学生成绩数据，

1.简单的学生成绩录入，姓名 ，语文成绩，数学成绩，英语成绩，
2.新增，总分排名，英语成绩排名，数学成绩排名，语文成绩排名，
3.存储过程随机生成一万条数据
4.用python生成排名后的表格文件，存在本地
step1:数据结构

```bash
仿照下面的样式，给我生成十条sql数据，建表，添加数据

{
  "students": [
    {
      "id": 0,
      "name": "Zhang San",
      "scores": {
        "chinese": 88,
        "math": 92,
        "english": 65
      }
    },
    {
      "id": 1,
      "name": "Li Si",
      "scores": {
        "chinese": 69,
        "math": 72,
        "english": 38
      }
    }
  ]
}
```

step2:sql 存储过程

```sql


show databases;

DROP TABLE users;


SHOW CREATE TABLE db_school.users;

show tables;

use db_school;

SELECT * FROM db_school.jewelry_categories;

CREATE DATABASE db_school;


-- 创建用户表 (对应 POST/PUT/DELETE 操作)
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY COMMENT '用户唯一ID',
    name VARCHAR(50) NOT NULL COMMENT '用户姓名',
    email VARCHAR(255) NOT NULL UNIQUE COMMENT '唯一邮箱地址',
    age INT CHECK (age BETWEEN 0 AND 150) COMMENT '年龄（0-150岁）',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

TRUNCATE users;

SELECT *FROM users;

-- 学生表
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
);

-- 成绩表（使用 student_id 关联学生表）
CREATE TABLE scores (
    student_id INT,
    chinese INT CHECK (chinese BETWEEN 0 AND 100),
    math INT CHECK (math BETWEEN 0 AND 100),
    english INT CHECK (english BETWEEN 0 AND 100),
    FOREIGN KEY (student_id) REFERENCES students(id)
);

-- 插入学生信息
INSERT INTO students (id, name) VALUES
(0, 'Zhang San'),
(1, 'Li Si'),
(2, 'Wang Wu'),
(3, 'Zhao Liu'),
(4, 'Qian Qi'),
(5, 'Sun Ba'),
(6, 'Zhou Jiu'),
(7, 'Wu Shi'),
(8, 'Zheng Shiyi'),
(9, 'Feng Shier');

-- 插入成绩信息（与 students.id 严格对应）
INSERT INTO scores (student_id, chinese, math, english) VALUES
(0, 88, 92, 65),
(1, 69, 72, 38),
(2, 75, 85, 90),
(3, 82, 78, 62),
(4, 95, 88, 73),
(5, 63, 95, 41),
(6, 71, 68, 57),
(7, 84, 77, 82),
(8, 79, 83, 69),
(9, 91, 89, 76);


SELECT
    s.id,
    s.name,
    sc.chinese,
    sc.math,
    sc.english
FROM students s
JOIN scores sc ON s.id = sc.student_id
WHERE s.id = 2;  -- 替换为实际ID





SELECT
    s.id,
    s.name,
    sc.chinese,
    sc.math,
    sc.english
FROM students s
JOIN scores sc ON s.id = sc.student_id;




-- 创建学生表
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;



-- 创建成绩表（带外键约束）
CREATE TABLE scores (
    id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT NOT NULL,
    chinese INT CHECK (chinese BETWEEN 0 AND 100),
    math INT CHECK (math BETWEEN 0 AND 100),
    english INT CHECK (english BETWEEN 0 AND 100),
    FOREIGN KEY (student_id) REFERENCES students(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

DELIMITER $$
CREATE PROCEDURE InsertStudentsAndScores(IN num_rows INT)
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE student_name VARCHAR(50);



    -- 事务开始
    START TRANSACTION;
    WHILE i < num_rows DO

        -- 生成随机学生姓名（示例用简单随机组合）
        SET student_name = CONCAT(
            ELT(FLOOR(1 + RAND() * 4), '张', '王', '李', '赵'),
            ELT(FLOOR(1 + RAND() * 4), '三', '四', '五', '六')
        );



        -- 插入学生表
        INSERT INTO students (name) VALUES (student_name);



        -- 插入成绩表（使用LAST_INSERT_ID()获取学生ID）
        INSERT INTO scores (student_id, chinese, math, english)
        VALUES (
            LAST_INSERT_ID(),
            FLOOR(RAND() * 101),  -- 生成0-100的随机分数
            FLOOR(RAND() * 101),
            FLOOR(RAND() * 101)
        );

        SET i = i + 1;
    END WHILE;
    COMMIT;
END$$
DELIMITER ;



-- 调用存储过程生成10000条数据
CALL InsertStudentsAndScores(10000);


-- 统计 students 表记录数
SELECT COUNT(*) AS student_count FROM students;

-- 统计 scores 表记录数
SELECT COUNT(*) AS score_count FROM scores;





# 分割线


-- 清空表重置自增ID（可选）
-- 创建学生表
CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;



-- 创建成绩表（带外键约束）
CREATE TABLE scores (
    id INT AUTO_INCREMENT PRIMARY KEY,
    student_id INT NOT NULL,
    chinese INT CHECK (chinese BETWEEN 0 AND 100),
    math INT CHECK (math BETWEEN 0 AND 100),
    english INT CHECK (english BETWEEN 0 AND 100),
    FOREIGN KEY (student_id) REFERENCES students(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 修改后的存储过程
DELIMITER $$
CREATE PROCEDURE InsertStudentsAndScores(IN num_rows INT)
BEGIN
    DECLARE i INT DEFAULT 0;
    DECLARE current_id INT;

    -- 事务每1000条提交一次
    DECLARE commit_frequency INT DEFAULT 1000;
    DECLARE commit_counter INT DEFAULT 0;

    START TRANSACTION;

    WHILE i < num_rows DO
        -- 插入学生表（生成 student_ 前缀 + 自增ID）
        INSERT INTO students (name) VALUES ('student_placeholder');

        -- 获取当前自增ID
        SET current_id = LAST_INSERT_ID();

        -- 更新学生名为 student_[id]
        UPDATE students
        SET name = CONCAT('student_', current_id)
        WHERE id = current_id;

        -- 插入成绩表
        INSERT INTO scores (student_id, chinese, math, english)
        VALUES (
            current_id,
            FLOOR(RAND() * 101),  -- 随机生成0-100
            FLOOR(RAND() * 101),
            FLOOR(RAND() * 101)
        );

        SET i = i + 1;
        SET commit_counter = commit_counter + 1;

        -- 分批提交事务
        IF commit_counter >= commit_frequency THEN
            COMMIT;
            START TRANSACTION;
            SET commit_counter = 0;
        END IF;
    END WHILE;

    COMMIT;
END
$$
DELIMITER ;

-- 调用存储过程生成10000条数据
CALL InsertStudentsAndScores(10000);

-- 验证数据
SELECT
    s.id,
    s.name,
    sc.chinese,
    sc.math,
    sc.english
FROM students s
JOIN scores sc ON s.id = sc.student_id
LIMIT 5;


SELECT
    s.id AS 学生ID,
    s.name AS 学生姓名,
    sc.chinese AS 语文,
    sc.math AS 数学,
    sc.english AS 英语,
    (sc.chinese + sc.math + sc.english) AS 总分,
    RANK() OVER (ORDER BY (sc.chinese + sc.math + sc.english) DESC) AS 总分排名,
    RANK() OVER (ORDER BY sc.chinese DESC) AS 语文排名,
    RANK() OVER (ORDER BY sc.math DESC) AS 数学排名,
    RANK() OVER (ORDER BY sc.english DESC) AS 英语排名
FROM students s
JOIN scores sc ON s.id = sc.student_id
 ;


SELECT
    s.id AS 'id',
    s.name AS '学生姓名',
    sc.chinese AS '语文',
    sc.math AS '数学',
    sc.english AS '英语',
    (sc.chinese + sc.math + sc.english) AS '总分',
    -- 总分排名（允许并列，例如 1,1,3）
    RANK() OVER (ORDER BY (sc.chinese + sc.math + sc.english) DESC) AS '总分排名',
    -- 语文排名（允许并列）
    RANK() OVER (ORDER BY sc.chinese DESC) AS '语文排名',
    -- 数学排名（允许并列）
    RANK() OVER (ORDER BY sc.math DESC) AS '数学排名',
    -- 英语排名（允许并列）
    RANK() OVER (ORDER BY sc.english DESC) AS '英语排名'
FROM students s
JOIN scores sc ON s.id = sc.student_id
ORDER BY '总分排名' ASC  -- 按总分从高到低排序
LIMIT 5;


SELECT
    s.id AS 学生ID,
    s.name AS 学生姓名,
    sc.chinese AS 语文,
    sc.math AS 数学,
    sc.english AS 英语,
    (sc.chinese + sc.math + sc.english) AS 总分,
    RANK() OVER (ORDER BY (sc.chinese + sc.math + sc.english) DESC) AS 总分排名,
    RANK() OVER (ORDER BY sc.chinese DESC) AS 语文排名,
    RANK() OVER (ORDER BY sc.math DESC) AS 数学排名,
    RANK() OVER (ORDER BY sc.english DESC) AS 英语排名
FROM students s
JOIN scores sc ON s.id = sc.student_id
LIMIT 5;
```

step3:python访问mysql，生成文档

```python
import pymysql.cursors
from typing import Dict, List, Optional
import json
import pandas as pd

# 数据库连接配置
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_school',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}


# ---------------------- 通用数据库操作函数 ----------------------
def execute_query(query: str, params=None, fetch: bool = True) -> Optional[List[Dict]]:
    """执行 SQL 查询并返回结果"""
    connection = pymysql.connect(**DB_CONFIG)
    try:
        with connection.cursor() as cursor:
            cursor.execute(query, params)
            result = cursor.fetchall() if fetch else None
            connection.commit()
        return result
    except Exception as e:
        connection.rollback()
        raise RuntimeError(f"数据库操作失败: {str(e)}")
    finally:
        connection.close()


# ---------------------- 业务函数 ----------------------
def get_student_ranking(limit: int = None) -> List[Dict]:
    """
    获取学生成绩及排名信息
    :param limit: 限制返回条数（可选）
    :return: 原始数据列表
    """
    ranking_query = """
    SELECT
        s.id AS '学号',
        s.name AS '姓名',
        sc.chinese AS '语文',
        sc.math AS '数学',
        sc.english AS '英语',
        (sc.chinese + sc.math + sc.english) AS '总分',
        RANK() OVER (ORDER BY (sc.chinese + sc.math + sc.english) DESC) AS '总分排名',
        RANK() OVER (ORDER BY sc.chinese DESC) AS '语文排名',
        RANK() OVER (ORDER BY sc.math DESC) AS '数学排名',
        RANK() OVER (ORDER BY sc.english DESC) AS '英语排名'
    FROM students s
    JOIN scores sc ON s.id = sc.student_id
    ORDER BY 总分排名 ASC
    """
    if limit is not None:
        ranking_query += f" LIMIT {limit}"

    try:
        return execute_query(ranking_query)
    except Exception as e:
        raise RuntimeError(f"查询排名失败: {str(e)}")


def export_to_excel(data: List[Dict], filename: str = "学生排名.xlsx") -> str:
    """
    导出数据到Excel文件
    :param data: 要导出的数据列表
    :param filename: 输出文件名（默认：学生排名.xlsx）
    :return: 文件保存路径
    """
    try:
        # 转换为DataFrame
        df = pd.DataFrame(data)

        # 调整列顺序（可选）
        columns_order = ['学号', '姓名', '语文', '数学', '英语', '总分',
                         '总分排名', '语文排名', '数学排名', '英语排名']
        df = df[columns_order]

        # 保存Excel文件
        df.to_excel(filename, index=False, engine='openpyxl')

        print(f"Excel文件已生成: {filename}")
        return filename
    except ImportError:
        raise RuntimeError("请先安装依赖库：pip install pandas openpyxl")
    except Exception as e:
        raise RuntimeError(f"导出Excel失败: {str(e)}")


def print_json(data: List[Dict]) -> None:
    """打印格式化JSON数据"""
    try:
        json_str = json.dumps(
            data,
            ensure_ascii=False,
            indent=2,
            default=str
        )
        print("JSON格式数据:")
        print(json_str)
    except Exception as e:
        print(f"JSON格式化失败: {str(e)}")


# ---------------------- 示例用法 ----------------------
if __name__ == "__main__":
    try:
        # 获取排名数据（默认获取全部）
        ranking_data = get_student_ranking()

        # 打印JSON格式
        print("\n===== JSON格式预览 =====")
        print_json(ranking_data[:5])  # 仅打印前5条

        # 导出完整数据到Excel
        print("\n===== 导出Excel文件 =====")
        excel_path = export_to_excel(ranking_data)
        print(f"文件路径: {excel_path}")

    except RuntimeError as e:
        print(f"操作失败: {str(e)}")
    except Exception as e:
        print(f"未知错误: {str(e)}")
```

step4:运行结果，可以用wps打开本地文档查看


```bash
id	学生姓名		语文	数学	英语	总分	总分排名	语文排名	数学排名	英语排名
12	student_12	98		100		97		295		1		2		1		1
45	student_45	95		99		95		289		2		5		3		4
78	student_78	96		98		93		287		3		3		5		7
23	student_23	94		97		96		287		3		8		8		3
56	student_56	97		95		94		286		5		4		10		5
```


end