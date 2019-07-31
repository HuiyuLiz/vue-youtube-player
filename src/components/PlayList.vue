<template>
  <div>
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
export default {};
</script>

<style>
</style>
