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
  emits: ["addItemToPaper"],
  setup(_, { emit }) {
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
  background-color: #f9f9f9;
  padding: 10px;
  border-radius: 4px;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  grid-gap: 10px;
}

.item {
  cursor: grab;
  padding: 10px;
  border: 1px solid #ddd;
  background-color: white;
  color: #333;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.palette-image {
  max-width: 100%;
  max-height: 50px;
  margin-bottom: 10px;
}
</style>
