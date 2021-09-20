<template>
  <div>
    <li
      class="list-group-item pb-0 border-0 mb-2 bg-transparent rounded-0 text-left d-flex justify-content-between align-items-center"
    >
      <div class="text-white d-flex align-items-center">
        <small class="text-secondary">#{{ index + 1 }}</small>
        <div>
          <div
            class="nav-link text-white d-flex flex-column"
            :class="{ 'text-success': index === currentIndex }"
            style="                                                                 cursor:pointer"
            @dblclick.prevent="selectHandler(item, index)"
          >
            <div class="d-flex">
              {{ item.song }}
              <div v-if="index === currentIndex" v-show="isBuffering">
                <i class="fas fa-spinner fa-spin ml-2"></i>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="text-secondary d-flex">
        <small>{{ item.duration }}</small>
        <div class="mx-4 text-center" style="width:1rem">
          <i
            class="material-icons text-secondary"
            v-show="!item.like"
            @click="addFavoriteListHandler"
            >favorite_border</i
          >
          <i
            class="material-icons text-success"
            v-show="item.like"
            @click="removeListHandler({ item, index })"
            >favoriter</i
          >
        </div>
        <div class="position-relative">
          <i
            class="material-icons text-secondary"
            ref="more"
            @click="showPopovers(index)"
            >more_horiz</i
          >
          <slot></slot>
        </div>
      </div>
    </li>
  </div>
</template>
<script>
export default {
  name: "PlayListItem",
  props: {
    item: {
      type: Object,
      required: true
    },
    index: {
      type: Number,
      required: true
    },
    isBuffering: {
      type: Boolean,
      required: true
    },
    currentIndex: {
      type: Number,
      required: true
    },
    currentSong: {
      type: String
    },
    musicType: {
      type: String
    }
  },
  data() {
    return {};
  },

  methods: {
    showPopovers(index) {
      this.$emit("showPopovers", index);
    },
    selectHandler(item, index) {
      this.$emit("selectSong", { item, index });
    },
    addFavoriteListHandler() {
      this.$set(this.item, "like", true);
    },
    removeListHandler() {
      this.$set(this.item, "like", false);
    }
  }
};
</script>
