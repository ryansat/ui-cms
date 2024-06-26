<template>
  <div id="app">
    <PageControls
      :pages="pages"
      :currentPageIndex="currentPageIndex"
      @updatePages="updatePages"
      @updateCurrentPageIndex="updateCurrentPageIndex"
    />
    <Workspace
      :pageData="currentPage"
      @updateItem="updateItem"
      @selectItem="selectItem"
    />
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import Workspace from "./components/Workspace.vue";
import PageControls from "./components/PageControls.vue";

const pages = ref([{ items: [] }]);
const currentPageIndex = ref(0);
const selectedItem = ref(null);

const currentPage = computed(() => {
  return {
    ...pages.value[currentPageIndex.value],
    pageIndex: currentPageIndex.value,
  };
});

const updatePages = (newPages) => {
  pages.value = newPages;
};

const updateCurrentPageIndex = (newIndex) => {
  currentPageIndex.value = newIndex;
};

const updateItem = (pageIndex, items) => {
  pages.value[pageIndex].items = items;
};

const selectItem = (item) => {
  selectedItem.value = item;
};
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
