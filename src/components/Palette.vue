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
      } catch (error) {
        console.error("Fetch error:", error);
        // Fallback to default items or handle the error as needed
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
  width: 200px;
  height: 100vh;
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
  max-height: 50px; /* Adjust as needed */
  margin-right: 10px;
}
</style>
