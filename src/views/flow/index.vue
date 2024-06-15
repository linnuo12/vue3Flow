<script setup lang="ts">

import { onMounted, reactive  } from 'vue'
import '@vue-flow/core/dist/style.css'
import '@vue-flow/core/dist/theme-default.css'
import '@vue-flow/controls/dist/style.css'
import { VueFlow, Position, useVueFlow } from '@vue-flow/core'
import { Background } from '@vue-flow/background'
import { Controls } from '@vue-flow/controls'
// import { MiniMap } from '@vue-flow/minimap'
import CustomNode from './components/CustomNode.vue'
 
const { onNodeClick } = useVueFlow()
const treeData = [
  {
    id: '1',
    children: [
      { id: '2', children: [{ id: '4' }, { id: '5' }, { id: '9' }] },
      { id: '3', children: [{ id: '6' }, { id: '10', children: [{ id: '11' }, { id: '12' }, { id: '13' }] }] },
      { id: '7', children: [{ id: '8' }] }
    ]
  }
]
 
const buildNodes = (tree: any, parentNode: any, level: any, levels: any) => {
  const nodes = [] as any
  const edges = [] as any
 
  tree.forEach((node: any) => {
    // levels 记录 递归过程中 每层级的node数量 初始{}
    levels[level] = (levels[level] || 0) + 1
    // 如果子集存在 递归 并合并递归函数返回结果
    if (node.children?.length) {
      const childs = buildNodes(node.children, node, level + 1, levels)
      nodes.push(...childs.nodes)
      edges.push(...childs.edges)
    }
    // 计算在当前递归level时，最长（最多node）的level的node数量
    const top = Math.max(...Object.keys(levels).map(cur => {
      return levels[cur]
    }))
    // 当前node数据处理 业务数据放置data中（customNode接收的node的props有过滤，放入data可带入）
    const temp = {
      id: node.id,
      data: { ...node },
      type: 'custom',
      // style: { background: '#D6D8D9', color: '#333', border: '1px solid #ccc' },
      // 手动指定位置 位置计算规则:
      // x:间隔(leveln-1) 
      // y: 递归计算当前level最大node数top,(递归线路上 leveln时的top < level1时的top),故而,从0向坐标轴反方向增加y -间隔(top-1)
      position: { x: 230 * (level - 1), y: -100 * (top - 1) },
      label: String(node.id),
      sourcePosition: Position.Right,
      targetPosition: Position.Left
    }
    // 连线处理
    if (parentNode) {
      temp.parentId = parentNode.id
      const edge = {
        id: parentNode.id + '-' + temp.id,
        source: parentNode.id,
        target: temp.id,
        type: 'smoothstep',
        animated: true,
        labelBgStyle: { fill: '#10b981', color: '#fff', fillOpacity: 1 },
      }
      edges.push(edge)
    }
    nodes.push(temp)
 
  })
  return {
    nodes,
    edges
  }
}
const mainData = reactive({
  nodes: [],
  edges: []
})

// 全局 id 计数器
let currentId = '' // 从现有的最大 id 开始
onNodeClick((event: any) => {
  return
  // 全局 id 计数器
  currentId = !currentId ? event.node.id : currentId // 从现有的最大 id 开始

  // 递归函数，找到目标节点并添加子项
  function addChildToNode(node, targetId, newChildData) {
    if (node.id === targetId) {
      currentId++;
      const newChild = {
        id: currentId.toString(),
        data: newChildData,
        children: []
      };
      node.children = []
      node.children.push(newChild);
      return true;
    }
    for (let child of node.children) {
      if (addChildToNode(child, targetId, newChildData)) {
        return true;
      }
    }
    return false;
  }

  // 要添加的新子项数据
  let newChildData = {
    name: '新子项'
  };

  // 调用函数添加子项
  addChildToNode(treeData[0], event.node.id, newChildData);

  console.log(treeData)
  // Object.assign(mainData, buildNodes(treeData, null, 1, {}))
  Object.assign(mainData, buildNodes(treeData, null, 1, {}, 0))
})

const addNodes = () => {

}

onMounted(() => {
  Object.assign(mainData, buildNodes(treeData, null, 1, {}))
})
</script>
 
<template>
  <VueFlow :nodes="mainData.nodes" :edges="mainData.edges" fitViewOnInit :nodes-draggable="false">
    <Controls :show-interactive="false" />
    <Background />
    <template #node-custom="customNodeProps">
      <CustomNode v-bind="customNodeProps" @add="addNodes" />
    </template>
    <!-- <MiniMap pannable zoomable /> -->
  </VueFlow>
</template>