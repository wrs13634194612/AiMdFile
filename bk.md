说明：
我希望用vue做一款记忆卡牌游戏
游戏规则如下：
游戏设置：使用3x4的网格，包含3对字母（A,B,C,D,E,F）。
随机洗牌：初始字母对被打乱顺序，生成随机布局。
游戏流程：玩家依次翻开两张牌，匹配则保留，否则隐藏。全部匹配后获胜

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7fa525630f3a444d998f40a92228e372.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Cards.vue

```typescript
<template>
  <div class="game-container">
    <!-- 游戏网格 -->
    <div class="grid">
      <div
        v-for="(card, index) in cards"
        :key="index"
        class="card"
        :class="{ flipped: card.revealed, matched: card.matched }"
        @click="selectCard(index)"
      >
        <div class="front">{{ card.id }}</div>
        <div class="back">{{ card.value }}</div>
      </div>
    </div>

    <!-- 游戏状态提示 -->
    <div v-if="message" class="message">{{ message }}</div>
    <div v-if="gameWon" class="win-message">You Win!</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rows: 3,
      cols: 4,
      cards: [],
      selected: [],
      message: '',
      gameWon: false,
      processing: false
    }
  },
  mounted() {
    this.initializeGame();
  },
  methods: {
    // 初始化游戏
    initializeGame() {
      // 生成成对字母
      const pairs = Array.from({ length: (this.rows * this.cols) / 2 }, (_, i) =>
        String.fromCharCode(65 + i)
      ).flatMap(c => [c, c]);

      // 打乱顺序
      for (let i = pairs.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [pairs[i], pairs[j]] = [pairs[j], pairs[i]];
      }

      // 初始化卡片数据
      this.cards = pairs.map((value, id) => ({
        id: id + 1,
        value,
        revealed: false,
        matched: false
      }));
    },

    // 处理卡片点击
    async selectCard(index) {
      if (
        this.processing ||              // 正在处理中
        this.cards[index].matched ||    // 已匹配卡片
        this.selected.includes(index)   // 重复选择同一卡片
      ) return;

      this.cards[index].revealed = true;
      this.selected.push(index);

      // 选择两张卡片后进行处理
      if (this.selected.length === 2) {
        this.processing = true;
        const [first, second] = this.selected;
        const match = this.cards[first].value === this.cards[second].value;

        if (match) {
          this.message = "Match!";
          this.cards[first].matched = true;
          this.cards[second].matched = true;

          // 检查胜利条件
          this.gameWon = this.cards.every(c => c.matched);
        } else {
          this.message = "No match...";
          // 延迟后翻转回来
          await new Promise(resolve => setTimeout(resolve, 1000));
          this.cards[first].revealed = false;
          this.cards[second].revealed = false;
        }

        // 重置状态
        await new Promise(resolve => setTimeout(resolve, 500));
        this.selected = [];
        this.message = '';
        this.processing = false;
      }
    }
  }
}
</script>

<style>
.game-container {
  max-width: 600px;
  margin: 20px auto;
}

.grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.card {
  height: 100px;
  border: 2px solid #ccc;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  cursor: pointer;
  position: relative;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.card.flipped {
  transform: rotateY(180deg);
}

.card.matched {
  background-color: #90EE90;
}

.front, .back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
}

.back {
  transform: rotateY(180deg);
}

.message {
  margin-top: 20px;
  font-size: 24px;
  text-align: center;
}

.win-message {
  margin-top: 20px;
  font-size: 32px;
  color: green;
  text-align: center;
  font-weight: bold;
}
</style>

```

end