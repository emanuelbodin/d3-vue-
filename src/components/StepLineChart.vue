<template>
  <div class="chart" :id="id">
    <svg :id="`svg_${id}`"></svg>
    <div class="tooltip-window" :id="`stepLineChartTooltipDiv_${id}`">
      <svg :id="`stepLineChartTooltip_${id}`" height="100%" width="100%"></svg>
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
export default {
  name: 'StepLineChart',
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
        .scaleLinear()
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
      const mergedDataset = this.dataset.data.map((el) => el.values);
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
      if (!this.mergedData.length) return;
      const yAxis = d3
        .select(`#axisGroupYStepLineChart_${this.id}`)
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
        .select(`#axisGroupXStepLineChart_${this.id}`)
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
      d3.select(`#tooltipBisectorStepLineChart_${this.id}`)
        .on('touchmouse mousemove', function(event) {
          that.onHoverChart(event, this);
        })
        .on('mouseleave', function() {
          that.onHoverLeaveChart();
        });
    },
    onHoverChart(event, rectObj) {
      const tooltip = d3.select(`#stepLineChartTooltipDiv_${this.id}`);
      const mousePos = d3.pointer(event, rectObj);
      const xValue = this.xScale.invert(mousePos[0]);
      // Custom Bisector - left, center, right
      const bisector = d3.bisector(this.xAccessor).left;
      const index = bisector(this.dataset.data[0].values, xValue);
      const points = [];
      this.dataset.data.forEach((el) => {
        points.push({ ...el.values[index - 1], name: el.name });
      });
      if (points.findIndex((el) => el == null) !== -1) return;
      // Update Image
      d3.select(`#tooltipDotsStepLineChart_${this.id}`)
        .selectAll('circle')
        .data(points)
        .style('opacity', 1)
        .attr('cx', (d) => this.xScale(this.xAccessor(d)))
        .attr('cy', (d) => this.yScale(this.yAccessor(d)))
        .raise();
      tooltip
        .style('display', 'block')
        .style('top', this.tooltipPosition.top + 'px')
        .style('left', this.ctrWidth - this.tooltipPosition.right + 'px');
      d3.select(`#stepLineTooltipWindowYLabels_${this.id}`)
        .selectAll('text')
        .data(points)
        .text((d) => `${d.name}: ${Math.round(d.y)}%`);
      d3.select(`#stepLineTooltipWindowXLabel_${this.id}`)
        .selectAll('text')
        .data([xValue])
        .join((enter) => enter.append('text'))
        .text(Math.round(xValue) + '%');
    },
    onHoverLeaveChart() {
      const tooltip = d3.select(`#stepLineChartTooltipDiv_${this.id}`);
      d3.select(`#tooltipDotsStepLineChart_${this.id}`)
        .selectAll('circle')
        .style('opacity', 0);
      tooltip.style('display', 'none');
    },
    createToolTipDots() {
      const filtredData = this.dataset.data.filter((el) => el.values.length);
      d3.select(`#tooltipDotsStepLineChart_${this.id}`)
        .raise()
        .selectAll('circle')
        .data(filtredData)
        .join((enter) => enter.append('circle'))
        .attr('r', 5)
        .attr('fill', (d) => d.color)
        .attr('stroke', 'black')
        .attr('stroke-width', 2)
        .style('opacity', 0)
        .style('pointer-events', 'none');
    },
    createTooltipWindow() {
      const tooltipWindow = d3.select(`#stepLineChartTooltip_${this.id}`);
      tooltipWindow
        .append('g')
        .attr('id', `stepLineTooltipWindowYLabels_${this.id}`);
      tooltipWindow
        .append('g')
        .attr('id', `stepLineTooltipWindowXLabel_${this.id}`);
    },
    updateTooltipWindow() {
      const filtredData = this.dataset.data.filter((el) => el.values.length);
      const tooltipWindowYlabels = d3.select(
        `#stepLineTooltipWindowYLabels_${this.id}`
      );
      tooltipWindowYlabels
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
      tooltipWindowYlabels
        .selectAll('text')
        .data(filtredData)
        .join((enter) => enter.append('text'))
        .attr('x', 30)
        .attr('y', (_d, i) => 8 + i * 12)
        .text((d) => d.name)
        .attr('fill', '#6A6A6A')
        .style('font-size', '10px');

      const tooltipWindowXlabels = d3.select(
        `#stepLineTooltipWindowXLabel_${this.id}`
      );
      tooltipWindowXlabels
        .selectAll('text')
        .data([1])
        .join((enter) => enter.append('text'))
        .attr('x', 50)
        .attr('y', 8 + (filtredData.length + 1) * 12)
        .text(100 + '%')
        .attr('fill', '#6A6A6A')
        .style('font-size', '14px');
    },
    drawLine() {
      const lines = [];
      this.dataset.data.forEach((data, index) => {
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
        if (!data.values.length) return;
        lines.push({
          line,
          color: this.dataset.data[index].color,
          visible: this.dataset.data[index].visible,
        });
      });
      d3.select(`#lineGroupStepLineChart_${this.id}`)
        .selectAll('path')
        .data(lines)
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
      ctr.append('g').attr('id', `tooltipDotsStepLineChart_${this.id}`);
      ctr.append('g').attr('id', `lineGroupStepLineChart_${this.id}`);
      ctr.append('g').attr('id', `axisGroupYStepLineChart_${this.id}`);
      ctr.append('g').attr('id', `axisGroupXStepLineChart_${this.id}`);
      ctr
        .append('rect')
        .attr('width', this.ctrWidth)
        .attr('height', this.ctrHeight)
        .style('opacity', 0)
        .attr('id', `tooltipBisectorStepLineChart_${this.id}`);
    },
    init() {
      this.initSvg();
      this.drawLine();
      this.drawAxis();
      this.createToolTipDots();
      this.createTooltipWindow();
      this.updateTooltipWindow();
      this.drawTooltipBisector(this);
    },
    draw() {
      this.drawLine();
      this.drawAxis();
      this.updateTooltipWindow();
      this.createToolTipDots();
    },
  },
  mounted() {
    this.init();
    this.resizeListener = () => {
      this.drawSvg();
      d3.select(`#tooltipBisectorStepLineChart_${this.id}`)
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
