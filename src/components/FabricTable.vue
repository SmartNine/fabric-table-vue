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
      @keyup.enter="applyInput" @keyup.esc="cancelEdit" />
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
let editingRow = null
let editingCol = null

// 双击防抖相关
let isDoubleClick = false
let doubleClickTimer = null

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

      // 确保从 table.value 获取最新数据
      const cellValue = table.value[r][c] || ''
      const text = new fabric.Textbox(cellValue, {
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
  if (editingRow !== null && editingCol !== null) {
    // 更新数据模型
    table.value[editingRow][editingCol] = inputValue.value

    // 更新 fabric.js 文本对象
    if (editingText) {
      editingText.set('text', inputValue.value)
    }

    // 清理编辑状态
    showInput.value = false
    editingText = null
    editingRow = null
    editingCol = null

    canvas.requestRenderAll()
  }
}

function cancelEdit() {
  showInput.value = false
  editingText = null
  editingRow = null
  editingCol = null
}

function handleRowHeaderClick(index) {
  if (!showInput.value) { // 如果不在编辑状态才允许选择行
    selectedRowIndex.value = index
    toolbarTop.value = tableTop + index * props.cellHeight + 20
    highlightRowCol()
  }
}

function handleColHeaderClick(index) {
  if (!showInput.value) { // 如果不在编辑状态才允许选择列
    selectedColIndex.value = index
    toolbarLeft.value = tableLeft + index * props.cellWidth + 20
    highlightRowCol()
  }
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

  // 修改 mouse:down 事件，避免双击时触发
  canvas.on('mouse:down', (opt) => {
    // 如果是双击或正在编辑，不处理这个事件
    if (isDoubleClick || showInput.value) {
      return
    }

    // 延迟处理，给双击事件留时间
    doubleClickTimer = setTimeout(() => {
      const target = opt.target
      if (!target || target === tableGroup) {
        selectedRowIndex.value = null
        selectedColIndex.value = null
        highlightRowCol()
        canvas.requestRenderAll()
      }
    }, 200) // 200ms 延迟
  })

  // 修复后的双击事件处理
  canvas.on('mouse:dblclick', (opt) => {
    // 设置双击标志
    isDoubleClick = true

    // 清除单击的延迟处理
    if (doubleClickTimer) {
      clearTimeout(doubleClickTimer)
      doubleClickTimer = null
    }

    // 重置双击标志
    setTimeout(() => {
      isDoubleClick = false
    }, 300)

    const pointer = canvas.getPointer(opt.e)

    // 检查是否点击在表格区域内
    if (tableGroup && tableGroup.containsPoint(pointer)) {
      // 计算相对于表格的坐标
      const relativeX = pointer.x - tableGroup.left
      const relativeY = pointer.y - tableGroup.top

      // 计算行列索引
      const c = Math.floor(relativeX / props.cellWidth)
      const r = Math.floor(relativeY / props.cellHeight)

      // 检查索引是否有效
      if (r >= 0 && r < table.value.length && c >= 0 && c < table.value[0].length) {
        // 不要设置选中的行列，只进入编辑模式
        // 清除之前的选中状态
        selectedRowIndex.value = null
        selectedColIndex.value = null
        highlightRowCol()

        // 设置编辑状态
        editingRow = r
        editingCol = c
        editingText = textRefs.value[r][c]
        inputValue.value = table.value[r][c] // 从数据源获取当前值

        // 计算输入框的绝对位置
        const inputLeft = tableGroup.left + c * props.cellWidth + 5
        const inputTop = tableGroup.top + r * props.cellHeight + 5

        inputStyle.value = {
          position: 'absolute',
          left: `${inputLeft}px`,
          top: `${inputTop}px`,
          width: `${props.cellWidth - 10}px`,
          height: `${props.cellHeight - 10}px`,
          fontSize: '16px',
          padding: '2px 4px',
          border: '2px solid #007acc',
          background: '#fff',
          zIndex: 1000,
          boxSizing: 'border-box'
        }

        showInput.value = true

        nextTick(() => {
          const input = document.querySelector('.floating-input')
          if (input) {
            input.focus()
            input.select()
          }
        })
      }
    }
  })

  // 添加键盘事件监听
  document.addEventListener('keydown', (e) => {
    if (showInput.value) {
      if (e.key === 'Escape') {
        cancelEdit()
      }
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