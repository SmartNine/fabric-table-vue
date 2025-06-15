
<template>
  <canvas ref="canvasEl" :width="canvasWidth" :height="canvasHeight" />
</template>

<script setup>
import { ref, onMounted, defineProps, defineExpose } from 'vue'
import { fabric } from 'fabric'

const props = defineProps({
  rows: { type: Number, default: 3 },
  cols: { type: Number, default: 4 },
  cellWidth: { type: Number, default: 100 },
  cellHeight: { type: Number, default: 40 },
  tableData: { type: Array, default: () => [] },
})

const canvasEl = ref(null)
const canvasWidth = 800
const canvasHeight = 600
let canvas, tableGroup

function createTable(data, rows, cols, cellWidth, cellHeight) {
  const cells = []
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      const left = c * cellWidth
      const top = r * cellHeight
      const rect = new fabric.Rect({
        left,
        top,
        width: cellWidth,
        height: cellHeight,
        stroke: '#555',
        strokeWidth: 1,
        fill: '#fff',
        selectable: false,
      })
      const content = data[r]?.[c] || ''
      const text = new fabric.IText(content, {
        left: left + 5,
        top: top + 5,
        fontSize: 16,
        width: cellWidth - 10,
        height: cellHeight - 10,
        editable: true,
        selectable: true,
      })
      cells.push(rect, text)
    }
  }
  return new fabric.Group(cells, {
    left: 100,
    top: 100,
    subTargetCheck: true,
    hasControls: true,
    hasBorders: true,
  })
}

onMounted(() => {
  canvas = new fabric.Canvas(canvasEl.value, {
    selection: true,
  })
  const table = createTable(props.tableData, props.rows, props.cols, props.cellWidth, props.cellHeight)
  tableGroup = table
  canvas.add(table)
  canvas.setActiveObject(table)
  canvas.renderAll()
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
</style>
