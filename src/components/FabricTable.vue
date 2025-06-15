<template>
  <div style="position: relative">
    <RowHeader :rows="table.length" :cellHeight="props.cellHeight" :tableTop="tableTop"
      :totalHeight="table.length * props.cellHeight" :selectedRow="selectedRowIndex"
      @select-row="handleRowHeaderClick" />

    <ColHeader :cols="table[0]?.length || 0" :cellWidth="props.cellWidth" :tableLeft="tableLeft"
      :totalWidth="table[0]?.length * props.cellWidth || 0" :selectedCol="selectedColIndex"
      @select-col="handleColHeaderClick" />

    <canvas ref="canvasEl" :width="canvasWidth" :height="canvasHeight" />

    <!-- 行工具栏 -->
    <RowToolbar v-if="showRowToolbar" :top="toolbarTop" @insert-above="insertRowAbove" @insert-below="insertRowBelow"
      @delete="deleteSelectedRow" />

    <!-- 列工具栏 -->
    <ColToolbar v-if="showColToolbar" :top="20" :left="toolbarLeft" @insert-left="insertColLeft"
      @insert-right="insertColRight" @delete="deleteSelectedCol" @align="setColAlign" />

    <!-- 浮动输入框 -->
    <input v-if="showInput" v-model="inputValue" :style="inputStyle" class="floating-input" @blur="applyInput"
      @keyup.enter="applyInput" />
  </div>
</template>

<script setup>
import { ref, onMounted, defineProps, defineExpose, watch, nextTick } from 'vue'
import { fabric } from 'fabric'
import RowToolbar from './RowToolbar.vue'
import ColToolbar from './ColToolbar.vue'
import RowHeader from './RowHeader.vue'
import ColHeader from './ColHeader.vue'

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
const textRefs = ref([])
const rectRefs = ref([])
const selectedRowIndex = ref(null)
const selectedColIndex = ref(null)
const toolbarTop = ref(0)
const toolbarLeft = ref(0)
const tableTop = 100
const tableLeft = 100
const showRowToolbar = ref(false)
const showColToolbar = ref(false)

// 输入框相关
const showInput = ref(false)
const inputValue = ref('')
const inputStyle = ref({})
let editingText = null

watch([selectedRowIndex, selectedColIndex], () => {
  showRowToolbar.value = selectedRowIndex.value !== null
  showColToolbar.value = selectedColIndex.value !== null
})

function highlightRowCol() {
  for (let r = 0; r < rectRefs.value.length; r++) {
    for (let c = 0; c < rectRefs.value[r].length; c++) {
      const rect = rectRefs.value[r][c]
      const isRow = selectedRowIndex.value === r
      const isCol = selectedColIndex.value === c
      rect.set('fill', isRow || isCol ? '#e6f4ff' : '#fff')
    }
  }
  canvas.requestRenderAll()
}

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
  rectRefs.value = []
  const cells = []
  const rows = table.value.length
  const cols = table.value[0].length

  for (let r = 0; r < rows; r++) {
    const textRow = []
    const rectRow = []
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

      const text = new fabric.Textbox(table.value[r][c], {
        left: left + 5,
        top: top + 5,
        fontSize: 16,
        width: props.cellWidth - 10,
        height: props.cellHeight - 10,
        editable: false,
        selectable: true,
        textAlign: 'center',
      })

      textRow.push(text)
      rectRow.push(rect)
      cells.push(rect, text)
    }
    textRefs.value.push(textRow)
    rectRefs.value.push(rectRow)
  }

  tableGroup = new fabric.Group(cells, {
    left: tableLeft,
    top: tableTop,
    subTargetCheck: true,
    hasControls: true,
    hasBorders: true,
  })

  canvas.add(tableGroup)
  canvas.setActiveObject(tableGroup)
  canvas.renderAll()
}

function applyInput() {
  if (editingText && selectedRowIndex.value !== null && selectedColIndex.value !== null) {
    editingText.text = inputValue.value
    table.value[selectedRowIndex.value][selectedColIndex.value] = inputValue.value
    showInput.value = false
    canvas.requestRenderAll()
  }
}

function handleRowHeaderClick(index) {
  selectedRowIndex.value = index
  toolbarTop.value = tableTop + index * props.cellHeight + 20
  highlightRowCol()
}

function handleColHeaderClick(index) {
  selectedColIndex.value = index
  toolbarLeft.value = tableLeft + index * props.cellWidth + 20
  highlightRowCol()
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

  canvas.on('mouse:down', (opt) => {
    const target = opt.target
    if (!target || target === tableGroup) {
      selectedRowIndex.value = null
      selectedColIndex.value = null
      showInput.value = false
      highlightRowCol()
      canvas.requestRenderAll()
    }
  })

  canvas.on('mouse:dblclick', (opt) => {
    const target = opt.target
    if (target && target.type === 'textbox') {
      const r = Math.floor(target.top / props.cellHeight)
      const c = Math.floor(target.left / props.cellWidth)
      selectedRowIndex.value = r
      selectedColIndex.value = c
      highlightRowCol()

      editingText = textRefs.value[r][c] // 使用已缓存的 TextBox 引用
      inputValue.value = editingText.text

      inputStyle.value = {
        position: 'absolute',
        left: `${tableLeft + editingText.left}px`,
        top: `${tableTop + editingText.top}px`,
        width: `${editingText.width}px`,
        height: `${props.cellHeight - 10}px`,
        fontSize: '16px',
        padding: '2px 4px',
        border: '1px solid #888',
      }

      showInput.value = true
      nextTick(() => {
        document.querySelector('.floating-input')?.focus()
      })
    }
  })
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
  display: none;
}

button {
  margin-right: 10px;
  padding: 6px 10px;
  font-size: 14px;
}

.floating-input {
  position: absolute;
  z-index: 10;
  box-sizing: border-box;
  outline: none;
}
</style>
