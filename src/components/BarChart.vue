<template>
  <div v-if="dataset != null" class="chart">
    <svg :height="dimensions.height" :width="dimensions.width">
      <g
        :transform="
          `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
        "
      >
        <g :id="`barGroup_${id}`"></g>
        <g :id="`axisGroupY_${id}`"></g>
        <g :id="`axisGroupX_${id}`"></g>
      </g>
    </svg>
    <div class="tooltip" :id="`tooltip_${id}`">
      <div class="data"></div>
      <div class="date"></div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'BarChart',
  props: {
    dataset: {
      type: Object,
      required: false,
    },
    id: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      dimensions: { width: 600, height: 300, margins: 50 },
    };
  },
  computed: {
    ctrWidth() {
      return this.dimensions.width - this.dimensions.margins * 2;
    },
    ctrHeight() {
      return this.dimensions.height - this.dimensions.margins * 2;
    },
    stackGenerator() {
      return d3.stack().keys(this.dataset.values.columns.slice(1));
    },
    stackData() {
      return this.stackGenerator(this.dataset.values).map((dataGroup) => {
        dataGroup.forEach((state) => {
          state.key = dataGroup.key;
        });
        return dataGroup;
      });
    },
    xScale() {
      return d3
        .scaleBand()
        .domain(this.dataset.values.map((d) => this.xAccessor(d)))
        .range([this.dimensions.margins, this.ctrWidth])
        .padding(0.1);
    },
    yScale() {
      return d3
        .scaleLinear()
        .domain([
          0,
          d3.max(this.stackData, (dataGroup) => {
            return d3.max(dataGroup, (state) => state[1]);
          }),
        ])
        .range([this.ctrHeight, 0])
        .nice();
    },
    colorScale() {
      return d3
        .scaleOrdinal()
        .domain(this.stackData.map((d) => d.key))
        .range(d3.schemeSpectral[this.stackData.length])
        .unknown('#ccc');
    },
    yAxis() {
      return d3
        .axisLeft(this.yScale)
        .tickFormat((d) => d + '%')
        .tickValues([0, 50, 100]);
    },
    xAxis() {
      return d3.axisBottom(this.xScale).tickFormat(d3.timeFormat('%a'));
    },
  },
  methods: {
    xAccessor(d) {
      const parseDate = d3.timeParse('%Y-%m-%d');
      return parseDate(d.date);
    },
    yAccessor(d) {
      return parseInt(d.y);
    },
    drawAxis() {
      d3.select(`#axisGroupY_${this.id}`)
        .classed('noStroke', true)
        .classed('axis', true)
        .classed('axis-grid', true)
        .call(this.yAxis);
      d3.select(`#axisGroupX_${this.id}`)
        .style('transform', `translateY(${this.ctrHeight}px`)
        .classed('noStroke', true)
        .call(this.xAxis);
    },
    drawBars() {
      const barGroups = d3
        .select(`#barGroup_${this.id}`)
        .selectAll('g')
        .data(this.stackData)
        .join('g');

      barGroups
        .selectAll('rect')
        .data((d) => d)
        .join('rect')
        .attr('x', (d) => this.xScale(this.xAccessor(d.data)))
        .attr('y', (d) => this.yScale(d[1]))
        .attr('width', this.xScale.bandwidth())
        .attr('height', (d) => this.yScale(d[0]) - this.yScale(d[1]))
        .attr('fill', (d) => this.dataset.colors[d.key])
        .on('mouseover', function(event, d) {
          const name = d3.select(this.parentNode).datum().key;
          const value = d.data[name];
          const tooltip = d3.select(`#tooltip_bar1`);
          tooltip.style('display', 'block');
          tooltip.select('.data').text(`${name}: ${value}`);
        })
        .on('mouseout', function() {
          const tooltip = d3.select(`#tooltip_bar1`);
          tooltip.style('display', 'none');
        });
    },
    draw() {
      this.drawBars();
      this.drawAxis();
    },
  },
  watch: {
    dataset: {
      deep: true,
      handler() {
        this.draw();
      },
    },
  },
};
</script>

<style scoped>
.chart {
  height: 350px;
  margin: 25px auto;
  position: relative;
  background-color: #efefef;
}
.tooltip {
  border: 1px solid #ccc;
  position: absolute;
  padding: 10px;
  background-color: #fff;
  display: none;
  pointer-events: none;
  border-radius: 10px;
  width: 120px;
  height: 40px;
}

.data {
  font-weight: bold;
}

.noStroke path,
.noStroke line {
  stroke: none;
}
.axis {
  font-style: 'SemiBold';
  font-size: 14px;
}
.axis-grid line {
  stroke: #ccc;
}
</style>
