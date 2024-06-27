<template>
  <div
    class="paper"
    @dragover.prevent
    @drop="onDrop"
    @mousedown="deselectAll"
    :style="{
      width: paperSize.width + 'mm',
      height: paperSize.height + 'mm',
    }"
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
        zIndex: index,
      }"
      @mousedown.stop="startDrag(item, $event)"
      @click.stop="selectItem(item)"
      @dragover.prevent="onDragOverItem(item, $event)"
      @drop="onDropItem(item, $event)"
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

const props = defineProps({
  droppedItems: {
    type: Array,
    default: () => [],
  },
  paperSize: {
    type: Object,
    default: () => ({ width: 210, height: 297 }),
  },
});
const emit = defineEmits([
  "update-items",
  "selectItem",
  "addItemToPaper",
  "deleteItem",
]);

const selectedItem = ref(null);
const dragStart = ref({ x: 0, y: 0 });
const resizing = ref(false);
const resizingHandle = ref("");
const initialAspectRatio = ref(null);
const isShiftPressed = ref(false);

const startDrag = (item, event) => {
  selectedItem.value = item;
  dragStart.value = { x: event.clientX - item.x, y: event.clientY - item.y };
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
  initialAspectRatio.value = item.width / item.height;
  isShiftPressed.value = event.shiftKey;
  document.addEventListener("mousemove", onResize);
  document.addEventListener("mouseup", stopResize);
};

const onResize = (event) => {
  if (selectedItem.value && resizing.value) {
    const deltaX = event.clientX - dragStart.value.x;
    const deltaY = event.clientY - dragStart.value.y;

    if (resizingHandle.value.includes("right")) {
      selectedItem.value.width = dragStart.value.width + deltaX;
      if (isShiftPressed.value) {
        selectedItem.value.height =
          selectedItem.value.width / initialAspectRatio.value;
      }
    }
    if (resizingHandle.value.includes("left")) {
      selectedItem.value.width = dragStart.value.width - deltaX;
      selectedItem.value.x = event.clientX;
      if (isShiftPressed.value) {
        selectedItem.value.height =
          selectedItem.value.width / initialAspectRatio.value;
      }
    }
    if (resizingHandle.value.includes("bottom")) {
      selectedItem.value.height = dragStart.value.height + deltaY;
      if (isShiftPressed.value) {
        selectedItem.value.width =
          selectedItem.value.height * initialAspectRatio.value;
      }
    }
    if (resizingHandle.value.includes("top")) {
      selectedItem.value.height = dragStart.value.height - deltaY;
      selectedItem.value.y = event.clientY;
      if (isShiftPressed.value) {
        selectedItem.value.width =
          selectedItem.value.height * initialAspectRatio.value;
      }
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
    item.x = event.clientX - 50; // Adjust position
    item.y = event.clientY - 50; // Adjust position
    emit("addItemToPaper", item);
  }
};

const onDragOverItem = (item, event) => {
  event.preventDefault();
};

const onDropItem = (item, event) => {
  const file = event.dataTransfer.files[0];
  if (file && file.type.startsWith("image/")) {
    const reader = new FileReader();
    reader.onload = (e) => {
      item.imageUrl = e.target.result;
      emit("update-items", props.droppedItems);
    };
    reader.readAsDataURL(file);
  }
};

const deselectAll = () => {
  selectedItem.value = null;
  emit("selectItem", null);
};

const deleteItem = (itemId) => {
  const index = props.droppedItems.findIndex((item) => item.id === itemId);
  if (index !== -1) {
    props.droppedItems.splice(index, 1);
    emit("update-items", props.droppedItems);
    deselectAll();
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

watch(
  () => props.paperSize,
  (newSize) => {
    console.log("Paper size updated:", newSize);
  },
  { immediate: true, deep: true }
);
</script>

<style scoped>
.paper {
  position: relative;
  background-color: #fff;
  overflow: hidden;
  border: 1px solid #ccc;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
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
