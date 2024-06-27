<template>
  <div class="properties">
    <div v-if="selectedItem">
      <div class="property">
        <label>Name:</label>
        <input
          type="text"
          v-model="itemName"
          @dblclick="enableEditing"
          @blur="disableEditing"
          @keyup.enter="disableEditing"
          :readonly="!editing"
        />
      </div>
      <div class="property">
        <label>Label:</label>
        <input
          type="text"
          v-model="itemLabel"
          @input="updateProperty('label', itemLabel)"
        />
      </div>
      <div class="property">
        <label>X:</label>
        <input
          type="number"
          v-model.number="selectedItem.x"
          @input="updateProperty('x', selectedItem.x)"
        />
      </div>
      <div class="property">
        <label>Y:</label>
        <input
          type="number"
          v-model.number="selectedItem.y"
          @input="updateProperty('y', selectedItem.y)"
        />
      </div>
      <div class="property">
        <label>Width:</label>
        <input
          type="number"
          v-model.number="selectedItem.width"
          @input="updateProperty('width', selectedItem.width)"
        />
      </div>
      <div class="property">
        <label>Height:</label>
        <input
          type="number"
          v-model.number="selectedItem.height"
          @input="updateProperty('height', selectedItem.height)"
        />
      </div>
      <div class="property">
        <label>Image URL:</label>
        <input
          type="text"
          v-model="selectedItem.imageUrl"
          @input="updateProperty('imageUrl', selectedItem.imageUrl)"
        />
        <button @click="showUploadModal">Upload Image</button>
      </div>
      <div class="current-size">
        <p>Current Size:</p>
        <p>Width: {{ selectedItem.width }}px</p>
        <p>Height: {{ selectedItem.height }}px</p>
      </div>
    </div>
    <div class="layers">
      <h3>Layers</h3>
      <ul>
        <li
          v-for="(item, index) in droppedItems"
          :key="item.id"
          :draggable="true"
          @dragstart="onDragStart(index, $event)"
          @dragover.prevent
          @drop="onDrop(index)"
          :class="{ selected: selectedItem && selectedItem.id === item.id }"
        >
          <input
            v-model="item.name"
            @change="renameItem(item, item.name)"
          />
          <button @click="deleteItem(item.id)">Delete</button>
        </li>
      </ul>
    </div>
    <ImageUploadModal
      :visible="isUploadModalVisible"
      @upload="handleUpload"
      @close="hideUploadModal"
    />
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits, watch } from "vue";
import ImageUploadModal from "../common/ImageUploadModal.vue";

const props = defineProps(["selectedItem", "droppedItems"]);
const emit = defineEmits([
  "updateProperty",
  "updateItemsOrder",
  "deleteItem",
  "renameItem",
]);

const itemName = ref("");
const itemLabel = ref("");
const editing = ref(false);
const draggedIndex = ref(null);
const isUploadModalVisible = ref(false);

const updateProperty = (property, value) => {
  emit("updateProperty", property, value);
};

const enableEditing = () => {
  editing.value = true;
};

const disableEditing = () => {
  editing.value = false;
  emit("updateProperty", "name", itemName.value);
};

const editLabel = (item) => {
  item.editingLabel = true;
  emit("updateProperty", "name", item.name);
};

const onDragStart = (index, event) => {
  draggedIndex.value = index;
};

const onDrop = (index) => {
  const draggedItem = props.droppedItems.splice(draggedIndex.value, 1)[0];
  props.droppedItems.splice(index, 0, draggedItem);
  draggedIndex.value = null;
  emit("updateItemsOrder", props.droppedItems);
};

const showUploadModal = () => {
  isUploadModalVisible.value = true;
};

const hideUploadModal = () => {
  isUploadModalVisible.value = false;
};

const handleUpload = (imageUrl) => {
  if (props.selectedItem) {
    props.selectedItem.imageUrl = imageUrl;
    updateProperty("imageUrl", imageUrl);
  }
  hideUploadModal();
};

const renameItem = (item, newName) => {
  item.name = newName;
  emit("renameItem", item);
};

const deleteItem = (itemId) => {
  emit("deleteItem", itemId);
};

watch(
  () => props.selectedItem,
  (newItem) => {
    if (newItem) {
      itemName.value = newItem.name || "";
      itemLabel.value = newItem.label || "";
      emit("updateProperty", "x", newItem.x);
      emit("updateProperty", "y", newItem.y);
      emit("updateProperty", "width", newItem.width);
      emit("updateProperty", "height", newItem.height);
    } else {
      itemName.value = "";
      itemLabel.value = "";
    }
  },
  { immediate: true }
);
</script>

<style scoped>
.properties {
  width: 150%;
  background: #f9f9f9;
  padding: 10px;
  border-radius: 4px;
  display: grid;
  grid-template-columns: auto 1fr;
  grid-gap: 10px;
}

.property {
  display: contents;
}

.property label {
  font-weight: bold;
  align-self: center;
}

.property input {
  width: 100%;
  padding: 5px;
  box-sizing: border-box;
}

.current-size {
  grid-column: span 2;
  margin-top: 10px;
}

.current-size p {
  margin: 0;
  font-size: 14px;
}

.layers {
  grid-column: span 2;
  margin-top: 20px; /* Adds a gap between properties and layers */
}

.layers h3 {
  margin-top: 0;
  font-size: 18px;
}

.layers ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.layers li {
  padding: 5px;
  cursor: pointer;
  display: flex;
  align-items: center;
}

.layers li input {
  flex-grow: 1;
  margin-right: 10px;
}

.layers li button {
  background-color: red;
  color: white;
  border: none;
  cursor: pointer;
}

.layers li.selected {
  background-color: #ddd;
}
</style>
