<template>
  <div class="chart" :id="id">
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
    <svg :height="dimensions.height" :width="dimensions.width">
      <g
        id="ctr"
        :transform="
          `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
        "
      >
        <g :id="`tooltipDotsStepLineChart_${id}`"></g>
        <g :id="`lineGroupStepLineChart_${id}`"></g>
        <g :id="`axisGroupYStepLineChart_${id}`"></g>
        <g :id="`axisGroupXStepLineChart_${id}`"></g>
        <rect
          :id="`tooltipBisectorStepLineChart_${id}`"
          :width="ctrWidth"
          :height="ctrHeight"
          style="opacity: 0;"
        ></rect>
      </g>
    </svg>
    <div class="tooltip" :id="`tooltipStepLineChart_${id}`">
      <svg
        :id="`lineStepChartTooltip_${id}`"
        :height="'100%'"
        :width="'100%'"
      ></svg>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'StepLineChart',
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
        .scaleLinear()
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
      return d3
        .axisBottom(this.xScale)
        .tickFormat((d) => `${d}%`)
        .tickValues([0, 50, 100]);
    },
    mergedData() {
      const mergedDataset = this.dataset.map((el) => el.values);
      return mergedDataset.flat();
    },
  },
  methods: {
    xAccessor(d) {
      return parseInt(d.x);
    },
    yAccessor(d) {
      return parseInt(d.y);
    },
    drawAxis() {
      d3.select(`#axisGroupYStepLineChart_${this.id}`)
        .call(this.yAxis)
        .call((g) => g.select('.domain').remove());
      d3.select(`#axisGroupXStepLineChart_${this.id}`)
        .style('transform', `translateY(${this.ctrHeight}px`)
        .call(this.xAxis)
        .call((g) => g.select('.domain').remove())
        .call((g) => g.selectAll('line').remove());
    },
    drawTooltipBisector(that) {
      const tooltip = d3.select(`#tooltipStepLineChart_${this.id}`);
      d3.select(`#tooltipBisectorStepLineChart_${this.id}`)
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
          d3.select(`#tooltipDotsStepLineChart_${that.id}`)
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

          tooltip.select('.date').text(Math.floor(date) + '%');
        })
        // eslint-disable-next-line no-unused-vars
        .on('mouseleave', function(event) {
          d3.select(`#tooltipDotsStepLineChart_${that.id}`)
            .selectAll('circle')
            .style('opacity', 0);
          tooltip.style('display', 'none');
        });
    },
    createToolTipDots() {
      d3.select(`#tooltipDotsStepLineChart_${this.id}`)
        .selectAll('circle')
        .data(this.dataset)
        .join((enter) => enter.append('circle'))
        .attr('r', 5)
        .attr('fill', (d) => d.color)
        .attr('stroke', 'black')
        .attr('stroke-width', 2)
        .style('opacity', 0)
        .style('pointer-events', 'none');

      d3.select(`#lineStepChartTooltip_${this.id}`)
        .selectAll('line')
        .data(this.dataset)
        .join((enter) => enter.append('line'))
        .attr('x1', 0)
        .attr('x2', 25)
        .attr('y1', (_d, i) => 10 + i * 20)
        .attr('y2', (_d, i) => 10 + i * 20)
        .attr('stroke', (d) => d.color)
        .attr('stroke-width', 5)
        .attr('stroke-dasharray', (d) => (d.dotted ? '5,5' : null));
      d3.select(`#lineStepChartTooltip_${this.id}`)
        .selectAll('text')
        .data(this.dataset)
        .join((enter) => enter.append('text'))
        .attr('x', 30)
        .attr('y', (_d, i) => 13 + i * 20)
        .text((d) => d.name)
        .attr('fill', '#000')
        .style('font-size', '10px');
    },
    drawLine() {
      let usageLine, bookingLine;
      this.dataset.forEach((data, index) => {
        let line = 'M';
        data.values.forEach((d, i) => {
          const y0 = this.yScale(this.yAccessor(d));
          const x0 = this.xScale(this.xAccessor(d));
          if (i === 0) {
            line += `${x0},${y0}H${x0}`;
          } else {
            line += `H${x0}`;
          }
          if (data.values[i + 1]) {
            line += `V${this.yScale(this.yAccessor(data.values[i + 1]))}`;
          }
        });
        if (index === 0) usageLine = line;
        else if (index === 1) bookingLine = line;
      });
      const data = [
        {
          line: usageLine,
          color: this.dataset[0].color,
          visible: this.dataset[0].visible,
        },
        {
          line: bookingLine,
          color: this.dataset[1].color,
          visible: this.dataset[1].visible,
        },
      ];
      d3.select(`#lineGroupStepLineChart_${this.id}`)
        .selectAll('path')
        .data(data)
        .join((enter) => enter.append('path'))
        .transition()
        .duration(500)
        .attr('d', (d) => d.line)
        .attr('id', 1)
        .attr('fill', 'none')
        .attr('stroke', (d) => d.color)
        .attr('stroke-width', 2)
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

.grid line {
  stroke: red;
  stroke-opacity: 0.7;
  shape-rendering: crispEdges;
}
.grid path {
  stroke-width: 1;
}
</style>
