说明：
c++算法实现贪吃蛇
效果图：

step0：游戏规则和思路

```bash
用c++写贪吃蛇ai算法

0.在控制台展示位置坐标，把游戏界面作为x轴和y轴，左下角为(0,0) ,右上角为(x_max,y_max)
1. 让ai自动控制蛇的移动，通过吃掉地图上的食物（如苹果🍎）使蛇身变长。
2. ai需要避免碰撞，蛇头不能触碰墙壁、障碍物或自身身体，否则游戏结束。
3. 尽可能延长蛇身并存活更长时间。
4.ai必须保持智能，聪明，不能犯低级错误

为了实现一个智能的贪吃蛇AI，我们需要分步骤构建游戏框架并实现AI算法。以下是详细的实现过程：

第一步：游戏框架和基本结构
我们首先定义游戏中的基本元素，包括点、方向、蛇和游戏状态。


第二步：AI算法实现
AI的核心是路径寻找和决策逻辑，使用BFS寻找最短路径，洪水填充评估生存空间。

第三步：主函数和游戏循环
最后，设置游戏的主循环，处理输入、更新和渲染

说明
路径寻找：使用BFS算法寻找当前蛇头到食物的最短路径，确保路径安全。
生存策略：当无法找到路径时，使用洪水填充算法评估每个方向的可行区域，选择最安全的方向。
碰撞检测：精确模拟蛇移动后的身体状态，避免自碰撞和墙壁碰撞。
动态更新：每次移动后更新蛇的状态，检查是否吃到食物或发生碰撞。
该实现能够确保AI智能地寻找食物并尽可能延长生存时间，同时避免低级错误。


# 贪吃蛇AI实现思路分析
## 第一步：游戏框架构建
1. **基础数据结构**
   - `Point`结构体表示坐标点，实现哈希支持容器操作
   - `Direction`枚举定义移动方向，配套`opposite()`获取反向

2. **蛇类实现(Snake)**
   - 使用`deque`存储身体坐标
   - 方向控制逻辑：禁止立即反向移动
   - 移动机制：
     - `getNextHead()`计算头部预测位置
     - `move()`处理身体增长/正常移动

3. **游戏状态管理(Game)**
   - 地图边界管理(x_max/y_max)
   - 食物生成：
     - 随机生成并排除与蛇身重叠
   - 碰撞检测：
     - 边界碰撞
     - 自碰撞检测
   - 状态更新：
     - 同步食物增长与身体变化

## 第二步：AI核心算法
1. **决策流程**
   ```mermaid
   graph TD
     A[获取可能方向] --> B{路径可达?}
     B -- 是 --> C[选择最短路径]
     B -- 否 --> D[洪水填充找安全区]
     C --> E[执行最优方向]
     D --> E

2.路径寻找策略
BFS最短路径算法：
考虑模拟移动后的蛇身作为障碍
四方向广度优先搜索
模拟：
完全克隆蛇对象
预测未来n步的障碍物分布

3.生存策略保障
洪水填充算法：
评估每个方向的可行区域面积
优先选择最大安全区域

碰撞预判：
提前模拟移动后的自碰撞
排除会导致立即死亡的方向

第三步：游戏循环控制
渲染系统

控制台字符图形化：
'O'表示蛇头
'o'表示蛇身
'F'表示食物
坐标系处理：

Y轴反向显示（符合控制台从上到下特性）

2.运行逻辑
while(true){
    渲染界面 -> AI决策 -> 更新状态 -> 碰撞检测
}

3.失败处理
碰撞即时检测机制
游戏终止条件：
自碰撞/越界立即结束


结束提示显示"Game Over"

实现特点
双策略保障系统

主策略：积极寻食的最短路径
备用策略：生存优先的安全区域

精确状态模拟
完全克隆游戏对象进行预判
考虑食物增长带来的身体变化

高效算法组合
BFS时间复杂度：O(n)
洪水填充空间评估：O(n)
方向选择复杂度：O(4)
```

step1:C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include <iostream>
#include <vector>
#include <deque>
#include <queue>
#include <unordered_map>
#include <unordered_set>
#include <algorithm>
#include <thread>
#include <chrono>
#include <cstdlib>
#include <ctime>

struct Point {
    int x, y;
    Point(int x_, int y_) : x(x_), y(y_) {}
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
};

namespace std {
    template<> struct hash<Point> {
        size_t operator()(const Point& p) const {
            return hash<int>()(p.x) ^ hash<int>()(p.y);
        }
    };
}

enum Direction { UP, DOWN, LEFT, RIGHT };

Direction opposite(Direction dir) {
    switch (dir) {
    case UP: return DOWN;
    case DOWN: return UP;
    case LEFT: return RIGHT;
    case RIGHT: return LEFT;
    }
}

class Snake {
public:
    std::deque<Point> body;
    Direction dir;
    bool grow;

    Snake(int startX, int startY) : dir(RIGHT), grow(false) {
        body.emplace_back(startX, startY);
    }

    Point getHead() const { return body.back(); }

    void changeDirection(Direction newDir) {
        if (newDir != opposite(dir)) {
            dir = newDir;
        }
    }

    Point getNextHead() const {
        const Point& head = body.back();
        switch (dir) {
        case UP:    return Point(head.x, head.y + 1);
        case DOWN:  return Point(head.x, head.y - 1);
        case LEFT:  return Point(head.x - 1, head.y);
        case RIGHT: return Point(head.x + 1, head.y);
        }
        return head;
    }

    void move(bool willGrow) {
        body.push_back(getNextHead());
        if (!willGrow) body.pop_front();
        else grow = false;
    }
};

class Game {
public:
    int x_max, y_max;
    Snake snake;
    Point food;
    std::vector<Point> obstacles;

    Game(int x, int y) : x_max(x), y_max(y), snake(x / 2, y / 2), food(0, 0) {
        spawnFood();
    }

    void spawnFood() {
        do {
            food.x = rand() % x_max;
            food.y = rand() % y_max;
        } while (std::find(snake.body.begin(), snake.body.end(), food) != snake.body.end());
    }

    bool isCollision() {
        const Point& head = snake.getHead();
        if (head.x < 0 || head.y < 0 || head.x >= x_max || head.y >= y_max) return true;
        for (auto it = snake.body.begin(); it != prev(snake.body.end()); ++it)
            if (*it == head) return true;
        return false;
    }

    void update() {
        bool willGrow = snake.getHead() == food;
        snake.move(willGrow);
        if (willGrow) spawnFood();
    }
};

class AI {
public:
    Direction getNextDirection(const Game& game) {
        const auto& snake = game.snake;
        auto possibleDirs = getPossibleDirections(snake, game.x_max, game.y_max, game.food);
        if (possibleDirs.empty()) return snake.dir;

        Direction bestDir = possibleDirs.front();
        int shortest = INT_MAX;
        for (Direction dir : possibleDirs) {
            int pathLen = simulatePath(snake, dir, game);
            if (pathLen != -1 && pathLen < shortest) {
                shortest = pathLen;
                bestDir = dir;
            }
        }

        if (shortest != INT_MAX) return bestDir;
        return findSafestDirection(snake, possibleDirs, game);
    }

private:
    std::vector<Direction> getPossibleDirections(const Snake& snake, int x_max, int y_max, const Point& food) {
        std::vector<Direction> dirs;
        const Point head = snake.getHead();

        for (Direction dir : {UP, DOWN, LEFT, RIGHT}) {
            if (dir == opposite(snake.dir)) continue;

            Point newHead = moveHead(head, dir);
            if (newHead.x < 0 || newHead.y < 0 || newHead.x >= x_max || newHead.y >= y_max) continue;

            bool willGrow = newHead == food;
            auto newBody = simulateBody(snake.body, newHead, willGrow);
            if (!hasCollision(newHead, newBody)) dirs.push_back(dir);
        }
        return dirs;
    }

    int simulatePath(const Snake& snake, Direction dir, const Game& game) {
        Snake simSnake = snake;
        simSnake.changeDirection(dir);
        Point newHead = simSnake.getNextHead();
        bool willGrow = newHead == game.food;
        auto newBody = simulateBody(simSnake.body, newHead, willGrow);

        std::vector<Point> obstacles(newBody.begin(), newBody.end() - 1);
        return bfs(newHead, game.food, obstacles, game.x_max, game.y_max);
    }

    int bfs(const Point& start, const Point& end, const std::vector<Point>& obstacles, int x_max, int y_max) {
        std::queue<Point> q;
        std::unordered_map<Point, int> dist;
        q.push(start);
        dist[start] = 0;

        while (!q.empty()) {
            Point p = q.front(); q.pop();
            if (p == end) return dist[p];

            for (Direction dir : {UP, DOWN, LEFT, RIGHT}) {
                Point next = moveHead(p, dir);
                if (next.x < 0 || next.y < 0 || next.x >= x_max || next.y >= y_max) continue;
                if (dist.count(next) || std::find(obstacles.begin(), obstacles.end(), next) != obstacles.end()) continue;

                dist[next] = dist[p] + 1;
                q.push(next);
            }
        }
        return -1;
    }

    Direction findSafestDirection(const Snake& snake, const std::vector<Direction>& dirs, const Game& game) {
        Direction bestDir = dirs.front();
        int maxArea = 0;

        for (Direction dir : dirs) {
            Snake simSnake = snake;
            simSnake.changeDirection(dir);
            Point newHead = simSnake.getNextHead();
            bool willGrow = newHead == game.food;
            auto newBody = simulateBody(simSnake.body, newHead, willGrow);

            std::vector<Point> obstacles(newBody.begin(), newBody.end() - 1);
            int area = floodFill(newHead, obstacles, game.x_max, game.y_max);
            if (area > maxArea) {
                maxArea = area;
                bestDir = dir;
            }
        }
        return bestDir;
    }

    int floodFill(const Point& start, const std::vector<Point>& obstacles, int x_max, int y_max) {
        std::unordered_set<Point> visited;
        std::queue<Point> q;
        q.push(start);
        visited.insert(start);

        int count = 0;
        while (!q.empty()) {
            Point p = q.front(); q.pop();
            count++;

            for (Direction dir : {UP, DOWN, LEFT, RIGHT}) {
                Point next = moveHead(p, dir);
                if (next.x < 0 || next.y < 0 || next.x >= x_max || next.y >= y_max) continue;
                if (visited.count(next) || std::find(obstacles.begin(), obstacles.end(), next) != obstacles.end()) continue;

                visited.insert(next);
                q.push(next);
            }
        }
        return count;
    }

    Point moveHead(const Point& p, Direction dir) const {
        switch (dir) {
        case UP:    return Point(p.x, p.y + 1);
        case DOWN:  return Point(p.x, p.y - 1);
        case LEFT:  return Point(p.x - 1, p.y);
        case RIGHT: return Point(p.x + 1, p.y);
        }
        return p;
    }

    std::deque<Point> simulateBody(const std::deque<Point>& body, const Point& newHead, bool grow) const {
        std::deque<Point> newBody = body;
        newBody.push_back(newHead);
        if (!grow) newBody.pop_front();
        return newBody;
    }

    bool hasCollision(const Point& head, const std::deque<Point>& body) const {
        for (auto it = body.begin(); it != prev(body.end()); ++it)
            if (*it == head) return true;
        return false;
    }
};

int main() {
    srand(time(nullptr));
    const int WIDTH = 20, HEIGHT = 20;
    Game game(WIDTH, HEIGHT);
    AI ai;

    while (true) {
        system("cls");
        for (int y = HEIGHT - 1; y >= 0; --y) {
            for (int x = 0; x < WIDTH; ++x) {
                Point p(x, y);
                if (p == game.snake.getHead()) std::cout << "O";
                else if (std::find(game.snake.body.begin(), game.snake.body.end(), p) != game.snake.body.end()) std::cout << "o";
                else if (p == game.food) std::cout << "F";
                else std::cout << ".";
            }
            std::cout << "\n";
        }

        Direction dir = ai.getNextDirection(game);
        game.snake.changeDirection(dir);
        game.update();

        if (game.isCollision()) {
            std::cout << "Game Over!\n";
            break;
        }

        std::this_thread::sleep_for(std::chrono::milliseconds(200));
    }
    return 0;
}
```
////我是分割线
另一种实现方案
step101:

```cpp
#include <iostream>
#include <vector>
#include <deque>
#include <queue>
#include <unordered_set>
#include <cstdlib>
#include <ctime>
#include <windows.h>
#include <conio.h>

using namespace std;

// 方向枚举
enum class Direction { UP, DOWN, LEFT, RIGHT };

// 坐标点结构体
struct Point {
    int x, y;
    Point(int x = 0, int y = 0) : x(x), y(y) {}
    bool operator==(const Point& other) const {
        return x == other.x && y == other.y;
    }
    bool operator!=(const Point& other) const {
        return !(*this == other);
    }
};

// 哈希函数用于unordered_set
namespace std {
    template<> struct hash<Point> {
        size_t operator()(const Point& p) const {
            return hash<int>()(p.x) ^ hash<int>()(p.y);
        }
    };
}

// 蛇类
class Snake {
private:
    deque<Point> body;
    Direction direction;
    bool growNextMove = false;

public:
    Snake(const Point& startPos) {
        body.push_back(startPos);
        direction = Direction::RIGHT;
    }

    // 获取蛇头位置
    Point getHead() const {
        return body.front();
    }

    // 获取蛇身
    const deque<Point>& getBody() const {
        return body;
    }

    // 设置方向（不能直接反向）
    void setDirection(Direction newDir) {
        if ((direction == Direction::UP && newDir != Direction::DOWN) ||
            (direction == Direction::DOWN && newDir != Direction::UP) ||
            (direction == Direction::LEFT && newDir != Direction::RIGHT) ||
            (direction == Direction::RIGHT && newDir != Direction::LEFT)) {
            direction = newDir;
        }
    }

    // 获取下一个头部位置
    Point getNextHead() const {
        Point head = getHead();
        switch (direction) {
        case Direction::UP: return Point(head.x, head.y + 1);
        case Direction::DOWN: return Point(head.x, head.y - 1);
        case Direction::LEFT: return Point(head.x - 1, head.y);
        case Direction::RIGHT: return Point(head.x + 1, head.y);
        }
        return head;
    }

    // 移动蛇
    void move() {
        Point newHead = getNextHead();
        body.push_front(newHead);
        if (growNextMove) {
            growNextMove = false;
        }
        else {
            body.pop_back();
        }
    }

    // 标记下次移动时增长
    void grow() {
        growNextMove = true;
    }

    // 检查是否与自身碰撞
    bool checkSelfCollision() const {
        Point head = getHead();
        for (size_t i = 1; i < body.size(); ++i) {
            if (body[i] == head) {
                return true;
            }
        }
        return false;
    }

    // 克隆蛇（用于AI预测）
    Snake clone() const {
        Snake newSnake(body.front());
        newSnake.body = deque<Point>(body.begin(), body.end());
        newSnake.direction = direction;
        newSnake.growNextMove = growNextMove;
        return newSnake;
    }
};

// 游戏类
class Game {
private:
    int width, height;
    Snake snake;
    Point food;
    bool gameOver;
    int score;

    // 生成新食物
    void spawnFood() {
        unordered_set<Point> occupied;
        for (const auto& p : snake.getBody()) {
            occupied.insert(p);
        }

        vector<Point> available;
        for (int x = 0; x < width; ++x) {
            for (int y = 0; y < height; ++y) {
                Point p(x, y);
                if (occupied.find(p) == occupied.end()) {
                    available.push_back(p);
                }
            }
        }

        if (!available.empty()) {
            food = available[rand() % available.size()];
        }
        else {
            // 没有可用空间，游戏胜利
            gameOver = true;
        }
    }

    // 绘制游戏界面
    void draw() {
        system("cls");
        cout << "Score: " << score << endl;

        // 绘制上边界
        cout << "+";
        for (int x = 0; x < width; ++x) cout << "-";
        cout << "+" << endl;

        // 绘制游戏区域
        for (int y = height - 1; y >= 0; --y) {
            cout << "|";
            for (int x = 0; x < width; ++x) {
                Point p(x, y);
                if (p == snake.getHead()) {
                    cout << "O";
                }
                else if (find(snake.getBody().begin(), snake.getBody().end(), p) != snake.getBody().end()) {
                    cout << "o";
                }
                else if (p == food) {
                    cout << "F";
                }
                else {
                    cout << " ";
                }
            }
            cout << "|" << endl;
        }

        // 绘制下边界
        cout << "+";
        for (int x = 0; x < width; ++x) cout << "-";
        cout << "+" << endl;

        // 调试信息：显示坐标
        cout << "Head: (" << snake.getHead().x << "," << snake.getHead().y << ") ";
        cout << "Food: (" << food.x << "," << food.y << ")" << endl;
    }

    // 使用BFS寻找最短路径
    Direction findPathToFood() {
        Point target = food;
        Point start = snake.getHead();

        // 障碍物集合（蛇身）
        unordered_set<Point> obstacles;
        for (const auto& p : snake.getBody()) {
            if (p != start) { // 头部不算障碍物
                obstacles.insert(p);
            }
        }

        // BFS队列：存储(位置, 初始方向)
        queue<pair<Point, Direction>> q;
        // 访问标记
        unordered_set<Point> visited;

        // 初始化四个方向
        vector<pair<Point, Direction>> neighbors = {
            {Point(start.x, start.y + 1), Direction::UP},
            {Point(start.x, start.y - 1), Direction::DOWN},
            {Point(start.x - 1, start.y), Direction::LEFT},
            {Point(start.x + 1, start.y), Direction::RIGHT}
        };

        for (const auto& neighbor : neighbors) {
            Point next = neighbor.first;
            Direction dir = neighbor.second;
            if (next.x >= 0 && next.x < width && next.y >= 0 && next.y < height &&
                obstacles.find(next) == obstacles.end()) {
                q.push({ next, dir });
                visited.insert(next);
                if (next == target) {
                    return dir;
                }
            }
        }

        // BFS搜索
        while (!q.empty()) {
            auto current = q.front();
            q.pop();

            vector<pair<Point, Direction>> nextNeighbors = {
                {Point(current.first.x, current.first.y + 1), current.second},
                {Point(current.first.x, current.first.y - 1), current.second},
                {Point(current.first.x - 1, current.first.y), current.second},
                {Point(current.first.x + 1, current.first.y), current.second}
            };

            for (const auto& neighbor : nextNeighbors) {
                Point next = neighbor.first;
                Direction dir = neighbor.second;
                if (next.x >= 0 && next.x < width && next.y >= 0 && next.y < height &&
                    obstacles.find(next) == obstacles.end() &&
                    visited.find(next) == visited.end()) {

                    if (next == target) {
                        return dir;
                    }

                    visited.insert(next);
                    q.push({ next, dir });
                }
            }
        }

        // 没有找到路径，返回无效方向
        return Direction::UP; // 默认值，实际会被后续逻辑覆盖
    }

    // 洪水填充评估安全区域大小
    int floodFillSize(const Point& start, const unordered_set<Point>& obstacles) {
        unordered_set<Point> visited;
        queue<Point> q;
        q.push(start);
        visited.insert(start);
        int count = 0;

        while (!q.empty()) {
            Point p = q.front();
            q.pop();
            count++;

            vector<Point> neighbors = {
                Point(p.x, p.y + 1), // UP
                Point(p.x, p.y - 1), // DOWN
                Point(p.x - 1, p.y), // LEFT
                Point(p.x + 1, p.y)  // RIGHT
            };

            for (const auto& next : neighbors) {
                if (next.x >= 0 && next.x < width && next.y >= 0 && next.y < height &&
                    obstacles.find(next) == obstacles.end() &&
                    visited.find(next) == visited.end()) {
                    visited.insert(next);
                    q.push(next);
                }
            }
        }

        return count;
    }

    // 选择最安全的方向
    Direction findSafestDirection() {
        Point head = snake.getHead();
        Direction bestDir = Direction::UP;
        int maxSpace = -1;

        // 障碍物集合（蛇身）
        unordered_set<Point> obstacles;
        for (const auto& p : snake.getBody()) {
            if (p != head) { // 头部不算障碍物
                obstacles.insert(p);
            }
        }

        // 测试四个方向
        vector<pair<Point, Direction>> neighbors = {
            {Point(head.x, head.y + 1), Direction::UP},
            {Point(head.x, head.y - 1), Direction::DOWN},
            {Point(head.x - 1, head.y), Direction::LEFT},
            {Point(head.x + 1, head.y), Direction::RIGHT}
        };

        for (const auto& neighbor : neighbors) {
            Point next = neighbor.first;
            Direction dir = neighbor.second;
            if (next.x >= 0 && next.x < width && next.y >= 0 && next.y < height &&
                obstacles.find(next) == obstacles.end()) {

                int space = floodFillSize(next, obstacles);
                if (space > maxSpace) {
                    maxSpace = space;
                    bestDir = dir;
                }
            }
        }

        return bestDir;
    }

public:
    Game(int w, int h) : width(w), height(h), snake(Point(w / 2, h / 2)), gameOver(false), score(0) {
        srand(static_cast<unsigned int>(time(nullptr)));
        spawnFood();
    }

    // AI决策
    void aiMove() {
        // 尝试寻找食物路径
        Direction dir = findPathToFood();

        // 验证路径是否安全
        Snake testSnake = snake.clone();
        testSnake.setDirection(dir);
        Point nextHead = testSnake.getNextHead();

        // 检查是否会撞墙或自身
        bool validMove = true;
        if (nextHead.x < 0 || nextHead.x >= width || nextHead.y < 0 || nextHead.y >= height) {
            validMove = false;
        }
        else {
            for (const auto& p : snake.getBody()) {
                if (p == nextHead) {
                    validMove = false;
                    break;
                }
            }
        }

        if (validMove) {
            snake.setDirection(dir);
        }
        else {
            // 找不到安全路径，选择最安全的方向
            snake.setDirection(findSafestDirection());
        }
    }

    // 更新游戏状态
    void update() {
        // 移动蛇
        snake.move();

        // 检查碰撞
        Point head = snake.getHead();
        if (head.x < 0 || head.x >= width || head.y < 0 || head.y >= height) {
            gameOver = true;
            return;
        }

        if (snake.checkSelfCollision()) {
            gameOver = true;
            return;
        }

        // 检查是否吃到食物
        if (head == food) {
            snake.grow();
            score++;
            spawnFood();
        }
    }

    // 运行游戏
    void run() {
        while (!gameOver) {
            draw();
            aiMove();
            update();
            Sleep(200); // 控制游戏速度
        }
        cout << "Game Over! Final Score: " << score << endl;
    }
};

int main() {
    Game game(20, 20);
    game.run();
    return 0;
}

```


end