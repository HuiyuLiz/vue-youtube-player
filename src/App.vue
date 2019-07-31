<template>
  <div id="app">
    <div class="btn btn-primary">111</div>
    <div class="progress">
      <div class="progress__pointer" :style="{'left':timeLine - 1 +'%'}"></div>
      <div class="progress__bar" ref="progressBar" @click="setCurrentTime">
        <div class="progress__bar--fill" :style="{'width':timeLine+'%'}"></div>
      </div>
    </div>

    <audio
      ref="audio"
      :src="currentSong.src+musicType"
      :loop="isLoop"
      :autoplay="isAutoPlay"
      controls
      @timeupdate="currentDuration"
      @ended="loopSong"
      @canplaythrough="durationTimeDisplay"
    ></audio>
    <div class="player">
      <h2>歌手:{{currentSong.singer}}</h2>
      <h2>歌名:{{currentSong.song}}</h2>
      <h3>時間:{{currentTime}} 總長度:{{duration}}</h3>
    </div>
    <div>
      <input
        type="range"
        name="volume"
        min="0"
        max="1"
        step="0.05"
        @change="changeVolume"
        ref="volume"
        value="0.3"
      />Volume
    </div>
    <ul class="list" v-for="(item,index) in playList" :key="item.singer">
      <li
        class="list__item"
        @dblclick="selectSong(item,index)"
        :class="{'list__item--active':index===currentIndex}"
      >歌手{{item.singer}}----歌曲{{item.song}}</li>
    </ul>
    <div class="container">
      <div class="btn" @click="isShuffle = !isShuffle">
        <i class="material-icons">shuffle</i>
        <p>{{isShuffle}}</p>
      </div>
      <div class="btn" @click="changeSong(-1)">
        <i class="material-icons">keyboard_arrow_left</i>
      </div>
      <div class="btn" @click="togglePlay">
        <div v-if="!isPlay">
          <i class="material-icons">play_arrow</i>
        </div>
        <div v-else>
          <i class="material-icons">pause</i>
        </div>
      </div>
      <div class="btn" @click="changeSong(1)">
        <i class="material-icons">keyboard_arrow_right</i>
      </div>
      <div class="btn" @click="isLoop=!isLoop">
        <div v-if="!isLoop">
          <i class="material-icons">repeat</i>
        </div>
        <div v-else>
          <i class="material-icons">repeat_one</i>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";

export default {
  name: "app",
  data() {
    return {
      currentIndex: 0,
      isPlay: false,
      musicType: ".mp3",
      currentSong: null,
      timeLine: 0,
      duration: "00:00",
      currentTime: "00:00",
      isDrag: false,
      isGrab: false,
      isAutoPlay: false,
      isLoop: false,
      isShuffle: false,
      playList: [
        {
          singer: "Ed Sheeran ",
          song: "Antisocial",
          src: "/audio/Ed Sheeran Travis Scott - Antisocial"
        },
        {
          singer: "Sam Smith",
          song: "How Do You Sleep?",
          src: "/audio/Sam Smith-How Do You Sleep"
        },
        {
          singer: "BTS",
          song: "Boy With Luv",
          src: "/audio/BTS-Boy With Luv"
        },
        {
          singer: " CHUNG HA ",
          song: "Chica",
          src: "/audio/CHUNG HA  - Chica"
        }
      ]
    };
  },
  components: {
    HelloWorld
  },
  computed: {
    audio() {
      return this.$refs.audio;
    }
  },
  watch: {},
  mounted() {
    let progressBar = this.$refs.progressBar;
    progressBar.addEventListener("mouseover", () => {
      this.isDrag = true;
    });
    progressBar.addEventListener("mouseout", () => {
      this.isDrag = false;
      this.isGrab = false;
    });
    progressBar.addEventListener("mousedown", e => {
      this.isGrab = true;
      this.isDrag = true;
    });
    progressBar.addEventListener("mouseup", () => {
      this.isGrab = false;
    });
    progressBar.addEventListener("mousemove", e => {
      if (this.isDrag && this.isGrab) this.setCurrentTime(e);
    });
  },
  created() {
    this.currentSong = this.playList[this.currentIndex];
  },
  watch: {},
  methods: {
    autoPlay() {
      if (!this.isPlay) {
        this.isPlay = true;
        this.audio.play();
      }
      this.isAutoPlay = true;
    },
    togglePlay() {
      this.isPlay = !this.isPlay;
      this.$refs.audio.play();
      if (!this.isPlay) {
        this.audio.pause();
        this.isAutoPlay = false;
      }
    },
    selectSong(item, index) {
      this.currentSong = item;
      this.currentIndex = index;
      this.autoPlay();
    },
    changeSong(click) {
      let total = this.playList.length;
      this.currentIndex = (this.currentIndex + click + total) % total;
      this.currentSong = this.playList[this.currentIndex];
      this.autoPlay();
    },
    changeVolume() {
      this.audio.volume = this.$refs.volume.value;
    },
    currentTimeDisplay() {
      // 調整時間 & 歌曲總長度
      let currentTime = this.audio.currentTime;
      let mins = Math.floor(currentTime / 60);
      let secs = Math.floor(currentTime % 60);
      this.currentTime = `${mins < 10 ? 0 : ""}${mins}:${
        secs < 10 ? 0 : ""
      }${secs}`;
    },
    currentDuration() {
      // 調整進度軸長度
      let timeLine = (this.audio.currentTime / this.audio.duration) * 100;
      this.timeLine = timeLine;
      this.currentTimeDisplay();
    },
    setCurrentTime(e) {
      let progressWidth = this.$refs.progressBar.offsetWidth;
      let x = e.offsetX;
      this.audio.currentTime = (x / progressWidth) * this.audio.duration;
    },
    durationTimeDisplay() {
      let duration = this.audio.duration;
      let totalMins = Math.floor(duration / 60);
      let totalSecs = Math.floor(duration % 60);
      this.duration = `${totalMins < 10 ? 0 : ""}${totalMins}:${
        totalSecs < 10 ? 0 : ""
      }${totalSecs}`;
    },
    loopSong() {
      let suffleIndex = Math.floor(Math.random() * this.playList.length);
      if (this.isShuffle && this.currentIndex !== suffleIndex) {
        this.currentIndex = suffleIndex;
      } else {
        this.currentIndex++;
      }
      if (this.currentIndex === this.playList.length) this.currentIndex = 0;
      this.currentSong = this.playList[this.currentIndex];
      this.autoPlay();
    }
  }
};
</script>

<style lang="scss">
@import "assets/style";
</style>
