<template>
  <div v-if="dataset != null" class="chart">
    <svg :height="dimensions.height" :width="dimensions.width">
      <g
        :transform="
          `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
        "
      >
        <g :id="`axisGroupY_${id}`"></g>
        <g :id="`barGroup_${id}`"></g>
        <g :id="`axisGroupX_${id}`"></g>
      </g>
    </svg>
    <div class="tooltip" :id="`barChartTooltipDiv_${id}`">
      <svg :id="`barChartTooltip_${id}`" :height="'100%'" :width="'100%'"></svg>
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
        .range([0, this.ctrWidth])
        .padding(0.5);
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
    yAxis() {
      return d3
        .axisLeft(this.yScale)
        .tickFormat((d) => d + '%')
        .tickSize(-this.ctrWidth, 0, 0)
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
      const yAxis = d3
        .select(`#axisGroupY_${this.id}`)
        .classed('axis', true)
        .call(this.yAxis)
        .call((g) => g.select('.domain').remove());
      yAxis
        .selectAll('g')
        .selectAll('line')
        .attr('stroke', '#BABABA');
      yAxis
        .selectAll('g')
        .selectAll('text')
        .attr('fill', '#6A6A6A');
      const xAxis = d3
        .select(`#axisGroupX_${this.id}`)
        .style('transform', `translateY(${this.ctrHeight}px`)
        .call(this.xAxis)
        .call((g) => g.select('.domain').remove())
        .call((g) => g.selectAll('line').remove());
      xAxis
        .selectAll('g')
        .selectAll('text')
        .attr('fill', '#6A6A6A');
    },
    drawBars() {
      const that = this;
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
        .attr('rx', 1)
        .attr('ry', 1)
        .on('mouseover', function(event, d) {
          const name = that.dataset.names[d.key];
          const color = that.dataset.colors[d.key];
          const value = d.data[d.key];
          const tooltip = d3.select(`#barChartTooltipDiv_${that.id}`);
          const mousePos = d3.pointer(event, this);
          tooltip
            .style('display', 'block')
            .style('top', mousePos[1] + 15 + 'px')
            .style('left', mousePos[0] + 25 + 'px');
          tooltip.selectAll('circle').attr('fill', color);
          tooltip.selectAll('#tooltipText').text(`${name}: ${value}%`);
          d3.select(this).style('opacity', 0.7);
        })
        .on('mouseout', function() {
          const tooltip = d3.select(`#barChartTooltipDiv_${that.id}`);
          tooltip.style('display', 'none');
          d3.select(this).style('opacity', 1);
        });
    },
    drawToolTip() {
      d3.select(`#barChartTooltip_${this.id}`)
        .append('circle')
        .attr('cx', 10)
        .attr('cy', 10)
        .attr('r', 6)
        .attr('fill', '#000');
      d3.select(`#barChartTooltip_${this.id}`)
        .append('text')
        .attr('x', 20)
        .attr('y', 13)
        .attr('id', 'tooltipText')
        .attr('fill', '#000')
        .style('font-size', '10px');
    },
    draw() {
      this.drawToolTip();
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
  height: 20px;
  width: 170px;
  border: 1px solid #ccc;
  position: absolute;
  padding: 5px;
  background-color: #fff;
  display: none;
  pointer-events: none;
  border-radius: 10px;
}
.data {
  font-weight: bold;
}
.axis {
  font-style: 'SemiBold';
  font-size: 14px;
}
</style>
