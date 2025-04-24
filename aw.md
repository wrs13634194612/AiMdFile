说明：
vue录制视频保存本地
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f29c32fa2a294fd7a3c0bea3f890fcdc.png#pic_center)

step1:C:\Users\wangrusheng\PycharmProjects\untitled3\src\views\Video.vue

```typescript
<template>
  <div class="camera-recorder">
    <video ref="videoPreview" autoplay playsinline></video>
    <div class="controls">
      <button @click="startRecording" :disabled="isRecording">开始录制</button>
      <button @click="stopRecording" :disabled="!isRecording">停止并保存</button>
    </div>
    <div v-if="downloadUrl" class="download-section">
      <a :href="downloadUrl" download="recording.webm">下载视频</a>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      mediaStream: null,
      mediaRecorder: null,
      recordedChunks: [],
      isRecording: false,
      downloadUrl: null
    };
  },
  async mounted() {
    await this.initializeCamera();
  },
  beforeUnmount() {
    this.stopCamera();
  },
  methods: {
    async initializeCamera() {
      try {
        this.mediaStream = await navigator.mediaDevices.getUserMedia({
          video: {
            width: { ideal: 1280 },
            height: { ideal: 720 }
          },
          audio: true
        });
        this.$refs.videoPreview.srcObject = this.mediaStream;
      } catch (error) {
        console.error('无法访问摄像头/麦克风:', error);
        alert('无法访问摄像头/麦克风，请检查权限设置');
      }
    },

    startRecording() {
      if (!this.mediaStream) return;

      this.recordedChunks = [];
      this.mediaRecorder = new MediaRecorder(this.mediaStream, {
        mimeType: 'video/webm; codecs=vp9'
      });

      this.mediaRecorder.ondataavailable = event => {
        if (event.data.size > 0) {
          this.recordedChunks.push(event.data);
        }
      };

      this.mediaRecorder.onstop = () => {
        const blob = new Blob(this.recordedChunks, {
          type: 'video/webm'
        });
        this.downloadUrl = URL.createObjectURL(blob);
        this.isRecording = false;
      };

      this.mediaRecorder.start();
      this.isRecording = true;
    },

    stopRecording() {
      if (this.mediaRecorder && this.isRecording) {
        this.mediaRecorder.stop();
        this.stopCamera();
      }
    },

    stopCamera() {
      if (this.mediaStream) {
        this.mediaStream.getTracks().forEach(track => track.stop());
        this.mediaStream = null;
        this.$refs.videoPreview.srcObject = null;
      }
    }
  }
};
</script>

<style scoped>
.camera-recorder {
  max-width: 800px;
  margin: 20px auto;
}

video {
  width: 100%;
  max-height: 600px;
  background: #000;
  border-radius: 4px;
}

.controls {
  margin: 20px 0;
  display: flex;
  gap: 10px;
}

button {
  padding: 10px 20px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.download-section {
  margin-top: 20px;
}

.download-section a {
  color: #28a745;
  text-decoration: none;
  font-weight: bold;
}
</style>
```

end