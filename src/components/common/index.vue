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
    @addSort="addSort"
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
import { orderBy } from 'lodash/collection';
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
      currentPage: 1,
      isSorted: false,
      isFiltered: false,
      sortedList: [],
      filteredList: [],
      sortFilterInfo: {},
    };
  },
  computed: {
    getTotalPages() {
      return this.list.length;
    },
  },
  methods: {
    sortList() {
      let array = [];
      if (this.isFiltered) {
        array = this.filteredList;
      } else {
        array = this.allPages;
      }
      
      this.sortedList =  orderBy(array, [this.sortFilterInfo.sortProp], [this.sortFilterInfo.sortDirection]);
    },
    filterList() {
      let array = [];
      if (this.isSorted) {
        array = this.sortedList;
      } else {
        array = this.allPages;
      }
      
      this.filteredList =  array.filter(row => row[this.sortFilterInfo.filterProp].search(this.sortFilterInfo.filterText) > -1);
    },
    addFilter(value) {
      this.isFiltered = true;
      this.sortFilterInfo = value;
      this.filterList();
      this.preparePages(this.filteredList);
      this.getPage(this.currentPage);
    },
    removeFilter(value) {
      this.isFiltered = false;
      this.sortFilterInfo = value;
      this.sortList();
      this.preparePages(this.sortedList);
      this.getPage(this.currentPage);
    },
    addSort(value) {
      this.isSorted = true;
      this.sortFilterInfo = value;
      this.sortList();
      this.preparePages(this.sortedList);
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
      // console.log('in prepare pages', this.list);
    },
    async getPage(number) {
      // console.log('in get page this.list: ', this.list);
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
