说明：
我计划用python的fastapi框架，实现操作MySQL数据库的表，实现增删改查的操作，并且在postman里面测试
step1: 安装数据库依赖

```bash
pip install fastapi uvicorn pymysql
```

step2:C:\Users\Administrator\PycharmProjects\FastAPIProject\main.py

```python
from fastapi import FastAPI, HTTPException, Body, Path
import pymysql.cursors
from typing import Optional, Dict
app = FastAPI()
# 数据库连接配置
DB_CONFIG = {
    'host': 'localhost',
    'user': 'root',
    'password': '123456',
    'db': 'db_spring',
    'charset': 'utf8mb4',
    'cursorclass': pymysql.cursors.DictCursor
}
# 查询数据库的函数
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
# 查询表数据的 API 端点
@app.get("/query")
async def query_table(table_name: str):
    query = f"SELECT * FROM {table_name}"
    try:
        data = query_database(query)
        return {"status": "success", "data": data}
    except HTTPException as e:
        return {"status": "error", "message": e.detail}

# 新增用户 (POST)
@app.post("/users")
async def create_user(
    user_data: Dict = Body(..., example={
        "name": "诸葛亮",
        "email": "zhugeliang@shu.com",
        "age": 54
    })
):
    try:
        # 检查邮箱唯一性
        check_query = "SELECT id FROM users WHERE email = %s"
        exist = query_database(check_query, (user_data["email"],))
        if exist:
            raise HTTPException(409, "邮箱已存在")

        # 执行插入操作
        insert_query = """
        INSERT INTO users (name, email, age)
        VALUES (%s, %s, %s)
        """
        with pymysql.connect(**DB_CONFIG) as conn:
            with conn.cursor() as cursor:
                cursor.execute(insert_query, (
                    user_data["name"],
                    user_data["email"],
                    user_data["age"]
                ))
                new_id = cursor.lastrowid
                conn.commit()
        return {"status": "success", "id": new_id}
    except KeyError as e:
        raise HTTPException(400, f"缺失必要字段: {e}")


# 更新用户 (PUT)
@app.put("/users/{user_id}")
async def update_user(
        user_id: int = Path(..., gt=0),
        update_data: Dict = Body(..., example={
            "name": "更新名称",
            "age": 99
        })
):
    try:
        # 检查用户是否存在
        exist = query_database("SELECT id FROM users WHERE id = %s", (user_id,))
        if not exist:
            raise HTTPException(404, "用户不存在")

        # 动态生成更新语句
        set_clause = ", ".join([f"{k}=%s" for k in update_data.keys()])
        update_query = f"UPDATE users SET {set_clause} WHERE id = %s"

        with pymysql.connect(**DB_CONFIG) as conn:
            with conn.cursor() as cursor:
                cursor.execute(update_query, (
                    *update_data.values(),
                    user_id
                ))
                conn.commit()
        return {"status": "success", "affected_rows": cursor.rowcount}
    except pymysql.err.IntegrityError:
        raise HTTPException(409, "邮箱冲突或数据约束失败")
# 删除用户 (DELETE)
@app.delete("/users/{user_id}")
async def delete_user(user_id: int = Path(..., gt=0)):
    try:
        with pymysql.connect(**DB_CONFIG) as conn:
            with conn.cursor() as cursor:
                cursor.execute(
                    "DELETE FROM users WHERE id = %s",
                    (user_id,)
                )
                conn.commit()
                if cursor.rowcount == 0:
                    raise HTTPException(404, "用户不存在")
        return {"status": "success", "deleted_id": user_id}
    except pymysql.err.Error as e:
        raise HTTPException(500, f"数据库错误: {e}")
# 启动应用
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

step3:运行

```bash
C:\Users\Administrator\PycharmProjects\FastAPIProject\.venv\Scripts\python.exe -m uvicorn main:app --reload 
INFO:     Will watch for changes in these directories: ['C:\\Users\\Administrator\\PycharmProjects\\FastAPIProject']
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Started reloader process [12132] using StatReload
INFO:     Started server process [2000]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     127.0.0.1:53269 - "DELETE /users/16 HTTP/1.1" 200 OK
```

 在postman里面，测试验证成功

end