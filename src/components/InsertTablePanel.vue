<template>
  <div class="insert-panel">
    <h3>Insert Table</h3>
    <div class="control">
      <label>Columns</label>
      <button @click="cols = Math.max(1, cols - 1)">－</button>
      <span>{{ cols }}</span>
      <button @click="cols++">＋</button>
    </div>
    <div class="control">
      <label>Rows</label>
      <button @click="rows = Math.max(1, rows - 1)">－</button>
      <span>{{ rows }}</span>
      <button @click="rows++">＋</button>
    </div>
    <button class="add-button" @click="insert">Add Table</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const rows = ref(2)
const cols = ref(2)

const emit = defineEmits(['insert'])

function insert() {
  emit('insert', { rows: rows.value, cols: cols.value })
}
</script>

<style scoped>
.insert-panel {
  padding: 16px;
  background: #f7f7f7;
  border: 1px solid #ccc;
  border-radius: 8px;
  width: 240px;
  margin-bottom: 16px;
}
.control {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
label {
  width: 80px;
}
button {
  margin: 0 4px;
}
.add-button {
  background-color: #00aaff;
  color: white;
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
