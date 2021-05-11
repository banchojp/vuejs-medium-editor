<template>
  <div
    class="image-handler"
    v-if="handler.isShow"
    v-bind:style="handler.position"
  >
    画像サイズ:
    <div class="image-hander-menu">
      <button v-on:click="large">
        大
      </button>
    </div>
    <div class="image-hander-menu">
      <button v-on:click="small">
        小
      </button>
    </div>
    <div class="image-hander-menu">
      <button v-on:click="remove">
        <span>ゴミ箱</span>
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
    large() {
      this.handler.isShow = false;
      this.$emit("onLarge");
    },
    small() {
      this.handler.isShow = false;
      this.$emit("onSmall");
    },
    remove() {
      this.handler.isShow = false;
      this.$emit("onRemove");
    }
  }
};
</script>
