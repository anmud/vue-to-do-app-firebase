<template>
  <div id="app">
  <app-tasks></app-tasks>
  </div>
</template>

<script>
import tasksList from '../src/tasks-list'
export default {
  name: 'app',
  data () {
    return {
     
    }
  },
  components: {
    'app-tasks': tasksList
  }
}
</script>

<style>

</style>
