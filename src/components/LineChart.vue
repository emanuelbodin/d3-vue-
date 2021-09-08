<template>
  <div v-if="dataset != null" class="chart" :id="id">
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
    <svg
      id="svg"
      :height="dimensions.height"
      :width="dimensions.width"
      reserveAspectRatio="none"
    >
      <g
        id="ctr"
        :transform="
          `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
        "
      >
        <g :id="`tooltipDotsLineChart_${id}`"></g>
        <g :id="`lineGroupLineChart_${id}`"></g>
        <g :id="`axisGroupYLineChart_${id}`"></g>
        <g :id="`axisGroupXLineChart_${id}`"></g>
        <rect
          :id="`tooltipBisectorLineChart_${id}`"
          :width="ctrWidth"
          :height="ctrHeight"
          style="opacity: 0;"
        ></rect>
      </g>
    </svg>
    <div class="tooltip" :id="`lineChartTooltipDiv_${id}`">
      <svg
        :id="`lineChartTooltip_${id}`"
        :height="'100%'"
        :width="'100%'"
      ></svg>
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
    id: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
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
      const yAxis = d3
        .select(`#axisGroupYLineChart_${this.id}`)
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
        .select(`#axisGroupXLineChart_${this.id}`)
        .style('transform', `translateY(${this.ctrHeight}px`)
        .call(this.xAxis)
        .call((g) => g.select('.domain').remove())
        .call((g) => g.selectAll('line').remove());
      xAxis
        .selectAll('g')
        .selectAll('text')
        .attr('fill', '#6A6A6A');
    },
    drawTooltipBisector(that) {
      const tooltip = d3.select(`#lineChartTooltipDiv_${this.id}`);
      d3.select(`#tooltipBisectorLineChart_${this.id}`)
        .on('touchmouse mousemove', function(event) {
          const mousePos = d3.pointer(event, this);
          const date = that.xScale.invert(mousePos[0]);
          // Custom Bisector - left, center, right
          const bisector = d3.bisector(that.xAccessor).left;
          const index = bisector(that.dataset[0].values, date);
          const points = [];
          that.dataset.forEach((el) => {
            points.push({ ...el.values[index - 1], name: el.name });
          });
          if (points.findIndex((el) => el == null) !== -1) return;
          // Update Image
          d3.select(`#tooltipDotsLineChart_${that.id}`)
            .selectAll('circle')
            .data(points)
            .style('opacity', 1)
            .attr('cx', (d) => that.xScale(that.xAccessor(d)))
            .attr('cy', (d) => that.yScale(that.yAccessor(d)))
            .raise();
          tooltip
            .style('display', 'block')
            .style('top', that.tooltipPosition.top + 'px')
            .style('left', that.ctrWidth - that.tooltipPosition.right + 'px');
          tooltip
            .selectAll('text')
            .data(points)
            .text((d) => `${d.name}: ${d.y}%`);

          const dateFormatter = d3.timeFormat('%B %-d, %Y');
          tooltip.select('.date').text(`${dateFormatter(date)}`);
        })
        .on('mouseleave', function() {
          d3.select(`#tooltipDotsLineChart_${that.id}`)
            .selectAll('circle')
            .style('opacity', 0);
          tooltip.style('display', 'none');
        });
    },
    createToolTip() {
      d3.select(`#tooltipDotsLineChart_${this.id}`)
        .selectAll('circle')
        .data(this.dataset)
        .join((enter) => enter.append('circle'))
        .attr('r', 5)
        .attr('fill', (d) => d.color)
        .attr('stroke', 'black')
        .attr('stroke-width', 2)
        .style('opacity', 0)
        .style('pointer-events', 'none');

      d3.select(`#lineChartTooltip_${this.id}`)
        .selectAll('line')
        .data(this.dataset)
        .join((enter) => enter.append('line'))
        .attr('x1', 0)
        .attr('x2', 25)
        .attr('y1', (_d, i) => 5 + i * 10)
        .attr('y2', (_d, i) => 5 + i * 10)
        .attr('stroke', (d) => d.color)
        .attr('stroke-width', 4)
        .attr('stroke-dasharray', (d) => (d.dotted ? '5,5' : null));
      d3.select(`#lineChartTooltip_${this.id}`)
        .selectAll('text')
        .data(this.dataset)
        .join((enter) => enter.append('text'))
        .attr('x', 30)
        .attr('y', (_d, i) => 8 + i * 10)
        .text((d) => d.name)
        .attr('fill', '#6A6A6A')
        .style('font-size', '10px');
    },
    drawLine() {
      d3.select(`#lineGroupLineChart_${this.id}`)
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
      this.createToolTip();
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
.tooltip-text {
  font-size: 4;
}
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

.grid line {
  stroke: red;
  stroke-opacity: 0.7;
  shape-rendering: crispEdges;
}
.grid path {
  stroke-width: 1;
}
</style>
