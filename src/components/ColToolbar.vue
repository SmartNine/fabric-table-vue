<template>
    <div class="col-toolbar" :style="{
        position: 'absolute',
        top: top + 'px',
        left: left + 'px',
    }">
        <button @click="$emit('insert-left')">⬅ 插左列</button>
        <button @click="$emit('insert-right')">➡ 插右列</button>
        <button @click="$emit('delete')">🗑 删除列</button>
        <select @change="$emit('align', $event.target.value)">
            <option value="left">⬅ 左对齐</option>
            <option value="center" selected>⬍ 居中</option>
            <option value="right">➡ 右对齐</option>
        </select>
    </div>
</template>

<script setup>
defineProps({
    top: Number,
    left: Number,
})

defineEmits(['insert-left', 'insert-right', 'delete', 'align'])
</script>

<style scoped>
.col-toolbar {
    display: flex;
    gap: 6px;
    padding: 4px 6px;
    background-color: #eef6ff;
    border: 1px solid #99c1e0;
    border-radius: 4px;
    box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.1);
    z-index: 100;
}

button {
    font-size: 12px;
    padding: 4px 6px;
    cursor: pointer;
}

select {
    font-size: 12px;
}
</style>
