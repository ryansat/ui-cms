<template>
  <div class="page-controls">
    <button @click="addNewPage">Add New Page</button>
    <button @click="saveConfiguration">Save Configuration</button>
    <input
      type="file"
      @change="loadConfiguration"
    />
    <button
      @click="prevPage"
      :disabled="currentPageIndex === 0"
    >
      Previous Page
    </button>
    <button
      @click="nextPage"
      :disabled="currentPageIndex === pages.length - 1"
    >
      Next Page
    </button>
    <input
      type="number"
      v-model.number="jumpPageIndex"
      @input="jumpToPage"
      min="1"
      :max="pages.length"
    />
    <p>Current Page: {{ currentPageIndex + 1 }} / {{ pages.length }}</p>
  </div>
</template>

<script>
import { ref } from "vue";

export default {
  props: {
    pages: Array,
    currentPageIndex: Number,
  },
  emits: ["updatePages", "updateCurrentPageIndex"],
  setup(props, { emit }) {
    const jumpPageIndex = ref(1);
    const pages = ref(props.pages);
    const currentPageIndex = ref(props.currentPageIndex);

    const addNewPage = () => {
      pages.value.push({ items: [] });
      currentPageIndex.value = pages.value.length - 1;
      emit("updatePages", pages.value);
      emit("updateCurrentPageIndex", currentPageIndex.value);
    };

    const saveConfiguration = () => {
      const json = JSON.stringify(pages.value);
      const blob = new Blob([json], { type: "application/json" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "configuration.json";
      a.click();
      URL.revokeObjectURL(url);
    };

    const loadConfiguration = (event) => {
      const file = event.target.files[0];
      const reader = new FileReader();
      reader.onload = () => {
        pages.value = JSON.parse(reader.result);
        currentPageIndex.value = 0;
        emit("updatePages", pages.value);
        emit("updateCurrentPageIndex", currentPageIndex.value);
      };
      reader.readAsText(file);
    };

    const nextPage = () => {
      if (currentPageIndex.value < pages.value.length - 1) {
        currentPageIndex.value++;
        emit("updateCurrentPageIndex", currentPageIndex.value);
      }
    };

    const prevPage = () => {
      if (currentPageIndex.value > 0) {
        currentPageIndex.value--;
        emit("updateCurrentPageIndex", currentPageIndex.value);
      }
    };

    const jumpToPage = () => {
      if (
        jumpPageIndex.value > 0 &&
        jumpPageIndex.value <= pages.value.length
      ) {
        currentPageIndex.value = jumpPageIndex.value - 1;
        emit("updateCurrentPageIndex", currentPageIndex.value);
      }
    };

    return {
      jumpPageIndex,
      addNewPage,
      saveConfiguration,
      loadConfiguration,
      nextPage,
      prevPage,
      jumpToPage,
      currentPageIndex,
      pages,
    };
  },
};
</script>

<style scoped>
.page-controls {
  text-align: center;
  margin-bottom: 20px;
  color: white; /* Set font color to white */
}

.page-controls p {
  padding-top: 10px; /* Add padding between the paragraph and the previous element */
}
</style>
