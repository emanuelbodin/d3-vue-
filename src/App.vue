<template>
  <div class="container">
    <div class="row">
      <button class="btn" @click="loadData(1)">Data 1</button>
      <button class="btn" @click="loadData(2)">Data 2</button>
    </div>
    <div class="row">
      <div class="col">
        <Legends :dataset="lineData" @toggle="toggleLine" />
        <LineChart :id="'line1'" :dataset="lineData" />
      </div>
      <div class="col">
        <StepLineChart
          :id="'step1'"
          :dataset="stepLineData"
          @toggle="toggleLine2"
        />
      </div>
    </div>
    <div class="row">
      <div class="col">
        <BarChart :id="'bar1'" :dataset="barData" />
      </div>
      <div class="col">
        <HorizontalBarChart :id="'bar1'" :dataset="horizontalBarData" />
      </div>
    </div>
  </div>
</template>

<script>
import LineChart from './components/LineChart.vue';
import BarChart from './components/BarChart.vue';
import HorizontalBarChart from './components/HorizontalBarChart.vue';
import StepLineChart from './components/StepLineChart.vue';
import Legends from './components/Legends.vue';
import * as d3 from 'd3';
export default {
  name: 'App',
  components: {
    LineChart,
    BarChart,
    HorizontalBarChart,
    StepLineChart,
    Legends,
  },
  data() {
    return {
      datasetNumber: 1,
      lineData: {
        resolution: 'DAY',
        data: [
          {
            id: 'usage1',
            name: 'Användning',
            values: [],
            color: '#D1A617',
            dashed: false,
            visible: true,
            showLegend: true,
          },
          {
            id: 'usage2',
            name: 'Bokning',
            values: [],
            color: '#5A657D',
            dashed: false,
            visible: true,
            showLegend: true,
          },
          {
            id: 'usage3',
            name: 'Max Användning',
            values: [],
            color: '#D1A617',
            dashed: true,
            visible: true,
            showLegend: false,
          },
          {
            id: 'usage4',
            name: 'Max Bokning',
            values: [],
            color: '#5A657D',
            dashed: true,
            visible: true,
            showLegend: false,
          },
        ],
      },
      barData: {},
      horizontalBarData: {},
      stepLineData: [],
    };
  },
  methods: {
    toggleLine(id) {
      const lineToToggle = this.lineData.data.find((el) => el.id === id);
      lineToToggle.visible = !lineToToggle.visible;
      this.lineData = { ...this.lineData };
    },
    toggleLine2(id) {
      this.stepLineData = this.stepLineData.map((el) => {
        if (el.id === id) el.visible = !el.visible;
        return el;
      });
    },
    async loadData(num) {
      this.setLineData();
      const barData = {};
      const horizontalBarData = {};
      const usagePattern = {};
      const bookingPattern = {};
      if (num === 1) {
        barData.values = await d3.csv(
          `datasets/v${this.datasetNumber}/bookedAndUsed.csv`
        );
        horizontalBarData.values = await d3.csv(
          `datasets/v${this.datasetNumber}/roomData.csv`
        );
        usagePattern.values = await d3.csv(
          `datasets/v${this.datasetNumber}/usagePattern.csv`
        );
        bookingPattern.values = await d3.csv(
          `datasets/v${this.datasetNumber}/bookingPattern.csv`
        );
      }

      usagePattern.color = '#D1A617';
      bookingPattern.color = '#5A657D';
      usagePattern.name = 'Användning';
      bookingPattern.name = 'Bokning';
      usagePattern.visible = true;
      bookingPattern.visible = true;
      usagePattern.id = 'step1';
      bookingPattern.id = 'step2';
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
      this.stepLineData = [usagePattern, bookingPattern];
    },
    async setLineData() {
      const dataCopy = { ...this.lineData };
      dataCopy.data[0].values = await d3.csv(
        `datasets/v${this.datasetNumber}/usage.csv`
      );
      dataCopy.data[1].values = await d3.csv(
        `datasets/v${this.datasetNumber}/booking.csv`
      );
      dataCopy.data[2].values = await d3.csv(
        `datasets/v${this.datasetNumber}/maxUsage.csv`
      );
      dataCopy.data[3].values = await d3.csv(
        `datasets/v${this.datasetNumber}/maxBooking.csv`
      );
      this.lineData = dataCopy;
      this.lineData.resolution = 'DAY';
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
  color: 'blue';
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
  margin-left: 10px;
  margin-right: 10px;
}
.btn {
  cursor: pointer;
  padding: 5px;
  margin: 10px;
}
</style>
