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
      <h2>Adjust Page Size</h2>
      <div class="size-options">
        <button @click="adjustSize('A4')">A4</button>
        <button @click="adjustSize('A3')">A3</button>
      </div>
      <div class="custom-size">
        <label>Custom Size</label>
        <label>Width:</label>
        <input
          type="number"
          v-model.number="customWidth"
        />
        <label>Height:</label>
        <input
          type="number"
          v-model.number="customHeight"
        />
        <button @click="adjustSize('custom')">Apply</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  visible: Boolean,
});
const emit = defineEmits(["adjust", "close"]);

const customWidth = ref(210);
const customHeight = ref(297);

const adjustSize = (size) => {
  let dimensions;
  if (size === "A4") {
    dimensions = { width: 210, height: 297 };
  } else if (size === "A3") {
    dimensions = { width: 297, height: 420 };
  } else {
    dimensions = { width: customWidth.value, height: customHeight.value };
  }
  emit("adjust", dimensions);
  emit("close");
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

.size-options button {
  margin: 5px;
}

.custom-size {
  margin-top: 20px;
}

.custom-size label {
  display: block;
  margin-bottom: 5px;
}

.custom-size input {
  width: 100px;
  margin-bottom: 10px;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  cursor: pointer;
}
</style>
