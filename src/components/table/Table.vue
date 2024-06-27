<template>
    <div class="table" :style="tableStyle">
      <table>
        <thead>
          <tr>
            <th v-for="(header, index) in data.headers" :key="index">{{ header }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in displayedRows" :key="rowIndex">
            <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      data: Object,
      pageIndex: Number,
      itemsPerPage: Number,
      style: Object
    },
    computed: {
      displayedRows() {
        const start = this.pageIndex * this.itemsPerPage;
        const end = start + this.itemsPerPage;
        return this.data.rows.slice(start, end);
      },
      tableStyle() {
        return {
          ...this.style,
          width: '100%'
        };
      }
    }
  };
  </script>
  
  <style scoped>
  .table {
    overflow: auto;
  }
  table {
    border-collapse: collapse;
    width: 100%;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 8px;
  }
  th {
    background-color: #f2f2f2;
  }
  </style>