<template>
  <div class="palette">
    <div
      v-for="item in items"
      :key="item.id"
      draggable="true"
      @dragstart="startDrag(item, $event)"
      class="item"
    >
      <img
        v-if="item.imageUrl"
        :src="item.imageUrl"
        alt="Image"
        class="palette-image"
      />
      <span v-else>{{ item.name }}</span>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  setup() {
    const items = ref([]);
    const fetchItems = async () => {
      try {
        const response = await fetch("/mockData.json");
        if (!response.ok) throw new Error("Network response was not ok.");
        const data = await response.json();
        items.value = data;
        
        // Fetch table data
        const tableResponse = await fetch("/tableData.json");
        if (!tableResponse.ok) throw new Error("Network response was not ok.");
        const tableData = await tableResponse.json();
        items.value.push({ ...tableData, name: "Table", type: "table" });
      } catch (error) {
        console.error("Fetch error:", error);
      }
    };

    onMounted(fetchItems);

    const startDrag = (item, event) => {
      event.dataTransfer.setData("application/json", JSON.stringify(item));
    };

    return { items, startDrag };
  },
};
</script>

<style scoped>
.palette {
  width: 100%;
  height: 100%;
  background-color: #f0f0f0;
}

.item {
  cursor: grab;
  margin: 10px;
  padding: 10px;
  border: 1px solid #ddd;
  background-color: white;
  color: #333;
  display: flex;
  align-items: center;
}

.palette-image {
  max-width: 100%;
  max-height: 50px;
  margin-right: 10px;
}
</style>