说明：我希望用vue实现猜词游戏
Vue Wordle 游戏规则总结

核心规则
单词选择
目标单词从预设词库（DEFAULT_WORDS）中随机选取，均为5字母单词（如apple、zebra等）。
输入要求
长度限制：必须输入恰好5个字母
有效性验证：单词需存在于预设词库中
输入方式：通过文本输入框输入，按回车键提交
尝试次数
最多允许 6次猜测，用尽后游戏失败。
颜色反馈机制
每个字母根据匹配状态显示不同颜色：
绿色：字母位置完全正确
黄色：字母存在但位置错误
灰色：字母不在目标单词中
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/4d0bc517c37c4775ad218a1f8f6996bd.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Wordle.vue

```typescript
<template>
  <div class="container">

    <div v-if="!gameStarted" class="start-screen">
      <button class="start-btn" @click="startGame">Start New Game</button>
    </div>

    <div v-else>
      <div class="game-board">
        <div v-for="(guess, index) in guesses" :key="index" class="guess-row">
          <div
            v-for="(char, charIndex) in guess.chars"
            :key="charIndex"
            :class="['char-box', colorClass(guess.results[charIndex])]"
          >
            {{ char }}
          </div>
        </div>
      </div>

      <div v-if="!gameOver" class="input-area">
        <input
          v-model="currentGuess"
          @keyup.enter="submitGuess"
          maxlength="5"
          placeholder="Enter 5-letter word"
          class="guess-input"
          autocomplete="off"
        />
        <p v-if="message" class="message">{{ message }}</p>
      </div>

      <div v-else class="game-over">
        <h3 v-if="isWin" class="result-text">🎉 Congratulations! You won!</h3>
        <h3 v-else class="result-text">😞 Game Over! The word was: {{ targetWord.toUpperCase() }}</h3>
        <button class="play-again-btn" @click="startGame">Play Again</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'

const DEFAULT_WORDS = [
  'apple', 'brain', 'chair', 'dance', 'earth',
  'flame', 'grape', 'happy', 'igloo', 'jelly',
  'koala', 'lemon', 'music', 'noble', 'ocean',
  'piano', 'quiet', 'river', 'smile', 'tiger',
  'urban', 'vivid', 'water', 'xenon', 'yacht', 'zebra'
]

// 游戏状态
const gameStarted = ref(false)
const gameOver = ref(false)
const isWin = ref(false)
const currentGuess = ref('')
const guesses = reactive([])
const message = ref('')

// 游戏数据
const targetWord = ref('')
const words = ref(DEFAULT_WORDS)

const COLOR = {
  GREEN: 'green',
  YELLOW: 'yellow',
  GRAY: 'gray'
}

const startGame = () => {
  gameStarted.value = true
  gameOver.value = false
  isWin.value = false
  currentGuess.value = ''
  guesses.splice(0)
  message.value = ''
  targetWord.value = words.value[Math.floor(Math.random() * words.value.length)]
}

const colorClass = (result) => ({
  [COLOR.GREEN]: result === COLOR.GREEN,
  [COLOR.YELLOW]: result === COLOR.YELLOW,
  [COLOR.GRAY]: result === COLOR.GRAY
})

const validateWord = (word) => words.value.includes(word.toLowerCase())

const calculateResults = (guess) => {
  const results = Array(5).fill(COLOR.GRAY)
  const targetChars = [...targetWord.value]
  const guessChars = [...guess]

  // 第一遍检查正确位置（绿色）
  guessChars.forEach((char, i) => {
    if (char === targetChars[i]) {
      results[i] = COLOR.GREEN
      targetChars[i] = null
    }
  })

  // 第二遍检查存在但位置错误（黄色）
  guessChars.forEach((char, i) => {
    if (results[i] !== COLOR.GREEN) {
      const foundIndex = targetChars.findIndex(c => c === char)
      if (foundIndex > -1) {
        results[i] = COLOR.YELLOW
        targetChars[foundIndex] = null
      }
    }
  })

  return results
}

const submitGuess = () => {
  const guess = currentGuess.value.toLowerCase().trim()

  if (guess.length !== 5) {
    message.value = 'Word must be 5 letters'
    return
  }

  if (!validateWord(guess)) {
    message.value = 'Not in word list'
    return
  }

  message.value = ''
  const results = calculateResults(guess)

  guesses.push({
    chars: [...guess.toUpperCase()],
    results
  })

  // 检查胜利条件
  if (results.every(r => r === COLOR.GREEN)) {
    isWin.value = true
    gameOver.value = true
  } else if (guesses.length >= 6) {
    gameOver.value = true
  }

  currentGuess.value = ''
}
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 2rem;
  background: #f8f9fa;
  border-radius: 16px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.12);
  font-family: 'Arial', sans-serif;
}

h1 {
  color: #2c3e50;
  margin-bottom: 2rem;
  font-size: 2.5rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}

.game-board {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
  margin: 1.5rem 0;
}

.guess-row {
  display: flex;
  justify-content: center;
  gap: 0.75rem;
  margin: 0.75rem 0;
}

.char-box {
  width: 60px;
  height: 60px;
  border: 2px solid #d3d6da;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 2rem;
  font-weight: bold;
  background: white;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

.char-box.green {
  background: #6aaa64;
  border-color: #6aaa64;
  color: white;
}

.char-box.yellow {
  background: #c9b458;
  border-color: #c9b458;
  color: white;
}

.char-box.gray {
  background: #787c7e;
  border-color: #787c7e;
  color: white;
}

.input-area {
  margin: 2rem 0;
}

.guess-input {
  padding: 1rem;
  font-size: 1.2rem;
  border: 2px solid #d3d6da;
  border-radius: 8px;
  width: 220px;
  text-align: center;
  transition: all 0.3s ease;
  text-transform: lowercase;
}

.guess-input:focus {
  outline: none;
  border-color: #6aaa64;
  box-shadow: 0 0 8px rgba(106, 170, 100, 0.3);
}

button {
  padding: 1rem 2rem;
  font-size: 1.1rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  background: #4a90e2;
  color: white;
  transition: all 0.2s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

button:hover {
  background: #357abd;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.start-btn {
  background: #6aaa64;
}

.start-btn:hover {
  background: #5a9c54;
}

.message {
  color: #e74c3c;
  margin-top: 1rem;
  font-weight: 500;
  min-height: 1.5rem;
}

.game-over {
  margin-top: 2rem;
  padding: 1.5rem;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05);
}

.result-text {
  margin: 1rem 0;
  color: #2c3e50;
  font-size: 1.2rem;
}

.play-again-btn {
  background: #6aaa64;
  margin-top: 1rem;
}

.play-again-btn:hover {
  background: #5a9c54;
}
</style>
```

end