<template>
  <div class="app-container">
    <div class="top-row">
      <PageControls
        :pages="pages"
        :currentPageIndex="currentPageIndex"
        @updatePages="updatePages"
        @updateCurrentPageIndex="updateCurrentPageIndex"
        @adjustPageSize="adjustPageSize"
      />
    </div>
    <div class="bottom-row">
      <div class="left-panel">
        <Palette
          class="palette-section"
          @addItemToPaper="addItemToPaper"
        />
      </div>
      <div class="paper-section">
        <Paper
          :droppedItems="pageData.items"
          :paperSize="paperSize"
          ref="paperRef"
          @update-items="updateDroppedItems"
          @selectItem="selectItem"
          @addItemToPaper="addItemToPaper"
          @deleteItem="deleteItem"
          @editItem="editItem"
        />
      </div>
      <div class="properties-panel">
        <PropertiesPanel
          :selectedItem="selectedItem"
          :droppedItems="pageData.items"
          @updateProperty="updateProperty"
          @updateItemsOrder="updateItemsOrder"
          @renameItem="renameItem"
          @deleteItem="deleteItem"
          @uploadImage="showImageUploadModal"
        />
      </div>
    </div>
    <ImageUploadModal
      :visible="isImageUploadModalVisible"
      @upload="handleImageUpload"
      @close="hideImageUploadModal"
    />
  </div>
</template>

<script>
import Palette from "./components/palette/Palette.vue";
import Paper from "./components/paper/Paper.vue";
import PropertiesPanel from "./components/layout/PropertiesPanel.vue";
import PageControls from "./components/layout/PageControls.vue";
import ImageUploadModal from "./components/common/ImageUploadModal.vue";
import { ref } from "vue";
import { v4 as uuidv4 } from "uuid";

export default {
  components: {
    Palette,
    Paper,
    PropertiesPanel,
    PageControls,
    ImageUploadModal,
  },
  props: {
    pageData: Object,
  },
  setup(props, { emit }) {
    const pages = ref([{ items: [] }]);
    const currentPageIndex = ref(0);
    const pageData = ref(pages.value[currentPageIndex.value]);
    const selectedItem = ref(null);
    const paperSize = ref({ width: 210, height: 297 });
    const isImageUploadModalVisible = ref(false);

    const updateDroppedItems = (items) => {
      pageData.value.items = items;
    };

    const addItemToPaper = (item) => {
      const newItem = {
        ...item,
        id: uuidv4(),
        x: 100,
        y: 100,
        width: item.width || 100,
        height: item.height || 100,
        label: item.label || "",
      };
      pageData.value.items.push(newItem);
      updateDroppedItems(pageData.value.items);
      selectItem(newItem);
    };

    const selectItem = (item) => {
      selectedItem.value = item;
    };

    const updateProperty = (property, value) => {
      if (selectedItem.value) {
        selectedItem.value[property] = value;
        updateDroppedItems(pageData.value.items);
      }
    };

    const updatePages = (newPages) => {
      pages.value = newPages;
    };

    const updateCurrentPageIndex = (newIndex) => {
      currentPageIndex.value = newIndex;
      pageData.value = pages.value[newIndex];
    };

    const adjustPageSize = (size) => {
      paperSize.value = size;
    };

    const showImageUploadModal = () => {
      isImageUploadModalVisible.value = true;
    };

    const hideImageUploadModal = () => {
      isImageUploadModalVisible.value = false;
    };

    const handleImageUpload = (imageUrl) => {
      if (selectedItem.value) {
        updateProperty("imageUrl", imageUrl);
      }
      hideImageUploadModal();
    };

    const deleteItem = (itemId) => {
      const index = pageData.value.items.findIndex(
        (item) => item.id === itemId
      );
      if (index !== -1) {
        pageData.value.items.splice(index, 1);
        updateDroppedItems(pageData.value.items);
        if (selectedItem.value && selectedItem.value.id === itemId) {
          selectedItem.value = null;
        }
      }
    };

    const renameItem = (item) => {
      const index = pageData.value.items.findIndex((i) => i.id === item.id);
      if (index !== -1) {
        pageData.value.items[index].name = item.name;
        updateDroppedItems(pageData.value.items);
      }
    };

    const editItem = (item) => {
      selectedItem.value = item;
    };

    return {
      pages,
      currentPageIndex,
      pageData,
      selectedItem,
      paperSize,
      updateDroppedItems,
      addItemToPaper,
      updatePages,
      updateCurrentPageIndex,
      adjustPageSize,
      updateProperty,
      isImageUploadModalVisible,
      showImageUploadModal,
      handleImageUpload,
      hideImageUploadModal,
      deleteItem,
      renameItem,
      editItem,
    };
  },
};
</script>

<style scoped>
.app-container {
  display: grid;
  grid-template-rows: auto 1fr; /* Two rows: auto for top row, 1fr for bottom row */
  grid-template-columns: 1fr; /* Single column layout */
  height: 100vh;
  gap: 1em; /* Gap between the rows */
}

.top-row {
  display: flex;
  justify-content: center;
  align-items: center;
}

.bottom-row {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* Create a three-column grid */
  gap: 1em; /* Gap between the columns */
  align-items: start;
}

.palette-section,
.paper-section,
.properties-panel {
  height: 100%;
}

.palette-section {
  background-color: #f0f0f0;
  overflow-y: auto;
  padding: 1em;
  margin-left: 1em; /* Add margin to the left */
}

.paper-section {
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #ffffff; /* Set background color for paper */
  padding: 1em;
  border: 1px solid #ccc; /* Optional: Add a border to the paper section */
}

.paper-section .paper {
  background-color: #fff; /* Paper background color */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* Optional: Add a shadow for better visibility */
}
</style>
