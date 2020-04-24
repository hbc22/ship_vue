<template>
  <div
    class="custom-video_container"
    ref="custom-video_container"
    @mouseover="handleControls($event, 'start')"
    @mouseleave="handleControls($event, 'end')"
  >
    <video
      class="custom-video_video"
      ref="custom-video"
      :poster="videoOption.poster"
    >
      <source :src="videoOption.src" type="video/mp4">
      <p>设备不支持</p>
    </video>
    <span
      v-if="videoState.play"
      class="custom-video_play custom-video_play-pause iconfont icon-zanting"
      @click="pause('btn')"
    >
    </span>
    <span
      v-else
      class="custom-video_play custom-video_play-play iconfont icon-bofang"
      @click="play('btn')"
    >
    </span>
    <!-- 控制区域背景 -->
    <!-- Vue 提供了 transition 的封装组件，在下列情形中，可以给任何元素和组件添加进入/离开过渡 -->
    <transition
      name="fade"
    >
      <div
        class="custom-video_control"
        v-show="!videoState.hideControl || !videoState.play"
      >
        <!-- 进度条 -->
        <!-- 鼠标按钮被按下时mousedown事件就会被发送 -->
        <!-- 当鼠标指针在元素内移动时，mousemove事件就会被触发 -->
        <!-- 当在元素上放松鼠标按钮时，会发生 mouseup 事件。 -->
        <div
          class="custom-video_control-bg"
          @mousedown="handlePrograssDown"
          @mousemove="handlePrograssMove"
          @mouseup="handlePrograssUp"
        >
          <div 
            class="custom-video_control-bg-outside"
            ref="custom-video_control-bg-outside"
          >
            <span 
              class="custom-video_control-bg-inside"
              ref="custom-video_control-bg-inside"
            ></span>
            <span 
              class="custom-video_control-bg-inside-point"
              ref="custom-video_control-bg-inside-point"
            ></span>
          </div>
        </div>
        <!-- 声音 -->
        <div
          class="custom-video_control-voice"
        >
          <span
            class="custom-video_control-voice-play iconfont icon-shengyin"
          ></span>
          <div
            class="custom-video_control-voice-bg"
            ref="custom-video_control-voice-bg"
            @mousedown="handleVolPrograssDown"
            @mousemove="handleVolPrograssMove"
            @mouseup="handleVolPrograssUp"
          >
            <div 
              class="custom-video_control-voice-bg-outside"
              ref="custom-video_control-voice-bg-outside"
            >
              <span 
                class="custom-video_control-voice-bg-inside"
                ref="custom-video_control-voice-bg-inside"
              ></span>
              <span 
                class="custom-video_control-voice-bg-point"
                ref="custom-video_control-voice-bg-point"
              ></span>
            </div>
          </div>
        </div>
        <!-- 时间 -->
        <div
          class="custom-video_control-time"
        >
          <span>{{currentTime ? currentTime : "00:00"}}</span>
           / 
          <span>{{duration ? duration : "00:00"}}</span>
        </div>
        <!-- 全屏缩放 -->
        <span
          class="custom-video_control-full iconfont icon-quanping"
          @click="handleScreen"
        ></span>
      </div>
    </transition>
  </div>
</template>
<script>
export default {
  data() {
    return {
      videoOption: {
        src: require("../../static/media/taru.mp4"), //视频
        poster: require("../../static/images/poster.jpg"), // 初始化占位图片
        volume: 20
      },
      videoState: {
        play: false, //播放状态
        hideControl: false, // 控制栏状态
        distance: 0, // 移动的距离
        downState: false, // 鼠标点击进度条
        playState: false,
        leftInit: 0, // 当前进度初始偏移量
        screenState: false
      },
      voiceState: { // 同上
        distance: 0,
        downState: false,
        topInit: 0
      },
      videoDom: null, // video
      videoProOut: null, // 视频总进度条
      videoPro: null, // 视频进度条
      videoPoi: null, // 视频进度点
      duration: 0, // 视频总时长
      currentTime: 0, // 视频当前播放时长
      processWidth: 0, // 视频进度条总长度
      voiceProOut: null, // 音频总进度条
      voicePro: null, // 音频进度条
      voicePoi: null, // 音频进度点
      volProcessHeight: 0,
    }
  },
  mounted() {
    // 初始化相关元数据
    this.videoDom = this.$refs["custom-video"]//视频的demo,整个demo元素都显示出来了。
    this.videoProOut = this.$refs['custom-video_control-bg-outside']//进度条的这条线
    this.videoPro = this.$refs['custom-video_control-bg-inside']//进度条的这条线上面的刚刚开始的这个点
    this.videoPoi = this.$refs['custom-video_control-bg-inside-point']//进度条的这条线上面的刚刚开始的这个圆点和上面的是并列的span

    this.voiceProOut = this.$refs['custom-video_control-voice-bg-outside']//声音进度条的这条线
    this.voicePro = this.$refs['custom-video_control-voice-bg-inside']//声音进度条的这条线上面的刚刚开始的这个点
    this.voicePoi = this.$refs['custom-video_control-voice-bg-point']//声音进度条的这条线上面的刚刚开始的这个圆点和上面的是并列的span

    //内联元素以及没有 CSS 样式的元素的 clientWidth 属性值为 0。Element.clientWidth 属性表示元素的内部宽度，以像素计。该属性包括内边距 padding，但不包括边框 border、外边距 margin 和垂直滚动条（如果有的话）
    this.processWidth = this.videoProOut.clientWidth
    console.log('111', this.processWidth)
    this.videoState.leftInit = this.getOffset(this.videoProOut).left
    console.log('222', this.videoState.leftInit)//初始偏移量
    this.videoDom.volume = this.videoOption.volume / 100 // 设置初始化声音volumedefault:20
    this.initMedaData()
  },
  methods: {
    initMedaData() { // 初始化video相关事件
      //当指定的音频/视频的元数据已加载时，会发生 loadedmetadata 事件。音频/视频的元数据包括：时长、尺寸（仅视频）以及文本轨道。
      this.videoDom.addEventListener('loadedmetadata', () => { // 获取视频总时长
        console.log('long', this.videoDom.duration)
        this.duration = this.timeTranslate(this.videoDom.duration)
      })
      this.videoDom.addEventListener("click", () => { // 点击视频区域可以进行播放或者暂停
      //设置或返回视频是否暂停。如果视频已暂停返回 true，否则为 false；paused, ended返回视频的播放是否已结束。
        if(this.videoDom.paused || this.videoDom.ended) {
            if(this.videoDom.ended) {
              //currentTime 属性设置或返回视频播放的当前位置（以秒计）。
              this.videoDom.currentTime = 0
            }
            this.play('btn')
        } else {
          this.pause('btn')
        }
      })
      //当currentTime更新时会触发timeupdate事件。
      this.videoDom.addEventListener("timeupdate", () => { // 监听视频播放过程中的时间
        const percentage = 100 * this.videoDom.currentTime / this.videoDom.duration
        this.videoPro.style.width = percentage + '%'  //播放进度从点到currentTime的百分比宽度
        this.videoPoi.style.left = percentage - 1 + '%' //这个表示的是小圆点距离开始位置的left的距离
        this.currentTime = this.timeTranslate(this.videoDom.currentTime)
      })
      this.videoDom.addEventListener("ended", () => { // 监听结束播放事件
        this.videoPro.style.width = 0 //播放条的宽度
        this.videoPoi.style.left = 0  //播放条上面的小圆点距离left的距离
        this.currentTime = 0          //播放时间
        this.videoState.play = false  //播放状态
        this.videoState.hideControl = false
      })
      this.videoDom.addEventListener("volumechange", () => {
        const percentage =  this.videoDom.volume * 100
        this.voicePro.style.height = percentage + '%' //音量条的高度，相对应播放进度条的宽度
        this.voicePoi.style.bottom = percentage + '%' //音量开始的小圆点距离原来的距离
      })
    },
    play(flag) { // 播放按钮事件
      console.log('ppp', flag)
      // playState play不知道为什么要定义两个状态而且都是false一开始的时候
      if(flag) this.videoState.playState = true
      this.videoState.play = true //播放状态
      this.videoDom.play() //开始播放视频。
    },
    pause(flag) { // 暂停按钮事件
      console.log('paapp', flag)
      if(flag) this.videoState.playState = false
      this.videoDom.pause() //暂停当前播放的视频。
      this.videoState.play = false
    },
    handlePrograssDown(ev) { // 监听点击进度条事件，方便获取初始点击的位置
    console.log('Down', ev)
      // 视频暂停
      this.videoState.downState = true //按下鼠标标志
      this.pause()
      // clientX当前（DOM 元素）坐标系下鼠标指针的 X 轴坐标值
      this.videoState.distance = ev.clientX - this.videoState.leftInit
    },
    handlePrograssMove(ev) { // 监听移动进度条事件，同步播放相关事件
    // console.log('move', ev)
      if(!this.videoState.downState) return
      let disX = ev.clientX - this.videoState.leftInit
      //processWidth进度条这条线的宽度
      // clientX应该是从同一个y轴的。x为0的时候开始计算的
      if(disX > this.processWidth) {
        disX = this.processWidth
      }
      if(disX < 0) {
        disX = 0
      }
      this.videoState.distance = disX
      // 移动的距离 / 进度条的宽度 * 视频总时长 = 这个时候小圆点在进度条上面的位置
      this.videoDom.currentTime = this.videoState.distance / this.processWidth * this.videoDom.duration
    },
    handlePrograssUp() { //松开鼠标，播放当前进度条视频
      this.videoState.downState = false
      // 视频播放
      this.videoDom.currentTime = this.videoState.distance / this.processWidth * this.videoDom.duration
      this.currentTime = this.timeTranslate(this.videoDom.currentTime)
      if(this.videoState.playState) {
        this.play()
      }
    },
    handleVolPrograssDown(ev) { // 监听声音点击事件
      this.voiceState.topInit = this.getOffset(this.voiceProOut).top//this.voiceProOut声音进度条的这条线Demo
      this.volProcessHeight = this.voiceProOut.clientHeight
      this.voiceState.downState = true //按下鼠标标志
      this.voiceState.distance = ev.clientY - this.voiceState.topInit
    },
    handleVolPrograssMove(ev) { // 监听声音进度条移动事件
      if(!this.voiceState.downState) return
      let disY = this.voiceState.topInit + this.volProcessHeight - ev.clientY
      if(disY > this.volProcessHeight - 2) {
        disY = this.volProcessHeight - 2
      }
      if(disY < 0) {
        disY = 0
      }
      this.voiceState.distance = disY
      this.videoDom.volume = this.voiceState.distance / this.volProcessHeight
      this.videoOption.volume = Math.round(this.videoDom.volume * 100)
    },
    handleVolPrograssUp() { // 监听声音鼠标离开事件
      this.voiceState.downState = false //按下鼠标标志
      this.videoDom.volume = this.voiceState.distance / this.volProcessHeight
      this.videoOption.volume = Math.round(this.videoDom.volume * 100)
    },
    handleControls(ev, flag) { // 监听离开或者进入视频区域隐藏或者展示控制栏
    // console.log('sssss', ev)
    // console.log('eneeen', flag)
      switch (flag) {
        case 'start':
          this.videoState.hideControl = false
          break;
        case 'end':
          this.videoState.hideControl = true
          break;
        default:
          break;
      }
    },
    handleScreen() { // 全屏操作
      this.videoState.screenState = !this.videoState.screenState
      if(this.videoState.screenState) {
        this.fullScreen()
      } else {
        this.exitFullscreen()
      }
    },
    timeTranslate(t) { // 时间转化
      let m = Math.floor(t / 60)
      m < 10 && (m = '0' + m)
      return m + ":" + (t % 60 / 100 ).toFixed(2).slice(-2)
    },
    getOffset(node, offset) { // 获取当前屏幕下进度条的左偏移量和又偏移量
    console.log('aa', node)
    console.log('bb', offset)
    // console.log('cc', offset.offsetParent)
      if(!offset) {
        offset = {}
        offset.left = 0
        offset.top = 0
      }
      if(node === document.body || node === null) {
        console.log('body11')
        return offset
      }
      offset.top += node.offsetTop
      offset.left += node.offsetLeft
      return this.getOffset(node.offsetParent, offset)
    },
    fullScreen() {
      let ele = document.documentElement //全部的文档流要看下文档流是否允许全屏，这是一个promise的。
      console.log('909', ele)
      console.log('909', ele .requestFullscreen)
      if (ele .requestFullscreen) {
        ele .requestFullscreen()
      } else if (ele .mozRequestFullScreen) {
        ele .mozRequestFullScreen()
      } else if (ele .webkitRequestFullScreen) {
        ele .webkitRequestFullScreen()
      }
      this.$refs['custom-video_container'].style.width = "100%"
      this.$refs['custom-video_container'].style.height = "100%"
    },
    exitFullscreen() {
      let de = document
      if (de.exitFullscreen) {
        de.exitFullscreen();
      } else if (de.mozCancelFullScreen) {
        de.mozCancelFullScreen();
      } else if (de.webkitCancelFullScreen) {
        de.webkitCancelFullScreen();
      }
      this.$refs['custom-video_container'].style.width = "500px"
      this.$refs['custom-video_container'].style.height = "300px"
    }
  }
}
</script>
<style scoped>
/* 总容器 */
.custom-video_container{
  width: 500px;
  height: 300px;
  margin: 0 auto;
  position: relative;
  overflow: hidden;
}
/* 视频标签 */
.custom-video_video{
  width: 100%;
  height: 100%;
  object-fit: fill;
}
/* 暂停 或者 播放 */
.custom-video_play{
  display: inline-block;
  position: absolute;
  right: 50px;
  bottom: 50px;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  font-size: 30px;
  color: cornflowerblue;
}
/* 暂停隐藏 */
.custom-video_play-pause{
  display: none;
}
/* hover 显示 */
.custom-video_container:hover > .custom-video_play-pause{
  display: inline-block;
}
/* hover 播放按钮动画 */
.custom-video_play:hover{
  box-shadow: 0 0 10px #5A4180;
  transition: all 0.4s;
}
/* 控制栏 */
.custom-video_control{
  position: absolute;
  width: 100%;
  height: 50px;
  left: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, .55);
  display: flex;
  flex-direction: row;
  align-items: center;
}
/* 控制栏进度条 */
.custom-video_control-bg{
  flex: 1;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0 10px;
}
/* 控制栏进度条 —— 总长度 */
.custom-video_control-bg-outside{
  width: 100%;
  height: 5px;
  border-radius: 2.5px;
  background-color: #aaa;
  position: relative;
  cursor: pointer;
}
/* 控制栏进度条 —— 播放长度 */
.custom-video_control-bg-inside{
  position: absolute;
  display: inline-block;
  width: 0;
  height: 100%;
  border-radius: 2.5px;
  left: 0;
  top: 0;
  background-color: #fff;
  transition: all 0.2s;
}
/* 控制栏进度条 —— 播放点 */
.custom-video_control-bg-inside-point{
  display: inline-block;
  width: 10px;
  height: 10px;
  background-color: #fff;
  border-radius: 50%;
  position: absolute;
  top: -2.5px;
  left: -1%;
  transition: all 0.2s;
}
/* 控制栏 —— 声音、时间、全屏缩放 */
.custom-video_control-voice,
.custom-video_control-time,
.custom-video_control-full{
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  color: #fff;
  position: relative;
}
.custom-video_control-voice:hover > .custom-video_control-voice-bg{
  display: block;
}
.custom-video_control-voice-play{
  z-index: 10;
}
.custom-video_control-voice-bg{
  display: none;
  position: absolute;
  width: 30px;
  height: 100px;
  background-color: rgba(0, 0, 0, .55);
  left: 0;
  bottom: 0px;
  border-radius: 15px;
}
.custom-video_control-voice-bg-outside{
  width: 5px;
  height: 70px;
  border-radius: 2.5px;
  background-color: #aaa;
  position: absolute;
  left: 50%;
  transform: translate3d(-50%, 10%, 0);
  cursor: pointer;
}
.custom-video_control-voice-bg-inside{
  display: inline-block;
  position: absolute;
  width: 100%;
  bottom: 0;
  left: 0;
  border-radius: 2.5px;
  background-color: #fff;
  height: 0;
}
.custom-video_control-voice-bg-point{
  display: inline-block;
  width: 10px;
  height: 10px;
  background-color: #fff;
  border-radius: 50%;
  position: absolute;
  left: -2.5px;
  bottom: -1px;
}
.custom-video_control-time{
  font-size: 12px;
}
.custom-video_control-full{
  font-size: 14px;
}
.custom-video_control-voice,
.custom-video_control-full{
  width: 30px;
  height: 30px;
  cursor: pointer;
}
/* 控制栏隐藏动画 */
.fade-enter-active {
  transition: all .3s ease;
}
.fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.fade-enter, .fade-leave-to {
  transform: translateY(50px);
  opacity: 0;
}
</style>