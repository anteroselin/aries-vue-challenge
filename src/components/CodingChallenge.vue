<template>
  <div class="chart-container">
    <div ref="chartCanvas" style="width: 800px; height: 600px"></div>
    <div v-if="mxP !== null" class="results">
      <p><strong>Maximun Profit:</strong> {{ mxP }}</p>
      <p><strong>Maximun Loss:</strong> {{ mxL }}</p>
      <p>
        <strong>Break Even Points:</strong>
        {{ breakPt.length > 0 ? breakPt.join(", ") : "None" }}
      </p>
    </div>
  </div>
</template>

<script>
import * as echarts from "echarts";

export default {
  name: "CodingChallenge",
  props: {
    optionsData: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      mxP: null,
      mxL: null,
      breakPt: [],
      chart: null,
    };
  },
  mounted() {
    this.calcRiskReward();
    this.initChart();
  },
  methods: {
    calcRiskReward() {
      const results = this.optionsData.map((option) => {
        const profitLoss =
          option.long_short === "long" ? option.bid : -option.ask;
        return {
          strike_price: option.strike_price,
          profitLoss,
        };
      });

      this.mxP = Math.max(...results.map((res) => res.profitLoss));
      this.mxL = Math.min(...results.map((res) => res.profitLoss));
      this.breakPt = this.calcBreakPt(results);
    },
    calcBreakPt(results) {
      const breakPt = [];

      for (let i = 1; i < results.length; i++) {
        const prev = results[i - 1];
        const curr = results[i];
        if (prev.profitLoss * curr.profitLoss < 0) {
          const breakEven =
            prev.strike_price +
            ((curr.strike_price - prev.strike_price) * (0 - prev.profitLoss)) /
              (curr.profitLoss - prev.profitLoss);
          breakPt.push(parseFloat(breakEven.toFixed(2)));
        }
      }

      return breakPt;
    },
    initChart() {
      this.chart = echarts.init(this.$refs.chartCanvas);

      // Prepare data for ECharts
      const xAxisData = this.optionsData.map((option) => option.strike_price);
      const seriesData = this.optionsData.map((option) =>
        option.long_short === "long" ? option.bid : -option.ask
      );

      // Set up ECharts options
      const option = {
        title: {
          text: "Options Strategy Risk & Reward Analysis",
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["line", "bar"] },
            restore: { show: true },
            saveAsImage: { show: true },
          },
        },
        tooltip: {
          trigger: "axis",
          formatter: "{b0} : {c0}",
        },
        xAxis: {
          type: "category",
          data: xAxisData,
          axisLabel: {
            formatter: "{value}",
          },
        },
        yAxis: {
          type: "value",
          axisLabel: {
            formatter: "{value}",
          },
        },
        series: [
          {
            type: "line",
            data: seriesData,
            markPoint: {
              data: [
                { type: "max", name: "Max", itemStyle: { color: "#71ffbe" } },
                { type: "min", name: "Min", itemStyle: { color: "#ff758e" } },
              ],
            },
            markLine: {
              data: [{ type: "average", name: "Avg" }],
            },
            smooth: true,
          },
        ],
      };

      // Set options and render chart
      this.chart.setOption(option);
    },
  },
};
</script>

<style scoped>
.chart-container {
  max-width: 600px;
  margin: 20px auto;
  text-align: center;
}

.results {
  margin-top: 20px;
  text-align: left;
}
</style>
