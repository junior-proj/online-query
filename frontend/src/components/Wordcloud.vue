<template>
  <div v-show="showed">
    <p v-if="isLoading">loading...</p>
    <div :id="`wordcloud-${userId}`">
      <p>{{ userId }}</p>
      <svg :width="width" :height="height" />
    </div>
  </div>
</template>

<script>
import * as d3 from 'd3';
import agent from '../api/agent';

const originalColor = '#003f5c';
const highlightColor = '#ff6361';

export default {
  props: {
    userId: {
      type: String,
      default: 'g10',
    },
    focusedContent: {
      type: String,
      default: '',
    },
    wordSize: {
      type: Array,
      default: () => [15, 60],
    },
    width: {
      type: Number,
      default: 600,
    },
    height: {
      type: Number,
      default: 400,
    },
  },

  data() {
    return {
      plot: null,
      isLoading: true,
      svg: null,
      showed: true,
    };
  },

  mounted() {
    agent
      .getWordcloud(this.userId)
      .then((resp) => {
        this.plot = resp.data;
        this.draw();
      })
      .finally(() => {
        this.isLoading = false;
      });
  },

  watch: {
    focusedContent() {
      if (!this.svg) return;
      let contentNotFound = true;
      this.svg.selectAll('text').style('fill', (d) => {
        if (this.focusedContent && d.word.includes(this.focusedContent)) {
          contentNotFound = false;
          return '#ff6361';
        }
        return originalColor;
      });
      this.showed = !contentNotFound || !this.focusedContent;
    },
  },

  methods: {
    draw() {
      const data = this.plot.sort((a, b) => (a.freq > b.freq ? -1 : 1));
      // set the dimensions and margins of the graph
      const margin = {
        top: 20,
        right: 20,
        bottom: 20,
        left: 20,
      };
      const width = this.width - margin.left - margin.right;
      const size = d3
        .scaleLinear()
        .domain(d3.extent(data.map((d) => d.freq)))
        .range(this.wordSize);

      // append the svg object to the body of the page
      const svg = d3
        .select(`#wordcloud-${this.userId} > svg`)
        .attr('style', 'outline: thin solid black;')
        .append('g')
        .attr('transform', `translate(${margin.left}, ${margin.top})`);
      this.svg = svg;

      svg
        .selectAll('text')
        .data(data)
        .enter()
        .append('text')
        .attr('text-anchor', 'start')
        .style('font-size', (d) => size(d.freq))
        .style('fill', originalColor)
        .text((d) => d.word)
        .append('tspan')
        .style('font-size', (d) => (size(d.freq) / 3) * 2)
        .style('fill', '#bc5090')
        .text((d) => d.freq)
        .attr('dy', '-0.6em');

      let x = 0;
      let y = 0;
      let previousRight = 0;
      svg.selectAll('text').attr('transform', function (d, i) {
        x = previousRight;
        if (i === 0) {
          y += parseInt(size(Number(d.freq)), 10) - 15;
        } else if (x + this.getComputedTextLength() > width) {
          x = 0;
          y += this.getBoundingClientRect().height;
        }
        previousRight = x + this.getComputedTextLength() + 5;
        return `translate(${x}, ${y})`;
      });

      const self = this;
      function handleMouseOver() { d3.select(this).style('fill', highlightColor); }
      function handleMouseOut() {
        d3.select(this).style('fill', (d) => {
          if (self.focusedContent && d.word.includes(self.focusedContent)) return highlightColor;
          return originalColor;
        });
      }

      svg.selectAll('text')
        .on('mouseover', handleMouseOver)
        .on('mouseout', handleMouseOut)
        .on('click', (d) => { this.$emit('update-focusedContent', d.word); });
    },
  },
};
</script>
