说明:
 用vue，将name.mp3这段录音文件，添加背景音乐，bg.mp3，然后生成新的文件 
 提前准备好两个mp3文件，一个录音文件，一个背景音乐，放在public目录里
step1:下载依赖

```bash
{
  "name": "untitled3",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "audiobuffer-to-wav": "^1.0.0",
    "axios": "^1.8.4",
    "vue": "^3.5.13"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.2.1",
    "vite": "^6.2.0"
  }
}

```

step2:

```typescript
<template>
  <div>
    <button @click="mergeAudio">合并音频并下载</button>
  </div>
</template>

<script>
import audioBufferToWav from 'audiobuffer-to-wav';

export default {
  methods: {
    async mergeAudio() {
      try {
        const [mainBuffer, bgBuffer] = await Promise.all([
          this.loadAudio('/name.mp3'),  // 修改为public路径
          this.loadAudio('/bg.mp3')     // 修改为public路径
        ]);

        // 创建离线音频上下文
        const offlineContext = new OfflineAudioContext(
          mainBuffer.numberOfChannels,
          mainBuffer.length,
          mainBuffer.sampleRate
        );

        // 创建主音频源
        const mainSource = offlineContext.createBufferSource();
        mainSource.buffer = mainBuffer;

        // 创建背景音乐源
        const bgSource = offlineContext.createBufferSource();
        bgSource.buffer = bgBuffer;
        bgSource.loop = true;

        // 创建音量控制节点
        const bgGainNode = offlineContext.createGain();
        bgGainNode.gain.value = 0.3; // 背景音乐音量设置为30%

        // 连接节点（移除了背景音乐的直接连接）
        mainSource.connect(offlineContext.destination);
        bgSource.connect(bgGainNode);
        bgGainNode.connect(offlineContext.destination);

        // 启动播放
        mainSource.start(0);
        bgSource.start(0);

        // 渲染音频
        const mixedBuffer = await offlineContext.startRendering();

        // 转换并下载
        this.saveAudio(mixedBuffer);
      } catch (error) {
        console.error('合并失败:', error);
      }
    },

    async loadAudio(url) {
      const response = await fetch(url);
      const arrayBuffer = await response.arrayBuffer();
      return new AudioContext().decodeAudioData(arrayBuffer);
    },

    saveAudio(buffer) {
      // 转换为WAV格式
      const wav = audioBufferToWav(buffer);
      const blob = new Blob([wav], { type: 'audio/wav' });

      // 创建下载链接
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.style.display = 'none';
      a.href = url;
      a.download = `mixed-${Date.now()}.wav`;
      document.body.appendChild(a);
      a.click();
      URL.revokeObjectURL(url);
      document.body.removeChild(a);
    }
  }
};
</script>
```

end