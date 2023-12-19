<template>
  <div class="paper" @dragover.prevent @drop="onDrop">
    <div
      v-for="item in droppedItems"
      :key="item.id"
      class="dropped-item"
      :style="{ left: item.x + 'px', top: item.y + 'px' }"
      draggable="true"
      @dragstart="startDrag(item, $event)"
    >
      {{ item.name }}
      <div class="popup" :class="{ 'popup-visible': item.showPopup }">
        <button @click="deleteItem(item)">Delete</button>
        <div class="position-adjust">
          <label for="adjustX">Adjust X:</label>
          <input
            id="adjustX"
            v-model="item.adjustX"
            type="number"
            placeholder="X (px)"
          />
          <label for="adjustY">Adjust Y:</label>
          <input
            id="adjustY"
            v-model="item.adjustY"
            type="number"
            placeholder="Y (px)"
          />
          <button @click="adjustPosition(item)">Confirm</button>
        </div>
      </div>
    </div>
    <button @click="exportAsJPG">Export as JPG</button>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import html2canvas from 'html2canvas';

export default {
  props: {
    initialX: Number,
    initialY: Number,
  },
  setup(props) {
    const droppedItems = ref([]);

    const onDrop = (event) => {
      event.preventDefault();
      const itemData = event.dataTransfer.getData('application/json');
      const item = JSON.parse(itemData);

      const existingItem = droppedItems.value.find((existing) => existing.id === item.id);

      if (!existingItem) {
        const rect = event.target.getBoundingClientRect();
        item.x = event.clientX - rect.left + window.scrollX;
        item.y = event.clientY - rect.top + window.scrollY;
        item.adjustX = 0;
        item.adjustY = 0;

        droppedItems.value.push(item);
      }
    };

    const startDrag = (item, event) => {
      event.dataTransfer.setData('application/json', JSON.stringify(item));
    };

    const deleteItem = (item) => {
      const index = droppedItems.value.findIndex((existing) => existing.id === item.id);
      if (index !== -1) {
        droppedItems.value.splice(index, 1);
      }
    };

    const adjustPosition = (item) => {
      item.x += item.adjustX;
      item.y += item.adjustY;
      item.showPopup = false;
    };

    const exportAsJPG = () => {
      const paperElement = document.querySelector('.paper');

      html2canvas(paperElement).then((canvas) => {
        // Convert the canvas to a data URL with JPG format
        const dataURL = canvas.toDataURL('image/jpeg');

        // Create a link element to trigger the download
        const link = document.createElement('a');
        link.href = dataURL;
        link.download = 'paper-export.jpg';
        link.click();
      });
    };

    onMounted(() => {
      document.addEventListener('click', (event) => {
        if (!event.target.closest('.paper') && !event.target.closest('.popup')) {
          droppedItems.value.forEach((item) => {
            item.showPopup = false;
          });
        }
      });
    });

    if (props.initialX !== undefined && props.initialY !== undefined) {
      const item = {
        id: 'unique-id',
        name: 'Item',
        x: props.initialX,
        y: props.initialY,
        adjustX: 0,
        adjustY: 0,
        showPopup: false,
      };
      droppedItems.value.push(item);
    }

    return { droppedItems, onDrop, startDrag, deleteItem, adjustPosition, exportAsJPG };
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
</style>
