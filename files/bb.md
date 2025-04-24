说明：vue用D3.js实现轮盘抽奖
 vue 用  D3.js  绘制圆形，分为六个扇形 每个扇形不同颜色

1.每个扇形添加文字
2.圆心添加指针 可以指向扇形
3.添加按钮 开始动画
4.点击按钮，开始让轮盘旋转
5.弹窗显示最终指针指向的扇形
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a7505a9a018c41d5977d953800836ad8.png#pic_center)

step1:添加d3依赖 

```bash
npm install d3

  "dependencies": {
    "audiobuffer-to-wav": "^1.0.0",
    "axios": "^1.8.4",
    "d3": "^7.9.0",
    "html2canvas": "^1.4.1",
    "phaser": "^3.88.2",
    "vue": "^3.5.13",
    "vue-router": "^4.5.0"
  },
  
```

step2:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Lottery.vue

```typescript
<template>
  <div class="pie-chart">
    <svg ref="svg"></svg>
    <button class="spin-button" @click="startSpin">开始旋转</button>
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <p>抽奖结果：{{ selectedLabel }}</p>
        <button @click="showModal = false">确定</button>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  data() {
    return {
      chartData: [
        { color: '#FF0000', probability: 0.01, label: '特等奖' },
        { color: '#FF7F00', probability: 0.03, label: '一等奖' },
        { color: '#FFFF00', probability: 0.05, label: '二等奖' },
        { color: '#00FF00', probability: 0.15, label: '三等奖' },
        { color: '#0000FF', probability: 0.18, label: '四等奖' },
        { color: '#4B0082', probability: 0.20, label: '五等奖' },
        { color: '#8B00FF', probability: 0.38, label: '谢谢参与' }
      ],
      currentRotation: 0,
      showModal: false,
      resultSector: null,
      radius: 0,
      width: 300,
      height: 300,
      angleMap: [] // 新增角度映射表
    };
  },
  computed: {
    selectedLabel() {
      return this.resultSector !== null
        ? this.chartData[this.resultSector].label
        : '';
    }
  },
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      const svg = d3.select(this.$refs.svg)
        .attr('width', this.width)
        .attr('height', this.height);

      // 添加箭头标记
      const defs = svg.append('defs');
      defs.append('marker')
        .attr('id', 'arrowhead')
        .attr('viewBox', '0 0 10 10')
        .attr('refX', 5)
        .attr('refY', 5)
        .attr('markerWidth', 6)
        .attr('markerHeight', 6)
        .attr('orient', 'auto')
        .append('path')
        .attr('d', 'M 0 0 L 10 5 L 0 10 z')
        .attr('fill', '#333');

      // 创建转盘组
      const chartGroup = svg.append('g')
        .attr('transform', `translate(${this.width/2},${this.height/2})`);

      // 创建指针
      svg.append('g')
        .attr('transform', `translate(${this.width/2},${this.height/2})`)
        .append('line')
        .attr('x1', 0)
        .attr('y1', 0)
        .attr('x2', 0)
        .attr('y2', -this.radius + 15)
        .attr('stroke', '#333')
        .attr('stroke-width', 2)
        .attr('marker-end', 'url(#arrowhead)');

      // 创建饼图（使用概率值）
      const pie = d3.pie()
        .value(d => d.probability)
        .sort(null);
      const arc = d3.arc()
        .innerRadius(0)
        .outerRadius(this.radius);

      // 绘制扇形
      chartGroup.selectAll('path')
        .data(pie(this.chartData))
        .join('path')
        .attr('d', arc)
        .attr('fill', d => d.data.color)
        .attr('stroke', '#fff')
        .attr('stroke-width', 2);

      // 添加文字标签
      chartGroup.selectAll('text')
        .data(pie(this.chartData))
        .join('text')
        .attr('transform', d => `translate(${arc.centroid(d)})`)
        .attr('text-anchor', 'middle')
        .attr('dy', '0.3em')
        .style('font-size', '12px')
        .style('fill', '#333')
        .text(d => d.data.label);
    },

    startSpin() {
    // 概率抽奖逻辑（保持原C++算法）
    const total = this.chartData.reduce((sum, item) => sum + item.probability, 0);
    const random = Math.random() * total;
    let accumulated = 0;
    let selectedIndex = 0;

    for (let i = 0; i < this.chartData.length; i++) {
      accumulated += this.chartData[i].probability;
      if (random <= accumulated) {
        selectedIndex = i;
        break;
      }
    }

    // 生成角度映射表（关键修复）
    const pieGenerator = d3.pie()
      .value(d => d.probability)
      .sort(null);
    const pieData = pieGenerator(this.chartData);

    // 转换为角度映射表（保存每个扇区的起始和结束角度）
    this.angleMap = pieData.map(d => ({
      start: d.startAngle * (180 / Math.PI),
      end: d.endAngle * (180 / Math.PI)
    }));

    // 计算目标旋转角度（精确角度对齐）
    const targetAngle = 360 - ((pieData[selectedIndex].startAngle + pieData[selectedIndex].endAngle) / 2 * (180 / Math.PI));
    const targetRotation = 5 * 360 + targetAngle;

    // 执行旋转动画
    const chartGroup = d3.select(this.$refs.svg).select('g');

    chartGroup.transition()
      .duration(3000)
      .ease(d3.easeCubicOut)
      .attrTween('transform', () => {
        const i = d3.interpolate(this.currentRotation, targetRotation);
        return t => `translate(${this.width/2},${this.height/2}) rotate(${i(t)})`;
      })
      .on('end', () => {
        // 精确计算最终指向（关键修复）
        this.currentRotation = targetRotation % 360;
        const pointerAngle = (360 - this.currentRotation) % 360;

        // 根据实际角度查找对应扇区
        this.resultSector = this.angleMap.findIndex(({ start, end }) => {
          if (end < start) { // 处理跨0度情况
            return pointerAngle >= start || pointerAngle < end;
          }
          return pointerAngle >= start && pointerAngle < end;
        });

        this.showModal = true;
        // 在.on('end')回调内添加
        console.log('理论索引:', selectedIndex, '实际索引:', this.resultSector);
        console.log('角度分布:', this.angleMap);
        console.log('指针角度:', pointerAngle);
      });
  }
  },
  created() {
    this.radius = Math.min(this.width, this.height) / 2;
  }
}
</script>

<style>
/* 原有样式保持不变 */
.pie-chart {
  margin: 20px;
  position: relative;
}

.spin-button {
  margin-top: 20px;
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.spin-button:hover {
  background: #45a049;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}

.modal-content button {
  margin-top: 10px;
  padding: 5px 15px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}
</style>
```
到这里，抽奖功能就完成了，直接运行就可以
step3:c++算法  可选  仅供参考 
C:\Users\wangrusheng\source\repos\CMakeProject1\CMakeProject1\CMakeProject1.cpp

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <random>
#include <iomanip>
#include <cstdint>  // 添加必要的头文件

struct Prize {
    std::string name;
    std::string color;
    double probability;
};

class LotterySystem {
private:
    std::vector<Prize> prizes;
    std::mt19937_64 random_engine;

    // 修正的初始化方法
    void initialize_random_engine() {
        std::random_device rd;
        std::seed_seq seed_seq{
            static_cast<uint_fast64_t>(rd()),
            static_cast<uint_fast64_t>(rd()),
            static_cast<uint_fast64_t>(rd())
        };
        random_engine.seed(seed_seq);
    }

public:
    explicit LotterySystem(const std::vector<Prize>& prize_list)
        : prizes(prize_list) {
        initialize_random_engine();
    }

    // 核心抽奖方法
    const Prize& draw() {
        double total = 0.0;
        for (const auto& prize : prizes) {
            total += prize.probability;
        }

        std::uniform_real_distribution<double> dist(0.0, total);
        double random = dist(random_engine);

        double accumulated = 0.0;
        for (const auto& prize : prizes) {
            accumulated += prize.probability;
            if (random <= accumulated) {
                return prize;
            }
        }

        return prizes.back();
    }

    // 概率验证方法（保持原有实现）
    void validateProbabilities(int trials = 1000000) {
        std::vector<int> counts(prizes.size(), 0);

        for (int i = 0; i < trials; ++i) {
            const Prize& result = draw();
            for (size_t j = 0; j < prizes.size(); ++j) {
                if (prizes[j].name == result.name) {
                    counts[j]++;
                    break;
                }
            }
        }

        std::cout << "概率验证 (" << trials << " 次测试):\n";
        for (size_t i = 0; i < prizes.size(); ++i) {
            double actual = static_cast<double>(counts[i]) / trials;
            std::cout << std::setw(8) << prizes[i].name << ": "
                << "预期 " << std::setw(5) << prizes[i].probability * 100 << "%, "
                << "实际 " << std::setw(5) << actual * 100 << "%\n";
        }
    }
};

int main() {
    // 保持原有配置
    std::vector<Prize> prizes = {
        { "特等奖", "#FF0000", 0.01 },
        { "一等奖", "#FF7F00", 0.03 },
        { "二等奖", "#FFFF00", 0.05 },
        { "三等奖", "#00FF00", 0.15 },
        { "四等奖", "#0000FF", 0.18 },
        { "五等奖", "#4B0082", 0.20 },
        { "谢谢参与", "#8B00FF", 0.38 }
    };

    LotterySystem lottery(prizes);

    // 示例运行
    const Prize& result = lottery.draw();
    std::cout << "抽奖结果：" << result.name << "\n\n";

    lottery.validateProbabilities();

    return 0;
}
```
///我是分割线
step101:旧版本等分抽奖程序

```typescript
参考c++的算法逻辑，修改我的vue抽奖程序


#include <iostream>
#include <vector>
#include <string>
#include <random>
#include <iomanip>
#include <cstdint>  // 添加必要的头文件

struct Prize {
    std::string name;
    std::string color;
    double probability;
};

class LotterySystem {
private:
    std::vector<Prize> prizes;
    std::mt19937_64 random_engine;

    // 修正的初始化方法
    void initialize_random_engine() {
        std::random_device rd;
        std::seed_seq seed_seq{
            static_cast<uint_fast64_t>(rd()),
            static_cast<uint_fast64_t>(rd()),
            static_cast<uint_fast64_t>(rd())
        };
        random_engine.seed(seed_seq);
    }

public:
    explicit LotterySystem(const std::vector<Prize>& prize_list)
        : prizes(prize_list) {
        initialize_random_engine();
    }

    // 核心抽奖方法
    const Prize& draw() {
        double total = 0.0;
        for (const auto& prize : prizes) {
            total += prize.probability;
        }

        std::uniform_real_distribution<double> dist(0.0, total);
        double random = dist(random_engine);

        double accumulated = 0.0;
        for (const auto& prize : prizes) {
            accumulated += prize.probability;
            if (random <= accumulated) {
                return prize;
            }
        }

        return prizes.back();
    }

    // 概率验证方法（保持原有实现）
    void validateProbabilities(int trials = 1000000) {
        std::vector<int> counts(prizes.size(), 0);

        for (int i = 0; i < trials; ++i) {
            const Prize& result = draw();
            for (size_t j = 0; j < prizes.size(); ++j) {
                if (prizes[j].name == result.name) {
                    counts[j]++;
                    break;
                }
            }
        }

        std::cout << "概率验证 (" << trials << " 次测试):\n";
        for (size_t i = 0; i < prizes.size(); ++i) {
            double actual = static_cast<double>(counts[i]) / trials;
            std::cout << std::setw(8) << prizes[i].name << ": "
                << "预期 " << std::setw(5) << prizes[i].probability * 100 << "%, "
                << "实际 " << std::setw(5) << actual * 100 << "%\n";
        }
    }
};

int main() {
    // 保持原有配置
    std::vector<Prize> prizes = {
        { "特等奖", "#FF0000", 0.01 },
        { "一等奖", "#FF7F00", 0.03 },
        { "二等奖", "#FFFF00", 0.05 },
        { "三等奖", "#00FF00", 0.15 },
        { "四等奖", "#0000FF", 0.18 },
        { "五等奖", "#4B0082", 0.20 },
        { "谢谢参与", "#8B00FF", 0.38 }
    };

    LotterySystem lottery(prizes);

    // 示例运行
    const Prize& result = lottery.draw();
    std::cout << "抽奖结果：" << result.name << "\n\n";

    lottery.validateProbabilities();

    return 0;
}
 
 <template>
  <div class="pie-chart">
    <svg ref="svg"></svg>
    <button class="spin-button" @click="startSpin">开始旋转</button>
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <p>指针指向的扇形是：{{ selectedLabel }}</p>
        <button @click="showModal = false">确定</button>
      </div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';

export default {
  data() {
    return {
      chartData: [
        { value: 1, color: '#FF6B6B', label: 'A' }, // 0-60
        { value: 1, color: '#4ECDC4', label: 'B' }, // 60-120
        { value: 1, color: '#45B7D1', label: 'C' }, // 120-180
        { value: 1, color: '#96CEB4', label: 'D' }, // 180-240
        { value: 1, color: '#FFEEAD', label: 'E' }, // 240-300
        { value: 1, color: '#D4A5A5', label: 'F' }  // 300-360
      ],
      currentRotation: 0,
      showModal: false,
      resultSector: null,
      radius: 0,
      width: 200,
      height: 200
    };
  },
  computed: {
   selectedLabel() {
      return this.resultSector !== null
        ? this.chartData[this.resultSector].label
        : '';
    }
  },
  mounted() {
    this.drawChart();
  },
  methods: {
    drawChart() {
      const svg = d3.select(this.$refs.svg)
        .attr('width', this.width)
        .attr('height', this.height);

      // 添加箭头标记
      const defs = svg.append('defs');
      defs.append('marker')
        .attr('id', 'arrowhead')
        .attr('viewBox', '0 0 10 10')
        .attr('refX', 5)
        .attr('refY', 5)
        .attr('markerWidth', 6)
        .attr('markerHeight', 6)
        .attr('orient', 'auto')
        .append('path')
        .attr('d', 'M 0 0 L 10 5 L 0 10 z')
        .attr('fill', '#333');

      // 创建转盘组
      const chartGroup = svg.append('g')
        .attr('transform', `translate(${this.width/2},${this.height/2})`);

      // 创建指针
      svg.append('g')
        .attr('transform', `translate(${this.width/2},${this.height/2})`)
        .append('line')
        .attr('x1', 0)
        .attr('y1', 0)
        .attr('x2', 0)
        .attr('y2', -this.radius + 15)
        .attr('stroke', '#333')
        .attr('stroke-width', 2)
        .attr('marker-end', 'url(#arrowhead)');

      // 创建饼图
      const pie = d3.pie().value(d => d.value).sort(null);
      const arc = d3.arc()
        .innerRadius(0)
        .outerRadius(this.radius);

      // 绘制扇形
      chartGroup.selectAll('path')
        .data(pie(this.chartData))
        .join('path')
        .attr('d', arc)
        .attr('fill', d => d.data.color)
        .attr('stroke', '#fff')
        .attr('stroke-width', 2);

      // 添加文字标签
      chartGroup.selectAll('text')
        .data(pie(this.chartData))
        .join('text')
        .attr('transform', d => `translate(${arc.centroid(d)})`)
        .attr('text-anchor', 'middle')
        .attr('dy', '0.3em')
        .style('font-size', '12px')
        .style('fill', '#333')
        .text(d => d.data.label);
    },
    startSpin() {
      const targetSector = Math.floor(Math.random() * 6);
      const sectorSize = 360 / this.chartData.length;
      const midAngle = targetSector * sectorSize + sectorSize/2;

      // 修正旋转角度计算
      const targetRotation = 5 * 360 + (360 - midAngle);
      const additionalRotation = targetRotation - this.currentRotation;

      const chartGroup = d3.select(this.$refs.svg).select('g');

      chartGroup.transition()
        .duration(3000)
        .ease(d3.easeCubicOut)
        .attrTween('transform', () => {
          const i = d3.interpolate(this.currentRotation, targetRotation);
          return t => `translate(${this.width/2},${this.height/2}) rotate(${i(t)})`;
        })
        .on('end', () => {
          // 修正结果计算逻辑
          this.currentRotation = targetRotation % 360;
          const pointerAngle = (360 - this.currentRotation) % 360;
          this.resultSector = Math.floor(pointerAngle / sectorSize);
          this.showModal = true;
        });
    }
  },
  created() {
   this.radius = Math.min(this.width, this.height) / 2;
  }
}
</script>

<style>
.pie-chart {
  margin: 20px;
  position: relative;
}

.spin-button {
  margin-top: 20px;
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.spin-button:hover {
  background: #45a049;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}

.modal-content button {
  margin-top: 10px;
  padding: 5px 15px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
}
</style>
```

end