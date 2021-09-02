<template>
  <div class="chart" id="barChart">
    <div class="tooltip" id="barChartTooltip">
      <div class="price"></div>
      <div class="date"></div>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'BarChart',
  data() {
    return {
      data: null,
      dimensions: { width: 1000, height: 500, margins: 50 },
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
        .domain(d3.extent(this.data, this.xAccessor))
        .range([0, this.ctrWidth]);
    },
    yScale() {
      return d3
        .scaleLinear()
        .domain(d3.extent(this.data, this.yAccessor))
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
      return d3.axisLeft(this.yScale).tickFormat((d) => `$${d}`);
    },
    xAxis() {
      return d3.axisBottom(this.xScale);
    },
  },
  methods: {
    xAccessor(d) {
      const parseDate = d3.timeParse('%Y-%m-%d');
      return parseDate(d.date);
    },
    yAccessor(d) {
      return parseInt(d.close);
    },
    draw() {
      const svg = d3
        .select('#barChart')
        .append('svg')
        .attr('width', this.dimensions.width)
        .attr('height', this.dimensions.height);

      const ctr = svg
        .append('g')
        .attr(
          'transform',
          `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
        );

      const tooltip = d3.select('#barChartTooltip');
      const tooltipDot = ctr
        .append('circle')
        .attr('r', 5)
        .attr('fill', '#fc8781')
        .attr('stroke', 'black')
        .attr('stroke-width', 2)
        .style('opacity', 0)
        .style('pointer-events', 'none');

      ctr
        .append('path')
        .datum(this.data)
        .attr('d', this.lineGenerator)
        .attr('fill', 'none')
        .attr('stroke', '#30475e')
        .attr('stroke-width', 2);
      ctr.append('g').call(this.yAxis);
      ctr
        .append('g')
        .style('transform', `translateY(${this.ctrHeight}px`)
        .call(this.xAxis);
      const that = this;

      // tooltip
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
          const index = bisector(that.data, date);
          const stock = that.data[index - 1];
          if (stock == null) return;
          // Update Image
          tooltipDot
            .style('opacity', 1)
            .attr('cx', that.xScale(that.xAccessor(stock)))
            .attr('cy', that.yScale(that.yAccessor(stock)))
            .raise();
          tooltip
            .style('display', 'block')
            .style('top', that.yScale(that.yAccessor(stock)) - 20 + 'px')
            .style('left', that.xScale(that.xAccessor(stock)) + 'px');
          tooltip.select('.price').text(`$${that.yAccessor(stock)}`);

          const dateFormatter = d3.timeFormat('%B %-d, %Y');
          tooltip
            .select('.date')
            .text(`${dateFormatter(that.xAccessor(stock))}`);
        })
        // eslint-disable-next-line no-unused-vars
        .on('mouseleave', function(event) {
          tooltipDot.style('opacity', 0);

          tooltip.style('display', 'none');
        });
    },
  },
  async mounted() {
    //this.data = await d3.csv('data.csv');
    //this.draw();
  },
};
</script>

<style scoped>
.chart {
  margin: 25px auto;
  width: 1000px;
  position: relative;
}
.tooltip {
  border: 1px solid #ccc;
  position: absolute;
  padding: 10px;
  background-color: #fff;
  display: none;
  pointer-events: none;
}

.price {
  font-weight: bold;
}
</style>
