<template>
  <div>
    <button
      class="btn btn-outline-light mx-2"
      :class="{ 'text-success': item.isFollow }"
      @click="follow"
    >
      <small>{{ !item.isFollow ? "FOLLOW" : "UNFOLLOW" }}</small>
    </button>
  </div>
</template>
<script>
export default {
  name: "PlayListFollowerBtn",
  props: ["item"],
  methods: {
    follow() {
      this.$emit("followHandler", this.item);
    }
  }
};
</script>
