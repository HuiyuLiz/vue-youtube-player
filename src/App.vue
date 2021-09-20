<template>
  <div id="app">
    <div
      class="modal bg-dark-50 bd-example-modal-lg"
      id="exampleModalCenter"
      tabindex="-1"
      role="dialog"
      aria-labelledby="exampleModalCenterTitle"
      aria-hidden="true"
      ref="modal"
      :class="{ 'd-block': modalShow }"
      @click="closeModal"
    >
      <div class="modal-dialog modal-dialog-centered modal-lg" role="document">
        <div class="modal-content">
          <div
            class="modal-body text-left text-white p-5 d-flex align-items-left flex-column justify-content-center"
          >
            <h1 class="display-4 font-weight-bold text-success ">
              只要 NT$19.00
            </h1>
            <h1 class="h2 font-weight-bold mb-4">
              即享 3 個月熱門音樂
            </h1>
            <p class="text-secondary">
              Premium 可讓你隨時隨地播放喜歡的歌曲，沒有廣告。
            </p>
            <button
              class="btn btn-success my-3 font-weight-bold"
              @click="modalShow = false"
            >
              我知道了!
            </button>
          </div>
        </div>
      </div>
    </div>
    <PlayList></PlayList>
  </div>
</template>
<script>
import PlayList from "@/components/PlayList/PlayList";
export default {
  name: "app",
  data() {
    return {
      modalShow: true
    };
  },
  methods: {
    closeModal(e) {
      let modal = this.$refs.modal;
      if (e.target === modal) {
        this.modalShow = false;
      }
    }
  },
  components: {
    PlayList
  }
};
</script>
<style lang="scss">
@import url("https://use.fontawesome.com/releases/v5.6.3/css/all.css");
@import "assets/style";
</style>
