<template>
  <div class="app-container">
    <Palette class="palette-section" @addItemToPaper="addItemToPaper" />
    <div class="paper-section">
      <Paper :droppedItems="droppedItems" ref="paperRef" @update-items="updateDroppedItems" />
    </div>
    <div class="side-buttons">
      <button class="export-button" @click="exportAsJPG">Export as JPG</button>
      <button class="export-json-button" @click="exportLayoutToJson">
        Export Layout to JSON
      </button>
      <input
        type="file"
        class="import-json-button"
        @change="importLayoutFromJson"
        accept=".json"
      />
    </div>
  </div>
</template>


<script>
import Palette from "./Palette.vue";
import Paper from "./Paper.vue";
import { ref, nextTick } from "vue";
import html2canvas from "html2canvas";
import { v4 as uuidv4 } from "uuid";

export default {
  components: {
    Palette,
    Paper,
  },
  setup() {
    const droppedItems = ref([]);
    const paperRef = ref(null);

    const updateDroppedItems = (updatedItems) => {
      droppedItems.value = updatedItems;
    };

    const addItemToPaper = (item) => {
      droppedItems.value.push({ ...item, id: uuidv4(), x: 100, y: 100 });
    };

    const importLayoutFromJson = async (event) => {
      const file = event.target.files[0];
      if (!file) {
        return;
      }

      const reader = new FileReader();
      reader.onload = (e) => {
        try {
          const importedData = JSON.parse(e.target.result);
          if (Array.isArray(importedData)) {
            droppedItems.value = importedData;
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
      let layoutData = paperRef.value
        ? paperRef.value.serializeCurrentState()
        : JSON.stringify(droppedItems.value, null, 2);
      await paperRef.value.ensureImagesLoaded();
      if (droppedItems.value.length === 0) {
        // Get the layout data directly from Paper.vue
        layoutData = paperRef.value.serializeCurrentState();
      } else {
        // Use the existing droppedItems data
        layoutData = JSON.stringify(droppedItems.value, null, 2);
      }

      const blob = new Blob([layoutData], { type: "application/json" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "layout.json";
      link.click();
      URL.revokeObjectURL(link.href);
    };

    const exportAsJPG = async () => {
      // Ensures that Vue has updated the DOM before capturing the image
      await nextTick();

      // Call the method from Paper.vue to ensure all images have loaded
      await paperRef.value.ensureImagesLoaded();

      // Use html2canvas to capture the content
      const paperElement = paperRef.value.$el;
      html2canvas(paperElement, { allowTaint: true, useCORS: true })
        .then((canvas) => {
          const dataURL = canvas.toDataURL("image/jpeg");
          const link = document.createElement("a");
          link.href = dataURL;
          link.download = "paper-export.jpg";
          document.body.appendChild(link); // Append the link to the body
          link.click();
          document.body.removeChild(link); // Clean up and remove the link
        })
        .catch((error) => {
          console.error("Error exporting as JPG:", error);
        });
    };

    return {
      droppedItems,
      addItemToPaper,
      exportAsJPG,
      paperRef,
      exportLayoutToJson,
      updateDroppedItems,
      importLayoutFromJson,
    };
  },
};
</script>

<style>
.app-container {
  display: grid;
  grid-template-columns: auto 1fr auto;
  height: 100vh;
  gap: 1em;
  padding: 1em;
}

.palette-section {
  grid-column: 1;
  background-color: #f0f0f0;
  overflow-y: auto;
  padding: 1em;
}

.paper-section {
  grid-column: 2;
  display: flex;
  justify-content: center;
  align-items: center;
}

.export-button,
.export-json-button,
.import-json-button {
  display: block;
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
  margin-bottom: 10px; /* Space between buttons */
}

/* Additional container for side buttons */
.side-buttons {
  grid-column: 3;
  display: flex;
  flex-direction: column;
  align-items: flex-start; /* Align buttons to the start of the column */
}

</style>


