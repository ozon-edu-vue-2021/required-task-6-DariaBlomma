<template>
  <!--  @getPage="infGetPage" для бесконечной пагиинации -->
  <!-- @getPage="getPage" для тстатической пагинации -->
  <oz-table
    :rows="rows"
    :all-pages="allPages"
    :total-pages="getTotalPages"
    :current-page="currentPage"
    :static-paging="staticPaging"
    :empty-message="emptyMessage"

    @getPage="infGetPage"

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
  // для сброса фильтров или сортировки к исходному состоянию должен быть неизменяемый массив - fetchedRows, allPages 
  data() {
    return {
      staticPaging: false,
      rows: [],
      fetchedRows: [],
      newRows: [],
      afterNewRows: [],
      renderedRows: 0,
      allPages: [],
      list: [],
      currentPage: 1,
      constantCurrentPage: 1,
      pageSize: 5,
      requiredRowsLength: 5,
      isSorted: false,
      isFiltered: false,
      pageRendered: false,
      newRowsFetched: false,
      sortedList: [],
      filteredList: [],
      sortFilterInfo: {},
      rememberLengthCount: 0,
      rememberedCurrentPage: 0,
      emptyMessage: '',
    };
  },
  computed: {
    getTotalPages() {
      return this.list.length;
    },
  },
  methods: {
    async fetchAllPages() {
      try {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments`);
        this.allPages = await res.json();
        this.preparePages(this.allPages);
        this.getPage(1);
      } catch (e) {
        console.warn('Could not fetch all pages', e);
      }
    },
    async fetchFirstPage() {
      try {
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=1`);
        this.fetchedRows = await res.json();
        this.rows = this.fetchedRows;
      } catch (e) {
        console.warn('Could not fetch first page', e);
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
    async filterList() {
      console.log('in filter list');
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
        for (let i = 0; i < Math.ceil(array.length/this.pageSize); i++) {
            list[i] = array.slice((i*this.pageSize), (i*this.pageSize) + this.pageSize);          
        }
        this.list = list;
      }
    },
    async getPage(number) {
      this.rows = this.list[number - 1];
      this.currentPage = number;
    },
    async fetchNextPage() {
      try { 
        this.newRowsFetched = false;
        const res = await fetch(`https://jsonplaceholder.typicode.com/comments?postId=${this.currentPage + 1}`);
        this.newRows = await res.json();
        this.newRowsFetched = true;
      } catch (e) {
        console.warn('Could not fetch next page', e);
      }
    },
    // * при самом первом вызове запоминаем номер страницы. После is equal увеличивает номер страницы на 1.
    // * это нужно для правильного подсчета требуемых рядов на странице. 
    // * Они зависят не от currentPage, то есть нужного id поста, а от разбиения по 5штук на страницу уже отфильтрованных данных
    // * Получается, после набора требуемого размера рядов, в следующий вызов прибавится 5
    rememberCurrentPage(updatePageNumber = false) {
      this.rememberedCurrentPage++;
      if (this.rememberedCurrentPage === 1) {
        this.rememberedPageNumber = this.currentPage;
      }
      if (updatePageNumber) {
        this.rememberedPageNumber++;
      }
      return this.rememberedPageNumber;
    },
    getRequiredRowsLength() {
      this.rememberLengthCount++;
      
      if (this.rememberLengthCount === 1) {
        this.requiredRowsLength = this.pageSize * this.rememberedPageNumber;
        // console.log('rememberedPageNumber: ', this.rememberedPageNumber);
        // console.log('remembered this.currentPage: ', this.currentPage);
      }
      return this.requiredRowsLength;
    },
    renderNextPage() {
      this.fetchedRows = [...this.fetchedRows, ...this.newRows];
    },
    async infGetPage() {
      console.log('in infGetPage');
      // if (this.renderedRows === this.requiredRowsLength) {
      //   console.log('rendered')
      // } else {
      //  console.log('not rendered')
      // }
      this.blockingPromise && await this.blockingPromise;

      await this.fetchNextPage() && this.newRowsFetched;
      
      // this.renderedRows = this.$children[0].$refs.tbody.children.length;
      if (this.newRows.length) {
        this.fetchedRows = [...this.fetchedRows, ...this.newRows];

      if (!this.sortFilterInfo.filterProp && !this.sortFilterInfo.sortProp) {
        this.rows = this.fetchedRows;
      }
      
      // todo - после фильтрации текущая страница ненормально увеличивается из-за рекурсии. Запомнить страницы при первом фильтре и вернкться к ней
      this.currentPage++;
      if (this.sortFilterInfo.filterProp) {
        // повторный вызов нужен для фильтрации по новополученным полям
        this.rememberCurrentPage();
        this.getRequiredRowsLength();
        this.filterList();

        if (this.rows.length < this.requiredRowsLength) {
          console.log('less');

          if (this.requiredRowsLength > 70) {
            console.log('requiredRowsLength: ', this.requiredRowsLength);
            console.log('this.rows.length: ', this.rows.length);
          }
          // * при быстрой прокрутке элементы могут дублироваться. Поэтому фильтруем на уникальность
          this.rows = this.filteredList.filter((value, index, array) => {
            if (array[index - 1]) {
              return array[index - 1].id !== value.id;
            }
          });
          // рекурсия - это выход, но при текузем сравнении требуемое число постоянно растет. Как сравнивать с константой
          await this.infGetPage();
        } else {
          this.rememberLengthCount = 0;
          this.rememberCurrentPage(true);
          console.log('is equal');
          console.log('requiredRowsLength: ', this.requiredRowsLength);
          console.log('this.rows.length: ', this.rows.length);
          console.log('after equal this.rememberedPageNumber: ', this.rememberedPageNumber);
          console.log('after equal this.currentPage: ', this.currentPage);
        }
      }

      if (this.sortFilterInfo.sortProp) {
        this.sortList();
      }
      } else {
        // todo - сделать return , передавать в разметку нужное сообщение
        // todo - проверить, что действительно не должно быть данных на этом моменте по статической пагинации
        console.log('no more new pages');
        console.log('current Id', this.currentPage)
        console.log('rows length', this.rows.length)
        this.emptyMessage = 'There are no more pages left';
        return;
      }  


    }
  },
};
</script>
