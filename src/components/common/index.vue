<template>
  <oz-table
    :rows="rows"
    :all-pages="allPages"
    :total-pages="100"
    :current-page="currentPage"
    :static-paging="true"

    @getPage="getPage"
  >
    <oz-table-column prop="id" title="ID" />
    <oz-table-column prop="postId" title="Post ID" />

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
// todo - фильтры должны учитывать тип данные, м.б range, cheeckbox
import OzTable from './oz-table';
import OzTableColumn from './oz-table-column';

export default {
  name: 'Common',
  components: {
    OzTableColumn,
    OzTable
  },
  async created() {
    // нужно для получения первой страницы при загрузке
    this.blockingPromise = this.getPage(1);
    this.getAllPages();
  },
  data() {
    return {
      rows: [],
      allPages: [],
      currentPage: 1
    };
  },
  methods: {
    async getAllPages() {
      const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
      this.allPages = await res.json();
    },
    async getPage(number) {
      const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${number}`);
      this.rows = await res.json();
      this.currentPage = number;
    },
    async infGetPage() {
      this.blockingPromise && await this.blockingPromise;
      const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${this.currentPage + 1}`);
      const newRows = await res.json();
      this.rows = [...this.rows, ...newRows];
      this.currentPage++;
    }
  },
};
</script>
