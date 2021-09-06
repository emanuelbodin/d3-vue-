<template>
  <div v-if="dataset != null" class="chart" id="lineChart">
    <div class="legend-container">
      <div v-for="data in dataset" :key="data.id">
        <label>{{ data.name }}</label>
        <input
          type="checkbox"
          :id="data.id"
          :name="data.name"
          :checked="data.visible"
          @change="$emit('toggle', data.id)"
        />
      </div>
    </div>
    <svg id="svg" :height="dimensions.height" :width="dimensions.width">
      <g
        id="ctr"
        :transform="
          `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
        "
      >
        <g id="tooltipDots"></g>
        <g id="lineGroup"></g>
        <g id="axisGroupY"></g>
        <g id="axisGroupX"></g>
        <rect
          id="tooltipBisector"
          :width="ctrWidth"
          :height="ctrHeight"
          style="opacity: 0;"
        ></rect>
      </g>
    </svg>
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
      dimensions: { width: 600, height: 300, margins: 50, padding: 10 },
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
    drawAxis() {
      d3.select('#axisGroupY')
        .classed('noStroke', true)
        .classed('axis', true)
        .classed('axis-grid', true)
        .call(this.yAxis);
      d3.select('#axisGroupX')
        .style('transform', `translateY(${this.ctrHeight}px`)
        .classed('noStroke', true)
        .call(this.xAxis);
    },
    drawTooltipBisector(that) {
      const tooltip = d3.select('#lineChartTooltip');
      d3.select('#tooltipBisector')
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
          d3.select('#tooltipDots')
            .selectAll('circle')
            .data(points)
            .style('opacity', 1)
            .attr('cx', (d) => that.xScale(that.xAccessor(d)))
            .attr('cy', (d) => that.yScale(that.yAccessor(d)))
            .raise();
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
          d3.select('#tooltipDots')
            .selectAll('circle')
            .style('opacity', 0);
          tooltip.style('display', 'none');
        });
    },
    createToolTipDots() {
      d3.select('#tooltipDots')
        .selectAll('circle')
        .data(this.dataset)
        .join((enter) => enter.append('circle'))
        .attr('r', 5)
        .attr('fill', (d) => d.color)
        .attr('stroke', 'black')
        .attr('stroke-width', 2)
        .style('opacity', 0)
        .style('pointer-events', 'none');
    },
    drawLine() {
      d3.select('#lineGroup')
        .selectAll('path')
        .data(this.dataset)
        .join((enter) => enter.append('path'))
        .transition()
        .duration(500)
        .attr('d', (d) => this.lineGenerator(d.values))
        .attr('id', (d) => 'line' + d.id)
        .attr('fill', 'none')
        .attr('stroke', (d) => d.color)
        .attr('stroke-width', 2)
        .attr('stroke-dasharray', (d) => (d.dotted ? '5,5' : null))
        .style('opacity', (d) => (d.visible ? 1 : 0));
    },
    draw() {
      this.createToolTipDots();
      this.drawLine();
      this.drawAxis();
      this.drawTooltipBisector(this);
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
.legend-container {
  display: flex;
  justify-content: space-around;
}
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
