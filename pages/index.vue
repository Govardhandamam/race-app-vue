<template>
  <Dashboard />
</template>

<script lang="ts">
import Vue from 'vue'
import Dashboard from '../components/Dasbhoard.vue'
import RaceCard from '../components/RaceCard.vue'
const components = { Dashboard, RaceCard }

Object.entries(components).forEach(([name, component]) => {
  Vue.component(name, component)
})

export default Vue.extend({
  name: 'IndexPage',
})
</script>
