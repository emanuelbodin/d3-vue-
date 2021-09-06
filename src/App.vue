<template>
  <div class="container">
    <div class="row">
      <button class="btn" @click="loadData(1)">Data 1</button>
      <button class="btn" @click="loadData(2)">Data 2</button>
    </div>
    <div class="row">
      <div class="col">
        <LineChart :dataset="lineData" @toggle="toggleLine" />
      </div>
      <div class="col">
        <BarChart :id="'bar1'" :dataset="barData" />
      </div>
    </div>
  </div>
</template>

<script>
import LineChart from './components/LineChart.vue';
import BarChart from './components/BarChart.vue';
import * as d3 from 'd3';
export default {
  name: 'App',
  components: { LineChart, BarChart },
  data() {
    return {
      lineData: [],
      barData: {},
    };
  },
  methods: {
    toggleLine(id) {
      this.lineData = this.lineData.map((el) => {
        if (el.id === id) el.visible = !el.visible;
        return el;
      });
    },
    async loadData(num) {
      const usage = {};
      const bookings = {};
      const maxUsage = {};
      const maxBookings = {};
      const barData = {};
      if (num === 1) {
        usage.values = await d3.csv('datasets/v1/usage.csv');
        bookings.values = await d3.csv('datasets/v1/booking.csv');
        maxUsage.values = await d3.csv('datasets/v1/maxUsage.csv');
        maxBookings.values = await d3.csv('datasets/v1/maxBooking.csv');
        barData.values = await d3.csv('datasets/v1/bookedAndUsed.csv');
      }
      if (num === 2) {
        usage.values = await d3.csv('datasets/v2/usage.csv');
        bookings.values = await d3.csv('datasets/v2/booking.csv');
        maxUsage.values = await d3.csv('datasets/v2/maxUsage.csv');
        maxBookings.values = await d3.csv('datasets/v2/maxBooking.csv');
        barData.values = await d3.csv('datasets/v2/bookedAndUsed.csv');
      }
      usage.color = '#D1A617';
      maxUsage.color = '#D1A617';
      bookings.color = '#5A657D';
      maxBookings.color = '#5A657D';
      barData.colors = {
        used: '#D1A617',
        booked: '#5A657D',
        usedAndBooked: '#A5965F',
        notUsedNotBooked: '#B8B2B2',
      };

      usage.name = 'Usage';
      maxUsage.name = 'Max usage';
      bookings.name = 'Bookings';
      maxBookings.name = 'Max bookings';
      barData.names = [
        'Used',
        'Booked',
        'Booked and used',
        'Not booked not used',
      ];
      usage.id = 0;
      maxUsage.id = 1;
      bookings.id = 2;
      maxBookings.id = 3;
      usage.visible = true;
      maxUsage.visible = true;
      bookings.visible = true;
      maxBookings.visible = true;
      maxUsage.dotted = true;
      maxBookings.dotted = true;
      this.lineData = [usage, bookings, maxUsage, maxBookings];
      barData.values.forEach((el) => {
        el.used = parseInt(el.used);
        el.booked = parseInt(el.booked);
        el.usedAndBooked = parseInt(el.usedAndBooked);
        el.notUsedNotBooked = parseInt(el.notUsedNotBooked);
      });
      this.barData = barData;
    },
  },
  async mounted() {
    this.loadData(1);
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Raleway:wght@600&display=swap');

* {
  font-family: 'Raleway', sans-serif;
  display: border-box;
}
.row {
  width: 100%;
  display: flex;
  justify-content: center;
}
.col {
  width: 40%;
  margin: 10px;
}
.btn {
  cursor: pointer;
  padding: 5px;
  margin: 10px;
}
</style>
