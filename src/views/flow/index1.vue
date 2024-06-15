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
      { id: '7', children: [{ id: '8', children: [{ id: '14' }, { id: '15' }, { id: '16' }] }] }
    ]
  }
]
const buildNodes = (tree: any, parentNode: any, level: any, levels: any, mark: any) => {
  const nodes = [] as any
  const edges = [] as any
  tree.forEach((node: any) => {
    if (level == 2) { mark = mark + 1 }
    // levels 记录 递归过程中 每层级的node数量 初始{}
    // mark 记录 循环level2时的次数 避免各支路level数不一致，导致布局混乱的问题
    // 在level1只有一个的情况下，level2第一个迭代链路结束levels会存在已有的层级对应的个数
    // 用mark 区分非第一个链路，如果在后续链路中出现新的层级，就会给新的level加上一层的计数 并给父节点添加一个新增层级的标签 isNewLevel
    levels[level] = mark > 1 && !levels[level] ? (parentNode['isNewLevel'] = true, (levels[level - 1])) : ((levels[level] || 0) + 1)
    // 如果子集存在 递归 并合并递归函数返回结果
    if (node.children?.length) {
      const childs = buildNodes(node.children, node, level + 1, levels, mark)
      nodes.push(...childs.nodes)
      edges.push(...childs.edges)
    }
    // 计算在当前递归level时，最长（最多node）的level的node数量
    const top = Math.max(...Object.keys(levels).map(cur => {
      return levels[cur]
    }))
    // 当前节点是否有子节点
    const childLength = node?.children?.length || 0
    // const more = childLength ? (node.children[childLength - 1].position.y - node.children[0].position.y) / 2 : 0
    // console.log(node.id, top, '---', more, '---mark', mark)
    // 当前node数据处理 业务数据放置data中（customNode接收的node的props有过滤，放入data可带入）
    const temp: any = {
      id: node.id,
      data: { ...node, level },
      type: 'custom',
      // 手动指定位置 位置计算规则:
      // x:间隔(leveln-1) 
      // y: 递归计算当前level最大node数top,(递归线路上 leveln时的top < level1时的top),故而,从0向坐标轴反方向增加y -间隔(top-1)
      // position: { x: 350 * (level - 1), y: -200 * (top - 1) - more },
      // 第二版位置设置 根据子节点位置设置居中位置，无子节点，位置照旧设置
      position: { x: 350 * (level - 1), y: childLength ? (node.children[childLength - 1].position.y + node.children[0].position.y) / 2 : -200 * (top - 1) },
      label: String(node.id),
      sourcePosition: Position.Right,
      targetPosition: Position.Left
    }
    // 记录节点位置于原始数据链 便于父节点递归回去后取值 设置自己的position
    node.position = temp.position
    // 连线处理
    if (parentNode) {
      temp.parentId = parentNode.id
      const edge = {
        id: parentNode.id + '-' + temp.id,
        source: parentNode.id,
        target: temp.id,
        type: 'smoothstep',
        animated: true
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
onNodeClick((event: any) => {
  console.log(event)
})
onMounted(() => {
  Object.assign(mainData, buildNodes(treeData, null, 1, {}, 0))
})
</script>
 
<template>
  <VueFlow :nodes="mainData.nodes" :edges="mainData.edges"  :default-viewport="{ zoom: 2.5 }"
      :min-zoom="1"
      :max-zoom="2" fitViewOnInit :nodes-draggable="false">
    <Controls :show-interactive="false" />
    <Background />
    <template #node-custom="customNodeProps">
      <CustomNode v-bind="customNodeProps" />
    </template>
    <!-- <MiniMap pannable zoomable /> -->
  </VueFlow>
</template>