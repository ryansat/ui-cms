<template>
  <div
    class="paper"
    @dragover.prevent
    @drop="onDrop"
  >
    <div
      v-for="(item, index) in droppedItems"
      :key="item.id"
      class="draggable-item"
      :style="{
        top: item.y + 'px',
        left: item.x + 'px',
        width: item.width + 'px',
        height: item.height + 'px',
      }"
      @mousedown="startDrag(item, $event)"
      @click="selectItem(item)"
    >
      <img
        v-if="item.imageUrl"
        :src="item.imageUrl"
        :alt="item.label || item.name"
        class="paper-image"
      />
      <span v-else>{{ item.label || item.name }}</span>
      <div
        v-if="item === selectedItem"
        class="resize-handles"
      >
        <div
          class="resize-handle top-left"
          @mousedown.stop="startResize(item, 'top-left', $event)"
        ></div>
        <div
          class="resize-handle top-right"
          @mousedown.stop="startResize(item, 'top-right', $event)"
        ></div>
        <div
          class="resize-handle bottom-left"
          @mousedown.stop="startResize(item, 'bottom-left', $event)"
        ></div>
        <div
          class="resize-handle bottom-right"
          @mousedown.stop="startResize(item, 'bottom-right', $event)"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, defineEmits, watch } from "vue";
import { v4 as uuidv4 } from "uuid";

const props = defineProps(["droppedItems"]);
const emit = defineEmits(["update-items", "selectItem", "addItemToPaper"]);

const selectedItem = ref(null);
const dragStart = ref({ x: 0, y: 0 });
const resizing = ref(false);
const resizingHandle = ref("");
const dragging = ref(false);

const startDrag = (item, event) => {
  selectedItem.value = item;
  dragStart.value = { x: event.clientX - item.x, y: event.clientY - item.y };
  dragging.value = true;
  document.addEventListener("mousemove", onDrag);
  document.addEventListener("mouseup", stopDrag);
};

const onDrag = (event) => {
  if (selectedItem.value && !resizing.value) {
    selectedItem.value.x = event.clientX - dragStart.value.x;
    selectedItem.value.y = event.clientY - dragStart.value.y;
    emit("update-items", props.droppedItems);
  }
};

const stopDrag = () => {
  dragging.value = false;
  document.removeEventListener("mousemove", onDrag);
  document.removeEventListener("mouseup", stopDrag);
};

const selectItem = (item) => {
  selectedItem.value = item;
  emit("selectItem", item);
};

const startResize = (item, handle, event) => {
  resizing.value = true;
  resizingHandle.value = handle;
  dragStart.value = {
    x: event.clientX,
    y: event.clientY,
    width: item.width,
    height: item.height,
  };
  document.addEventListener("mousemove", onResize);
  document.addEventListener("mouseup", stopResize);
};

const onResize = (event) => {
  if (selectedItem.value && resizing.value) {
    const deltaX = event.clientX - dragStart.value.x;
    const deltaY = event.clientY - dragStart.value.y;
    if (resizingHandle.value.includes("right")) {
      selectedItem.value.width = dragStart.value.width + deltaX;
    }
    if (resizingHandle.value.includes("left")) {
      selectedItem.value.width = dragStart.value.width - deltaX;
      selectedItem.value.x = event.clientX;
    }
    if (resizingHandle.value.includes("bottom")) {
      selectedItem.value.height = dragStart.value.height + deltaY;
    }
    if (resizingHandle.value.includes("top")) {
      selectedItem.value.height = dragStart.value.height - deltaY;
      selectedItem.value.y = event.clientY;
    }
    emit("update-items", props.droppedItems);
  }
};

const stopResize = () => {
  resizing.value = false;
  resizingHandle.value = "";
  document.removeEventListener("mousemove", onResize);
  document.removeEventListener("mouseup", stopResize);
};

const onDrop = (event) => {
  const data = event.dataTransfer.getData("application/json");
  if (data) {
    const item = JSON.parse(data);
    const rect = event.target.getBoundingClientRect();
    item.id = uuidv4();
    item.x = event.clientX - rect.left - 50; // Adjust position
    item.y = event.clientY - rect.top - 50; // Adjust position
    item.width = item.width || 100;
    item.height = item.height || 100;
    emit("addItemToPaper", item);
  }
};

watch(
  () => props.droppedItems,
  (newItems) => {
    if (!newItems.includes(selectedItem.value)) {
      selectedItem.value = null;
    }
  },
  { deep: true }
);
</script>

<style scoped>
.paper {
  position: relative;
  width: 100%;
  height: 100%;
}

.draggable-item {
  position: absolute;
  cursor: grab;
  border: 1px solid transparent;
}

.draggable-item.selected {
  border: 1px dashed red;
}

.resize-handles {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
}

.resize-handle {
  position: absolute;
  width: 10px;
  height: 10px;
  background: red;
  z-index: 1;
}

.resize-handle.top-left {
  top: -5px;
  left: -5px;
  cursor: nwse-resize;
}

.resize-handle.top-right {
  top: -5px;
  right: -5px;
  cursor: nesw-resize;
}

.resize-handle.bottom-left {
  bottom: -5px;
  left: -5px;
  cursor: nesw-resize;
}

.resize-handle.bottom-right {
  bottom: -5px;
  right: -5px;
  cursor: nwse-resize;
}

.paper-image {
  width: 100%;
  height: 100%;
}
</style>
