forms科学计算器
MC：清除内存
MR：读取内存值
MS：存储当前显示值到内存
M+：将当前显示值累加到内存
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3036449522204515b399167f236dc6d1.png#pic_center)

step1:C:\Users\wangrusheng\RiderProjects\WinFormsApp39\WinFormsApp39\Form1.cs

```csharp
using System;
using System.Drawing;
using System.Windows.Forms;

namespace WinFormsApp39
{
    public partial class Form1 : Form
    {
        private TextBox txtDisplay;
        private TableLayoutPanel tableLayoutPanel;
        private Button btnBackspace, btnClear, btnClearAll, btnMC, btnMR, btnMS, btnMAdd;
        private Button[] digitButtons = new Button[10];
        private Button btnPoint, btnSign, btnAdd, btnSubtract, btnMultiply, btnDivide;
        private Button btnSqrt, btnSquare, btnReciprocal, btnEqual;
        
        // 计算器状态变量
        private double sumInMemory = 0.0;
        private double sumSoFar = 0.0;
        private double factorSoFar = 0.0;
        private bool waitingForOperand = true;
        private string pendingAdditiveOperator = "";
        private string pendingMultiplicativeOperator = "";

        public Form1()
        {
            InitializeComponent();
            
                  // 组件初始化
            this.txtDisplay = new System.Windows.Forms.TextBox();
            this.tableLayoutPanel = new System.Windows.Forms.TableLayoutPanel();
            
            // 创建按钮
            this.btnMC = CreateButton("MC");
            this.btnMR = CreateButton("MR");
            this.btnMS = CreateButton("MS");
            this.btnMAdd = CreateButton("M+");
            this.btnClearAll = CreateButton("C");
            
            this.btnSqrt = CreateButton("√");
            this.btnSquare = CreateButton("x²");
            this.btnReciprocal = CreateButton("1/x");
            this.btnDivide = CreateButton("÷");
            this.btnMultiply = CreateButton("×");
            
            this.digitButtons = new System.Windows.Forms.Button[10];
            for (int i = 0; i < 10; i++)
                this.digitButtons[i] = CreateButton(i.ToString());
            
            this.btnBackspace = CreateButton("←");
            this.btnClear = CreateButton("CE");
            this.btnSubtract = CreateButton("-");
            this.btnAdd = CreateButton("+");
            this.btnEqual = CreateButton("=");
            this.btnPoint = CreateButton(".");
            this.btnSign = CreateButton("±");

            // 布局设置
            this.SuspendLayout();
            
            // txtDisplay
            this.txtDisplay.Dock = DockStyle.Top;
            this.txtDisplay.Font = new Font("Microsoft Sans Serif", 16F);
            this.txtDisplay.Height = 60;
            
            // tableLayoutPanel
            this.tableLayoutPanel.ColumnCount = 5;
            this.tableLayoutPanel.RowCount = 6;
            this.tableLayoutPanel.Dock = DockStyle.Fill;
            for (int i = 0; i < 5; i++)
                this.tableLayoutPanel.ColumnStyles.Add(new ColumnStyle(SizeType.Percent, 20F));
            for (int i = 0; i < 6; i++)
                this.tableLayoutPanel.RowStyles.Add(new RowStyle(SizeType.Percent, 16.666F));

            // 添加控件到布局
            AddControlsToTable();

            // Form设置
            this.ClientSize = new Size(400, 500);
            this.Controls.Add(this.tableLayoutPanel);
            this.Controls.Add(this.txtDisplay);
            this.Text = "Calculator";
            this.ResumeLayout();
            
            SetupCalculator();
        }

        private void SetupCalculator()
        {
            // 初始化控件并绑定事件
            foreach (Control c in tableLayoutPanel.Controls)
            {
                if (c is Button btn)
                {
                    if (char.IsDigit(btn.Text[0]))
                        btn.Click += DigitButtonClick;
                }
            }

            btnAdd.Click += AdditiveOperatorClick;
            btnSubtract.Click += AdditiveOperatorClick;
            btnMultiply.Click += MultiplicativeOperatorClick;
            btnDivide.Click += MultiplicativeOperatorClick;
            
            btnPoint.Click += BtnPoint_Click;
            btnSign.Click += BtnSign_Click;
            btnBackspace.Click += BtnBackspace_Click;
            btnClear.Click += BtnClear_Click;
            btnClearAll.Click += BtnClearAll_Click;
            btnEqual.Click += BtnEqual_Click;
            
            btnMC.Click += BtnMC_Click;
            btnMR.Click += BtnMR_Click;
            btnMS.Click += BtnMS_Click;
            btnMAdd.Click += BtnMAdd_Click;
            
            btnSqrt.Click += UnaryOperatorClick;
            btnSquare.Click += UnaryOperatorClick;
            btnReciprocal.Click += UnaryOperatorClick;

            txtDisplay.Text = "0";
            txtDisplay.ReadOnly = true;
            txtDisplay.TextAlign = HorizontalAlignment.Right;
        }

        #region 事件处理方法
        private void DigitButtonClick(object sender, EventArgs e)
        {
            Button btn = (Button)sender;
            string digit = btn.Text;

            if (txtDisplay.Text == "0" && digit == "0") return;

            if (waitingForOperand)
            {
                txtDisplay.Text = "";
                waitingForOperand = false;
            }

            txtDisplay.Text += digit;
        }

        private void BtnPoint_Click(object sender, EventArgs e)
        {
            if (waitingForOperand)
                txtDisplay.Text = "0";

            if (!txtDisplay.Text.Contains("."))
                txtDisplay.Text += ".";
            
            waitingForOperand = false;
        }

        private void BtnSign_Click(object sender, EventArgs e)
        {
            if (double.TryParse(txtDisplay.Text, out double value))
                txtDisplay.Text = (-value).ToString();
        }

        private void BtnBackspace_Click(object sender, EventArgs e)
        {
            if (txtDisplay.Text.Length > 1)
            {
                txtDisplay.Text = txtDisplay.Text.Substring(0, txtDisplay.Text.Length - 1);
            }
            else
            {
                txtDisplay.Text = "0";
                waitingForOperand = true;
            }
        }

        private void BtnClear_Click(object sender, EventArgs e)
        {
            txtDisplay.Text = "0";
            waitingForOperand = true;
        }

        private void BtnClearAll_Click(object sender, EventArgs e)
        {
            sumSoFar = 0.0;
            factorSoFar = 0.0;
            pendingAdditiveOperator = "";
            pendingMultiplicativeOperator = "";
            txtDisplay.Text = "0";
            waitingForOperand = true;
        }

  
        
        private void BtnEqual_Click(object sender, EventArgs e)
        {
            if (double.TryParse(txtDisplay.Text, out double operand))
            {
                HandlePendingMultiplication(); // 处理可能存在的乘除运算

                // 重新获取处理后的操作数
                if (!double.TryParse(txtDisplay.Text, out operand))
                {
                    AbortOperation();
                    return;
                }

                HandleFinalAddition(operand); // 使用最新的操作数处理加减运算

                txtDisplay.Text = sumSoFar.ToString();
                sumSoFar = 0.0;
                waitingForOperand = true;
            }
        }

        private void AdditiveOperatorClick(object sender, EventArgs e)
        {
            Button btn = (Button)sender;
            string operation = btn.Text;

            if (double.TryParse(txtDisplay.Text, out double operand))
            {
                HandlePendingMultiplication(); // 处理可能存在的乘除运算

                // 重新获取处理后的操作数
                if (!double.TryParse(txtDisplay.Text, out operand))
                {
                    AbortOperation();
                    return;
                }

                HandlePendingAddition(operand); // 使用最新的操作数处理加减运算

                pendingAdditiveOperator = operation;
                waitingForOperand = true;
            }
        }

        private void MultiplicativeOperatorClick(object sender, EventArgs e)
        {
            Button btn = (Button)sender;
            string operation = btn.Text;

            if (double.TryParse(txtDisplay.Text, out double operand))
            {
                if (!string.IsNullOrEmpty(pendingMultiplicativeOperator))
                {
                    if (!Calculate(operand, pendingMultiplicativeOperator))
                        return;
                    
                    txtDisplay.Text = factorSoFar.ToString();
                }
                else
                {
                    factorSoFar = operand;
                }

                pendingMultiplicativeOperator = operation;
                waitingForOperand = true;
            }
        }
 

        private bool Calculate(double operand, string operation)
        {
            switch (operation)
            {
                case "+":
                    sumSoFar += operand;
                    break;
                case "-":
                    sumSoFar -= operand;
                    break;
                case "×":
                    factorSoFar *= operand;
                    break;
                case "÷":
                    if (operand == 0) return false;
                    factorSoFar /= operand;
                    break;
                default:
                    return false;
            }
            return true;
        }

        private void HandlePendingMultiplication()
        {
            if (!string.IsNullOrEmpty(pendingMultiplicativeOperator))
            {
                if (!Calculate(double.Parse(txtDisplay.Text), pendingMultiplicativeOperator))
                {
                    AbortOperation();
                    return;
                }
                txtDisplay.Text = factorSoFar.ToString();
                pendingMultiplicativeOperator = null;
            }
        }

        private void HandlePendingAddition(double operand)
        {
            if (!string.IsNullOrEmpty(pendingAdditiveOperator))
            {
                if (!Calculate(operand, pendingAdditiveOperator))
                {
                    AbortOperation();
                    return;
                }
                txtDisplay.Text = sumSoFar.ToString();
            }
            else
            {
                sumSoFar = operand;
            }
        }

        private void HandleFinalAddition(double operand)
        {
            if (!string.IsNullOrEmpty(pendingAdditiveOperator))
            {
                if (!Calculate(operand, pendingAdditiveOperator))
                {
                    AbortOperation();
                    return;
                }
                pendingAdditiveOperator = null;
            }
            else
            {
                sumSoFar = operand;
            }
        }

        private void AbortOperation()
        {
            BtnClearAll_Click(null, EventArgs.Empty);
            txtDisplay.Text = "ERR";
        }

        #region 内存功能
        private void BtnMC_Click(object sender, EventArgs e) => sumInMemory = 0.0;
        private void BtnMR_Click(object sender, EventArgs e) => txtDisplay.Text = sumInMemory.ToString();
        private void BtnMS_Click(object sender, EventArgs e) => sumInMemory = double.Parse(txtDisplay.Text);
        private void BtnMAdd_Click(object sender, EventArgs e) => sumInMemory += double.Parse(txtDisplay.Text);
        #endregion

        #region 单目运算
        private void UnaryOperatorClick(object sender, EventArgs e)
        {
            Button btn = (Button)sender;
            if (double.TryParse(txtDisplay.Text, out double operand))
            {
                try
                {
                    switch (btn.Text)
                    {
                        case "√":
                            txtDisplay.Text = Math.Sqrt(operand).ToString();
                            break;
                        case "x²":
                            txtDisplay.Text = Math.Pow(operand, 2).ToString();
                            break;
                        case "1/x":
                            txtDisplay.Text = (1.0 / operand).ToString();
                            break;
                    }
                }
                catch
                {
                    AbortOperation();
                }
                waitingForOperand = true;
            }
        }
        #endregion
        #endregion

        #region Windows Form Designer generated code
      

        private Button CreateButton(string text)
        {
            return new Button
            {
                Text = text,
                Dock = DockStyle.Fill,
                Font = new Font("Microsoft Sans Serif", 12F),
                Margin = new Padding(2)
            };
        }

        private void AddControlsToTable()
        {
            // 行0
            tableLayoutPanel.Controls.Add(btnMC, 0, 0);
            tableLayoutPanel.Controls.Add(btnMR, 1, 0);
            tableLayoutPanel.Controls.Add(btnMS, 2, 0);
            tableLayoutPanel.Controls.Add(btnMAdd, 3, 0);
            tableLayoutPanel.Controls.Add(btnClearAll, 4, 0);

            // 行1
            tableLayoutPanel.Controls.Add(btnSqrt, 0, 1);
            tableLayoutPanel.Controls.Add(btnSquare, 1, 1);
            tableLayoutPanel.Controls.Add(btnReciprocal, 2, 1);
            tableLayoutPanel.Controls.Add(btnDivide, 3, 1);
            tableLayoutPanel.Controls.Add(btnMultiply, 4, 1);

            // 行2
            tableLayoutPanel.Controls.Add(digitButtons[7], 0, 2);
            tableLayoutPanel.Controls.Add(digitButtons[8], 1, 2);
            tableLayoutPanel.Controls.Add(digitButtons[9], 2, 2);
            tableLayoutPanel.Controls.Add(btnSubtract, 3, 2);
            tableLayoutPanel.Controls.Add(btnClear, 4, 2);

            // 行3
            tableLayoutPanel.Controls.Add(digitButtons[4], 0, 3);
            tableLayoutPanel.Controls.Add(digitButtons[5], 1, 3);
            tableLayoutPanel.Controls.Add(digitButtons[6], 2, 3);
            tableLayoutPanel.Controls.Add(btnAdd, 3, 3);
            tableLayoutPanel.Controls.Add(btnEqual, 4, 3);

            // 行4
            tableLayoutPanel.Controls.Add(digitButtons[1], 0, 4);
            tableLayoutPanel.Controls.Add(digitButtons[2], 1, 4);
            tableLayoutPanel.Controls.Add(digitButtons[3], 2, 4);
            tableLayoutPanel.Controls.Add(btnPoint, 3, 4);
            tableLayoutPanel.Controls.Add(btnSign, 4, 4);

            // 行5 (0按钮跨两列)
            tableLayoutPanel.Controls.Add(digitButtons[0], 0, 5);
            tableLayoutPanel.SetColumnSpan(digitButtons[0], 2);
            tableLayoutPanel.Controls.Add(btnBackspace, 2, 5);
        }
        #endregion
    }
}
```

end