<template>
  <div id="app">
    <Common v-if="common"/>
    <Paging v-else/>
  </div>
</template>

<script>

import Common from '@/components/common';
import Paging from '@/components/with-paging';

export default {
  name: 'App',
  components: {
    Common,
    Paging,
  },
  data() {
    return {
      common: true,
    }
  },
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
