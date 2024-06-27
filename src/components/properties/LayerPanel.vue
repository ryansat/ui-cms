<template>
  <div class="layer-panel">
    <h3>Layers</h3>
    <ul>
      <li
        v-for="(item, index) in items"
        :key="item.id"
        draggable="true"
        @dragstart="dragStart(index, $event)"
        @dragover.prevent
        @drop="drop(index, $event)"
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
</template>

<script>
import { ref, watch } from "vue";

export default {
  props: {
    items: Array,
    selectedItem: Object,
  },
  emits: ["updateItemsOrder", "renameItem", "deleteItem"],
  setup(props, { emit }) {
    const items = ref([...props.items]);
    let dragSrcIndex = ref(null);

    watch(props.items, (newItems) => {
      items.value = [...newItems];
    });

    const dragStart = (index, event) => {
      dragSrcIndex.value = index;
      event.dataTransfer.effectAllowed = "move";
    };

    const drop = (index) => {
      if (dragSrcIndex.value === null) return;

      const srcItem = items.value[dragSrcIndex.value];
      items.value.splice(dragSrcIndex.value, 1);
      items.value.splice(index, 0, srcItem);

      emit("updateItemsOrder", items.value);
      dragSrcIndex.value = null;
    };

    const renameItem = (item, newName) => {
      item.name = newName;
      emit("renameItem", item);
    };

    const deleteItem = (itemId) => {
      emit("deleteItem", itemId);
    };

    return {
      items,
      dragStart,
      drop,
      renameItem,
      deleteItem,
    };
  },
};
</script>

<style scoped>
.layer-panel {
  margin-top: 20px;
}

.layer-item {
  padding: 10px;
  margin: 5px 0;
  background-color: #fff;
  border: 1px solid #ccc;
  cursor: grab;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.layer-item input {
  flex-grow: 1;
  margin-right: 10px;
}

.layer-item button {
  background-color: red;
  color: white;
  border: none;
  cursor: pointer;
}
</style>
