<template>
  <oz-table
    :rows="rows"
    :all-pages="allPages"
    :total-pages="getTotalPages"
    :current-page="currentPage"
    :static-paging="true"

    @getPage="getPage"
    @addFilter="addFilter"
    @removeFilter="removeFilter"
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
    const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
    this.allPages = await res.json();
    this.preparePages(this.allPages);
    // нужно для получения первой страницы при загрузке
    this.blockingPromise = this.getPage(1);
  },
  data() {
    return {
      rows: [],
      allPages: [],
      list: [],
      currentPage: 1
    };
  },
  computed: {
    getTotalPages() {
      return this.list.length;
    }
  },
  methods: {
    addFilter(value) {
      const res = this.allPages.filter(row => row[value.filterProp].search(value.filterText) > -1)
      this.preparePages(res);
      this.getPage(this.currentPage);
    },
    removeFilter() {
      this.preparePages(this.allPages);
      this.getPage(this.currentPage);
    },
    // создает разбитый на страницы массив
    preparePages(array) {
      if (array.length) {
        const list = [];
        let size = 5; //раземер страницы
        for (let i = 0; i < Math.ceil(array.length/size); i++) {
            list[i] = array.slice((i*size), (i*size) + size);          
        }
        this.list = list;
      }
      console.log('in prepare pages', this.list);
    },
    async getPage(number) {
      console.log('in get page this.list: ', this.list);
      this.rows = this.list[number - 1];
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
