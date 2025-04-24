python兴趣匹配算法，用于推荐好友，短视频推荐等等领域


功能列表：

1.用户类的定义，存储用户的基本信息和兴趣。
2.计算两个用户之间兴趣的匹配分数，比较每一位是否相同。
3.根据匹配分数对候选人进行排序。
4.提供交互式输入，让用户输入自己的兴趣模式。
5.预定义候选人列表，并进行匹配演示。
6.输出匹配结果的排名，显示每个候选人的匹配分数。


step1:C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py

```python
class User:
    def __init__(self, name, gender, interests):
        self.name = name
        self.gender = gender
        self.interests = interests

    def get_match_score(self, other):
        return sum(c1 == c2 for c1, c2 in zip(self.interests, other.interests))

def match_users(current_user, candidates):
    return sorted(candidates, 
                 key=lambda u: u.get_match_score(current_user), 
                 reverse=True)

def run_demo():
    my_interest = input("请输入你的兴趣模式（7位0/1，例如1011001）：\n")
    current_user = User("我", "MALE", my_interest)

    candidates = [
        User("张三", "MALE", "1011001"),
        User("李四", "MALE", "1110001"),
        User("王五", "MALE", "0011011"),
        User("赵六", "MALE", "1011000")
    ]

    matched = match_users(current_user, candidates)

    print("\n匹配结果排名：")
    for i, user in enumerate(matched, 1):
        score = current_user.get_match_score(user)
        print(f"{i}. {user.name} (匹配度：{score}/7)")

if __name__ == "__main__":
    run_demo()
```

step2:运行结果

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py
请输入你的兴趣模式（7位0/1，例如1011001）：
1011001

匹配结果排名：
1. 张三 (匹配度：7/7)
2. 赵六 (匹配度：6/7)
3. 李四 (匹配度：5/7)
4. 王五 (匹配度：5/7)
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py
请输入你的兴趣模式（7位0/1，例如1011001）：
1100001

匹配结果排名：
1. 李四 (匹配度：6/7)
2. 张三 (匹配度：4/7)
3. 赵六 (匹配度：3/7)
4. 王五 (匹配度：2/7)
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1>

```

end