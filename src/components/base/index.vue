<template>
  <oz-table :rows="rows">
    <!-- компонент колонки нужен для передачи нужных данных в компонент таблицы -->
    <oz-table-column prop="id" title="ID" />
    <oz-table-column prop="postId" title="Post ID" />

<!-- todo - можно делать пустой компонент и передавать слоты, не указывая их внутри компонента? -->
    <oz-table-column prop="email">
      <template #title>
        <b>Email</b>
      </template>

      <template #body="{ row }">
        <a :href="`mailto:${row.email}`">{{ row.email }}</a>
      </template>
    </oz-table-column>

    <oz-table-column prop="name" title="Name" />
  </oz-table>
</template>

<script>
import OzTable from './oz-table';
import OzTableColumn from './oz-table-column';

export default {
  name: 'BaseWrapper',
  components: {
    OzTableColumn,
    OzTable
  },
  async created() {
    const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
    this.rows = await res.json();
  },
  data() {
    return {
      rows: []
    };
  }
};
</script>
