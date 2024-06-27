<template>
  <div
    v-if="visible"
    class="modal-overlay"
    @click.self="closeModal"
  >
    <div class="modal-content">
      <h3>Upload Image</h3>
      <input
        type="file"
        @change="handleFileUpload"
      />
      <button @click="closeModal">Close</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  visible: Boolean,
});
const emit = defineEmits(["upload", "close"]);

const closeModal = () => {
  emit("close");
};

const handleFileUpload = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      const imageUrl = e.target.result;
      emit("upload", imageUrl);
    };
    reader.readAsDataURL(file);
  }
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* Ensure it is on top */
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1001; /* Ensure it is on top */
}
</style>
