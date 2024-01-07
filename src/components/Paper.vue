<template>
  <div class="paper" @dragover.prevent="allowDrop" @drop="onDrop">
    <div
      v-for="item in localDroppedItems"
      :key="item.id"
      class="dropped-item"
      :style="{ left: item.x + 'px', top: item.y + 'px' }"
      draggable="true"
      @dragstart="startDrag(item, $event)"
      @click="showPopup(item)"
    >
      <template v-if="item.imageUrl">
        <img
          :src="item.imageUrl"
          alt="Dropped Image"
          class="dropped-image"
          crossorigin="anonymous"
        />
      </template>
      <template v-else>
        {{ item.name }}
      </template>
      <div class="popup" :class="{ 'popup-visible': item.showPopup }">
        <button @click="deleteItem(item)">Delete</button>
        <div class="position-adjust">
          <label for="adjustX">X:</label>
          <input
            id="adjustX"
            v-model.number="item.x"
            type="number"
            placeholder="X (px)"
          />
          <label for="adjustY">Y:</label>
          <input
            id="adjustY"
            v-model.number="item.y"
            type="number"
            placeholder="Y (px)"
          />
          <button @click="hidePopup(item)">Close</button>
        </div>
      </div>
    </div>
    <button @click="exportAsJPG">Export as JPG</button>
  </div>
</template>

<script>
import { ref, onMounted, watch } from "vue";
import html2canvas from "html2canvas";
import { v4 as uuidv4 } from "uuid";

export default {
  props: {
    initialX: Number,
    initialY: Number,
    droppedItems: Array,
  },
  emits: ["update-items"],
  setup(props, { emit }) {
    const localDroppedItems = ref([...props.droppedItems]);

    watch(
      () => props.droppedItems,
      (newItems) => {
        localDroppedItems.value = [...newItems];
      }
    );

    const exportAsJPG = async () => {
      await ensureImagesLoaded();
      const paperElement = document.querySelector(".paper");
      html2canvas(paperElement).then((canvas) => {
        const dataURL = canvas.toDataURL("image/jpeg");
        const link = document.createElement("a");
        link.href = dataURL;
        link.download = "paper-export.jpg";
        link.click();
      });
    };

    const startDrag = (item, event) => {
      event.dataTransfer.setData("application/json", JSON.stringify(item));
    };

    const deleteItem = (item) => {
      const index = localDroppedItems.value.findIndex(
        (existing) => existing.id === item.id
      );
      if (index !== -1) {
        localDroppedItems.value.splice(index, 1);
        emit("update-items", localDroppedItems.value);
      }
    };

    const adjustPosition = (item) => {
      item.x += item.adjustX;
      item.y += item.adjustY;
      item.showPopup = false;
      emit("update-items", localDroppedItems.value);
    };

    const showPopup = (item) => {
      item.showPopup = true;
    };

    const hidePopup = (item) => {
      item.showPopup = false;
      emit("update-items", localDroppedItems.value); // Emit update after closing popup
    };

    //onDrop logic
    const onDrop = (event) => {
      event.preventDefault();
      const itemData = event.dataTransfer.getData("application/json");
      if (itemData) {
        const item = JSON.parse(itemData);
        const existingItemIndex = localDroppedItems.value.findIndex(
          (i) => i.id === item.id
        );

        if (existingItemIndex >= 0) {
          // Update position of the existing item
          localDroppedItems.value[existingItemIndex].x =
            event.clientX - event.currentTarget.getBoundingClientRect().left;
          localDroppedItems.value[existingItemIndex].y =
            event.clientY - event.currentTarget.getBoundingClientRect().top;
        } else {
          // Add new item
          item.x =
            event.clientX - event.currentTarget.getBoundingClientRect().left;
          item.y =
            event.clientY - event.currentTarget.getBoundingClientRect().top;
          localDroppedItems.value.push(item);
        }
        emit("update-items", localDroppedItems.value);
      }
    };

    const allowDrop = (event) => {
      event.preventDefault();
    };

    onMounted(() => {
      document.addEventListener("click", (event) => {
        if (
          !event.target.closest(".paper") &&
          !event.target.closest(".popup")
        ) {
          localDroppedItems.value.forEach((item) => {
            item.showPopup = false;
          });
        }
      });

      if (props.initialX !== undefined && props.initialY !== undefined) {
        const newItem = {
          id: uuidv4(),
          name: "New Item",
          x: props.initialX,
          y: props.initialY,
          adjustX: 0,
          adjustY: 0,
          showPopup: false,
        };
        localDroppedItems.value.push(newItem);
        this.$emit("update-items", localDroppedItems.value);
      }
    });

    const serializeCurrentState = () => {
      return JSON.stringify(localDroppedItems.value, null, 2);
    };

    const ensureImagesLoaded = () => {
      const images = document.querySelectorAll(".paper img");
      return Promise.all(
        Array.from(images).map((img) => {
          if (img.complete && img.naturalHeight !== 0) {
            return Promise.resolve();
          }
          return new Promise((resolve) => {
            img.onload = resolve;
            img.onerror = resolve;
          });
        })
      );
    };

    const loadLayoutFromJson = (layoutData) => {
      localDroppedItems.value = layoutData.map((item) => ({
        ...item,
        id: item.id || uuidv4(), // Ensure each item has a unique ID
        x: item.x || 0,
        y: item.y || 0,
        adjustX: item.adjustX || 0,
        adjustY: item.adjustY || 0,
        showPopup: item.showPopup || false,
      }));
    };

    return {
      localDroppedItems,
      onDrop,
      startDrag,
      deleteItem,
      showPopup,
      hidePopup,
      exportAsJPG,
      ensureImagesLoaded,
      serializeCurrentState,
      loadLayoutFromJson,
      allowDrop,
    };
  },
};
</script>

<style scoped>
.paper {
  width: 210mm;
  height: 297mm;
  margin: 20px auto;
  background-color: #fff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  position: relative;
  overflow: hidden;
}

.dropped-item {
  position: absolute;
  cursor: move;
  background-color: #f0f0f0;
  padding: 10px;
  border: 1px solid #ddd;
  color: #333;
  transition: background-color 0.3s ease;
  user-select: none;
}

.popup {
  position: absolute;
  background-color: #fff;
  border: 1px solid #ddd;
  padding: 5px;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  display: none;
}

.popup-visible {
  display: block !important;
}

.dropped-item:hover .popup {
  display: block;
}

/* Style for dropped images */
.dropped-image {
  width: 200px; /* Set the width to 200px */
  height: 200px; /* Set the height to 200px */
  max-width: 100%; /* Ensure it doesn't exceed the container's width */
  max-height: 100%; /* Ensure it doesn't exceed the container's height */
  display: block; /* Make the image a block element */
}
</style>
