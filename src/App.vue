<template>
  <div id="app">
    <Grid />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import Grid from './components/Grid.vue';

@Component({
  components: {
    Grid,
  },
})
export default class App extends Vue {}
</script>

<style lang="scss">
#app {
  width: 80%;
  margin: auto;
}
</style>
