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
    <div class="row">
      <div class="col">
        <HorizontalBarChart :id="'bar1'" :dataset="horizontalBarData" />
      </div>
      <div class="col">
        <StepLineChart :id="'step1'" :dataset="lineData" />
      </div>
    </div>
  </div>
</template>

<script>
import LineChart from './components/LineChart.vue';
import BarChart from './components/BarChart.vue';
import HorizontalBarChart from './components/HorizontalBarChart.vue';
import StepLineChart from './components/StepLineChart.vue';
import * as d3 from 'd3';
export default {
  name: 'App',
  components: { LineChart, BarChart, HorizontalBarChart, StepLineChart },
  data() {
    return {
      lineData: [],
      barData: {},
      horizontalBarData: {},
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
      const horizontalBarData = {};
      if (num === 1) {
        usage.values = await d3.csv('datasets/v1/usage.csv');
        bookings.values = await d3.csv('datasets/v1/booking.csv');
        maxUsage.values = await d3.csv('datasets/v1/maxUsage.csv');
        maxBookings.values = await d3.csv('datasets/v1/maxBooking.csv');
        barData.values = await d3.csv('datasets/v1/bookedAndUsed.csv');
        horizontalBarData.values = await d3.csv('datasets/v1/roomData.csv');
      }
      if (num === 2) {
        usage.values = await d3.csv('datasets/v2/usage.csv');
        bookings.values = await d3.csv('datasets/v2/booking.csv');
        maxUsage.values = await d3.csv('datasets/v2/maxUsage.csv');
        maxBookings.values = await d3.csv('datasets/v2/maxBooking.csv');
        barData.values = await d3.csv('datasets/v2/bookedAndUsed.csv');
        horizontalBarData.values = await d3.csv('datasets/v2/roomData.csv');
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
      horizontalBarData.names = {
        used: 'Användning',
        booked: 'Bokning',
        usedAndBooked: 'Användning och bokning',
        notUsedNotBooked: 'Inte använd inte bokad',
      };
      horizontalBarData.colors = {
        used: '#D1A617',
        booked: '#5A657D',
        usedAndBooked: '#A5965F',
        notUsedNotBooked: '#B8B2B2',
      };
      barData.names = {
        used: 'Användning',
        booked: 'Bokning',
        usedAndBooked: 'Användning och bokning',
        notUsedNotBooked: 'Inte använd inte bokad',
      };
      usage.name = 'Usage';
      maxUsage.name = 'Max usage';
      bookings.name = 'Bookings';
      maxBookings.name = 'Max bookings';
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
      horizontalBarData.values.forEach((el) => {
        el.used = parseInt(el.used);
        el.booked = parseInt(el.booked);
        el.usedAndBooked = parseInt(el.usedAndBooked);
        el.notUsedNotBooked = parseInt(el.notUsedNotBooked);
      });
      this.barData = barData;
      this.horizontalBarData = horizontalBarData;
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
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  width: 100%;
  justify-content: center;
}
.col {
  display: flex;
  flex-direction: column;
  flex-basis: 100%;
  flex: 1;
}
.btn {
  cursor: pointer;
  padding: 5px;
  margin: 10px;
}
</style>
