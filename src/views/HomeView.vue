<template>
  <div class="home">
   <bpmn />
  </div>
</template>

<script>
// @ is an alias to /src
import bpmn from '@/components/bpmn'

export default {
  name: 'HomeView',
  components: {
    bpmn
  }
}
</script>

<style lang="scss" scoped>
.home {
  width: 100%;
  height: 100%;
}
</style>
