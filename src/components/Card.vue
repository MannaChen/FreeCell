<template>
  <img
    class="card-image"
    draggable
    @dragstart="drag"
    :id="cardId"
    :src="require(`@/assets/card-${cardId}.svg`)"
    alt="cardId">
</template>

<script>
export default {
  props: {
    cardId: {
      type: String,
      default: '',
    },
    columnIndex: {
      type: Number,
      default: -1,
    },
    cardIndex: {
      type: Number,
      default: -1,
    },
    // isDraggable: {
    //   type: Boolean,
    //   default: false,
    // }
  },
  data() {
    return {
    }
  },
  methods: {
    drag() {
      // if (this.isDraggable) {
      this.$emit('drag', this.cardId, this.cardIndex, this.columnIndex);
      // };
    },
  },
}
</script>

<style lang="scss" scoped>
.card-image {
  width: 100%;
  height: 231.7px;
  &:not(:first-child) {
    margin-top: -196px;
  }
  &:last-child {
    z-index: 1;
  }
}
</style>


