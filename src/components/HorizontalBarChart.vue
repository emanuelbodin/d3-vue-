<template>
  <div v-if="dataset != null" class="chart">
    <svg :height="50" :width="dimensions.width">
      <g
        :id="`horizontalBarAxisGroupX_${id}`"
        :transform="`translate(${this.dimensions.margins}, ${30})`"
      ></g>
    </svg>
    <div class="scroll-container">
      <svg
        :height="svgHeight"
        :width="dimensions.width"
        reserveAspectRatio="none"
      >
        <g :transform="`translate(${this.dimensions.margins}, 20)`">
          <g id="barLabels"></g>

          <g :id="`horizontalBarGroup_${id}`"></g>
          <g :id="`horizontalBarAxisGroupY_${id}`"></g>
        </g>
      </svg>
    </div>

    <div class="tooltip" :id="`horizontalBarChartTooltipDiv_${id}`">
      <svg
        :id="`horizontalBarChartTooltip_${id}`"
        :height="'100%'"
        :width="'100%'"
      ></svg>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'HorizontalBarChart',
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
      dominantBaseline: 'text-bottom',
      textOffset: 10,
    };
  },
  computed: {
    svgHeight() {
      if (this.dataset.values == null) return 0;
      return this.dataset.values.length * 40 + 100;
    },
    ctrWidth() {
      return this.dimensions.width - this.dimensions.margins * 2;
    },
    ctrHeight() {
      return this.svgHeight - this.dimensions.margins * 2;
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
        .scaleLinear()
        .domain([
          0,
          d3.max(this.stackData, (dataGroup) => {
            return d3.max(dataGroup, (state) => state[1]);
          }),
        ])
        .range([0, this.ctrWidth]);
    },
    yScale() {
      return d3
        .scaleBand()
        .domain(this.dataset.values.map((d) => this.xAccessor(d)))
        .range([0, this.ctrHeight])
        .padding(0.1);
    },
    yAxis() {
      return d3.axisLeft(this.yScale);
    },
    xAxis() {
      return d3
        .axisBottom(this.xScale)
        .tickFormat((d) => d + '%')
        .tickValues([0, 50, 100]);
    },
  },
  methods: {
    xAccessor(d) {
      return d.room;
    },
    yAccessor(d) {
      return parseInt(d.y);
    },
    drawAxis() {
      d3.select(`#horizontalBarAxisGroupX_${this.id}`)
        .style('position', 'fixed')
        .call(this.xAxis)
        .call((g) => g.select('.domain').remove())
        .call((g) => g.selectAll('line').remove())
        .selectAll('.tick')
        .selectAll('text')
        .attr('fill', '#6A6A6A')
        .style('font-size', 12);
    },
    drawBars() {
      const that = this;
      const barGroups = d3
        .select(`#horizontalBarGroup_${this.id}`)
        .selectAll('g')
        .data(this.stackData)
        .join('g');
      barGroups
        .selectAll('rect')
        .data((d) => d)
        .join('rect')
        .attr('x', (d) => this.xScale(d[0]))
        .attr(
          'y',
          (d) => this.yScale(d.data.room) + this.yScale.bandwidth() / 5
        )
        .attr('width', (d) => this.xScale(d[1]) - this.xScale(d[0]))
        .attr('height', this.yScale.bandwidth() / 4)
        .attr('fill', (d) => this.dataset.colors[d.key])
        .attr('rx', 1)
        .attr('ry', 1)
        .on('mouseover', function(event, d) {
          const name = that.dataset.names[d.key];
          const color = that.dataset.colors[d.key];
          const value = d.data[d.key];
          const tooltip = d3.select(`#horizontalBarChartTooltipDiv_${that.id}`);
          //const mousePos = d3.pointer(event, this);
          tooltip
            .style('display', 'block')
            .style('top', 0 + 'px')
            .style('left', that.ctrWidth - 100 + 'px');
          tooltip.selectAll('circle').attr('fill', color);
          tooltip.selectAll('#tooltipText').text(`${name}: ${value}%`);
          d3.select(this).style('opacity', 0.7);
        })
        .on('mouseout', function() {
          const tooltip = d3.select(`#horizontalBarChartTooltipDiv_${that.id}`);
          tooltip.style('display', 'none');
          d3.select(this).style('opacity', 1);
        });
      d3.select('#barLabels')
        .selectAll('text')
        .data(this.dataset.values)
        .join((enter) => enter.append('text'))
        .attr('x', this.xScale(0))
        .attr(
          'y',
          (d) =>
            this.yScale(d.room) + this.yScale.bandwidth() / 5 - this.textOffset
        )
        .attr('dominant-baseline', this.dominantBaseline)
        .style('fill', '#6A6A6A')
        .style('font-size', '12px')
        .text((d) => d.room);
    },
    drawToolTip() {
      d3.select(`#horizontalBarChartTooltip_${this.id}`)
        .append('circle')
        .attr('cx', 10)
        .attr('cy', 10)
        .attr('r', 6)
        .attr('fill', '#000');
      d3.select(`#horizontalBarChartTooltip_${this.id}`)
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
.scroll-container {
  height: 170px;
  overflow-y: scroll;
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
.bar-label {
  color: '#6A6A6A';
}
</style>
