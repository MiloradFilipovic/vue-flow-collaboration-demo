<script setup lang="ts">
import { onMounted, ref } from 'vue'
import Canvas from '@/components/Canvas.vue'
import Toolbar from '@/components/toolbar/Toolbar.vue'
import NodeSidebar from '@/components/nodeSidebar/NodeSidebar.vue'
import { useCanvasStore } from '@/stores/canvas.store'
import type { NodeType } from '../types/common'
import { useUIStore } from '../stores/ui.store'
import { useRoute } from 'vue-router'
import * as Y from 'yjs'
import { createYjsProvider } from '@y-sweet/client'

const route = useRoute()

const canvasStore = useCanvasStore()
const uiStore = useUIStore()

const currentDiagramId = ref<string | null>(null)

onMounted(() => {
  const id = route.params.id as string

  const existingDiagram = canvasStore.allDiagrams[id]
  if (existingDiagram) {
    currentDiagramId.value = id
    canvasStore.currentDiagramId = id
    return
  }
  canvasStore.createNewDiagram('Untitled', '1')
  currentDiagramId.value = canvasStore.currentDiagramId

  // Init Yjs
  const doc = new Y.Doc()
  const docId = 'my-doc-id'
  const authEndpoint = '/.netlify/functions/ysweet-auth'

  createYjsProvider(doc, docId, authEndpoint)
})

const onNodeTypeSelected = (nodeType: NodeType) => {
  canvasStore.addNode(nodeType, uiStore.lastClickedPosition)
}
</script>

<template>
  <div v-if="currentDiagramId" :class="$style.editor">
    <Toolbar />
    <Canvas :diagram-id="currentDiagramId" />
    <NodeSidebar @node-type-selected="onNodeTypeSelected" />
  </div>
  <div v-else>Loading...</div>
</template>

<style module lang="scss">
.editor {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
