<template>
  <div class="app-container">
    <Palette class="palette-section" @addItemToPaper="addItemToPaper" />
    <div class="paper-section">
      <Paper :droppedItems="droppedItems" ref="paperRef" />
      <!-- <button class="export-button" @click="exportToPDF">Export to PDF</button> -->
     
    </div>
    <button class="export-button" @click="exportAsJPG">Export as JPG</button>
  </div>
</template>

<script>
import Palette from './Palette.vue';
import Paper from './Paper.vue';
import html2pdf from 'html2pdf.js';
import { ref } from 'vue';
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

    const exportToPDF = async () => {
      const paperElement = document.querySelector('.paper');

      const pdfOptions = {
        margin: 10,
        filename: 'exported-paper.pdf',
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2 },
        jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' },
      };

      try {
        const pdf = await html2pdf().from(paperElement).set(pdfOptions).outputPdf();

        const blob = new Blob([pdf], { type: 'application/pdf' });
        const url = window.URL.createObjectURL(blob);
        window.open(url, '_blank');
      } catch (error) {
        console.error('PDF export error:', error);
      }
    };

    const exportAsJPG = async () => {
      const paperComponent = paperRef.value;

      if (paperComponent) {
        const canvas = await html2canvas(paperComponent.$el);
        const dataURL = canvas.toDataURL('image/jpeg');

        const link = document.createElement('a');
        link.href = dataURL;
        link.download = 'paper-export.jpg';
        link.click();
      }
    };

    return { droppedItems, addItemToPaper, exportToPDF, exportAsJPG, paperRef };
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
