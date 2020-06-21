<template>
  <suspense>
    <template #default>
      <FILP></FILP>
    </template>
    <template #fallback>
      <div>加载中...</div>
    </template>
  </suspense>
</template>

<script>
import FILP from './components/FILP.vue'

export default {
  name: 'App',
  components: {
    FILP
  }
}
</script>
