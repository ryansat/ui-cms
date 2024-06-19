<template>
  <div class="properties" v-if="selectedItem">
    <div>
      <label>X:</label>
      <input type="number" v-model.number="selectedItem.x" @input="updateProperty('x', selectedItem.x)" />
    </div>
    <div>
      <label>Y:</label>
      <input type="number" v-model.number="selectedItem.y" @input="updateProperty('y', selectedItem.y)" />
    </div>
    <div>
      <label>Width:</label>
      <input type="number" v-model.number="selectedItem.width" @input="updateProperty('width', selectedItem.width)" />
    </div>
    <div>
      <label>Height:</label>
      <input type="number" v-model.number="selectedItem.height" @input="updateProperty('height', selectedItem.height)" />
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits, watch } from 'vue'

const props = defineProps(['selectedItem'])
const emit = defineEmits(['updateProperty'])

const updateProperty = (property, value) => {
  emit('updateProperty', property, value)
}

watch(props.selectedItem, (newItem) => {
  if (newItem) {
    emit('updateProperty', 'x', newItem.x)
    emit('updateProperty', 'y', newItem.y)
    emit('updateProperty', 'width', newItem.width)
    emit('updateProperty', 'height', newItem.height)
  }
})
</script>

<style scoped>
.properties {
  width: 100%;
  /* background: #f9f9f9; */
  padding: 10px;
  border-radius: 4px;
}

.properties div {
  margin-bottom: 10px;
}

.properties label {
  display: block;
  font-weight: bold;
}

.properties input {
  width: 100%;
  padding: 5px;
  margin-top: 5px;
  box-sizing: border-box;
}
</style>