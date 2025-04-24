说明：
我希望用c#和c++写一个脚本解释器，用于科学运算
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/bde53506ac844f0f89aa68778c253678.png#pic_center)

step1: c# 
C:\Users\wangrusheng\RiderProjects\WinFormsApp3\WinFormsApp3\Form1.cs

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Text;
using System.Text.RegularExpressions;
using System.Windows.Forms;


namespace WinFormsApp3;

public partial class Form1 : Form
{
    private Dictionary<string, double> variables = new Dictionary<string, double>();

    // Windows Forms 设计器代码
    private TextBox txtScript;
    private Button btnExecute;
    private TextBox txtOutput;
    public Form1()
    {
        InitializeComponent();
    
        this.txtScript = new TextBox();
        this.btnExecute = new Button();
        this.txtOutput = new TextBox();
        this.SuspendLayout();
            
        // txtScript
        this.txtScript.Multiline = true;
        this.txtScript.Location = new System.Drawing.Point(12, 12);
        this.txtScript.Size = new System.Drawing.Size(400, 150);
        this.txtScript.ScrollBars = ScrollBars.Vertical;
            
        // btnExecute
        this.btnExecute.Location = new System.Drawing.Point(12, 170);
        this.btnExecute.Size = new System.Drawing.Size(75, 23);
        this.btnExecute.Text = "执行";
        this.btnExecute.Click += new System.EventHandler(this.btnExecute_Click);
            
        // txtOutput
        this.txtOutput.Multiline = true;
        this.txtOutput.Location = new System.Drawing.Point(12, 200);
        this.txtOutput.Size = new System.Drawing.Size(400, 150);
        this.txtOutput.ScrollBars = ScrollBars.Vertical;
        this.txtOutput.ReadOnly = true;
            
        // MainForm
        this.ClientSize = new System.Drawing.Size(424, 361);
        this.Controls.Add(this.txtOutput);
        this.Controls.Add(this.btnExecute);
        this.Controls.Add(this.txtScript);
        this.Text = "简单脚本解释器";
        this.ResumeLayout(false);
        this.PerformLayout();
    }
    
            private void btnExecute_Click(object sender, EventArgs e)
        {
            variables.Clear();
            txtOutput.Clear();
            
            string[] lines = txtScript.Text.Split(
                new[] { "\r\n", "\r", "\n" }, 
                StringSplitOptions.RemoveEmptyEntries
            );

            foreach (string line in lines)
            {
                try
                {
                    var result = ProcessLine(line.Trim());
                    if (!string.IsNullOrEmpty(result))
                    {
                        txtOutput.AppendText(result + "\r\n");
                    }
                }
                catch (Exception ex)
                {
                    txtOutput.AppendText($"错误: {ex.Message}\r\n");
                }
            }
        }

        private string ProcessLine(string line)
        {
            if (string.IsNullOrWhiteSpace(line)) return "";

            // 处理赋值语句
            if (line.Contains("="))
            {
                var match = Regex.Match(line, @"^\s*([a-zA-Z_]\w*)\s*=\s*(.+?)\s*$");
                if (!match.Success) throw new ArgumentException("无效的赋值语句");

                string varName = match.Groups[1].Value;
                double value = EvaluateExpression(match.Groups[2].Value);
                variables[varName] = value;
                return $"{varName} = {value}";
            }

            // 处理普通表达式
            return EvaluateExpression(line).ToString();
        }

        private double EvaluateExpression(string expr)
        {
            // 替换变量为实际值
            string resolvedExpr = Regex.Replace(expr, @"\b([a-zA-Z_]\w*)\b", match =>
            {
                string varName = match.Groups[1].Value;
                if (variables.TryGetValue(varName, out double value))
                {
                    return value.ToString();
                }
                throw new ArgumentException($"未定义的变量: {varName}");
            });

            // 计算数学表达式
            DataTable table = new DataTable();
            table.Columns.Add("expr", typeof(string), resolvedExpr);
            DataRow row = table.NewRow();
            table.Rows.Add(row);
            return Convert.ToDouble(row["expr"]);
        }
    
}
```

step2: C++代码 C:\Users\wangrusheng\CLionProjects\untitled28\main.cpp

```cpp
#include <iostream>
#include <string>
#include <map>
#include <regex>
#include <cctype>
#include <vector>
#include <algorithm>
#include <sstream>
#include <stdexcept>

using namespace std;

map<string, double> variables;

// 辅助函数：跳过空白字符
void skip_whitespace(const string& expr, size_t& pos) {
    while (pos < expr.size() && isspace(expr[pos])) pos++;
}

// 表达式解析函数声明
double eval_expression(const string& expr, size_t& pos);
double eval_term(const string& expr, size_t& pos);
double eval_factor(const string& expr, size_t& pos);
double eval_primary(const string& expr, size_t& pos);

// 主解析函数
double evaluate(const string& expr) {
    size_t pos = 0;
    double result = eval_expression(expr, pos);
    skip_whitespace(expr, pos);
    if (pos != expr.size()) {
        throw runtime_error("Unexpected characters in expression");
    }
    return result;
}

// 表达式解析实现
double eval_expression(const string& expr, size_t& pos) {
    return eval_term(expr, pos);
}

double eval_term(const string& expr, size_t& pos) {
    double left = eval_factor(expr, pos);
    skip_whitespace(expr, pos);

    while (pos < expr.size()) {
        char op = expr[pos];
        if (op != '+' && op != '-') break;
        pos++;
        double right = eval_factor(expr, pos);
        if (op == '+') left += right;
        else left -= right;
        skip_whitespace(expr, pos);
    }
    return left;
}

double eval_factor(const string& expr, size_t& pos) {
    double left = eval_primary(expr, pos);
    skip_whitespace(expr, pos);

    while (pos < expr.size()) {
        char op = expr[pos];
        if (op != '*' && op != '/') break;
        pos++;
        double right = eval_primary(expr, pos);
        if (op == '*') left *= right;
        else {
            if (right == 0) throw runtime_error("Division by zero");
            left /= right;
        }
        skip_whitespace(expr, pos);
    }
    return left;
}

double eval_primary(const string& expr, size_t& pos) {
    skip_whitespace(expr, pos);
    if (pos >= expr.size()) {
        throw runtime_error("Unexpected end of expression");
    }

    if (expr[pos] == '(') {
        pos++;
        double value = eval_expression(expr, pos);
        skip_whitespace(expr, pos);
        if (pos >= expr.size() || expr[pos] != ')') {
            throw runtime_error("Missing closing parenthesis");
        }
        pos++;
        return value;
    }

    if (expr[pos] == '+' || expr[pos] == '-') {
        bool negative = (expr[pos] == '-');
        pos++;
        double value = eval_primary(expr, pos);
        return negative ? -value : value;
    }

    if (isdigit(expr[pos]) || expr[pos] == '.') {
        size_t start = pos;
        bool has_decimal = false;
        while (pos < expr.size() && (isdigit(expr[pos]) || expr[pos] == '.')) {
            if (expr[pos] == '.') {
                if (has_decimal) throw runtime_error("Invalid number format");
                has_decimal = true;
            }
            pos++;
        }
        return stod(expr.substr(start, pos - start));
    }

    throw runtime_error("Unexpected character: " + string(1, expr[pos]));
}

// 变量替换函数
string replace_variables(const string& expr) {
    regex var_re(R"(\b([a-zA-Z_]\w*)\b)");
    smatch match;
    string result;
    size_t last = 0;

    auto begin = expr.cbegin();
    while (regex_search(begin, expr.cend(), match, var_re)) {
        result += string(begin, begin + match.position());
        string var = match[1];
        if (!variables.count(var)) {
            throw runtime_error("Undefined variable: " + var);
        }
        result += to_string(variables[var]);
        begin += match.position() + match.length();
        last = begin - expr.begin();
    }
    result += expr.substr(last);
    return result;
}

// 处理单行输入
void process_line(const string& line) {
    try {
        string trimmed = line;
        trimmed.erase(remove_if(trimmed.begin(), trimmed.end(), ::isspace), trimmed.end());
        if (trimmed.empty()) return;

        // 处理赋值语句
        smatch match;
        regex assign_re(R"(^([a-zA-Z_]\w*)=(.*)$)");
        if (regex_match(trimmed, match, assign_re)) {
            string var = match[1];
            string expr = replace_variables(match[2]);
            variables[var] = evaluate(expr);
            cout << var << " = " << variables[var] << endl;
        }
        // 处理普通表达式
        else {
            string expr = replace_variables(trimmed);
            double result = evaluate(expr);
            cout << result << endl;
        }
    } catch (const exception& e) {
        cerr << "Error: " << e.what() << endl;
    }
}

int main() {
    cout << "Simple C++ Script Interpreter (type 'exit' to quit)" << endl;

    string line;
    while (true) {
        cout << "> ";
        getline(cin, line);
        if (line == "exit") break;
        process_line(line);
    }

    return 0;
}
```
step3:运行

```bash
C:\Users\wangrusheng\CLionProjects\untitled28\cmake-build-debug\untitled28.exe
Simple C++ Script Interpreter (type 'exit' to quit)
>a=5*3

a = 15
>b=a*2

b = 30
>c=5/0

>Error: Division by zero
a=9

a = 9
>8

8
>3/4*((2-(5/6))/(7/12)+(1/3)*(9/5)-(2/15))

1.85
>-2*2*((3/(-4)+0.5))-(((-5)*2-3)/7)

2.85714
>1/(1+(1/(2+(1/(3+(1/4))))))

0.697674
>2.5*((6-(3/2))*(6-(3/2)))/1.25-(5*5-(4*3))/(2*-1)

47
>
```
手动分割线：
step101:C:\Users\wangrusheng\CLionProjects\untitled28\main.cpp

```cpp
#include <iostream>
#include <string>
#include <map>
#include <regex>
#include <cctype>
#include <vector>
#include <algorithm>
#include <sstream>
#include <stdexcept>
#include <fstream>  // 新增文件流支持

using namespace std;

map<string, double> variables;

// 辅助函数：跳过空白字符
void skip_whitespace(const string& expr, size_t& pos) {
    while (pos < expr.size() && isspace(expr[pos])) pos++;
}

// 表达式解析函数声明
double eval_expression(const string& expr, size_t& pos);
double eval_term(const string& expr, size_t& pos);
double eval_factor(const string& expr, size_t& pos);
double eval_primary(const string& expr, size_t& pos);

// 主解析函数
double evaluate(const string& expr) {
    size_t pos = 0;
    double result = eval_expression(expr, pos);
    skip_whitespace(expr, pos);
    if (pos != expr.size()) {
        throw runtime_error("Unexpected characters in expression");
    }
    return result;
}

// 表达式解析实现
double eval_expression(const string& expr, size_t& pos) {
    return eval_term(expr, pos);
}

double eval_term(const string& expr, size_t& pos) {
    double left = eval_factor(expr, pos);
    skip_whitespace(expr, pos);

    while (pos < expr.size()) {
        char op = expr[pos];
        if (op != '+' && op != '-') break;
        pos++;
        double right = eval_factor(expr, pos);
        if (op == '+') left += right;
        else left -= right;
        skip_whitespace(expr, pos);
    }
    return left;
}

double eval_factor(const string& expr, size_t& pos) {
    double left = eval_primary(expr, pos);
    skip_whitespace(expr, pos);

    while (pos < expr.size()) {
        char op = expr[pos];
        if (op != '*' && op != '/') break;
        pos++;
        double right = eval_primary(expr, pos);
        if (op == '*') left *= right;
        else {
            if (right == 0) throw runtime_error("Division by zero");
            left /= right;
        }
        skip_whitespace(expr, pos);
    }
    return left;
}

double eval_primary(const string& expr, size_t& pos) {
    skip_whitespace(expr, pos);
    if (pos >= expr.size()) {
        throw runtime_error("Unexpected end of expression");
    }

    if (expr[pos] == '(') {
        pos++;
        double value = eval_expression(expr, pos);
        skip_whitespace(expr, pos);
        if (pos >= expr.size() || expr[pos] != ')') {
            throw runtime_error("Missing closing parenthesis");
        }
        pos++;
        return value;
    }

    if (expr[pos] == '+' || expr[pos] == '-') {
        bool negative = (expr[pos] == '-');
        pos++;
        double value = eval_primary(expr, pos);
        return negative ? -value : value;
    }

    if (isdigit(expr[pos]) || expr[pos] == '.') {
        size_t start = pos;
        bool has_decimal = false;
        while (pos < expr.size() && (isdigit(expr[pos]) || expr[pos] == '.')) {
            if (expr[pos] == '.') {
                if (has_decimal) throw runtime_error("Invalid number format");
                has_decimal = true;
            }
            pos++;
        }
        return stod(expr.substr(start, pos - start));
    }

    throw runtime_error("Unexpected character: " + string(1, expr[pos]));
}

// 变量替换函数
string replace_variables(const string& expr) {
    regex var_re(R"(\b([a-zA-Z_]\w*)\b)");
    smatch match;
    string result;
    size_t last = 0;

    auto begin = expr.cbegin();
    while (regex_search(begin, expr.cend(), match, var_re)) {
        result += string(begin, begin + match.position());
        string var = match[1];
        if (!variables.count(var)) {
            throw runtime_error("Undefined variable: " + var);
        }
        result += to_string(variables[var]);
        begin += match.position() + match.length();
        last = begin - expr.begin();
    }
    result += expr.substr(last);
    return result;
}

// 处理单行输入
void process_line(const string& line) {
    try {
        string trimmed = line;
        trimmed.erase(remove_if(trimmed.begin(), trimmed.end(), ::isspace), trimmed.end());
        if (trimmed.empty()) return;

        // 处理赋值语句
        smatch match;
        regex assign_re(R"(^([a-zA-Z_]\w*)=(.*)$)");
        if (regex_match(trimmed, match, assign_re)) {
            string var = match[1];
            string expr = replace_variables(match[2]);
            variables[var] = evaluate(expr);
            cout << var << " = " << variables[var] << endl;
        }
        // 处理普通表达式
        else {
            string expr = replace_variables(trimmed);
            double result = evaluate(expr);
            cout << result << endl;
        }
    } catch (const exception& e) {
        cerr << "Error: " << e.what() << endl;
    }
}

int main() {
    // 设置默认文件路径（使用原始字符串避免转义）
    const string filepath = R"(C:\Users\wangrusheng\CLionProjects\untitled28\cmake-build-debug\input.txt)";

    // 打开输入文件
    ifstream input_file(filepath);

    if (!input_file.is_open()) {
        cerr << "Error: Could not open input file at:\n" << filepath << endl;
        return 1;
    }

    // 逐行读取并处理
    string line;
    while (getline(input_file, line)) {
        process_line(line);
    }

    input_file.close();
    return 0;
}
```

step102:C:\Users\wangrusheng\CLionProjects\untitled28\cmake-build-debug\input.txt

```bash
3/4*((2-(5/6))/(7/12)+(1/3)*(9/5)-(2/15))
-2*2*((3/(-4)+0.5))-(((-5)*2-3)/7)
1/(1+(1/(2+(1/(3+(1/4))))))

2.5*((6-(3/2))*(6-(3/2)))/1.25-(5*5-(4*3))/(2*-1)
```

step103:运行

```bash
C:\Users\wangrusheng\CLionProjects\untitled28\cmake-build-debug\untitled28.exe
1.85
2.85714
0.697674
47

Process finished with exit code 0
```
手动分割线
用python实现
step201:

```python
import re

class Evaluator:
    def __init__(self):
        self.variables = {}

    def skip_whitespace(self, expr, pos):
        while pos[0] < len(expr) and expr[pos[0]].isspace():
            pos[0] += 1

    def eval_expression(self, expr, pos):
        return self.eval_term(expr, pos)

    def eval_term(self, expr, pos):
        left = self.eval_factor(expr, pos)
        self.skip_whitespace(expr, pos)

        while pos[0] < len(expr):
            op = expr[pos[0]]
            if op not in '+-':
                break
            pos[0] += 1
            right = self.eval_factor(expr, pos)
            if op == '+':
                left += right
            else:
                left -= right
            self.skip_whitespace(expr, pos)
        return left

    def eval_factor(self, expr, pos):
        left = self.eval_primary(expr, pos)
        self.skip_whitespace(expr, pos)

        while pos[0] < len(expr):
            op = expr[pos[0]]
            if op not in '*/':
                break
            pos[0] += 1
            right = self.eval_primary(expr, pos)
            if op == '*':
                left *= right
            else:
                if right == 0:
                    raise RuntimeError("Division by zero")
                left /= right
            self.skip_whitespace(expr, pos)
        return left

    def eval_primary(self, expr, pos):
        self.skip_whitespace(expr, pos)
        if pos[0] >= len(expr):
            raise RuntimeError("Unexpected end of expression")

        if expr[pos[0]] == '(':
            pos[0] += 1
            value = self.eval_expression(expr, pos)
            self.skip_whitespace(expr, pos)
            if pos[0] >= len(expr) or expr[pos[0]] != ')':
                raise RuntimeError("Missing closing parenthesis")
            pos[0] += 1
            return value

        if expr[pos[0]] in '+-':
            sign = -1 if expr[pos[0]] == '-' else 1
            pos[0] += 1
            primary = self.eval_primary(expr, pos)
            return sign * primary

        if expr[pos[0]].isdigit() or expr[pos[0]] == '.':
            start = pos[0]
            has_decimal = False
            while pos[0] < len(expr) and (expr[pos[0]].isdigit() or expr[pos[0]] == '.'):
                if expr[pos[0]] == '.':
                    if has_decimal:
                        raise RuntimeError("Invalid number format")
                    has_decimal = True
                pos[0] += 1
            num_str = expr[start:pos[0]]
            try:
                return float(num_str)
            except ValueError:
                raise RuntimeError(f"Invalid number: {num_str}")

        raise RuntimeError(f"Unexpected character: {expr[pos[0]]} at position {pos[0]}")

    def replace_variables(self, expr):
        def replacer(match):
            var_name = match.group(1)
            if var_name not in self.variables:
                raise RuntimeError(f"Undefined variable: {var_name}")
            return str(self.variables[var_name])
        try:
            replaced_expr = re.sub(r'\b([a-zA-Z_]\w*)\b', replacer, expr)
        except RuntimeError as e:
            raise e
        return replaced_expr

    def process_line(self, line):
        stripped = line.replace(' ', '')
        if not stripped:
            return
        try:
            match = re.fullmatch(r'^([a-zA-Z_]\w*)=(.*)$', stripped)
            if match:
                var = match.group(1)
                expr = match.group(2)
                replaced_expr = self.replace_variables(expr)
                pos = [0]
                value = self.eval_expression(replaced_expr, pos)
                if pos[0] != len(replaced_expr):
                    raise RuntimeError(f"Unexpected characters in expression: {replaced_expr[pos[0]:]}")
                self.variables[var] = value
                print(f"{var} = {value}")
            else:
                replaced_expr = self.replace_variables(stripped)
                pos = [0]
                value = self.eval_expression(replaced_expr, pos)
                if pos[0] != len(replaced_expr):
                    raise RuntimeError(f"Unexpected characters in expression: {replaced_expr[pos[0]:]}")
                print(value)
        except Exception as e:
            print(f"Error: {e}")

def main():
    evaluator = Evaluator()
    filepath = r'C:\Users\wangrusheng\CLionProjects\untitled28\cmake-build-debug\input.txt'
    try:
        with open(filepath, 'r') as f:
            for line in f:
                evaluator.process_line(line.strip())
    except IOError as e:
        print(f"Error opening file: {e}")

if __name__ == '__main__':
    main()
```

step202:运行

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py
1.8499999999999996
2.857142857142857
0.6976744186046512
47.0
a = 5.0
b = -7.0
c = -2.0
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> 

```

end