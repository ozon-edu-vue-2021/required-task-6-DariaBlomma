<template>
  <!--  @getPage="infGetPage" для бесконечной пагиинации -->
  <!-- @getPage="getPage" для тстатической пагинации -->
  <oz-table
    :rows="rows"
    :all-pages="allPages"
    :total-pages="getTotalPages"
    :current-page="currentPage"
    :static-paging="staticPaging"

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
    if (this.staticPaging) {
      this.fetchAllPages();
    } else {
      // нужно для получения первой страницы при загрузке
      this.blockingPromise = this.fetchFirstPage();
    }
  },   
    // const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
    // this.allPages = await res.json();
    // this.preparePages(this.allPages);

    // для сброса фильтров или сортировки к исходному состоянию должен быть неизменяемый массив - fetchedRows, allPages 
  data() {
    return {
      staticPaging: true,
      rows: [],
      fetchedRows: [],
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
    async fetchAllPages() {
      if (this.staticPaging) {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
        this.allPages = await res.json();
        this.preparePages(this.allPages);
        this.getPage(1);
      }
    },
    async fetchFirstPage() {
      if (!this.staticPaging) {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=1`);
        this.fetchedRows = await res.json();
        this.rows = this.fetchedRows;
      }
    },
    sortList() {
      let array = [];
      if (this.isFiltered) {
        array = this.filteredList;
      } else {
          if (!this.staticPaging) {
            array = this.fetchedRows;
          } else {
            array = this.allPages;
          }
      }
      
      this.sortedList =  orderBy(array, [this.sortFilterInfo.sortProp], [this.sortFilterInfo.sortDirection]);

      if (!this.staticPaging) {
        this.rows = this.sortedList;
      }
    },
    filterList() {
      let array = [];
      if (this.isSorted) {
        array = this.sortedList;
      } else {
        if (!this.staticPaging) {
          array = this.fetchedRows;
        } else {
          array = this.allPages;
        }
      }
      
      this.filteredList =  array.filter(row => row[this.sortFilterInfo.filterProp].search(this.sortFilterInfo.filterText) > -1);
      // todo - так работает при бесконечном скролле, но лагает.
      if (!this.staticPaging) {
        this.rows = this.filteredList;
      }
    },
    addFilter(value) {
      this.isFiltered = true;
      this.sortFilterInfo = value;
      this.filterList();
      
      if (this.staticPaging) {
        this.preparePages(this.filteredList);
        this.getPage(this.currentPage);
      }
    },
    removeFilter(value) {
      this.isFiltered = false;
      this.sortFilterInfo = value;
      this.sortList();

      if (this.staticPaging) {
        this.preparePages(this.sortedList);
        this.getPage(this.currentPage);
      }
    },
    addSort(value) {
      this.isSorted = true;
      this.sortFilterInfo = value;
      this.sortList();

      if (this.staticPaging) {
        this.preparePages(this.sortedList);
        this.getPage(this.currentPage);
      }
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
    },
    async getPage(number) {
      this.rows = this.list[number - 1];
      this.currentPage = number;
    },
    async infGetPage() {
      this.blockingPromise && await this.blockingPromise;
      const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${this.currentPage + 1}`);
      const newRows = await res.json();
      this.fetchedRows = [...this.fetchedRows, ...newRows];
      this.rows = this.fetchedRows;
      this.currentPage++;
      if (this.sortFilterInfo.filterProp) {
        // повторный вызов нужен для фильтрации по новополученным полям
        this.filterList();
      }
      if (this.sortFilterInfo.sortProp) {
        this.sortList();
      }
    }
  },
};
</script>
