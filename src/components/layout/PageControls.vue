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
    <button @click="showExportModal">Export</button>
    <button @click="showSizeAdjustmentModal">Adjust Page Size</button>
    <ExportModal
      :visible="isExportModalVisible"
      @export="handleExport"
      @close="hideExportModal"
    />
    <SizeAdjustmentModal
      :visible="isSizeAdjustmentModalVisible"
      @adjust="handleSizeAdjustment"
      @close="hideSizeAdjustmentModal"
    />
  </div>
</template>

<script setup>
import { ref } from "vue";
import ExportModal from "../common/ExportModal.vue";
import SizeAdjustmentModal from "../layout/SizeAdjustmentModal.vue";
import html2canvas from "html2canvas";
import jsPDF from "jspdf";

const props = defineProps({
  pages: Array,
  currentPageIndex: Number,
});
const emit = defineEmits([
  "updatePages",
  "updateCurrentPageIndex",
  "adjustPageSize",
]);

const jumpPageIndex = ref(1);
const pages = ref(props.pages);
const currentPageIndex = ref(props.currentPageIndex);
const isExportModalVisible = ref(false);
const isSizeAdjustmentModalVisible = ref(false);

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
  if (jumpPageIndex.value > 0 && jumpPageIndex.value <= pages.value.length) {
    currentPageIndex.value = jumpPageIndex.value - 1;
    emit("updateCurrentPageIndex", currentPageIndex.value);
  }
};

const showExportModal = () => {
  isExportModalVisible.value = true;
};

const hideExportModal = () => {
  isExportModalVisible.value = false;
};

const showSizeAdjustmentModal = () => {
  isSizeAdjustmentModalVisible.value = true;
};

const hideSizeAdjustmentModal = () => {
  isSizeAdjustmentModalVisible.value = false;
};

const handleExport = async (format) => {
  const paperElement = document.querySelector(".paper");

  if (format === "jpeg" || format === "png") {
    const canvas = await html2canvas(paperElement, {
      allowTaint: true,
      useCORS: true,
    });
    const dataURL = canvas.toDataURL(`image/${format}`);
    const link = document.createElement("a");
    link.href = dataURL;
    link.download = `export.${format}`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  } else if (format === "pdf") {
    const canvas = await html2canvas(paperElement, {
      allowTaint: true,
      useCORS: true,
    });
    const dataURL = canvas.toDataURL("image/jpeg");
    const pdf = new jsPDF();
    pdf.addImage(dataURL, "JPEG", 0, 0, 210, 297);
    pdf.save("export.pdf");
  }
};

const handleSizeAdjustment = (size) => {
  emit("adjustPageSize", size);
  hideSizeAdjustmentModal();
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

.page-controls button {
  margin: 5px; /* Add margin between buttons */
}
</style>
