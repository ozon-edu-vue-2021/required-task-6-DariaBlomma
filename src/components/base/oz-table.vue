<script lang="jsx">
// todo - имеет доступ ко всем слотам любой вложенности?
export default {
  name: 'oz-table',
  props: {
    rows: {
      type: Array,
      default: () => []
    }
  },
  methods: {
    renderHead(h, columnsOptions) {
      // columnsOptions - массив из 4 колонок
      // todo - внутри vnode? программный параметр?
      // содержит атрибуты элементов и слоты
      // todo внутри дети, те слоты? 
      // console.log('columnsOptions: ', columnsOptions);
      return columnsOptions.map((column) => {
        // column.scopedSlots.title - слот с таким именем
        // todo - column.scopedSlots.title() - откуда в слоте такая функция?
        const renderedTitle = column.scopedSlots.title ? column.scopedSlots.title() : column.title;

        return (
          <th key={column.prop} class={this.$style.headerCell}>
            {renderedTitle}
          </th>
        );
      });
    },
    renderRows(h, columnsOptions) {
      // this.rows - props, получили через fetch
      return this.rows.map((row, index) => {
        return <tr key={row.id || index}>{...this.renderColumns(h, row, columnsOptions)}</tr>;
      });
    },
    renderColumns(h, row, columnsOptions) {
      // todo - откуда второй параметр row?
      // todo - this.$style - что это?
      // todo - this.$style - объект из классов внутри этого компонента и ?
      // column.scopedSlots.body - слот с другой разметкой td

      // console.log('this.$style: ', this.$style);
      return columnsOptions.map((column) => {
        return (
          <td key={column.prop} class={this.$style.cell}>
            {column.scopedSlots.body ? column.scopedSlots.body({ row }) : row[column.prop]}
          </td>
        );
      });
    },
    getColumnOptions() {
      console.log('this.$slots.default: ', this.$slots.default);
      // this.$slots.default - массив из 4 vnode
      // возвращает отфильтрованные vnode только с нужным нам содержанием. 
      // будет передаваться в renderHead, renderRows
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
  render(h) {
    // программная функция
    const columnsOptions = this.getColumnOptions();
    const columnsHead = this.renderHead(h, columnsOptions);
    const rows = this.renderRows(h, columnsOptions);

    return (
      <table class={this.$style.table}>
        <thead>{...columnsHead}</thead>
        <tbody>{...rows}</tbody>
      </table>
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
</style>
