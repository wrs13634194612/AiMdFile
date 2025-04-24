说明：
vue实现俄罗斯方块
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0560bc4e0eb648b1b406a0bbaac35fca.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Game.vue

```typescript
 <script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

const SHAPES = [
  [[1, 1, 1, 1]], // I
  [[1, 1], [1, 1]], // O
  [[1, 1, 1], [0, 1, 0]], // T
  [[1, 1, 0], [0, 1, 1]], // Z
  [[0, 1, 1], [1, 1, 0]], // S
  [[1, 1, 1], [0, 0, 1]], // L
  [[1, 1, 1], [1, 0, 0]]  // J
]

// 游戏状态
const board = ref(Array(20).fill().map(() => Array(10).fill(0)))
const currentBlock = ref(null)
const position = reactive({ x: 0, y: 0 })
const score = ref(0)
const gameLoopTimer = ref(null)
const isPlaying = ref(false)
const isPaused = ref(false)

// 生成新方块
const createBlock = () => {
  const shape = SHAPES[Math.floor(Math.random() * SHAPES.length)]
  currentBlock.value = shape
  position.x = Math.floor((10 - shape[0].length) / 2)
  position.y = 0
}

// 碰撞检测
const checkCollision = (dx, dy) => {
  return currentBlock.value?.some((row, i) =>
    row.some((cell, j) => {
      const nx = position.x + j + dx
      const ny = position.y + i + dy
      return cell && (
        nx < 0 || nx >= 10 ||
        ny >= 20 ||
        board.value[ny]?.[nx]
      )
    })
  )
}

// 移动方块
const moveDown = () => {
  if (!checkCollision(0, 1)) {
    position.y++
  } else {
    mergeBlock()
    clearLines()
    createBlock()
  }
}

// 合并方块到面板
const mergeBlock = () => {
  currentBlock.value.forEach((row, i) => {
    row.forEach((cell, j) => {
      const y = position.y + i
      const x = position.x + j
      if (cell && y >= 0) {
        board.value[y][x] = 1
      }
    })
  })
}

// 消除满行
const clearLines = () => {
  let lines = 0
  for (let y = 19; y >= 0; y--) {
    if (board.value[y].every(cell => cell)) {
      board.value.splice(y, 1)
      board.value.unshift(Array(10).fill(0))
      lines++
      y++
    }
  }
  score.value += lines * 100
}

// 游戏结束处理
const gameOver = () => {
  stopGame()
  alert('Game Over!')
}

// 游戏循环
const gameLoop = () => {
  moveDown()
  if (checkCollision(0, 0)) {
    gameOver()
  }
}

// 开始游戏
const startGame = () => {
  board.value = Array(20).fill().map(() => Array(10).fill(0))
  score.value = 0
  isPlaying.value = true
  isPaused.value = false
  createBlock()
  gameLoopTimer.value = setInterval(gameLoop, 300)
}

// 停止游戏
const stopGame = () => {
  isPlaying.value = false
  isPaused.value = false
  clearInterval(gameLoopTimer.value)
  board.value = Array(20).fill().map(() => Array(10).fill(0))
  score.value = 0
  currentBlock.value = null
}

// 暂停/恢复游戏
const togglePause = () => {
  if (!isPlaying.value) return
  if (isPaused.value) {
    gameLoopTimer.value = setInterval(gameLoop, 300)
    isPaused.value = false
  } else {
    clearInterval(gameLoopTimer.value)
    isPaused.value = true
  }
}

// 键盘控制
const handleKeyDown = (e) => {
  if (!isPlaying.value || isPaused.value) return
  switch(e.key) {
    case 'ArrowLeft':
      if (!checkCollision(-1, 0)) position.x--
      break
    case 'ArrowRight':
      if (!checkCollision(1, 0)) position.x++
      break
    case 'ArrowDown':
      moveDown()
      break
    case 'ArrowUp':
      rotateBlock()
      break
    case ' ':
      togglePause()
      break
  }
}

// 生命周期钩子
onMounted(() => {
  window.addEventListener('keydown', handleKeyDown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeyDown)
  clearInterval(gameLoopTimer.value)
})

// 判断当前方块位置
const isCurrentBlock = (x, y) => {
  return currentBlock.value?.some((row, i) =>
    row.some((cell, j) =>
      cell &&
      x === position.x + j &&
      y === position.y + i
    )
  )
}

// 旋转方块（保持不变）
const rotateBlock = () => {
  if (!currentBlock.value) return

  const rotated = currentBlock.value[0].map((_, i) =>
    currentBlock.value.map(row => row[i]).reverse()
  )

  if (JSON.stringify(rotated) === JSON.stringify(currentBlock.value)) return

  const originalBlock = currentBlock.value
  currentBlock.value = rotated

  let offset = 0
  while (checkCollision(0, 0)) {
    offset++
    position.x += position.x >= 5 ? -1 : 1
    if (offset > 2) {
      currentBlock.value = originalBlock
      return
    }
  }

  if (checkCollision(0, 0)) {
    currentBlock.value = originalBlock
  }
}

// 移动控制方法（保持不变）
const moveLeft = () => {
  if (!checkCollision(-1, 0)) position.x--
}

const moveRight = () => {
  if (!checkCollision(1, 0)) position.x++
}

const moveDownBtn = () => {
  moveDown()
}

const rotateBtn = () => {
  rotateBlock()
}
</script>

<template>
  <div class="game-container">
    <!-- 左侧游戏界面 -->
    <div class="game-board" :style="{ width: 10 * 25 + 'px' }">
      <div v-for="(row, y) in board" :key="y" class="row">
        <div
          v-for="(cell, x) in row"
          :key="x"
          class="cell"
          :class="{
            'filled': cell,
            'current-block': isCurrentBlock(x, y)
          }"
        ></div>
      </div>
    </div>

    <!-- 右侧控制面板 -->
    <div class="control-panel">
      <div class="game-info">
        <div class="score">Score: {{ score }}</div>
        <div class="action-buttons">
          <button @click="startGame">Start</button>
          <button @click="togglePause" :disabled="!isPlaying">
            {{ isPaused ? 'Resume' : 'Pause' }}
          </button>
          <button @click="stopGame" :disabled="!isPlaying">Stop</button>
        </div>
      </div>

      <!-- 控制按钮 -->
      <div class="control-buttons">
        <div class="button-row">
          <button class="control-btn rotate" @click="rotateBtn" :disabled="!isPlaying || isPaused">
            <svg width="24" height="24" viewBox="0 0 24 24">
              <path fill="currentColor" d="M12 6v3l4-4-4-4v3c-4.42 0-8 3.58-8 8c0 1.57.46 3.03 1.24 4.26L6.7 14.8c-.45-.83-.7-1.79-.7-2.8c0-3.31 2.69-6 6-6m6.76 1.74L17.3 9.2c.44.84.7 1.79.7 2.8c0 3.31-2.69 6-6 6v-3l-4 4l4 4v-3c4.42 0 8-3.58 8-8c0-1.57-.46-3.03-1.24-4.26Z"/>
            </svg>
          </button>
        </div>
        <div class="button-row">
          <button class="control-btn" @click="moveLeft" :disabled="!isPlaying || isPaused">←</button>
          <button class="control-btn" @click="moveDownBtn" :disabled="!isPlaying || isPaused">↓</button>
          <button class="control-btn" @click="moveRight" :disabled="!isPlaying || isPaused">→</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 50px;
  padding: 20px;
  background: #2d2d2d;
  min-height: 100vh;
}

/* 游戏板样式 */
.game-board {
  background: #000;
  border: 2px solid #444;
  box-shadow: 0 8px 24px rgba(0,0,0,0.3);
  border-radius: 8px;
  overflow: hidden;
}

.row {
  display: flex;
}

.cell {
  width: 25px;
  height: 25px;
  border: 1px solid #333;
  transition: background 0.1s ease;
}

.filled {
  background: #FF5252;
  box-shadow: inset 0 0 8px rgba(255,0,0,0.3);
}

.current-block {
  background: #69F0AE;
  box-shadow: inset 0 0 12px rgba(0,255,0,0.4);
}

/* 控制面板 */
.control-panel {
  background: rgba(255,255,255,0.1);
  padding: 30px;
  border-radius: 16px;
  backdrop-filter: blur(8px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.2);
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.game-info {
  display: flex;
  flex-direction: column;
  gap: 20px;
  color: white;
}

.score {
  font-size: 24px;
  font-weight: 600;
  text-shadow: 0 2px 4px rgba(0,0,0,0.2);
  color: #FFF;
}

/* 动作按钮组 */
.action-buttons {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.action-buttons button {
  width: 100%;
  padding: 14px 20px;
  font-size: 16px;
  font-weight: 500;
  border: none;
  border-radius: 10px;
  background: linear-gradient(145deg, #2196F3, #1976D2);
  color: white;
  box-shadow: 0 4px 6px rgba(0,0,0,0.15);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
}

/* 控制按钮组 */
.control-buttons {
  display: flex;
  flex-direction: column;
  gap: 25px;
  align-items: center;
}

.button-row {
  display: flex;
  gap: 20px;
}

/* 控制按钮 */
.control-btn {
  width: 75px;
  height: 75px;
  font-size: 28px;
  border: none;
  border-radius: 15px;
  background: linear-gradient(145deg, #4CAF50, #45a049);
  color: white;
  box-shadow: 0 5px 15px rgba(0,0,0,0.2);
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 特殊按钮样式 */
.control-btn.rotate {
  background: linear-gradient(145deg, #FF9800, #fb8c00);
}

.control-btn:nth-child(1),  /* 左 */
.control-btn:nth-child(3) { /* 右 */
  background: linear-gradient(145deg, #2196F3, #1976D2);
}

/* 交互效果 */
.action-buttons button:hover,
.control-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(0,0,0,0.3);
  filter: brightness(110%);
}

.action-buttons button:active,
.control-btn:active:not(:disabled) {
  transform: scale(0.96);
  box-shadow: 0 3px 6px rgba(0,0,0,0.2);
}

/* 禁用状态 */
.control-btn:disabled,
.action-buttons button:disabled {
  background: linear-gradient(145deg, #616161, #424242);
  color: #bdbdbd;
  box-shadow: none;
  cursor: not-allowed;
  filter: grayscale(70%);
}

/* 响应式调整 */
@media (max-width: 768px) {
  .game-container {
    flex-direction: column;
    align-items: center;
    padding: 15px;
  }

  .control-btn {
    width: 60px;
    height: 60px;
  }
}
</style>
```

end