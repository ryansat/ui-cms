<template>
  <div class="palette">
    <div
      v-if="loading"
      class="spinner"
    ></div>
    <div v-else>
      <div
        v-for="item in items"
        :key="item.id"
        draggable="true"
        @dragstart="startDrag(item, $event)"
        class="item"
      >
        <img
          v-if="item.type === 'table'"
          src="/tabledata.svg"
          alt="Table Data"
          class="palette-image"
        />
        <img
          v-else-if="item.type === 'image'"
          src="/imageupload.svg"
          alt="Image Upload"
          class="palette-image"
        />
        <img
          v-else
          src="/label.svg"
          alt="Label"
          class="palette-image"
        />
        <span>{{ item.name }}</span>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  emits: ["addItemToPaper"],
  setup(_, { emit }) {
    const items = ref([]);
    const loading = ref(true);

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
      } finally {
        loading.value = false;
      }
    };

    onMounted(fetchItems);

    const startDrag = (item, event) => {
      event.dataTransfer.setData("application/json", JSON.stringify(item));
    };

    return { items, loading, startDrag };
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
  position: relative;
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

.spinner {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 50px;
  height: 50px;
  margin: -25px 0 0 -25px;
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #333;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
