<template>
  <div class="chart" :id="id">
    <svg :id="`svg_${id}`"></svg>
    <div class="tooltip" :id="`barChartTooltipDiv_${id}`">
      <svg :id="`barChartTooltip_${id}`" height="100%" width="100%"></svg>
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
    timeFormat() {
      if (this.dataset.timeSetting === 'DAY') return '%H:%M';
      else if (this.dataset.timeSetting === 'WEEK') return '%a';
      else if (this.dataset.timeSetting === 'MONTH') return '%d/%m';
      else if (this.dataset.timeSetting === 'YEAR') return 'V.%V';
      else return '';
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
      return d3
        .axisBottom(this.xScale)
        .tickFormat(d3.timeFormat(this.timeFormat));
    },
  },
  methods: {
    xAccessor(d) {
      const date = new Date(Date.parse(d.date));
      return date;
    },
    yAccessor(d) {
      return parseInt(d.y);
    },
    drawAxis() {
      if (!this.dataset.values.length) return;
      const yAxis = d3
        .select(`#axisGroupYBarChart_${this.id}`)
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
        .select(`#axisGroupXBarChart_${this.id}`)
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
        .select(`#barGroupBarChart_${this.id}`)
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
          that.onHoverChart(event, this, d);
        })
        .on('mouseout', function() {
          that.onHoverLeaveChart(this);
        });
    },
    onHoverChart(event, rectEl, d) {
      const name = this.dataset.names[d.key];
      const color = this.dataset.colors[d.key];
      const value = d.data[d.key];
      const tooltip = d3.select(`#barChartTooltipDiv_${this.id}`);
      const mousePos = d3.pointer(event, rectEl);
      tooltip
        .style('display', 'block')
        .style('top', mousePos[1] + 15 + 'px')
        .style('left', mousePos[0] + 25 + 'px');
      tooltip.selectAll('circle').attr('fill', color);
      tooltip.selectAll('#tooltipText').text(`${name}: ${value}%`);
      d3.select(rectEl).style('opacity', 0.7);
    },
    onHoverLeaveChart(rectEl) {
      const tooltip = d3.select(`#barChartTooltipDiv_${this.id}`);
      tooltip.style('display', 'none');
      d3.select(rectEl).style('opacity', 1);
    },
    createTooltipWindow() {
      const tooltipWindow = d3.select(`#barChartTooltip_${this.id}`);
      tooltipWindow
        .append('g')
        .attr('id', `barTooltipWindowYLabels_${this.id}`)
        .append('circle')
        .attr('cx', 10)
        .attr('cy', 10)
        .attr('r', 6)
        .attr('fill', '#000');
      tooltipWindow
        .append('g')
        .attr('id', `barTooltipWindowXLabel_${this.id}`)
        .append('text')
        .attr('x', 20)
        .attr('y', 13)
        .attr('id', 'tooltipText')
        .attr('fill', '#000')
        .style('font-size', '10px');
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
      ctr.append('g').attr('id', `axisGroupYBarChart_${this.id}`);
      ctr.append('g').attr('id', `axisGroupXBarChart_${this.id}`);
      ctr.append('g').attr('id', `barGroupBarChart_${this.id}`);
    },
    init() {
      this.initSvg();
      this.drawBars();
      this.drawAxis();
      this.createTooltipWindow();
    },
    draw() {
      this.drawBars();
      this.drawAxis();
    },
  },
  mounted() {
    this.init();
    this.resizeListener = () => {
      this.drawSvg();
      d3.select(`#tooltipBisectorLineChart_${this.id}`)
        .attr('width', this.ctrWidth)
        .attr('height', this.ctrHeight);
      this.draw();
    };
    window.addEventListener('resize', this.resizeListener);
  },
  watch: {
    dataset() {
      this.draw();
    },
    destroyed() {
      window.removeEventListener('resize', this.resizeListener);
    },
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
