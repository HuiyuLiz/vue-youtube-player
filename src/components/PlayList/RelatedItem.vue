<template>
  <div class="d-flex flex-column justify-content-center align-items-center">
    <div
      class="img-singer-thumb img-cover rounded-circle"
      :style="`background-image: url(${publicPath}${item.backgroundImg})`"
      @click="selectHandler(item, index)"
    ></div>
    <small class="text-white mt-3" v-if="musicType !== 'Daily Mix'">{{
      item.singer
    }}</small>
    <small class="text-white mt-3" v-else>Daily Mix</small>
  </div>
</template>

<script>
export default {
  name: "RelatedItem",
  props: {
    item: {
      type: Object,
      required: true
    },
    index: {
      type: Number,
      required: true
    },
    musicType: {
      type: String
    }
  },
  data() {
    return {
      publicPath: "./",
      currentItem: {},
      currentIndex: null
    };
  },
  methods: {
    selectHandler(item, index) {
      this.currentIndex = index;
      this.currentItem = item;
      this.emitHandler(item, index);
    },
    emitHandler(item, index) {
      this.$emit("selectSinger", { item, index });
    }
  }
};
</script>

<style></style>
