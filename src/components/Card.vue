<template>
  <img
    class="card-image"
		:class="{'is-dragging': isDragging}"
    draggable
    @dragstart="drag"
		@dragend="isDragging = false"
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
  },
  data() {
    return {
	    isDragging: false,
    }
  },
  methods: {
    drag(e) {
    	this.isDragging = true;
      this.$emit('drag', this.cardId, this.cardIndex, this.columnIndex);
    },
  },
}
</script>

<style lang="scss" scoped>
.card-image {
  width: 100%;
	height: 231.7px;
	&.is-dragging {
		filter: invert(1);
	}
  &:not(:first-child) {
    margin-top: -196px;
  }
  &:last-child {
    z-index: 1;
  }
}
</style>


