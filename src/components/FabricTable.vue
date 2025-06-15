<template>
  <div style="position: relative">
    <div class="toolbar">
      <button @click="addRow">➕ 添加行</button>
      <button @click="addCol">➕ 添加列</button>
      <button @click="removeRow">➖ 删除行</button>
      <button @click="removeCol">➖ 删除列</button>
    </div>

    <canvas ref="canvasEl" :width="canvasWidth" :height="canvasHeight" />

    <!-- 行工具栏 -->
    <RowToolbar v-if="selectedRowIndex !== null" :top="toolbarTop" @insert-above="insertRowAbove"
      @insert-below="insertRowBelow" @delete="deleteSelectedRow" />

    <!-- 列工具栏 -->
    <ColToolbar v-if="selectedColIndex !== null" :top="20" :left="toolbarLeft" @insert-left="insertColLeft"
      @insert-right="insertColRight" @delete="deleteSelectedCol" @align="setColAlign" />
  </div>
</template>

<script setup>
import { ref, onMounted, defineProps, defineExpose } from 'vue'
import { fabric } from 'fabric'
import RowToolbar from './RowToolbar.vue'
import ColToolbar from './ColToolbar.vue'

const props = defineProps({
  rows: { type: Number, default: 3 },
  cols: { type: Number, default: 4 },
  cellWidth: { type: Number, default: 100 },
  cellHeight: { type: Number, default: 40 },
  tableData: { type: Array, default: () => [] },
})

const canvasEl = ref(null)
const canvasWidth = 1000
const canvasHeight = 600
let canvas, tableGroup
const table = ref([])
const textRefs = ref([]) // 记录每格 IText 对象
const selectedRowIndex = ref(null)
const selectedColIndex = ref(null)
const toolbarTop = ref(0)
const toolbarLeft = ref(0)

function initTableData(rows, cols, data = []) {
  const result = []
  for (let r = 0; r < rows; r++) {
    const row = []
    for (let c = 0; c < cols; c++) {
      row.push(data[r]?.[c] || '')
    }
    result.push(row)
  }
  return result
}

function renderTable() {
  canvas.clear()
  textRefs.value = []
  const cells = []
  const rows = table.value.length
  const cols = table.value[0].length

  for (let r = 0; r < rows; r++) {
    const textRow = []
    for (let c = 0; c < cols; c++) {
      const left = c * props.cellWidth
      const top = r * props.cellHeight

      const rect = new fabric.Rect({
        left,
        top,
        width: props.cellWidth,
        height: props.cellHeight,
        stroke: '#555',
        strokeWidth: 1,
        fill: '#fff',
        selectable: false,
      })

      const text = new fabric.IText(table.value[r][c], {
        left: left + 5,
        top: top + 5,
        fontSize: 16,
        width: props.cellWidth - 10,
        height: props.cellHeight - 10,
        editable: true,
        selectable: true,
        textAlign: 'center',
      })

      text.on('mousedown', () => {
        selectedRowIndex.value = r
        selectedColIndex.value = c
        toolbarTop.value = top + 20
        toolbarLeft.value = left + 20
      })

      text.on('changed', () => {
        table.value[r][c] = text.text
      })

      textRow.push(text)
      cells.push(rect, text)
    }
    textRefs.value.push(textRow)
  }

  tableGroup = new fabric.Group(cells, {
    left: 100,
    top: 100,
    subTargetCheck: true,
    hasControls: true,
    hasBorders: true,
  })

  canvas.add(tableGroup)
  canvas.setActiveObject(tableGroup)
  canvas.renderAll()
}

function addRow() {
  table.value.push(Array(table.value[0].length).fill(''))
  renderTable()
}

function addCol() {
  for (let row of table.value) row.push('')
  renderTable()
}

function removeRow() {
  if (table.value.length > 1) table.value.pop()
  renderTable()
}

function removeCol() {
  if (table.value[0].length > 1)
    table.value.forEach((row) => row.pop())
  renderTable()
}

// 行操作
function insertRowAbove() {
  if (selectedRowIndex.value !== null) {
    table.value.splice(selectedRowIndex.value, 0, Array(table.value[0].length).fill(''))
    renderTable()
  }
}
function insertRowBelow() {
  if (selectedRowIndex.value !== null) {
    table.value.splice(selectedRowIndex.value + 1, 0, Array(table.value[0].length).fill(''))
    renderTable()
  }
}
function deleteSelectedRow() {
  if (table.value.length > 1 && selectedRowIndex.value !== null) {
    table.value.splice(selectedRowIndex.value, 1)
    renderTable()
  }
}

// 列操作
function insertColLeft() {
  if (selectedColIndex.value !== null) {
    for (let row of table.value) row.splice(selectedColIndex.value, 0, '')
    renderTable()
  }
}
function insertColRight() {
  if (selectedColIndex.value !== null) {
    for (let row of table.value) row.splice(selectedColIndex.value + 1, 0, '')
    renderTable()
  }
}
function deleteSelectedCol() {
  if (selectedColIndex.value !== null && table.value[0].length > 1) {
    for (let row of table.value) row.splice(selectedColIndex.value, 1)
    renderTable()
  }
}
function setColAlign(alignType) {
  const col = selectedColIndex.value
  if (col === null) return
  for (let r = 0; r < textRefs.value.length; r++) {
    const text = textRefs.value[r][col]
    if (text) {
      text.textAlign = alignType
    }
  }
  canvas.requestRenderAll()
}

onMounted(() => {
  canvas = new fabric.Canvas(canvasEl.value, { selection: true })
  table.value = initTableData(props.rows, props.cols, props.tableData)
  renderTable()
})

defineExpose({
  getCanvas: () => canvas,
  exportJSON: () => canvas.toJSON(),
})
</script>

<style scoped>
canvas {
  border: 1px solid #ccc;
}

.toolbar {
  margin-bottom: 10px;
}

button {
  margin-right: 10px;
  padding: 6px 10px;
  font-size: 14px;
}
</style>
