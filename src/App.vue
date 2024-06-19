<template>
  <div id="app">
    <button @click="addNewPage">Add New Page</button>
    <button @click="saveConfiguration">Save Configuration</button>
    <input type="file" @change="loadConfiguration" />
    <button @click="prevPage" :disabled="currentPageIndex === 0">Previous Page</button>
    <button @click="nextPage" :disabled="currentPageIndex === pages.length - 1">Next Page</button>
    <input type="number" v-model.number="jumpPageIndex" @input="jumpToPage" min="1" :max="pages.length" />
    <p>Current Page: {{ currentPageIndex + 1 }} / {{ pages.length }}</p>
    <Workspace :pageData="currentPage" @updateItem="updateItem" @selectItem="selectItem" />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import Workspace from './components/Workspace.vue'

const pages = ref([{ items: [] }])
const currentPageIndex = ref(0)
const jumpPageIndex = ref(1)
const selectedItem = ref(null)

const currentPage = computed(() => {
  return { ...pages.value[currentPageIndex.value], pageIndex: currentPageIndex.value }
})

const addNewPage = () => {
  pages.value.push({ items: [] })
  currentPageIndex.value = pages.value.length - 1
}

const updateItem = (pageIndex, items) => {
  pages.value[pageIndex].items = items
}

const saveConfiguration = () => {
  const json = JSON.stringify(pages.value)
  const blob = new Blob([json], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = 'configuration.json'
  a.click()
  URL.revokeObjectURL(url)
}

const loadConfiguration = (event) => {
  const file = event.target.files[0]
  const reader = new FileReader()
  reader.onload = () => {
    pages.value = JSON.parse(reader.result)
    currentPageIndex.value = 0
  }
  reader.readAsText(file)
}

const nextPage = () => {
  if (currentPageIndex.value < pages.value.length - 1) {
    currentPageIndex.value++
  }
}

const prevPage = () => {
  if (currentPageIndex.value > 0) {
    currentPageIndex.value--
  }
}

const jumpToPage = () => {
  if (jumpPageIndex.value > 0 && jumpPageIndex.value <= pages.value.length) {
    currentPageIndex.value = jumpPageIndex.value - 1
  }
}

const selectItem = (item) => {
  selectedItem.value = item
}
</script>

<style scoped>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>