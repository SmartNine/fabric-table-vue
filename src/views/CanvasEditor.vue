<template>
  <InsertTablePanel @insert="handleInsertTable" />
  <FabricTable v-if="showTable" :rows="selectedRows" :cols="selectedCols" ref="fabricTableRef" />
  <button v-if="showTable" @click="downloadJSON">导出JSON</button>
</template>

<script setup>
import { ref } from 'vue'
import FabricTable from '../components/FabricTable.vue'
import InsertTablePanel from '../components/InsertTablePanel.vue'

const selectedRows = ref(0)
const selectedCols = ref(0)
const showTable = ref(false)
const fabricTableRef = ref()

function handleInsertTable({ rows, cols }) {
  selectedRows.value = rows
  selectedCols.value = cols
  showTable.value = true
}

function downloadJSON() {
  const json = fabricTableRef.value.exportJSON()
  const blob = new Blob([JSON.stringify(json, null, 2)], { type: 'application/json' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = 'table.json'
  link.click()
}
</script>
