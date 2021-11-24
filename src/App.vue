<template>
  <div id="app">
    <!-- <VirtalScrollWrapper /> -->
    <!-- <Base /> -->
    <Paging />
  </div>
</template>

<script>

// import VirtalScrollWrapper from '@/components/virtual-scroll';
// import Base from '@/components/base';
import Paging from '@/components/with-paging';
export default {
  name: 'App',
  components: {
    // VirtalScrollWrapper
    // Base,
    Paging,
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
