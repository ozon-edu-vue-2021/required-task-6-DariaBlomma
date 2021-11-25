<script lang="jsx">
import { orderBy } from 'lodash/collection';
import FilterDropdown from './filter-dropdown';
import OzTablePaginator from './oz-table-paginator';
import DotsLoaderIcon from './dost-loader.svg';

export default {
  name: 'oz-table',
  props: {
    rows: {
      type: Array,
      default: () => []
    },
    allPages: {
      type: Array,
      default: () => []
    },
    totalPages: {
      type: Number,
      default: 0
    },
    filteredPage: {
      type: Array,
      default: () => [],
    },
    currentPage: {
      type: Number,
      default: 0
    },
    staticPaging: {
      type: Boolean,
      default: true
    },
  },
  data() {
    return {
      // property for sorting
      sortProp: '',
      // asc desc
      sortDirection: '',
      filterProp: '',
      filterText: '',
      filter: {
        sortProp: '',
        sortDirection: '',
        filterProp: '',
        filterText: '',
      }
    };
  },
  computed: {
    // как вариант - переместить в индексный файл
    sortedRows() {
      let res;

      if (!this.sortProp) {
        // todo  - возвращать не просто ряды, а отфильтрованные
        res =  this.rows;
        // res =  this.allPages;
      }
      // для правильной фильтрации должна быть не текущая страница, а все 100
      res = orderBy(this.rows, [this.sortProp], [this.sortDirection]);
      // console.log('res: ', res);

      return res;
    }
  },
  methods: {
    toggleSort(prop) {
      this.filter.sortProp = prop;
      this.filter.sortDirection = (this.filter.sortDirection === 'desc' || !this.filter.sortDirection) ? 'asc' : 'desc';
      this.$emit('addSort', this.filter);
    },
    openFilterTooltip(prop = '') {
      console.log('in open filterTooltip')
      if (prop === '') {
        this.$emit('removeFilter', this.filter);
      }
      this.filter.filterProp = prop;
      this.filter.filterText = '';
    },
    setFilterText(e) {
      this.filter.filterText = e.target.value;
      if (this.filter.filterText) {
        this.$emit('addFilter', this.filter);
      }
    },
    renderHead(h, columnsOptions) {
      const { $style, filter } = this;

      return columnsOptions.map((column) => {
        const renderedTitle = column.scopedSlots.title ? column.scopedSlots.title() : column.title;
        let sortIcon = 'sort';

        if (filter.sortProp === column.prop) {
          sortIcon = filter.sortDirection === 'asc' ? 'sort-amount-down' : 'sort-amount-up';
        }

        return (
          <th key={column.prop} class={$style.headerCell}>
            <div class={$style.headerCellContent}>
              <span>{renderedTitle}</span>
              <font-awesome-icon
                class={$style.sortIcon}
                icon={sortIcon}
                on={{ click: () => this.toggleSort(column.prop) }}
              />
              <FilterDropdown
                columnProp={column.prop}
                shown={column.prop === filter.filterProp}
                filterText={filter.filterText}

                on={{
                  openFilterTooltip: () => this.openFilterTooltip(column.prop),
                  closeFilterTooltip: () => this.openFilterTooltip(),
                  setFilterText: this.setFilterText,
                }}
              />
            </div>
          </th>
        );
      });
    },
    renderRows(h, columnsOptions) {
      return this.rows.map((row, index) => {
        return <tr key={row.id || index}>{...this.renderColumns(h, row, columnsOptions)}</tr>;
      });
    },
    renderColumns(h, row, columnsOptions) {
      return columnsOptions.map((column) => {
        return (
          <td key={column.prop} class={this.$style.cell}>
            {column.scopedSlots.body ? column.scopedSlots.body({ row }) : row[column.prop]}
          </td>
        );
      });
    },
    getColumnOptions() {
      return this.$slots.default.
        filter(item => item.componentOptions && item.componentOptions.tag === 'oz-table-column').
        map(column =>
          Object.assign({}, column.componentOptions.propsData, {
              scopedSlots: column.data.scopedSlots || {}
            }
          )
        );
    }
  },
  renderInfPager() {
    const directives = [
      {
        name: 'detect-viewport',
        value: {
          callback: this.$listeners.getPage
        }
      }
    ];

    const style = {
      background: `url("${DotsLoaderIcon}") no-repeat center`
    };

    return (
      <div {...{ class: this.$style.infPager, style, directives }} />
    );
  },
  render(h) {
    const { $style, totalPages, currentPage, staticPaging, $listeners } = this;
    const { getPage } = $listeners;
    const columnsOptions = this.getColumnOptions();
    const columnsHead = this.renderHead(h, columnsOptions);
    const rows = this.renderRows(h, columnsOptions);

    return (
      <div>
        <table class={$style.table}>
          <thead>{...columnsHead}</thead>
          <tbody>{...rows}</tbody>
        </table>

        {staticPaging
          ? <OzTablePaginator totalPages={totalPages} currentPage={currentPage} on={{ getPage: getPage }} />
          : this.renderInfPager()
        }
      </div>
    );
  }
};
</script>

<style module>
  .table {
    border-spacing: 0;
    margin: 8px;
    width: 100%;
  }

  .cell {
    text-align: left;
    border-bottom: 1px solid #c8cacc;
    padding: 1rem 1rem;
  }

  .headerCell {
    composes: cell;
    background: #c7cbcb;
  }

  .headerCellContent {
    display: flex;
    align-items: center;
  }

  .sortIcon {
    margin-left: 8px;
    margin-right: 24px;
  }

  .sortIcon:hover {
    cursor: pointer;
  }

  .infPager {
    width: 100%;
    height: 32px;
  }
</style>
