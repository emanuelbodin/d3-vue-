<template>
  <div class="chart" :id="id">
    <svg :id="`xAxisSvg_${id}`"></svg>
    <div :id="`horizontalBarScrollContainer_${id}`" class="scroll-container">
      <svg :id="`svg_${id}`"></svg>
    </div>
    <div class="tooltip" :id="`horizontalBarChartTooltipDiv_${id}`">
      <svg
        :id="`horizontalBarChartTooltip_${id}`"
        height="100%"
        width="100%"
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
      tooltipPosition: { top: 10, right: 90 },
      xAxisSvgHeight: 50,
      width: 0,
      height: 0,
      margins: 30,
      resizeListener: null,
      dominantBaseline: 'text-bottom',
      textOffset: 10,
    };
  },
  computed: {
    svgHeight() {
      if (!this.dataset.values.length) return 0;
      return this.dataset.values.length * 40 + 100;
    },
    ctrWidth() {
      return this.width - this.margins * 2;
    },
    ctrHeight() {
      return this.svgHeight - this.margins * 2;
    },
    stackGenerator() {
      const columns = this.dataset.values.length
        ? this.dataset.values.columns.slice(1)
        : [];
      return d3.stack().keys(columns);
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
      if (!this.dataset.values.length) return;
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
          that.onHoverChart(d, this);
        })
        .on('mouseout', function() {
          that.onHoverLeaveChart();
        });
    },
    onHoverChart(d, rectObj) {
      const name = this.dataset.names[d.key];
      const color = this.dataset.colors[d.key];
      const value = d.data[d.key];
      const tooltip = d3.select(`#horizontalBarChartTooltipDiv_${this.id}`);
      //const mousePos = d3.pointer(event, this);
      tooltip
        .style('display', 'block')
        .style('top', 0 + 'px')
        .style('left', this.ctrWidth - 100 + 'px');
      tooltip.selectAll('circle').attr('fill', color);
      tooltip.selectAll('#tooltipText').text(`${name}: ${value}%`);
      d3.select(rectObj).style('opacity', 0.7);
    },
    onHoverLeaveChart(rectObj) {
      const tooltip = d3.select(`#horizontalBarChartTooltipDiv_${this.id}`);
      tooltip.style('display', 'none');
      d3.select(rectObj).style('opacity', 1);
    },
    drawBarLabels() {
      d3.select(`#horizontalBarLabels_${this.id}`)
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
    createTooltipWindow() {
      const tooltipWindow = d3.select(`#horizontalBarChartTooltip_${this.id}`);
      tooltipWindow
        .append('g')
        .attr('id', `horizontalBarTooltipWindowYLabels_${this.id}`)
        .append('circle')
        .attr('cx', 10)
        .attr('cy', 10)
        .attr('r', 6)
        .attr('fill', '#000');
      tooltipWindow
        .append('g')
        .attr('id', `horizontalBarTooltipWindowXLabel_${this.id}`)
        .append('text')
        .attr('x', 20)
        .attr('y', 13)
        .attr('id', 'tooltipText')
        .attr('fill', '#000')
        .style('font-size', '10px');
    },
    drawSvg() {
      const currentWidth = parseInt(
        d3.select(`#horizontalBarScrollContainer_${this.id}`).style('width'),
        10
      );
      this.width = currentWidth;
      const xAxisSvg = d3.select(`#xAxisSvg_${this.id}`);
      const barSvg = d3.select(`#svg_${this.id}`);
      xAxisSvg.attr('width', currentWidth);
      xAxisSvg.attr('height', this.xAxisSvgHeight);
      barSvg.attr('width', currentWidth);
      barSvg.attr('height', this.svgHeight);
    },
    initSvg() {
      this.drawSvg();
      const barSvg = d3.select(`#svg_${this.id}`);
      const xAxisSvg = d3.select(`#xAxisSvg_${this.id}`);
      xAxisSvg
        .append('g')
        .attr('transform', `translate(${this.margins}, ${this.margins})`)
        .attr('id', `horizontalBarAxisGroupX_${this.id}`);
      const ctr = barSvg
        .append('g')
        .attr('transform', `translate(${this.margins}, ${this.margins})`);
      ctr.append('g').attr('id', `horizontalBarLabels_${this.id}`);
      ctr.append('g').attr('id', `horizontalBarGroup_${this.id}`);
      ctr.append('g').attr('id', `horizontalBarAxisGroupY_${this.id}`);
    },
    init() {
      this.initSvg();
      this.drawBars();
      this.drawBarLabels();
      this.drawAxis();
      this.createTooltipWindow();
    },
    draw() {
      this.drawSvg();
      this.drawBars();
      this.drawBarLabels();
      this.drawAxis();
    },
  },
  mounted() {
    this.init();
    this.resizeListener = () => {
      this.draw();
    };
    window.addEventListener('resize', this.resizeListener);
  },
  watch: {
    dataset() {
      this.draw();
    },
  },
  destroyed() {
    window.removeEventListener('resize', this.resizeListener);
  },
};
</script>

<style scoped>
.chart {
  height: 100%;
  width: 100%;
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
</style>
