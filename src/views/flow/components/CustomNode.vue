<script setup lang="ts">
import type { NodeProps } from '@vue-flow/core'
import { Position, Handle } from '@vue-flow/core'
 
const props = defineProps<NodeProps>()
const emit = defineEmits(['add'])

const add = () => {
  emit('add')
}
</script>
 
<template>
  <div class="custom-node">
    <Handle type="target" :position="Position.Left" style="background: rgb(244, 164, 96);" />
    <div>{{ props.id }}</div>
    <Handle type="source" :position="Position.Right" style="background: rgb(244, 164, 96);" @click="add" />
  </div>
</template>
<style lang="less" scoped>
.custom-node {
  width: 100px;
  height: 50px;
  line-height: 50px;
  border-radius: 8px;
  border: 1px solid rgb(244, 164, 96);
  background: #fff;
}
</style>