<script setup>
import { ref, reactive } from 'vue'

import Recorder from 'js-audio-recorder'

const isRecording = ref(false);
const mediaRecorder = ref(null);
const audioChunks = ref([]);
const audioUrl = ref('');
const transcript = ref('');

const data = reactive({
  //用于存储创建的语音对象
  recorder: null,
  formData: null,
  // 控制录音动画的显示隐藏
  showAnima: false,
  mation: true,
  isHistory: true,
  // 录音时长
  duration: 0,
  submit() { // 发送语音的方法
    data.recorder.pause() // 暂停录音
    data.timer = null
    console.log('上传录音')// 上传录音
    var formData = new FormData()
    var blob = data.recorder.getWAVBlob()//获取wav格式音频数据
    //此处获取到blob对象后需要设置fileName满足当前项目上传需求，其它项目可直接传把blob作为		  file塞入formData
    var newbolb = new Blob([blob], { type: 'audio/wav' })
    var fileOfBlob = new File([newbolb], new Date().getTime() + '.wav')
    //formData是传给后端的对象,
    formData.append('file', fileOfBlob)
    //计算出录音时长
    data.duration = Math.ceil((new Date() - data.duration) / 1000);
    console.log(data.duration);
    //发送给后端的方法
    sendAudio(formData).then(res => {
      console.log(res);
    })
  },
  // 录音按钮的点击事件
  voice() {
    //实例化语音对象
    data.recorder = new Recorder({
      sampleBits: 16, // 采样位数，支持 8 或 16，默认是16
      sampleRate: 16000, // 采样率，支持 11025、16000、22050、24000、44100、48000，根据浏览器默认值，我的chrome是48000
      numChannels: 1 // 声道，支持 1 或 2， 默认是1
    })
    //记录开始录音的时间
    data.duration = new Date();
    Recorder.getPermission().then(() => {
      console.log('开始录音')
      data.recorder.start() // 开始录音
    }, (error) => {
      console.log(`${error.name} : ${error.message}`)
    })
  },
  handleStop() {
    console.log('停止录音')
    data.recorder.stop() // 停止录音
    data.mation = false;
  },
  handlePlay() {
    console.log('播放录音')
    data.recorder.play() // 播放录音
  },
  handleDestroy() {
    console.log('销毁实例')
    if (data.recorder) {
      data.recorder.destroy() // 毁实例
    }
  },
})
// 开始录音
const startRecording = async () => {
  try {
    // 请求麦克风权限并获取音频流
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    // 创建MediaRecorder实例
    mediaRecorder.value = new MediaRecorder(stream);
    mediaRecorder.value.ondataavailable = (event) => {
      // 将音频块数据收集起来
      audioChunks.value.push(event.data);
    };
    mediaRecorder.value.onstop = () => {
      // 录音停止时，处理音频块数据
      const audioBlob = new Blob(audioChunks.value);
      audioUrl.value = URL.createObjectURL(audioBlob);
      // 重置音频块
      // audioChunks.value = [];
    };
    // 开始录音
    mediaRecorder.value.start();
    isRecording.value = true;
  } catch (error) {
    console.error('无法获取麦克风输入:', error);
  }
}
const stopRecording = () => {
  if (mediaRecorder.value) {
    // 停止录音
    mediaRecorder.value.stop();
    isRecording.value = false;
    const blob = new Blob(audioChunks.value, { type: 'audio/webm' });
    const formData = new FormData();
    formData.append('audio', blob, 'recording.webm');
    fetch('/api/transcribe', { method: 'POST', body: formData })
      .then(response => response.json())
      .then(data => transcript.value = data.transcript)
      .catch(error => console.error(error));

    audioChunks.value = [];
    
  }
}
</script>

<template>
  <div class="card">
    <div class="mation" v-if="isRecording">
      <div class="ap">
        <div class="box">
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
        </div>
      </div>
      <div class="ap aps">
        <div class="box">
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
          <div></div>
        </div>
      </div>
    </div>
    <!-- <div class="wrap">
      <button @click="data.voice">开始录音</button>
      <button @click="data.handlePlay">播放录音</button>
      <button @click="data.handleStop">停止录音</button>
    </div> -->
    <div class="wrap">
      <button @click="startRecording">开始录音</button>
      <button @click="stopRecording">停止录音</button>
      <audio controls :src="audioUrl" v-if="audioUrl"></audio>
      <p>{{ transcript }}</p>
    </div>

  </div>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}

.wrap {
  display: flex;
  justify-content: space-between;
}

.ap {
  position: relative;
  height: 50px;
  width: 30px;
  margin: 0 auto;
}

.aps {
  transform-origin: center center;
  transform: rotate(180deg);
}

.box div {
  display: inline-block;
  position: absolute;
  bottom: 0;
  width: 10px;
  height: 30px;
  /* 不喜欢该颜色改成想要的动画颜色 */
  background-color: rgb(78, 176, 255);
  transform-origin: bottom;
  border-radius: 5px 5px 0 0;
}

.aps .box div {
  background-color: rgb(0, 141, 255);
  bottom: 2px;
}

.box div:nth-child(1) {
  left: -60px;
  animation: musicWave 0.5s infinite linear both alternate;
}

.box div:nth-child(2) {
  left: -40px;
  animation: musicWave 0.8s infinite linear both alternate;
}

.box div:nth-child(3) {
  left: -20px;
  animation: musicWave 0.6s infinite linear both alternate;
}

.box div:nth-child(4) {
  left: 0px;
  animation: musicWave 0.7s infinite linear both alternate;
}

.box div:nth-child(5) {
  left: 20px;
  animation: musicWave 0.7s infinite linear both alternate;
}

.box div:nth-child(6) {
  left: 40px;
  animation: musicWave 0.6s infinite linear both alternate;
}

.box div:nth-child(7) {
  left: 60px;
  animation: musicWave 0.8s infinite linear both alternate;
}

.box div:nth-child(8) {
  left: 80px;
  animation: musicWave 0.5s infinite linear both alternate;
}

@keyframes musicWave {
  0% {
    height: 8px;
  }

  100% {
    height: 30px;
  }
}
</style>
