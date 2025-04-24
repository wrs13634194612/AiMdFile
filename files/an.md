vue模拟扑克效果

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/83a689dca14f4469aa7b7ae49a816625.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled18\src\views\Home.vue

```typescript
<template>
  <div class="poker-container">
    <!-- 使用复合数据对象实现双行显示 -->
    <div
      v-for="(card, index) in POKER_CARDS"
      :key="index"
      class="poker-card"
      :class="[cardColors(card.suit), { 'joker-card': card.type === 'JOKER' }]"
    >
     <template v-if="card.type === 'JOKER'">
        <!-- 修改后的JOKER结构 -->
        <div class="card-top">{{ card.size === 'little' ? '🦈' : '🐬' }}</div>
        <div class="card-center" :class="card.size === 'big' ? 'red-text' : 'gray-text'">JOKER</div>
        <div class="card-bottom">{{ card.size === 'little' ? '🦈' : '🐬' }}</div>
      </template>
      <template v-else>
        <div class="card-top">{{ card.suit }}</div>
        <div class="card-center">{{ card.number }}</div>
        <div class="card-bottom">{{ card.suit }}</div>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      POKER_CARDS: [
        { type: 'JOKER', size: 'big' },    // 大王
        { type: 'JOKER', size: 'little' }, // 新增的小王
        { suit: '♠', number: '2' },
        { suit: '♣', number: '2' },
        { suit: '♥', number: 'A' },
        { suit: '♦', number: 'A' },
        { suit: '♠', number: 'A' },
        { suit: '♥', number: 'K' },
        { suit: '♣', number: 'K' },
        { suit: '♠', number: 'K' },
        { suit: '♦', number: '10' },
        { suit: '♠', number: '10' },
        { suit: '♥', number: '8' },
        { suit: '♦', number: '8' },
        { suit: '♥', number: '8' },
        { suit: '♠', number: '8' },
        { suit: '♦', number: '4' }
      ]
    }
  },
  methods: {
    cardColors(suit) {
      return {
        'black-text': ['♠', '♣'].includes(suit),
        'red-text': ['♥', '♦'].includes(suit)
      }
    }
  }
}
</script>

<style>
.poker-container {
  display: flex;
  gap: 12px;
  padding: 20px;
  background: white;
  flex-wrap: wrap;
  justify-content: center;
}

.poker-card {
  width: 80px;
  height: 120px;
  border: 1px solid #ddd;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.black-text { color: #333; }
.red-text { color: #c00; }


.card-top, .card-bottom {
  font-size: 1.2em;
  transform: scaleY(0.8);
}

.card-center {
  font-size: 1.8em;
  font-weight: bold;
  text-align: center;
}

.card-bottom {
  transform: rotate(180deg);
}
/* 新增灰色文本样式 */
.gray-text { color: #666; }

/* 保持JOKER文字与普通卡片一致 */
.joker-card .card-center {
  font-size:22px;
  font-weight: bold;
  text-align: center;
  /* 通过缩放保持原有布局效果 */
  transform: scaleY(0.8);
}


</style>
```

end