<template>
  <div
    v-if="visible"
    class="modal-overlay"
    @click.self="closeModal"
  >
    <div class="modal-content">
      <h3>Adjust Page Size</h3>
      <label for="preset">Select preset:</label>
      <select
        id="preset"
        v-model="selectedPreset"
        @change="applyPreset"
      >
        <option value="A4">A4 (210 x 297 mm)</option>
        <option value="A3">A3 (297 x 420 mm)</option>
        <option value="custom">Custom</option>
      </select>
      <div v-if="selectedPreset === 'custom'">
        <label for="width">Width (px):</label>
        <input
          type="number"
          id="width"
          v-model.number="customWidth"
        />
        <label for="height">Height (px):</label>
        <input
          type="number"
          id="height"
          v-model.number="customHeight"
        />
      </div>
      <button @click="adjustSize">Apply</button>
      <button @click="closeModal">Close</button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const props = defineProps({
  visible: Boolean,
});
const emit = defineEmits(["adjust", "close"]);

const selectedPreset = ref("A4");
const customWidth = ref(210);
const customHeight = ref(297);

const applyPreset = () => {
  if (selectedPreset.value === "A4") {
    customWidth.value = 210;
    customHeight.value = 297;
  } else if (selectedPreset.value === "A3") {
    customWidth.value = 297;
    customHeight.value = 420;
  }
};

const adjustSize = () => {
  emit("adjust", { width: customWidth.value, height: customHeight.value });
  closeModal();
};

const closeModal = () => {
  emit("close");
};

watch(
  () => props.visible,
  (newVal) => {
    if (newVal) {
      applyPreset();
    }
  }
);
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
  z-index: 1000;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  z-index: 1001;
}
</style>
