<template>
  <div v-if="dataset != null" class="chart">
    <svg :height="50" :width="dimensions.width">
      <g
        :id="`horizontalBarAxisGroupX_${id}`"
        :transform="`translate(${this.dimensions.margins}, ${30})`"
      ></g>
    </svg>
    <div class="scroll-container">
      <svg :height="svgHeight" :width="dimensions.width">
        <g
          :transform="
            `translate(${this.dimensions.margins}, ${this.dimensions.margins})`
          "
        >
          <g :id="`horizontalBarGroup_${id}`"></g>
          <g :id="`horizontalBarAxisGroupY_${id}`"></g>
        </g>
      </svg>
    </div>

    <div class="tooltip" :id="`horizontalBarTooltip_${id}`">
      <div class="data"></div>
      <div class="date"></div>
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
    };
  },
  computed: {
    svgHeight() {
      if (this.dataset.values == null) return 0;
      return this.dataset.values.length * 40 - this.dimensions.margins * 2;
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
      d3.select(`#horizontalBarAxisGroupY_${this.id}`)
        .classed('noStroke', true)
        .classed('axis', true)
        .classed('axis-grid', true)
        .call(this.yAxis)
        .call((g) => g.select('.domain').remove())
        .call((g) => g.selectAll('line').remove());
      d3.select(`#horizontalBarAxisGroupX_${this.id}`)
        .classed('noStroke', true)
        .style('position', 'fixed')
        .call(this.xAxis)
        .call((g) => g.select('.domain').remove())
        .call((g) => g.selectAll('line').remove());
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
        .attr('y', (d) => this.yScale(d.data.room))
        .attr('width', (d) => this.xScale(d[1]) - this.xScale(d[0]))
        .attr('height', this.yScale.bandwidth())
        .attr('fill', (d) => this.dataset.colors[d.key])
        .on('mouseover', function(event, d) {
          const name = that.dataset.names[d.key];
          const value = d.data[d.key];
          const tooltip = d3.select(`#horizontalBarTooltip_${that.id}`);
          const mousePos = d3.pointer(event, this);
          tooltip
            .style('display', 'block')
            .style('top', mousePos[1] + 'px')
            .style('left', mousePos[0] + 20 + 'px');
          tooltip.select('.data').text(`${name}: ${value}%`);
          d3.select(this).style('opacity', 0.7);
        })
        .on('mouseout', function() {
          const tooltip = d3.select(`#horizontalBarTooltip_${that.id}`);
          tooltip.style('display', 'none');
          d3.select(this).style('opacity', 1);
        });
    },
    draw() {
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
  margin: 10px auto;
}
.tooltip {
  border: 1px solid #ccc;
  position: absolute;
  padding: 5px;
  background-color: #fff;
  display: none;
  pointer-events: none;
  border-radius: 10px;
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
