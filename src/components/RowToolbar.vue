<template>
    <div class="row-toolbar" :style="{
        position: 'absolute',
        top: top + 'px',
        left: left + 'px',
    }">
        <button @click="$emit('insert-above')">⬆ 插入上方</button>
        <button @click="$emit('insert-below')">⬇ 插入下方</button>
        <button @click="$emit('delete')">🗑 删除行</button>
    </div>
</template>

<script setup>
defineProps({
    top: Number,
    left: {
        type: Number,
        default: 20,
    },
})

defineEmits(['insert-above', 'insert-below', 'delete'])
</script>

<style scoped>
.row-toolbar {
    display: flex;
    gap: 6px;
    padding: 4px 6px;
    background-color: #f9f9f9;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.1);
    z-index: 100;
}

button {
    font-size: 12px;
    padding: 4px 6px;
    cursor: pointer;
}
</style>
