<template>
  <div class="app-container">
    <Palette class="palette-section" @addItemToPaper="addItemToPaper" />
    <div class="paper-section">
      <Paper :droppedItems="droppedItems" ref="paperRef" />
    </div>
    <button class="export-button" @click="exportAsJPG">Export as JPG</button>
  </div>
</template>

<script>
import Palette from './Palette.vue';
import Paper from './Paper.vue';
import { ref, nextTick } from 'vue'; 
import html2canvas from 'html2canvas';

export default {
  components: {
    Palette,
    Paper,
  },
  setup() {
    const droppedItems = ref([]);
    const paperRef = ref(null);

    const addItemToPaper = (item) => {
      droppedItems.value.push(item);
    };

    const exportAsJPG = async () => {
  // Ensures that Vue has updated the DOM before capturing the image
  await nextTick();

  // Call the method from Paper.vue to ensure all images have loaded
  await paperRef.value.ensureImagesLoaded();

  // Use html2canvas to capture the content
  const paperElement = paperRef.value.$el;
  html2canvas(paperElement, { allowTaint: true, useCORS: true }).then((canvas) => {
    const dataURL = canvas.toDataURL('image/jpeg');
    const link = document.createElement('a');
    link.href = dataURL;
    link.download = 'paper-export.jpg';
    document.body.appendChild(link); // Append the link to the body
    link.click();
    document.body.removeChild(link); // Clean up and remove the link
  }).catch((error) => {
    console.error('Error exporting as JPG:', error);
  });
};


    return { droppedItems, addItemToPaper, exportAsJPG, paperRef };
  },
};
</script>

<style>
.app-container {
  display: flex;
  height: 100vh;
  position: relative;
}

.export-button {
  position: absolute;
  top: -30px;
  right: -200px;
  z-index: 2;
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 10px 20px;
  cursor: pointer;
}

.palette-section {
  width: 200px;
  background-color: #f0f0f0;
  overflow-y: auto;
}

.paper-section {
  flex-grow: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
