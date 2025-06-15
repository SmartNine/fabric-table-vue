<template>
  <div
    class="row-header"
    :style="{
      top: tableTop + 'px',
      height: totalHeight + 'px',
    }"
  >
    <div
      v-for="(r, index) in rows"
      :key="index"
      class="row-label"
      :style="{ height: cellHeight + 'px' }"
      :class="{ selected: index === selectedRow }"
      @click="$emit('select-row', index)"
    >
      {{ index + 1 }}
    </div>
  </div>
</template>

<script setup>
defineProps({
  rows: Number,
  cellHeight: Number,
  tableTop: Number,
  totalHeight: Number,
  selectedRow: Number,
})
defineEmits(['select-row'])
</script>

<style scoped>
.row-header {
  position: absolute;
  left: 0;
  width: 30px;
  background: #f2f2f2;
  border-right: 1px solid #ccc;
  z-index: 50;
}
.row-label {
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border-bottom: 1px solid #ccc;
  font-size: 12px;
  user-select: none;
}
.row-label.selected {
  background-color: #d0ebff;
  font-weight: bold;
}
</style>
