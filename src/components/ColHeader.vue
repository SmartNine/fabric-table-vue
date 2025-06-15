<template>
  <div
    class="col-header"
    :style="{
      left: tableLeft + 'px',
      width: totalWidth + 'px',
    }"
  >
    <div
      v-for="(c, index) in cols"
      :key="index"
      class="col-label"
      :style="{ width: cellWidth + 'px' }"
      :class="{ selected: index === selectedCol }"
      @click="$emit('select-col', index)"
    >
      {{ String.fromCharCode(65 + index) }}
    </div>
  </div>
</template>

<script setup>
defineProps({
  cols: Number,
  cellWidth: Number,
  tableLeft: Number,
  totalWidth: Number,
  selectedCol: Number,
})
defineEmits(['select-col'])
</script>

<style scoped>
.col-header {
  position: absolute;
  top: 0;
  height: 30px;
  background: #f2f2f2;
  border-bottom: 1px solid #ccc;
  display: flex;
  z-index: 50;
}
.col-label {
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border-right: 1px solid #ccc;
  font-size: 12px;
  user-select: none;
}
.col-label.selected {
  background-color: #d0ebff;
  font-weight: bold;
}
</style>
