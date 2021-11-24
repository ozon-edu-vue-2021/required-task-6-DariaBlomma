<script>
export default {
  name: 'filter-dropdown',
  props: {
    columnProp: {
      type: String,
      default: ''
    },
    shown: {
      type: Boolean,
      default: false
    },
    filterText: {
      type: String,
      default: ''
    }
  },
  render() {
    const { $style, shown, filterText, $listeners } = this;
    const { openFilterTooltip, setFilterText, closeFilterTooltip } = $listeners;

    return (
      <v-dropdown
        class={$style.filterIcon}
        triggers={[]}
        autoHide={false}
        shown={shown}
      >
        <font-awesome-icon
          icon="filter"
          on={{ click: openFilterTooltip }}
        />

        <div slot="popper">
          <input
            type="text"
            domProps={{ value: filterText }}
            on={{ input: setFilterText }}
          />

          <font-awesome-icon
            icon="times"
            class={$style.closeIcon}
            on={{ click: closeFilterTooltip }}
          />
        </div>
      </v-dropdown>
    );
  }
};
</script>

<style module>
  .filterIcon {
    margin-left: auto;
    margin-right: 8px;
  }

  .filterIcon:hover {
    cursor: pointer;
  }

  .closeIcon {
    position: absolute;
    top: 6px;
    right: 6px;
  }

  .closeIcon:hover {
    cursor: pointer;
  }
</style>
