<script>
import Vue from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
import AlbumList from "./AlbumList";
import PlayListFollowerBtn from "./PlayListFollowerBtn";
import PlayListItem from "./PlayListItem";
Vue.use(VueAxios, axios);
export default {
  name: "PlayList",
  data() {
    return {
      currentPopoverIndex: -1,
      currentIndex: 0,
      musicType: "KPOP",
      currentSong: {
        song: "Chica",
        videoId: "ccX6lLy2Ydw",
        duration: "00:00"
      },
      currentSinger: "Chung Ha",
      videoId: "ccX6lLy2Ydw",
      albumIndex: 0,
      isPlay: false,
      timeLine: 0,
      duration: 0,
      currentTime: 0,
      isDrag: false,
      isGrab: false,
      isAutoPlay: false,
      isLoop: false,
      isShuffle: false,
      isBuffering: false,
      progress: 0,
      newTime: 0,
      processInterval: null,
      playerState: -1,
      volume: 30,
      isUnMute: false,
      playList: [],
      currentPlayList: [],
      currentAlbumImg: "",
      currentBackGround: "",
      playListStatus: "OVERVIEW",
      navbar: "Playlist",
      copyright: "",
      publicPath: "./"
    };
  },
  components: {
    PlayListFollowerBtn,
    AlbumList,
    PlayListItem
  },
  created() {
    let vm = this;
    this.axios.get("./playlist.json").then(response => {
      vm.playList = response.data;
      this.init();
      this.player.setVolume(30);
    });
  },
  mounted() {
    let progressBar = this.$refs.progressBar;
    progressBar.addEventListener("mouseover", () => {
      this.isDrag = true;
    });
    progressBar.addEventListener("mouseout", () => {
      this.isDrag = false;
      this.isGrab = false;
    });
    progressBar.addEventListener("mousedown", () => {
      this.isGrab = true;
      this.isDrag = true;
    });
    progressBar.addEventListener("mouseup", () => {
      this.isGrab = false;
    });
    progressBar.addEventListener("mousemove", e => {
      if (this.isDrag && this.isGrab) this.processBarTime(e);
    });
  },

  watch: {
    playerState() {
      if (this.playerState !== 3) {
        setTimeout(() => {
          this.isBuffering = false;
        }, 500);
      } else {
        this.isBuffering = true;
      }
    },
    navbar() {
      if (this.navbar === "Mix") {
        this.currentBackGround =
          this.publicPath + this.playList["Mix"][0].backgroundImg;
        this.currentPlayList = this.playList["Mix"][0].songs;
      }
    }
  },
  computed: {
    audio() {
      return this.$refs.audio;
    },
    player() {
      return this.$refs.youtube.player;
    },
    sameTypeList() {
      return this.playList[this.musicType];
    },
    typeList() {
      return Object.keys(this.playList);
    },
    favoriteList() {
      return this.playList["Mix"][0].songs || [];
    }
  },
  filters: {
    formatTime(time) {
      // 調整時間 & 歌曲總長度
      let mins = Math.floor(time / 60);
      let secs = Math.floor(time % 60);
      time = `${mins < 10 ? 0 : ""}${mins}:${secs < 10 ? 0 : ""}${secs}`;
      return time;
    }
  },

  methods: {
    checkPopovers($event) {
      if ($event.target.nodeName !== "I") {
        this.currentPopoverIndex = -1;
      }
    },
    showPopovers(index) {
      this.currentPopoverIndex = index;
      setTimeout(() => {
        this.currentPopoverIndex = -1;
      }, 4000);
    },
    init() {
      this.$once("hook:beforeDestroy", () => {
        clearInterval(this.processInterval);
      });
      let item = this.playList[this.musicType][this.albumIndex];
      this.reset(item);
    },
    reset(item) {
      this.currentSong = item.songs[this.currentIndex].song;
      this.videoId = item.songs[this.currentIndex].videoId;
      this.currentPlayList = item.songs || [];
      this.currentAlbumImg = item.img;
      this.currentBackGround = this.publicPath + item.backgroundImg;
      this.currentSinger = item.singer;
      this.musicType = item.type;
      this.copyright = item.copyright;
    },
    autoPlay() {
      // 自動撥放
      if (!this.isPlay) {
        this.isPlay = true;
        this.player.playVideo();
      } else {
        this.player.playVideo();
      }
    },
    togglePlay() {
      // 切換撥放或停止
      this.isPlay = !this.isPlay;
      this.player.playVideo();
      if (!this.isPlay) {
        this.player.pauseVideo();
        this.isPlay = false;
      }
    },
    changeSong(click) {
      // 上一首或下一首
      let total = this.currentPlayList.length;
      this.currentIndex = (this.currentIndex + click + total) % total;
      this.init();
      this.player.nextVideo();

      this.$nextTick(function() {
        this.autoPlay();
      });
    },
    changeVolume() {
      // 改變聲音&聲音 Input 顏色
      let volume = parseInt(this.$refs.volume.value);
      if (this.isUnMute && this.volume > 0) {
        this.isUnMute = false;
        this.player.unMute();
      }
      this.player.setVolume(volume);
      this.volume = volume;
    },
    changeUnMute() {
      // 靜音&取消靜音
      this.isUnMute = !this.isUnMute;
      if (this.isUnMute) {
        this.volume = 0;
        this.player.mute();
      } else if (!this.isUnMute) {
        this.volume = 30;
        this.player.unMute();
      }
    },
    processBarTime(e) {
      // 調整進度條
      let progressWidth = this.$refs.progressBar.offsetWidth;
      let x = e.offsetX;
      this.newTime = Math.floor((x / progressWidth) * this.duration);
      this.player.seekTo(this.newTime);
    },
    async updateDuration() {
      // 計算歌曲總時間
      this.duration = await this.player.getDuration();
      this.processInterval = setInterval(() => {
        this.player.getCurrentTime().then(currentTime => {
          // 調整進度軸長度 time:currentTime/totaltime:duration
          this.timeLine = (currentTime / this.duration) * 100;
          this.currentTime = currentTime;
          this.getPlayerState();
        });
      }, 1000);
    },
    paused() {
      this.$once("hook:beforeDestroy", () => {
        clearInterval(this.processInterval);
      });
    },
    async getPlayerState() {
      // 監控影片狀態=>增加 loading icon
      // getPlayerState():Number
      // -1 – unstarted、0 – ended、1 – playing、2 – paused、3 – buffering、5 – video cued
      this.playerState = await this.player.getPlayerState();
    },
    loopCurrentSong() {
      let id = this.videoId;
      this.player.loadVideoById(id);
      this.$nextTick(function() {
        this.autoPlay();
      });
    },
    loopSong() {
      if (this.isLoop) {
        this.loopCurrentSong();
      } else {
        let total = this.currentPlayList.length;
        // 隨機撥放
        let suffleIndex = Math.floor(Math.random() * total);
        if (this.isShuffle && this.currentIndex !== suffleIndex) {
          this.currentIndex = suffleIndex;
        } else {
          this.currentIndex++;
        }
        if (this.currentIndex === total) this.currentIndex = 0;
        this.init();
        this.$nextTick(function() {
          this.autoPlay();
        });
        this.$once("hook:beforeDestroy", () => {
          clearInterval(this.processInterval);
        });
      }
    },
    followHandler(item) {
      item.isFollow = !item.isFollow;
      let follower = parseInt(item.followers.replace(/,/g, ""));
      if (item.isFollow) {
        item.followers = (follower + 1).toLocaleString("en-us");
      } else if (!item.isFollow) {
        item.followers = (follower - 1).toLocaleString("en-us");
      }
    },
    selectSinger({ item, index }) {
      this.currentIndex = 0;
      this.albumIndex = index;
      this.reset(item);
      if (this.musicType === "Mix") {
        this.currentSinger = this.currentPlayList[this.currentIndex].singer;
      }
      this.$nextTick(function() {
        this.autoPlay();
      });
    },
    selectSong({ item, index }) {
      // 點選歌曲
      this.currentSong = item.song;
      this.videoId = item.videoId;
      this.currentIndex = index;
      if (this.musicType === "Mix") {
        this.currentSinger = this.currentPlayList[this.currentIndex].singer;
      }
      this.$nextTick(function() {
        this.autoPlay();
      });
    }
  }
};
</script>
<template src="./template.html"></template>
<style lang="scss"></style>
