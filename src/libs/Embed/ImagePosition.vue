<template>
  <div
    class="image-handler"
    v-if="handler.isShow"
    v-bind:style="handler.position"
  >
    <div class="image-hander-menu">
      <button class="btn-toggle" v-on:click="remove">
        ゴミ箱
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["handler"],
  methods: {
    imageSizing(sizing) {
      this.handler.currentSize = sizing;
      this.handler.currentLine.className = "editor-image " + sizing;

      this.$emit("onPositionChange");
    },
    remove() {
      this.handler.isShow = false;
      this.$emit("onRemove");
    }
  }
};
</script>
