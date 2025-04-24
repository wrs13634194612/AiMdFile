说明：
forms计算器
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/eb61eff7ce604d33862121e03906863e.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp2\WinFormsApp2\Form1.cs

```csharp
using System;
using System.Windows.Forms;
namespace WinFormsApp2;

public partial class Form1 : Form
{
    
    private double _firstOperand = 0;
    private string _currentOperator = "";
    private bool _isNewNumber = true;
    
    public Form1()
    {
        InitializeComponent();
        CreateCalculatorLayout();
    }
    
    

        private void CreateCalculatorLayout()
        {
            // 创建显示文本框
            TextBox textBox = new TextBox
            {
                Name = "txtDisplay",
                ReadOnly = true,
                TextAlign = HorizontalAlignment.Right,
                Font = new Font("Arial", 18),
                Dock = DockStyle.Top,
                Height = 60
            };
            this.Controls.Add(textBox);

            // 创建按钮布局
            TableLayoutPanel panel = new TableLayoutPanel
            {
                Dock = DockStyle.Fill,
                ColumnCount = 4,
                RowCount = 5,
                Padding = new Padding(5)
            };

            // 设置行列比例
            for (int i = 0; i < 4; i++)
                panel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 25F));
            for (int i = 0; i < 5; i++)
                panel.RowStyles.Add(new RowStyle(SizeType.Percent, 20F));

            // 添加按钮
            string[] buttons = {
                "7", "8", "9", "/",
                "4", "5", "6", "*",
                "1", "2", "3", "-",
                "0", ".", "=", "+",
                "C"
            };

            int index = 0;
            for (int row = 0; row < 4; row++)
            {
                for (int col = 0; col < 4; col++)
                {
                    var btn = CreateButton(buttons[index++]);
                    panel.Controls.Add(btn, col, row);
                }
            }

            // 添加清除按钮（单独一行）
            var clearBtn = CreateButton(buttons[index]);
            panel.SetColumnSpan(clearBtn, 4);
            panel.Controls.Add(clearBtn, 0, 4);

            this.Controls.Add(panel);
        }

        private Button CreateButton(string text)
        {
            Button btn = new Button
            {
                Text = text,
                Font = new Font("Arial", 12),
                Dock = DockStyle.Fill,
                Margin = new Padding(3),
                Tag = text
            };
            btn.Click += CalculatorButton_Click;
            return btn;
        }

        private void CalculatorButton_Click(object sender, EventArgs e)
        {
            Button btn = (Button)sender;
            string input = btn.Tag.ToString();
            var txtDisplay = this.Controls["txtDisplay"] as TextBox;

            if (char.IsDigit(input, 0))
            {
                if (_isNewNumber)
                {
                    txtDisplay.Text = input;
                    _isNewNumber = false;
                }
                else
                {
                    txtDisplay.Text += input;
                }
            }
            else if (input == ".")
            {
                if (!txtDisplay.Text.Contains("."))
                {
                    if (_isNewNumber)
                    {
                        txtDisplay.Text = "0.";
                        _isNewNumber = false;
                    }
                    else
                    {
                        txtDisplay.Text += ".";
                    }
                }
            }
            else if (input == "C")
            {
                txtDisplay.Text = "0";
                _firstOperand = 0;
                _currentOperator = "";
                _isNewNumber = true;
            }
            else if (input == "=")
            {
                if (!string.IsNullOrEmpty(_currentOperator) && !_isNewNumber)
                {
                    CalculateResult(double.Parse(txtDisplay.Text));
                    _currentOperator = "";
                }
            }
            else // 运算符
            {
                if (!_isNewNumber)
                {
                    if (!string.IsNullOrEmpty(_currentOperator))
                    {
                        CalculateResult(double.Parse(txtDisplay.Text));
                    }
                    else
                    {
                        _firstOperand = double.Parse(txtDisplay.Text);
                    }
                    _currentOperator = input;
                    _isNewNumber = true;
                }
            }
        }

        private void CalculateResult(double secondOperand)
        {
            try
            {
                switch (_currentOperator)
                {
                    case "+":
                        _firstOperand += secondOperand;
                        break;
                    case "-":
                        _firstOperand -= secondOperand;
                        break;
                    case "*":
                        _firstOperand *= secondOperand;
                        break;
                    case "/":
                        if (secondOperand == 0)
                            throw new DivideByZeroException();
                        _firstOperand /= secondOperand;
                        break;
                }

                var txtDisplay = this.Controls["txtDisplay"] as TextBox;
                txtDisplay.Text = _firstOperand.ToString("G15");
                _isNewNumber = true;
            }
            catch (DivideByZeroException)
            {
                MessageBox.Show("Cannot divide by zero!", "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
                _firstOperand = 0;
                _currentOperator = "";
                _isNewNumber = true;
                (this.Controls["txtDisplay"] as TextBox).Text = "0";
            }
        }




}
```

end