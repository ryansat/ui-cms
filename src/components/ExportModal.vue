<template>
  <div
    v-if="visible"
    class="modal"
  >
    <div class="modal-content">
      <span
        class="close"
        @click="$emit('close')"
        >&times;</span
      >
      <h2>Select Export Format</h2>
      <div class="export-options">
        <button @click="exportAs('jpeg')">JPEG</button>
        <button @click="exportAs('png')">PNG</button>
        <button @click="exportAs('pdf')">PDF</button>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  visible: Boolean,
});
const emit = defineEmits(["export", "close"]);

const exportAs = (format) => {
  emit("export", format);
};
</script>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* Ensure modal is on top of other content */
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 4px;
  text-align: center;
}

.modal-content h2 {
  color: black; /* Set font color to black */
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
}
</style>
