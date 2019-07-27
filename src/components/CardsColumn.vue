<template>
  <div
    class="cards-wrap"
    @drop="onDropCardColumn"
    @dragover="allowDrop"
  >
    <slot />
  </div>
</template>

<script>
export default {
  props: {
    cards: {
      type: Array,
      default: [],
    },
    columnIndex: {
      type: Number,
      default: 0,
    }
  },
  data() {
    return {
      
    }
  },
  methods: {
    onDropCardColumn() {
      this.$emit('onDropCardColumn', this.columnIndex);
    },
    allowDrop(e) {
      e.preventDefault();
    },
  }
}
</script>

<style lang="scss" scoped>
.cards-wrap {
  width: 100%;
  height: 100%;
}
</style>


