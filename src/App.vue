<template>
  <div id="app">
    <Common />
    <!-- <Sort /> -->
  </div>
</template>

<script>

import Common from '@/components/common';
// import Sort from '@/components/with-sort';

export default {
  name: 'App',
  components: {
    Common,
    // Sort,
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
