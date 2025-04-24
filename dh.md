说明：
用vue做一款录音系统

1.点击按钮，开始录制音频

2.录制过程中，可以暂停和停止录制 有时长显示

3.点击停止录制 可以保存音频，保存在本地  

4.找到刚刚保存的音频路径，可以点击播放 ，需要显示音频总时长

5.可以下载音频

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/525ba7db655145ea9bfcbee8aeb814f8.png#pic_center)

step1:

```typescript
<template>
  <div class="container">
    <h1>录音系统</h1>

    <!-- 录音控制 -->
    <div class="controls">
      <button @click="toggleRecording" :disabled="isPaused">
        {{ isRecording ? '正在录音...' : '开始录音' }}
      </button>
      <button @click="togglePause" :disabled="!isRecording">
        {{ isPaused ? '继续录音' : '暂停录音' }}
      </button>
      <button @click="stopRecording" :disabled="!isRecording && !isPaused">
        停止并保存
      </button>
    </div>

    <!-- 录音时长 -->
    <div class="duration">当前时长: {{ formatTime(currentDuration) }}</div>

    <!-- 保存的录音 -->
    <div class="recordings" v-if="currentRecording">
      <h2>保存的录音</h2>
      <div class="audio-item">
        <audio ref="audioPlayer" controls @loadedmetadata="updateDuration"></audio>
        <div>
          <button @click="playRecording">播放</button>
          <span>时长: {{ formatTime(currentRecording.duration) }}</span>
          <a :href="currentRecording.url" download="recording.webm">下载</a>
          <button @click="deleteRecording" class="delete-btn">删除</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isRecording: false,
      isPaused: false,
      currentDuration: 0,
      timer: null,
      mediaRecorder: null,
      audioChunks: [],
      currentRecording: null
    };
  },
  methods: {
    async toggleRecording() {
      if (!this.isRecording) {
        try {
          const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
          this.mediaRecorder = new MediaRecorder(stream);

          this.mediaRecorder.ondataavailable = event => {
            this.audioChunks.push(event.data);
          };

          this.mediaRecorder.onstop = () => {
            const audioBlob = new Blob(this.audioChunks, { type: 'audio/webm' });
            const audioUrl = URL.createObjectURL(audioBlob);

            // 只保留最新录音
            this.currentRecording = {
              url: audioUrl,
              duration: this.currentDuration
            };

            this.audioChunks = [];
            stream.getTracks().forEach(track => track.stop());
          };

          this.mediaRecorder.start();
          this.startTimer();
          this.isRecording = true;
          this.isPaused = false;
        } catch (err) {
          console.error('无法访问麦克风:', err);
        }
      }
    },

    // 新增删除方法
    deleteRecording() {
      if (this.currentRecording) {
        URL.revokeObjectURL(this.currentRecording.url);
        this.currentRecording = null;
      }
    },

    togglePause() {
      if (this.isPaused) {
        this.mediaRecorder.resume();
        this.startTimer();
      } else {
        this.mediaRecorder.pause();
        clearInterval(this.timer);
      }
      this.isPaused = !this.isPaused;
    },

    stopRecording() {
      this.mediaRecorder.stop();
      clearInterval(this.timer);
      this.isRecording = false;
      this.isPaused = false;
      this.currentDuration = 0;
    },

    startTimer() {
      this.timer = setInterval(() => {
        this.currentDuration++;
      }, 1000);
    },

    playRecording() {
      const audioElement = this.$refs.audioPlayer;
      audioElement.src = this.currentRecording.url;
      audioElement.play();
    },

    updateDuration() {
      const audioElement = this.$refs.audioPlayer;
      if (this.currentRecording) {
        this.currentRecording.duration = Math.round(audioElement.duration);
      }
    },

    formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = seconds % 60;
      return `${mins}:${secs.toString().padStart(2, '0')}`;
    }
  },
  beforeUnmount() {
    clearInterval(this.timer);
    if (this.currentRecording) {
      URL.revokeObjectURL(this.currentRecording.url);
    }
  }
};
</script>

<style>
/* 新增删除按钮样式 */
.delete-btn {
  background-color: #ff4444;
  margin-left: 10px;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.controls {
  margin: 20px 0;
  display: flex;
  gap: 10px;
}

.audio-item {
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.duration {
  font-size: 1.2em;
  margin: 10px 0;
}

button {
  padding: 8px 16px;
  cursor: pointer;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

a {
  margin-left: 10px;
  color: #2196F3;
  text-decoration: none;
}

</style>
```

end