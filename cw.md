说明：
我计划用c++写算法，将两个本地文件进行差异对比，生成差异报告html，并将差异部分，用高亮颜色标注
效果图
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/9f324ea44f33489080ca1250a78f0885.png#pic_center)


step1:C:\Users\wangrusheng\CLionProjects\untitled21\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.15)
project(untitled21 CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(untitled21 main.cpp)

# 针对GNU/Clang编译器链接文件系统库
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_link_libraries(untitled21 PRIVATE stdc++fs)
endif()
```

step2:C:\Users\wangrusheng\CLionProjects\untitled21\main.cpp

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <sstream>
#include <algorithm>
#include <filesystem>

using namespace std;
namespace fs = std::filesystem;

// ANSI颜色代码（控制台输出用）
#define RED   "\033[31m"
#define GREEN "\033[32m"
#define RESET "\033[0m"

enum Operation { DEL, INS, EQ };
struct Diff { Operation op; string line; };

struct OutputLine {
    int orig_num;
    string orig_text;
    bool orig_highlight;
    int mod_num;
    string mod_text;
    bool mod_highlight;
};

vector<string> read_lines_from_file(const string& file_path) {
    vector<string> lines;
    ifstream file(file_path);

    if (!file.is_open()) {
        cerr << RED "错误：无法打开文件 " << file_path << RESET << endl;
        exit(EXIT_FAILURE);
    }

    lines.reserve(1000);  // 预分配内存
    string line;

    while (getline(file, line)) {
        if (!line.empty() && line.back() == '\r') {
            line.resize(line.size() - 1);  // 处理Windows换行符
        }
        lines.push_back(line);
    }

    return lines;
}

void compute_diff(vector<Diff>& diffs, const vector<string>& a, const vector<string>& b) {
    const size_t m = a.size(), n = b.size();
    vector<vector<size_t>> dp(m + 1, vector<size_t>(n + 1, 0));

    // 构建DP矩阵
    for (size_t i = 1; i <= m; ++i) {
        for (size_t j = 1; j <= n; ++j) {
            if (a[i-1] == b[j-1]) {
                dp[i][j] = dp[i-1][j-1] + 1;
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
            }
        }
    }

    // 回溯生成差异
    size_t i = m, j = n;
    vector<Diff> temp;
    temp.reserve(m + n);  // 预分配内存

    while (i > 0 || j > 0) {
        if (i > 0 && j > 0 && a[i-1] == b[j-1]) {
            temp.push_back({EQ, a[i-1]});
            --i; --j;
        } else if (j > 0 && (i == 0 || dp[i][j-1] >= dp[i-1][j])) {
            temp.push_back({INS, b[j-1]});
            --j;
        } else if (i > 0 && (j == 0 || dp[i-1][j] >= dp[i][j-1])) {
            temp.push_back({DEL, a[i-1]});
            --i;
        }
    }

    reverse(temp.begin(), temp.end());
    diffs = move(temp);
}

string escape_html(const string& s) {
    string escaped;
    escaped.reserve(s.size() * 1.2);  // 预分配内存

    for (char c : s) {
        switch(c) {
            case '&': escaped += "&amp;"; break;
            case '<': escaped += "&lt;"; break;
            case '>': escaped += "&gt;"; break;
            case '"': escaped += "&quot;"; break;
            case '\'': escaped += "&apos;"; break;
            default: escaped += c; break;
        }
    }
    return escaped;
}

int main(int argc, char* argv[]) {
    // 文件路径配置
    string old_file = R"(C:\Users\wangrusheng\Desktop\old.txt)";
    string new_file = R"(C:\Users\wangrusheng\Desktop\new.txt)";

    // 从文件读取内容
    auto original_lines = read_lines_from_file(old_file);
    auto modified_lines = read_lines_from_file(new_file);

    // 计算差异
    vector<Diff> diffs;
    compute_diff(diffs, original_lines, modified_lines);

    // 生成输出结构
    vector<OutputLine> output_lines;
    int orig_num = 1, mod_num = 1;
    output_lines.reserve(diffs.size());

    for (const auto& diff : diffs) {
        switch(diff.op) {
            case EQ:
                output_lines.push_back({orig_num++, diff.line, false, mod_num++, diff.line, false});
                break;
            case DEL:
                output_lines.push_back({orig_num++, diff.line, true, 0, "", false});
                break;
            case INS:
                output_lines.push_back({0, "", false, mod_num++, diff.line, true});
                break;
        }
    }

    // 生成HTML报告
    ofstream html_out("diff_result.html");
    if (!html_out.is_open()) {
        cerr << RED "错误：无法创建结果文件" RESET << endl;
        return EXIT_FAILURE;
    }

    // 使用ostringstream构建初始HTML
    ostringstream oss;
    oss << R"(<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>文件差异对比报告</title>
    <style>
        table { border-collapse: collapse; width: 100%; margin: 20px 0; }
        th, td {
            border: 1px solid #ddd;
            padding: 2px 5px;
            vertical-align: top;
            font-family: 'Consolas', monospace;
            white-space: pre-wrap;
            font-size: 14px;
        }
        .red { background-color: #ffe6e6; }
        .green { background-color: #e6ffe6; }
        .line-number {
            color: #666;
            display: inline-block;
            width: 40px;
            user-select: none;
        }
        del { background: #ffcccc; text-decoration: none; }
        ins { background: #ccffcc; text-decoration: none; }
    </style>
</head>
<body>
    <h2>文件差异对比报告</h2>
    <table>
        <tr><th style="width:50%;">原始文件 (${OLD_FILE})</th>
            <th style="width:50%;">修改文件 (${NEW_FILE})</th></tr>
)";

    string html_content = oss.str();

    // 替换文件名占位符
    size_t old_pos = html_content.find("${OLD_FILE}");
    if (old_pos != string::npos) {
        html_content.replace(old_pos, 11, fs::path(old_file).filename().string());
    }

    size_t new_pos = html_content.find("${NEW_FILE}");
    if (new_pos != string::npos) {
        html_content.replace(new_pos, 11, fs::path(new_file).filename().string());
    }

    html_out << html_content;

    // 写入表格内容
    for (const auto& line : output_lines) {
        html_out << "<tr>\n";

        // 原始文件列
        html_out << "<td" << (line.orig_highlight ? " class=\"red\"" : "") << ">";
        if (line.orig_num > 0) {
            html_out << "<span class=\"line-number\">" << line.orig_num << "</span>"
                     << escape_html("  " + line.orig_text);
        }
        html_out << "</td>\n";

        // 修改文件列
        html_out << "<td" << (line.mod_highlight ? " class=\"green\"" : "") << ">";
        if (line.mod_num > 0) {
            html_out << "<span class=\"line-number\">" << line.mod_num << "</span>"
                     << escape_html("  " + line.mod_text);
        }
        html_out << "</td>\n</tr>\n";
    }

    html_out << R"(</table>
</body>
</html>)";

    // 输出结果信息
    cout << GREEN "✓ 成功生成差异报告：" RESET
         << fs::absolute("diff_result.html") << "\n"
         << "原始文件: " << fs::absolute(old_file) << "\n"
         << "修改文件: " << fs::absolute(new_file) << "\n"
         << "差异数量: " << count_if(diffs.begin(), diffs.end(),
             [](const Diff& d) { return d.op != EQ; }) << " 处\n";

    return 0;
}
```

step3:运行

```bash
file:///C:/Users/wangrusheng/CLionProjects/untitled21/cmake-build-debug/diff_result.html
```
纯算法部分

step101:C:\Users\wangrusheng\CLionProjects\untitled19\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.30)
project(untitled19)

set(CMAKE_CXX_STANDARD 20)

add_executable(untitled19 main.cpp)

```

step102:C:\Users\wangrusheng\CLionProjects\untitled19\main.cpp

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <sstream>
#include <fstream>
#include <iomanip>

using namespace std;

#define RED   "\033[31m"
#define GREEN "\033[32m"
#define RESET "\033[0m"

enum Operation { DEL, INS, EQ };
struct Diff { Operation op; string line; };
struct OutputLine { int orig_num; string orig_text; bool orig_highlight; int mod_num; string mod_text; bool mod_highlight; };

vector<string> split(const string &s, char delimiter) {
    vector<string> tokens;
    string token;
    istringstream tokenStream(s);
    while (getline(tokenStream, token, delimiter)) tokens.push_back(token);
    return tokens;
}

// 核心改进：基于 LCS 的 Diff 生成算法
void compute_diff(vector<Diff>& diffs, const vector<string>& a, const vector<string>& b, int i, int j) {
    while (i < a.size() && j < b.size() && a[i] == b[j]) {
        diffs.push_back({EQ, a[i]});
        i++; j++;
    }

    if (i < a.size() && j < b.size()) {
        vector<Diff> branch1, branch2;
        compute_diff(branch1, a, b, i+1, j); // 尝试删除原行
        compute_diff(branch2, a, b, i, j+1); // 尝试插入新行

        if (branch1.size() <= branch2.size()) {
            diffs.push_back({DEL, a[i]});
            diffs.insert(diffs.end(), branch1.begin(), branch1.end());
        } else {
            diffs.push_back({INS, b[j]});
            diffs.insert(diffs.end(), branch2.begin(), branch2.end());
        }
    } else {
        while (i < a.size()) diffs.push_back({DEL, a[i++]}); // 剩余删除
        while (j < b.size()) diffs.push_back({INS, b[j++]}); // 剩余插入
    }
}

int main() {
    string original = "Hello world!\nWelcome to C++\n1\n2\n3";
    string modified = "Hello dtl!\nWelcome to C++\nDiff example\n1\n2\n3";

    vector<string> original_lines = split(original, '\n');
    vector<string> modified_lines = split(modified, '\n');

    vector<Diff> diffs;
    compute_diff(diffs, original_lines, modified_lines, 0, 0); // 生成更准确的差异

    vector<OutputLine> output_lines;
    int orig_num = 1, mod_num = 1;

    for (size_t k = 0; k < diffs.size(); k++) {
        switch(diffs[k].op) {
            case EQ:
                output_lines.push_back({orig_num, diffs[k].line, false, mod_num, diffs[k].line, false});
                orig_num++;
                mod_num++;
                break;
            case DEL:
                output_lines.push_back({orig_num, diffs[k].line, true, 0, "", false});
                orig_num++;
                break;
            case INS:
                output_lines.push_back({0, "", false, mod_num, diffs[k].line, true});
                mod_num++;
                break;
        }
    }

    ofstream out("diff_result.txt");
    const int LEFT_WIDTH = 40;

    for (const auto& line : output_lines) {
        ostringstream left, right;

        // 处理左侧
        if (line.orig_num > 0) {
            left << setw(4) << line.orig_num << " ";
            if (line.orig_highlight) left << RED << "- " << line.orig_text << RESET;
            else left << "  " << line.orig_text;
        }

        // 处理右侧
        if (line.mod_num > 0) {
            right << setw(4) << line.mod_num << " ";
            if (line.mod_highlight) right << GREEN << "+ " << line.mod_text << RESET;
            else right << "  " << line.mod_text;
        }

        // 对齐输出
        string output = left.str();
        output += string(LEFT_WIDTH - output.length(), ' ');
        output += "| " + right.str();

        cout << output << endl;
        out << output << endl;
    }

    out.close();
    return 0;
}
```

end