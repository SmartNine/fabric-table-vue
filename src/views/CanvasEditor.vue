
<template>
  <FabricTable
    :rows="4"
    :cols="3"
    :cellWidth="120"
    :cellHeight="40"
    :tableData="tableData"
    ref="fabricTableRef"
  />
  <button @click="downloadJSON">导出JSON</button>
</template>

<script setup>
import { ref } from 'vue'
import FabricTable from '../components/FabricTable.vue'

const tableData = [
  ['Name', 'Age', 'City'],
  ['Alice', '24', 'LA'],
  ['Bob', '32', 'NY'],
  ['Eve', '', 'SF'],
]

const fabricTableRef = ref()

function downloadJSON() {
  const json = fabricTableRef.value.exportJSON()
  const blob = new Blob([JSON.stringify(json, null, 2)], { type: 'application/json' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = 'table.json'
  link.click()
}
</script>
