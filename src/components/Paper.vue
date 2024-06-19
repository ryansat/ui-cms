<template>
  <div class="paper" @dragover.prevent="allowDrop" @drop="onDrop">
    <div
      v-for="item in localDroppedItems"
      :key="item.id"
      class="dropped-item"
      :style="{ left: item.x + 'px', top: item.y + 'px', width: item.width + 'px', height: item.height + 'px' }"
      draggable="true"
      @dragstart="startDrag(item, $event)"
      @click="selectItem(item)"
    >
      <template v-if="item.type === 'table'">
        <Table
          :data="item"
          :pageIndex="0"
          :itemsPerPage="10"
          :style="{ width: '100%', height: '100%' }"
        />
      </template>
      <template v-else-if="item.imageUrl">
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
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, watch } from "vue";
import { v4 as uuidv4 } from "uuid";
import Table from "./Table.vue";

export default {
  components: { Table },
  props: {
    droppedItems: Array,
  },
  emits: ["update-items", "selectItem"],
  setup(props, { emit }) {
    const localDroppedItems = ref([...props.droppedItems]);

    watch(
      () => props.droppedItems,
      (newItems) => {
        localDroppedItems.value = [...newItems];
      }
    );

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

    const selectItem = (item) => {
      emit("selectItem", item);
    };

    const onDrop = (event) => {
      event.preventDefault();
      const itemData = event.dataTransfer.getData("application/json");
      if (itemData) {
        const item = JSON.parse(itemData);
        const existingItemIndex = localDroppedItems.value.findIndex(
          (i) => i.id === item.id
        );

        if (existingItemIndex >= 0) {
          localDroppedItems.value[existingItemIndex].x =
            event.clientX - event.currentTarget.getBoundingClientRect().left;
          localDroppedItems.value[existingItemIndex].y =
            event.clientY - event.currentTarget.getBoundingClientRect().top;
        } else {
          item.id = uuidv4();
          item.x =
            event.clientX - event.currentTarget.getBoundingClientRect().left;
          item.y =
            event.clientY - event.currentTarget.getBoundingClientRect().top;
          item.width = 400;  // Default width for table
          item.height = 300; // Default height for table
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
    });

    return {
      localDroppedItems,
      onDrop,
      startDrag,
      deleteItem,
      selectItem,
      allowDrop,
    };
  },
};
</script>

<style scoped>
.paper {
  width: 100%;
  height: 100%;
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

.dropped-image {
  width: 200px;
  height: 200px;
  max-width: 100%;
  max-height: 100%;
  display: block;
}
</style>