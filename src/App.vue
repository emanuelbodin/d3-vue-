<template>
  <div class="container">
    <div class="row">
      <div class="col">
        <div>
          <button class="btn" @click="loadData(1)">Data 1</button>
          <button class="btn" @click="loadData(2)">Data 2</button>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <div class="card">
          <Legends :dataset="lineData" @toggle="toggleLine" />
          <LineChart id="line1" :dataset="lineData" />
        </div>
      </div>
      <div class="col">
        <div class="card">
          <StepLineChart id="step1" :dataset="stepLineData" />
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col">
        <div class="card">
          <BarChart id="bar1" :dataset="barData" />
        </div>
      </div>
      <div class="col">
        <div class="card">
          <HorizontalBarChart
            id="horizontalBar1"
            :dataset="horizontalBarData"
          />
        </div>
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
      stepLineData: {
        data: [
          {
            id: '1',
            name: 'Användning',
            values: [],
            color: '#D1A617',
            dashed: false,
            visible: true,
          },
          {
            id: '2',
            name: 'Bokning',
            values: [],
            color: '#5A657D',
            dashed: false,
            visible: true,
          },
        ],
      },
      barData: {
        values: [],
        timeSetting: 'WEEK',
        colors: {
          used: '#D1A617',
          booked: '#5A657D',
          usedAndBooked: '#A5965F',
          notUsedNotBooked: '#B8B2B2',
        },
        names: {
          used: 'Användning',
          booked: 'Bokning',
          usedAndBooked: 'Användning och bokning',
          notUsedNotBooked: 'Inte använd inte bokad',
        },
      },
      horizontalBarData: {},
    };
  },
  methods: {
    toggleLine(id) {
      const lineToToggle = this.lineData.data.find((el) => el.id === id);
      lineToToggle.visible = !lineToToggle.visible;
      this.lineData = { ...this.lineData };
    },
    async loadData(num) {
      this.setLineData(num);
      this.setStepLineData(num);
      this.setBarData(num);
      /*
      if (num === 1) {
        barData.values = await d3.csv(
          `datasets/v${this.datasetNumber}/bookedAndUsed.csv`
        );
        horizontalBarData.values = await d3.csv(
          `datasets/v${this.datasetNumber}/roomData.csv`
        );
      }
      
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
    
      
      horizontalBarData.values.forEach((el) => {
        el.used = parseInt(el.used);
        el.booked = parseInt(el.booked);
        el.usedAndBooked = parseInt(el.usedAndBooked);
        el.notUsedNotBooked = parseInt(el.notUsedNotBooked);
      });
      this.horizontalBarData = horizontalBarData;
      */
    },
    async setLineData(num) {
      const dataCopy = { ...this.lineData };
      dataCopy.data[0].values = await d3.csv(`datasets/v${num}/usage.csv`);
      dataCopy.data[1].values = await d3.csv(`datasets/v${num}/booking.csv`);
      dataCopy.data[2].values = await d3.csv(`datasets/v${num}/maxUsage.csv`);
      dataCopy.data[3].values = await d3.csv(`datasets/v${num}/maxBooking.csv`);
      this.lineData = dataCopy;
      this.lineData.resolution = 'DAY';
    },
    async setStepLineData(num) {
      const dataCopy = { ...this.stepLineData };
      dataCopy.data[0].values = await d3.csv(
        `datasets/v${num}/usagePattern.csv`
      );
      dataCopy.data[1].values = await d3.csv(
        `datasets/v${num}/bookingPattern.csv`
      );
      this.stepLineData = dataCopy;
    },
    async setBarData(num) {
      const dataCopy = { ...this.barData };
      dataCopy.values = await d3.csv(`datasets/v${num}/bookedAndUsed.csv`);
      this.barData = dataCopy;
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
  margin-bottom: 20px;
}
.col {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  flex-basis: 100%;
  flex: 1;
  margin-left: 10px;
  margin-right: 10px;
}

.card {
  height: 300px;
  width: 75%;
  padding: 10px;
}
.btn {
  cursor: pointer;
  padding: 5px;
  margin: 10px;
}
</style>
