<template>
  <div class="app-container">
    <div class="left-panel">
      <Palette
        class="palette-section"
        @addItemToPaper="addItemToPaper"
      />
    </div>
    <div class="paper-section">
      <Paper
        :droppedItems="pageData.items"
        ref="paperRef"
        @update-items="updateDroppedItems"
        @selectItem="selectItem"
        @addItemToPaper="addItemToPaper"
        size="A4"
      />
    </div>
    <div class="properties-panel">
      <PropertiesPanel
        :selectedItem="selectedItem"
        :droppedItems="pageData.items"
        @updateProperty="updateProperty"
        @updateItemsOrder="updateItemsOrder"
      />
    </div>
  </div>
</template>

<script>
import Palette from "./Palette.vue";
import Paper from "./Paper.vue";
import PropertiesPanel from "./PropertiesPanel.vue";
import { ref, nextTick } from "vue";
import html2canvas from "html2canvas";
import { v4 as uuidv4 } from "uuid";

export default {
  components: {
    Palette,
    Paper,
    PropertiesPanel,
  },
  props: {
    pageData: Object,
  },
  setup(props, { emit }) {
    const selectedItem = ref(null);
    const paperRef = ref(null);

    const updateDroppedItems = (updatedItems) => {
      emit("updateItem", props.pageData.pageIndex, updatedItems);
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
      props.pageData.items.push(newItem);
      updateDroppedItems(props.pageData.items);
      selectItem(newItem);
    };

    const selectItem = (item) => {
      selectedItem.value = item;
      emit("selectItem", item);
    };

    const updateProperty = (property, value) => {
      if (selectedItem.value) {
        selectedItem.value[property] = value;
        updateDroppedItems(props.pageData.items);
      }
    };

    const importLayoutFromJson = async (event) => {
      const file = event.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (e) => {
        try {
          const importedData = JSON.parse(e.target.result);
          if (Array.isArray(importedData)) {
            props.pageData.items = importedData;
            updateDroppedItems(props.pageData.items);
          } else {
            console.error("Invalid JSON format");
            alert("Invalid JSON format");
          }
        } catch (err) {
          console.error("Error reading JSON:", err);
          alert("Error reading JSON file");
        }
      };
      reader.readAsText(file);
    };

    const exportLayoutToJson = async () => {
      await nextTick();
      const layoutData = JSON.stringify(props.pageData.items, null, 2);
      const blob = new Blob([layoutData], { type: "application/json" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "layout.json";
      link.click();
      URL.revokeObjectURL(link.href);
    };

    const ensureImagesLoaded = async () => {
      const images = paperRef.value.$el.querySelectorAll("img");
      const promises = Array.from(images).map((img) => {
        return new Promise((resolve, reject) => {
          if (img.complete) {
            resolve();
          } else {
            img.onload = resolve;
            img.onerror = reject;
          }
        });
      });
      await Promise.all(promises);
    };

    const exportAsJPG = async () => {
      await nextTick();
      await ensureImagesLoaded();
      const paperElement = paperRef.value.$el;
      html2canvas(paperElement, { allowTaint: true, useCORS: true })
        .then((canvas) => {
          const dataURL = canvas.toDataURL("image/jpeg");
          const link = document.createElement("a");
          link.href = dataURL;
          link.download = "paper-export.jpg";
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
        })
        .catch((error) => {
          console.error("Error exporting as JPG:", error);
        });
    };

    const updateItemsOrder = (newOrder) => {
      props.pageData.items = newOrder;
      updateDroppedItems(newOrder);
    };

    return {
      selectedItem,
      addItemToPaper,
      exportAsJPG,
      paperRef,
      exportLayoutToJson,
      updateDroppedItems,
      importLayoutFromJson,
      selectItem,
      updateProperty,
      updateItemsOrder,
    };
  },
};
</script>

<style>
.app-container {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* Create a three-column grid */
  height: 100vh;
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
  width: 210mm; /* Width for A4 paper */
  height: 297mm; /* Height for A4 paper */
  background-color: #fff; /* Paper background color */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1); /* Optional: Add a shadow for better visibility */
}
</style>
