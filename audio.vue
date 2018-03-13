<template lang="html">
  <div class="mainpart">
    <!-- 音频轨迹 -->
    <div class="audioTrack" id="audioTrack">
      <!-- 进度百分比 -->
      <div class="audioPercentage" id="audioPercentage" :style="{ width: accomplishData + unit}"></div>
      <!-- 圆点 -->
      <div class="audioDot" id="audioDot" :style="{ left: dotData + unit}"></div>
    </div>
    <!-- 时间 -->
    <div class="timer">
      <div class="timerL">播放了：{{scheduleTimer}}</div>
      <div class="timerR">总时长:{{altogetherTimer}}</div>
    </div>
    <!-- 播放/暂停 -->
    <div class="playButt">
      <div class="playButtCon" id="playButtCon">{{playButtConText}}</div>
    </div>
    <!-- 原生音频文件 -->
    <div class="">
      <audio autobuffer controls="controls" id="myAudio">
        <source src="http://qgyz.test.mmsay.com/api-xcx/qgyz/resource?filePath=qgyz-dev:/wx/audio/9b4bcc8712254a2486a29a5cb88a8d55.mp3" type="audio/mpeg">
      </audio>
    </div>
  </div>
</template>

<script>
export default {
  name : "",
  data () {
    return{
      playButtConText: "播放",
      altogetherTimer: "", //音频总时长
      scheduleTimer: "", //当前播放进度时间
      accomplishData: "", // 当前播放进度比
      dotData: "", // 圆点进度的位置
      unit: "%", //单位
    }
  },
  mounted:function(){
    // window.onload=function(){
      let that = this;
      let myAudio = document.querySelector('#myAudio'); // 音频
      let audioTrack = document.querySelector('#audioTrack'); // 音频轨迹
      let audioPercentage = document.querySelector('#audioPercentage'); // 音频进度百分比
      let audioDot = document.querySelector('#audioDot'); // 音频圆点
      let playButtCon = document.querySelector('#playButtCon'); // 播放/暂停按钮
      let timer ;
      // that.altogetherTimer = myAudio.duration
      myAudio.oncanplaythrough = function(){
        that.altogetherTimer = Math.round(myAudio.duration);
        playButtCon.onclick = function(){
          //如果当前音频是暂停状态，那么就播放，否者暂停
          if(myAudio.paused){
            myAudio.play();
            clearInterval(timer); // 清楚计时器
            that.playButtConText = "暂停";
            timer=setInterval(function(){
              myAudioPlay(myAudio.currentTime/myAudio.duration); // 计算播放进度百分比
              that.scheduleTimer = Math.round(myAudio.currentTime);
            },1000)
          }else{
            myAudio.pause();
            clearInterval(timer); // 清楚计时器
            that.playButtConText = "播放";
          }
        }
        // 正在播放时音频轨迹的状态
        function myAudioPlay(PercentageData){
          that.unit = "%";
          that.accomplishData = PercentageData*100
          that.dotData  = PercentageData*100
          if(that.dotData>96){
            that.dotData = 96
          }
          if(myAudio.currentTime == myAudio.duration){
            that.playButtConText = "从新播放";
            // clearInterval(timer);
          }
        }
        // 点击音频轨迹
        audioTrack.addEventListener('touchstart',function(ev){
          let audioTrackW = audioTrack.offsetWidth; // 读取音频轨迹的长度
          that.unit = "px";
          that.accomplishData = ev.targetTouches[0].clientX;
          that.dotData = ev.targetTouches[0].clientX;
          let cilckPercentage = ev.targetTouches[0].clientX/audioTrackW;
          myAudio.currentTime=cilckPercentage*myAudio.duration;
          timer=setInterval(function(){
              myAudioPlay(myAudio.currentTime/myAudio.duration);
              that.scheduleTimer = Math.round(myAudio.currentTime);
          }, 100)
          if(myAudio.paused){
              that.scheduleTimer = Math.round(myAudio.currentTime);
              clearInterval(timer)
          }
          console.log(ev.targetTouches[0].clientX)
        },false)
        // 拖动音频进度
        audioDot.addEventListener('touchstart',function(ev){
          clearInterval(timer); // 清楚计时器
          let audioTrackW = audioTrack.offsetWidth; // 读取音频轨迹的长度
          let oldX = ev.targetTouches[0].clientX - audioDot.offsetLeft; // 当前点击的位置 - 目标相对父级元素左边的距离
          console.log(audioDot.offsetLeft)
          console.log(ev.targetTouches[0].clientX)
          audioTrack.addEventListener('touchmove',funMove,false); //音频轨迹添加 touchmove 事件
          audioTrack.addEventListener('touchend',funEnd,false); //音频轨迹添加 touchend 事件
          function funMove(ev){
              let x = ev.targetTouches[0].clientX - oldX;
              if(x >= audioTrackW){
                  x = audioTrackW;
              }else if(x <= 0){
                  x = 0;
              }
              that.unit = "px";
              that.accomplishData = x;
              that.dotData = x-15;

              let newX = ev.changedTouches[0].clientX;
              let oDiv2W = audioPercentage.offsetWidth;
              let oDuration = oDiv2W/audioTrackW;
              clearInterval(timer);

              myAudio.currentTime=oDuration*myAudio.duration;
              timer=setInterval(function(){
                  myAudioPlay(myAudio.currentTime/myAudio.duration);
                  that.scheduleTimer = Math.round(myAudio.currentTime);
              }, 100)
              if(myAudio.paused){
                  that.scheduleTimer = Math.round(myAudio.currentTime);
                  clearInterval(timer)
              }
          }

          function funEnd(ev){
              audioTrack.removeEventListener('touchend',funMove,false);
              audioTrack.removeEventListener('touchmove',funEnd,false);
          }

        },false)
      }

    // }
  }
}
</script>

<style lang="css" scoped>
.mainpart{
  width: 100%;
  padding-top: 50px;
}
.audioTrack{
  width: 95%;
  height: 8px;
  margin: 0 auto;
  position: relative;
  background-color: #eee;
}
.audioPercentage{
  /* width: 25%; */
  height: 100%;
  position: absolute;
  top: 0;
  left: 0 ;
  background-color: red;
}
.audioDot{
  width: 16px;
  height: 16px;
  position: absolute;
  left: 0px;
  top: -4px;
  border-radius: 100%;
  background-color: rgb(195, 72, 78);
}
.playButt{
  width: 100%;
  display: flex;
  justify-content: center;
  margin-top: 20px;
}
.playButtCon{
  width: 50px;
  height: 50px;
  text-align: center;
  line-height: 50px;
  font-size: 16px;
  color: #fff;
  border-radius: 100%;
  background-color:rgb(200, 203, 134);
}
.timer{
  width: 95%;
  margin: 0 auto;
  margin-top: 20px;
  font-size: 14px;
  display: flex;
  justify-content: space-between;
}
</style>
