<template>
  <div>
    <table v-if="items.length > 0" class="data-table">
      <thead>
        <tr>
          <th v-for="header in headers" :key="header.value">{{ header.text }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in items" :key="index">
          <td v-for="header in headers" :key="header.value">
            {{ formatCell(item, header) }}
          </td>
        </tr>
      </tbody>
    </table>
    <p v-else class="no-data-message">{{ noDataMessage }}</p>
  </div>
</template>

<script setup lang="ts">
defineProps({
  items: {
    type: Array,
    required: true
  },
  headers: {
    // eslint-disable-next-line @typescript-eslint/no-unsafe-function-type
    type: Array as () => Array<{ text: string; value: string; formatter?: Function }>,
    required: true
  },
  noDataMessage: {
    type: String,
    default: 'Nenhum registro foi encontrado'
  }
});

// eslint-disable-next-line @typescript-eslint/no-explicit-any
const formatCell = (item: any, header: any) => {
  if (header.formatter) {
    return header.formatter(item[header.value]);
  }
  return item[header.value];
};
</script>

<style scoped>
.data-table {
  width: 100%;
  border-collapse: collapse;
  margin: 1rem 0;
}

.data-table th, .data-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.no-data-message {
  color: #666;
  font-style: italic;
}
</style>
