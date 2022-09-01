<template>
  <div>
      <div class="voice-popup">
        <div class="voice-title">
          <!-- <div class="voice-txt" v-show="from == 'edit' || voiceRecordState === StateEnum.recording || voiceRecordState === StateEnum.fail">普通话</div>
          <div class="voice-txt voice-modify" v-show="from != 'edit' && (voiceRecordState === StateEnum.continue || voiceRecordState === StateEnum.finish)">轻触文字可修改信息</div> -->

        </div>
        <div class="voice-content">
          <!-- 正在收音 -->
          <div v-show="voiceRecordState === StateEnum.recording" class="voice-info" @click="handleStart">我在听，请说话</div>
          <!-- 未识别成功 -->
          <div v-show="voiceRecordState === StateEnum.fail" class="voice-info">没听清，请再说一遍</div>
          <!-- 语音转文字内容 -->
          <div v-show="voiceRecordState === StateEnum.continue || voiceRecordState === StateEnum.finish" class="voice-result" @click="handleEdit">{{ userVoiceComment }}</div>
          <!-- 波形图 -->
        </div>
  
        <div class="voice-record-finish" v-if="voiceRecordState === StateEnum.finish">
          <button class="voice-record-start" @click="handleContinue">开始说话</button>
          <button class="voice-record-send" @click="voiceCloseSend">发送</button>
        </div>
        <div v-else>
          <button v-show="voiceRecordState === StateEnum.recording || voiceRecordState === StateEnum.continue" class="voice-record" >

            <span @click="handleStop">正在收音 点击停止</span>
          </button>
          <button v-show="voiceRecordState === StateEnum.fail" class="voice-record" @click="handleContinue">

            <span v-show="voiceRecordState === StateEnum.fail">点击开始说话</span>
          </button>
        </div>
      </div>
      <audio :src="audioSrc" controls></audio>
  </div>
  </template>
  
  <script>
  import Recorder from 'js-audio-recorder'

  
  const StateEnum = {
    fail: 0,
    recording: 1,
    continue: 2,
    finish: 3
  }
  export default {
    name: "VoicePopup",
    data() {
      return {
        voiceShow: false,
        voiceRecordState: StateEnum.recording,
        StateEnum,
        audioSrc: '',
        recorder: null, // 用于存储创建的语音对象
        userVoiceComment: 'aaaa'
      };
    },
    mounted() {
      this.recorderInit()
    },
    beforeDestroy() {
      this.handleDestroy()
    },
    methods: {
      recorderInit() {
        //实例化语音对象
        this.recorder = new Recorder({
          sampleBits: 16, // 采样位数，支持 8 或 16，默认是16
          sampleRate: 16000, // 采样率，支持 11025、16000、22050、24000、44100、48000，根据浏览器默认值，我的chrome是48000
          numChannels: 1 // 声道，支持 1 或 2， 默认是1
        })
        console.log("实例化语音对象: recorder", this.recorder);
        // this.handleStart()
      },
      // 获取麦克风权限
      getPermission(){
        Recorder.getPermission().then(() => {
          console.log('给权限了');
            this.handleStart()
        }, error => {
            console.log('getPermission >> error', error);
        });
      },
      handleContinue() {
        this.handleStart()
        // 修改状态为 continue
        this.voiceRecordState = StateEnum.continue
      },
      handleStart() {
        console.log('开始录音')
        this.voiceRecordState = StateEnum.recording
        this.recorder.start().then(() => {
            // 开始录音
            this.handleProgress()
        }, (error) => {
            // 出错了
            console.log(error);
        });
      },
      handleProgress() {
        this.recorder.onprogress = (params)=> {
            // // console.log('录音时长(秒)', params.duration);
            console.log('录音大小(字节)', params.fileSize);
            // // console.log('录音音量百分比(%)', params.vol);
            // // 若超过10秒种，未检测到用户声音，自动进入【结束录音&未识别成功】状态
            // if(params.duration >= 10) {
            //   // 检测用户声音
            //   this.sendAudio('check')
            // }
            // // 录制最大时长限制为60秒，超过自动结束录制
            // if(params.duration > 60) {
            //   this.handleStop()
            // }
        }
      },
      handleStop () {
        console.log('停止录音')
        this.recorder.stop() // 停止录音
        // this.handleDestroy()
  
        this.sendAudio()
      },
      handleDestroy () {
        console.log('销毁实例')
        if (this.recorder) {
          // 销毁录音实例，置为null释放资源
          this.recorder.destroy()
          this.recorder = null;
        }
      },
      // 获取录音文件
      getRecorder(){
        let duration = this.recorder.duration;//录音总时长
        let fileSize = this.recorder.fileSize;//录音总大小
        console.log("duration==", duration);
        console.log("fileSize==", fileSize);
  
        //录音结束，获取取录音数据
        let PCMBlob = this.recorder.getPCMBlob();//获取 PCM 数据
        let wav = this.recorder.getWAVBlob();//获取 WAV 数据
  
        console.log("PCMBlob==", PCMBlob);
        console.log("wav==", wav);
  
      },
      /**
       *  下载录音文件
       * */
      //下载pcm
      downPCM(){
        //这里传参进去的是文件名
        this.recorder.downloadPCM('testpcm');
      },
      //下载wav
      downWAV(){
        //这里传参进去的是文件名
        this.recorder.downloadWAV('testwav');
      },
      sendAudio(check = '') {
        let blob = this.recorder.getWAVBlob()//获取wav格式音频数据
        console.log("blob：", blob);
        let newbolb = new Blob([blob], { type: 'audio/wav' })
        let fileOfBlob = new File([newbolb], new Date().getTime() + '.pcm')
        const url = window.URL.createObjectURL(blob)
        this.audioSrc = url


        console.log("fileOfBlob", url);
      },
      voiceClose() {
        this.voiceShow = false
        this.recorder.stop() // 停止录音
        this.$emit('voiceClose')
      },
      voiceCloseSend() {
        this.voiceClose()
        this.$emit('voiceCloseSend', this.userVoiceComment)
      },
      handleEdit() {
      }
    }
  };
  </script>
  
  <style scoped>
  </style>
  