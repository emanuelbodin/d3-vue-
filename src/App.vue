<template>
  <div class="container">
    <div class="row">
      <button class="btn" @click="loadData(1)">Data 1</button>
      <button class="btn" @click="loadData(2)">Data 2</button>
    </div>
    <div class="row">
      <LineChart :dataset="lineData" />
    </div>
  </div>
</template>

<script>
import LineChart from './components/LineChart.vue';
//import BarChart from './components/BarChart.vue';
import * as d3 from 'd3';
export default {
  name: 'App',
  components: { LineChart },
  data() {
    return {
      lineData: [],
    };
  },
  methods: {
    async loadData(num) {
      const usage = {};
      const bookings = {};
      const maxUsage = {};
      const maxBookings = {};
      if (num === 1) {
        usage.values = await d3.csv('datasets/v1/usage.csv');
        bookings.values = await d3.csv('datasets/v1/booking.csv');
        maxUsage.values = await d3.csv('datasets/v1/maxUsage.csv');
        maxBookings.values = await d3.csv('datasets/v1/maxBooking.csv');
      }
      if (num === 2) {
        usage.values = await d3.csv('datasets/v2/usage.csv');
        bookings.values = await d3.csv('datasets/v2/booking.csv');
        maxUsage.values = await d3.csv('datasets/v2/maxUsage.csv');
        maxBookings.values = await d3.csv('datasets/v2/maxBooking.csv');
      }
      usage.color = '#5A657D';
      maxUsage.color = '#5A657D';
      bookings.color = '#D1A617';
      maxBookings.color = '#D1A617';
      maxUsage.dotted = true;
      maxBookings.dotted = true;
      this.lineData = [usage, bookings, maxUsage, maxBookings];
    },
  },
  async mounted() {
    this.loadData(1);
  },
};
</script>

<style>
.container {
}

.row {
  width: 100%;
  display: flex;
  justify-content: center;
}

.btn {
  cursor: pointer;
  padding: 5px;
  margin: 10px;
}
</style>
