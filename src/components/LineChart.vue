<template>
  <div class="chart" :id="id">
    <svg :id="`svg_${id}`"></svg>
    <div class="tooltip-window" :id="`lineChartTooltipDiv_${id}`">
      <svg :id="`lineChartTooltip_${id}`" height="100%" width="100%"></svg>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'LineChart',
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
      width: 0,
      height: 0,
      margins: 30,
      xScalePadding: 10,
      resizeListener: null,
    };
  },
  computed: {
    ctrWidth() {
      return this.width - this.margins * 2;
    },
    ctrHeight() {
      return this.height - this.margins * 2;
    },
    xScale() {
      return d3
        .scaleUtc()
        .domain(d3.extent(this.mergedData, this.xAccessor))
        .range([0, this.ctrWidth - this.xScalePadding]);
    },
    yScale() {
      return d3
        .scaleLinear()
        .domain([0, 100])
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
      const dates = this.dataset.data[0].values.map((el) => new Date(el.x));
      let tickValues = [];
      let timeFormat = '';
      if (this.dataset.resolution === 'HOUR') {
        tickValues = dates.filter((el, i) => i % 2 === 0);
        timeFormat = '%H:%M';
      } else if (this.dataset.resolution === 'DAY') {
        tickValues = dates;
        timeFormat = '%a';
      } else if (this.dataset.resolution === 'WEEK') {
        tickValues = dates;
        timeFormat = 'V.%V';
      }
      return d3
        .axisBottom(this.xScale)
        .tickFormat(d3.timeFormat(timeFormat))
        .tickValues(tickValues);
    },
    mergedData() {
      const mergedDataset = this.dataset.data.map((el) => el.values);
      return mergedDataset.flat();
    },
  },
  methods: {
    xAccessor(d) {
      const date = new Date(Date.parse(d.x));
      return date;
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
      d3.select(`#tooltipBisectorLineChart_${that.id}`)
        .on('touchmouse mousemove', function(event) {
          that.onHoverChart(event, this);
        })
        .on('mouseleave', function() {
          that.onHoverLeaveChart();
        });
    },
    onHoverChart(event, rectObj) {
      const tooltip = d3.select(`#lineChartTooltipDiv_${this.id}`);
      const mousePos = d3.pointer(event, rectObj);
      const date = this.xScale.invert(mousePos[0]);
      // Custom Bisector - left, center, right
      const bisector = d3.bisector(this.xAccessor).left;
      const index = bisector(this.dataset.data[0].values, date);
      const points = [];
      this.dataset.data.forEach((el) => {
        points.push({
          ...el.values[index - 1],
          name: el.name,
          visible: el.visible,
        });
      });
      if (points.findIndex((el) => el == null) !== -1) return;
      // Update Image
      d3.select(`#tooltipDotsLineChart_${this.id}`)
        .selectAll('circle')
        .data(points)
        .style('opacity', (d) => (d.visible ? 1 : 0))
        .attr('cx', (d) => this.xScale(this.xAccessor(d)))
        .attr('cy', (d) => this.yScale(this.yAccessor(d)));
      tooltip
        .style('display', 'block')
        .style('top', this.tooltipPosition.top + 'px')
        .style('left', this.ctrWidth - this.tooltipPosition.right + 'px');
      tooltip
        .selectAll('text')
        .data(points)
        .text((d) => `${d.name}: ${Math.round(d.y)}%`);
    },
    onHoverLeaveChart() {
      const tooltip = d3.select(`#lineChartTooltipDiv_${this.id}`);
      d3.select(`#tooltipDotsLineChart_${this.id}`)
        .selectAll('circle')
        .style('opacity', 0);
      tooltip.style('display', 'none');
    },
    createToolTipDot() {
      const filtredData = this.dataset.data.filter((el) => el.values.length);
      d3.select(`#tooltipDotsLineChart_${this.id}`)
        .raise()
        .selectAll('circle')
        .data(filtredData)
        .join((enter) => enter.append('circle'))
        .attr('r', 5)
        .attr('fill', (d) => d.color)
        .attr('stroke', 'black')
        .attr('stroke-width', 1.5)
        .style('opacity', 0)
        .style('pointer-events', 'none');
    },
    createTooltipWindow() {
      const filtredData = this.dataset.data.filter((el) => el.values.length);
      const tooltipWindow = d3.select(`#lineChartTooltip_${this.id}`);
      tooltipWindow
        .selectAll('line')
        .data(filtredData)
        .join((enter) => enter.append('line'))
        .attr('x1', 0)
        .attr('x2', 25)
        .attr('y1', (_d, i) => 5 + i * 12)
        .attr('y2', (_d, i) => 5 + i * 12)
        .attr('stroke', (d) => d.color)
        .attr('stroke-width', 4)
        .attr('stroke-dasharray', (d) => (d.dashed ? '5,5' : null));
      tooltipWindow
        .selectAll('text')
        .data(filtredData)
        .join((enter) => enter.append('text'))
        .attr('x', 30)
        .attr('y', (_d, i) => 8 + i * 12)
        .text((d) => d.name)
        .attr('fill', '#6A6A6A')
        .style('font-size', '10px');
    },
    drawLine() {
      d3.select(`#lineGroupLineChart_${this.id}`)
        .selectAll('path')
        .data(this.dataset.data)
        .join((enter) => enter.append('path'))
        .transition()
        .duration(500)
        .attr('d', (d) => this.lineGenerator(d.values))
        .attr('id', (d) => 'line' + d.id)
        .attr('fill', 'none')
        .attr('stroke', (d) => d.color)
        .attr('stroke-width', 2)
        .attr('stroke-dasharray', (d) => (d.dashed ? '5,5' : null))
        .style('opacity', (d) => (d.visible ? 1 : 0));
    },
    drawSvg() {
      const currentWidth = parseInt(
        d3.select(`#${this.id}`).style('width'),
        10
      );
      const currentHeight = parseInt(
        d3.select(`#${this.id}`).style('height'),
        10
      );
      this.width = currentWidth;
      this.height = currentHeight;
      const svg = d3.select(`#svg_${this.id}`);
      svg.attr('width', currentWidth);
      svg.attr('height', currentHeight);
    },
    initSvg() {
      this.drawSvg();
      const svg = d3.select(`#svg_${this.id}`);
      const ctr = svg
        .append('g')
        .attr('transform', `translate(${this.margins}, ${this.margins})`);
      ctr.append('g').attr('id', `tooltipDotsLineChart_${this.id}`);
      ctr.append('g').attr('id', `lineGroupLineChart_${this.id}`);
      ctr.append('g').attr('id', `axisGroupYLineChart_${this.id}`);
      ctr.append('g').attr('id', `axisGroupXLineChart_${this.id}`);
      ctr
        .append('rect')
        .attr('width', this.ctrWidth)
        .attr('height', this.ctrHeight)
        .style('opacity', 0)
        .attr('id', `tooltipBisectorLineChart_${this.id}`);
    },
    init() {
      this.initSvg();
      this.drawLine();
      this.drawAxis();
      this.createToolTipDot();
      this.createTooltipWindow();
      this.drawTooltipBisector(this);
    },
    draw() {
      this.drawLine();
      this.drawAxis();
      this.createToolTipDot();
      this.createTooltipWindow();
    },
  },
  mounted() {
    this.init();
    this.resizeListener = () => {
      this.drawSvg();
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
.tooltip-window {
  border: 1px solid #ccc;
  position: absolute;
  padding: 5px;
  background-color: #fff;
  display: none;
  pointer-events: none;
  border-radius: 10px;
  width: 150px;
  height: 65px;
}
</style>
