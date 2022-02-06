<template>
  <div>
    <a-layout>
      <a-layout-header style="background: #fff; text-align: center">
        <h1>Race dashboard</h1>
      </a-layout-header>
      <a-layout-content class="layout-container">
        <div>
          <h3>Select category to filter, Unselect to clear</h3>
          <a-button-group>
            <a-button
              v-for="(category, key) in categories"
              :key="key"
              :type="selectedCategory === category.id ? 'primary' : null"
              @click="selectCategory(category.id)"
              >{{ category.name }}</a-button
            >
          </a-button-group>
        </div>
        <div class="cards-container">
          <h3>Race cards</h3>
          <a-row
            :v-if="races.length"
            type="flex"
            :gutter="16"
            justify="center"
            align="top"
          >
            <template v-for="(race, key) in filteredRaces">
              <a-col v-if="race" :key="key" :span="4">
                <RaceCard :race-data="race" @race-done="onRaceDone($event)" />
              </a-col>
            </template>
          </a-row>
        </div>
      </a-layout-content>
    </a-layout>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Race from '../interfaces/RaceInterface'
import Category from '../interfaces/CategoryInterface'
export default Vue.extend({
  name: 'MainDashboard',
  data() {
    return {
      races: [] as Race[],
      filteredRaces: [] as Race[],
      selectedCategory: '' as string,
      categories: [] as Category[],
    }
  },
  mounted() {
    this.categories = [
      { name: 'Greyhound racing', id: '9daef0d7-bf3c-4f50-921d-8e818c60fe61' },
      { name: 'Harness racing', id: '161d9be2-e909-4326-8c2c-35ed71fb460b' },
      { name: 'Horse racing', id: '4a2788f8-e825-4d36-9894-efd4baf1cfae' },
    ]
    this.getRaceData()
  },
  methods: {
    getRaceData() {
      this.$axios
        .$get(
          'https://api.neds.com.au/rest/v1/racing/?method=nextraces&count=5'
        )
        .then((response) => {
          const data = response.data
          const races = data.next_to_go_ids
            .map((id: string) => data.race_summaries[id])
            .sort(this.raceSort)
          this.races = JSON.parse(JSON.stringify(races))
          this.filteredRaces = [...races]
        })
    },
    onRaceDone(raceId: string) {
      if (raceId && this.races?.length) {
        this.getRaceData()
      }
    },
    selectCategory(id: string) {
      if (this.selectedCategory === id) {
        this.selectedCategory = ''
        const races = this.races.filter(() => true)
        this.filteredRaces = races.splice(0, 5)
        return
      }
      this.selectedCategory = id
      if (id && this.filteredRaces?.length) {
        const races = this.races.filter((race) => race.category_id === id)
        this.filteredRaces = races.splice(0, 5)
      }
    },
    raceSort(a: Race, b: Race) {
      if (a.advertised_start.seconds < b.advertised_start.seconds) return -1
      if (a.advertised_start.seconds > b.advertised_start.seconds) return 1
      return 0
    },
  },
})
</script>
<style scoped>
.layout-container {
  margin: 24px 16px;
  padding: 24px;
  background: #fff;
  min-height: 280px;
}
.cards-container {
  margin-top: 16px;
}
</style>
