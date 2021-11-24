<template>
  <div id="app">
    <!-- <VirtalScrollWrapper /> -->
    <!-- <Base /> -->
    <!-- <Paging /> -->
    <Sort />
  </div>
</template>

<script>

// import VirtalScrollWrapper from '@/components/virtual-scroll';
// import Base from '@/components/base';
// import Paging from '@/components/with-paging';
import Sort from '@/components/with-sort';

export default {
  name: 'App',
  components: {
    // VirtalScrollWrapper
    // Base,
    // Paging,
    Sort,
  }
};
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
