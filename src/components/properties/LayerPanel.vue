<template>
  <div class="layer-panel">
    <h3>Layers</h3>
    <div
      v-for="(item, index) in items"
      :key="item.id"
      draggable="true"
      @dragstart="dragStart(index, $event)"
      @dragover.prevent
      @drop="drop(index, $event)"
      class="layer-item"
    >
      {{ item.name }}
    </div>
  </div>
</template>

<script>
import { ref, watch } from "vue";

export default {
  props: {
    items: Array,
  },
  emits: ["updateItemsOrder"],
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

    return {
      items,
      dragStart,
      drop,
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
}
</style>
