<template>
  <a-card
    :v-if="raceData"
    class="race-card"
    :class="{
      started: timeToEnd <= 60 && timeToEnd > 20,
      ending: timeToEnd <= 20,
    }"
  >
    <p><b>Race Number:</b> {{ raceData.race_number }}</p>
    <p><b>Race Name:</b> {{ raceData.meeting_name }}</p>
    <p>
      <b>Race start:</b>
      {{
        raceData.advertised_start.seconds &&
        formatDateTime(raceData.advertised_start.seconds)
      }}
    </p>
    <p class="timer"><b>Time to End:</b> {{ getFormattedTime(timeToEnd) }}</p>
  </a-card>
</template>

<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  name: 'RaceCard',
  props: {
    raceData: {
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      timeToEnd: 0,
      timerRunning: false,
    }
  },
  watch: {
    raceData() {
      this.initializeTimeToEnd()
      if (!this.timerRunning) {
        this.countDownTimer()
        this.timerRunning = true
      }
    },
  },
  beforeMount() {
    this.initializeTimeToEnd()
  },

  mounted() {
    this.countDownTimer()
    this.timerRunning = true
  },
  methods: {
    formatDateTime(value: number): string {
      return new Date(value * 1000).toLocaleString()
    },
    getFormattedTime(value: number): string {
      if (value > 0) {
        const minutes = Math.floor(value / 60)
          .toString()
          .padStart(2, '0')
        const seconds = (value % 60).toString().padStart(2, '0')
        return `${minutes}:${seconds}`
      }
      return '00:00'
    },
    countDownTimer(): void {
      if (this.timeToEnd > 0) {
        setTimeout(() => {
          this.timeToEnd -= 1
          this.countDownTimer()
        }, 1000)
      } else if (this.raceData?.race_id) {
        this.timerRunning = false
        this.$emit('race-done', this.raceData.race_id)
      }
    },
    initializeTimeToEnd(): void {
      const currentTime = Math.round(new Date().getTime() / 1000)
      this.timeToEnd = this.raceData
        ? this.raceData?.advertised_start?.seconds - currentTime + 60
        : 0
    },
  },
})
</script>
<style scoped>
.race-card {
  background: #7ff5cf59;
  transition: background 0.2s ease-in-out;
}
.started {
  background: #f5e17f59;
}
.ending {
  background: #f57f7f59;
}
.timer {
  font-size: 1.5rem;
}
</style>
