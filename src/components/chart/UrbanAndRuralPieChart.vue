<template>
  <div
    id="UARdiv"
    class="h-full"
  ></div>
</template>

<script setup>
  import { watch, onMounted } from 'vue';
  import * as d3 from 'd3';
  /**
   * 接收来自父组件的数据
   */
  const props = defineProps({
    cityData: {
      type: Array,
      required: true,
    },
    villageData: {
      type: Array,
      required: true,
    },
    selectYear: {
      type: String,
      required: true,
    },
  });
  /**
   * 配置组件
   */
  defineOptions({
    name: 'RadarChart',
  });
  /**
   * 监听cityData数据变化
   * @returns
   */
  watch(
    () => [props.cityData, props.villageData],
    () => {
      createdChart();
    },
    { deep: true }
  );
  /**
   * 组件挂载时执行
   * @returns
   */
  onMounted(() => {
    createdChart();
  });

  /**
   * 创建图表
   */
  function createdChart() {
    const cityData = props.cityData.find((item) => item.年 == props.selectYear)?.人均可支配收入;
    const villageData = props.villageData.find((item) => item.年 == props.selectYear)?.人均可支配收入;
    const data = [cityData, villageData];
    const margin = { top: 30, left: 60, bottom: 20 };
    const width = 565;
    const height = 450;
    const radius = Math.min(width, height) / 2 - 50;
    // 获取div的宽高
    const uarDiv = d3.select('#UARdiv');
    // 检查并获取现有的 SVG 元素，如果没有则创建新的
    const svg = uarDiv.select('svg').empty() ? uarDiv.append('svg').attr('width', width).attr('height', height) : uarDiv.select('svg');
    creatTitle(svg, width, margin);
    updateChart(svg, radius, width, height, margin, data);
  }
  /**
   * 更新图表
   * @param {*} svg
   * @param {Number} radius
   * @param {Number} width
   * @param {Number} height
   * @param {Number} margin
   * @param {Array} data
   */
  function updateChart(svg, radius, width, height, margin, data) {
    // 创建饼图布局
    const pie = d3.pie();

    // 创建弧路径生成器
    const arc = d3.arc().innerRadius(0).outerRadius(radius);
    // 创建一个g元素，用于容纳所有的图形元素
    const g = svg.append('g').attr('transform', `translate(${width / 2}, ${height / 2 + margin.top / 2})`);
    // 绘制饼图
    const arcs = g.selectAll('.slice').data(pie(data)).enter().append('g').attr('class', 'slice');
    // 绘制饼图的弧路径
    arcs
      .append('path')
      .attr('d', arc)
      .attr('class', 'slice')
      .style('fill', (d, i) => d3.schemeCategory10[i]);
  }
  function creatTitle(svg, width, margin) {
    // 添加标题
    svg
      .append('text')
      .attr('x', width / 2)
      .attr('y', margin.top / 2)
      .attr('text-anchor', 'middle')
      .attr('font-size', '16px')
      .attr('font-weight', 'bold')
      .text('城乡收支对比');
  }
</script>
