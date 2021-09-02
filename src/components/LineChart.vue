<template>
  <div v-if="dataset != null" class="chart" id="lineChart">
    <div class="tooltip" id="lineChartTooltip">
      <div class="data"></div>
      <div class="date"></div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'LineChart',
  props: {
    dataset: {
      type: Array,
      required: false,
    },
  },
  data() {
    return {
      data: null,
      tooltipPosition: { top: 10, right: 50 },
      dimensions: { width: 1000, height: 500, margins: 50, padding: 10 },
    };
  },
  computed: {
    ctrWidth() {
      return this.dimensions.width - this.dimensions.margins * 2;
    },
    ctrHeight() {
      return this.dimensions.height - this.dimensions.margins * 2;
    },
    xScale() {
      return d3
        .scaleUtc()
        .domain(d3.extent(this.mergedData, this.xAccessor))
        .range([0, this.ctrWidth - this.dimensions.padding]);
    },
    yScale() {
      return d3
        .scaleLinear()
        .domain(d3.extent(this.mergedData, this.yAccessor))
        .range([this.ctrHeight, 0])
        .nice();
    },
    lineGenerator() {
      return d3
        .line()
        .x((d) => this.xScale(this.xAccessor(d)))
        .y((d) => this.yScale(this.yAccessor(d)));
    },
    yAxis() {
      return d3
        .axisLeft(this.yScale)
        .tickSize(-this.ctrWidth)
        .tickFormat((d) => `${d}%`)
        .tickValues([0, 50, 100]);
    },
    xAxis() {
      const tickValues = this.dataset[0].values.map((el) => new Date(el.x));
      return d3
        .axisBottom(this.xScale)
        .tickFormat(d3.timeFormat('%a'))
        .tickValues(tickValues);
    },
    mergedData() {
      const mergedDataset = this.dataset.map((el) => el.values);
      return mergedDataset.flat();
    },
  },
  methods: {
    xAccessor(d) {
      const parseDate = d3.timeParse('%Y-%m-%d');
      return parseDate(d.x);
    },
    yAccessor(d) {
      return parseInt(d.y);
    },
    drawContainer() {
      const svg = d3
        .select('#lineChart')
        .append('svg')
        .attr('width', this.dimensions.width)
        .attr('height', this.dimensions.height);

      const ctr = svg
        .append('g')
        .attr(
          'transform',
          `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
        );
      return ctr;
    },
    drawAxis(ctr) {
      ctr
        .append('g')
        .classed('noStroke', true)
        .classed('axis', true)
        .classed('axis-grid', true)
        .call(this.yAxis);
      ctr
        .append('g')
        .style('transform', `translateY(${this.ctrHeight}px`)
        .classed('noStroke', true)
        .call(this.xAxis);
    },
    drawTooltipBisector(ctr, tooltip, tooltipDots, that) {
      ctr
        .append('rect')
        .attr('width', this.ctrWidth)
        .attr('height', this.ctrHeight)
        .style('opacity', 0)
        .on('touchmouse mousemove', function(event) {
          const mousePos = d3.pointer(event, this);
          const date = that.xScale.invert(mousePos[0]);
          // Custom Bisector - left, center, right
          const bisector = d3.bisector(that.xAccessor).left;
          const index = bisector(that.dataset[0].values, date);
          const points = [];
          that.dataset.forEach((el) => {
            points.push(el.values[index - 1]);
          });
          if (points.findIndex((el) => el == null) !== -1) return;
          // Update Image
          tooltipDots.forEach((dot, dotIndex) => {
            dot
              .style('opacity', 1)
              .attr('cx', that.xScale(that.xAccessor(points[dotIndex])))
              .attr('cy', that.yScale(that.yAccessor(points[dotIndex])))
              .raise();
          });
          const text = points.map((el) => el.y);
          tooltip
            .style('display', 'block')
            .style('top', that.tooltipPosition.top + 'px')
            .style('left', that.ctrWidth - that.tooltipPosition.right + 'px');
          tooltip.select('.data').text(text);

          const dateFormatter = d3.timeFormat('%B %-d, %Y');
          tooltip.select('.date').text(`${dateFormatter(date)}`);
        })
        // eslint-disable-next-line no-unused-vars
        .on('mouseleave', function(event) {
          tooltipDots.forEach((dot) => {
            dot.style('opacity', 0);
          });
          tooltip.style('display', 'none');
        });
    },
    createToolTipDots(ctr, data) {
      const dots = [];
      data.forEach((el) => {
        const dot = ctr
          .append('circle')
          .attr('r', 5)
          .attr('fill', el.color)
          .attr('stroke', 'black')
          .attr('stroke-width', 2)
          .style('opacity', 0)
          .style('pointer-events', 'none');
        dots.push(dot);
      });
      return dots;
    },
    draw() {
      const ctr = this.drawContainer();
      const tooltip = d3.select('#lineChartTooltip');
      const tooltipDots = this.createToolTipDots(ctr, this.dataset);
      this.dataset.forEach((dataset) => {
        const line = ctr
          .append('path')
          .data([dataset.values])
          .attr('d', this.lineGenerator)
          .attr('fill', 'none')
          .attr('stroke', dataset.color)
          .attr('stroke-width', 2);

        if (dataset.dotted) line.attr('stroke-dasharray', '5,5');
      });
      this.drawAxis(ctr);

      // tooltip
      this.drawTooltipBisector(ctr, tooltip, tooltipDots, this);
    },
  },
  watch: {
    dataset() {
      this.draw();
    },
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Raleway:wght@600&display=swap');

* {
  font-family: 'Raleway', sans-serif;
}
.chart {
  margin: 25px auto;
  width: 1000px;
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
