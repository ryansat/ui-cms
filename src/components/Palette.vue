<template>
  <div class="palette">
    <div v-for="item in items" :key="item.id" draggable="true" @dragstart="startDrag(item, $event)" class="item">
      <template v-if="item.imageUrl">
        <img :src="item.imageUrl" alt="Image" class="image-item" />
      </template>
      <template v-else>
        {{ item.name }}
      </template>
    </div>
  </div>
</template>



<script>
import { ref, onMounted } from 'vue';

export default {
  setup() {
    const items = ref([]);

    onMounted(async () => {
      try {
        const response = await fetch('/mockData.json');
        if (!response.ok) throw new Error('Failed to fetch');
        items.value = await response.json();
      } catch (error) {
        console.error('Error fetching mock data:', error);
      }
    });

    const startDrag = (item, event) => {
      event.dataTransfer.setData('application/json', JSON.stringify(item));
    };

    return { items, startDrag };
  }
};
</script>

<style scoped>
.palette {
  width: 200px; /* Adjust width as needed */
  height: 100vh;
  position: fixed;
  left: 20%;
  transform: translateX(-100%);
  top: 0;
  background-color: #f0f0f0;
  /* Additional styling */
}

.item {
  cursor: grab;
  margin: 10px;
  padding: 10px;
  border: 1px solid #ddd;
  background-color: transparent; /* Transparent background */
  color: #333; /* Text color */
}

/* Style for image items */
.item img {
  max-width: 100%;
  max-height: 100px; /* Adjust height as needed */
}
</style>
