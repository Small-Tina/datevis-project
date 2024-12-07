<template>
  <div
    id="iaeDiv"
    class="h-full mt-2"
  ></div>
</template>

<script setup>
  import { watch, onMounted } from 'vue';
  import * as d3 from 'd3';
  /**
   * 配置组件
   */
  defineOptions({
    name: 'IAEbarChart',
  });
  /**
   * 接收来自父组件的数据
   */
  const props = defineProps({
    nationalData: {
      type: Array,
      required: true,
    },
  });
  /**
   * 监听nationalData数据变化
   * @returns
   */
  watch(
    () => props.nationalData,
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
   * @returns
   */
  function createdChart() {
    // 定义margin，确定与上下边的距离
    const margin = { top: 30, left: 60, bottom: 20 };
    const width = 1000;
    const height = 430;
    // 定义柱状图宽度
    const barWeight = 900;
    const preIAE = props.nationalData.map((item) => {
      return {
        year: item.年,
        preDI: item.人均可支配收入,
        prePCE: item.人均消费支出,
      };
    });
    const categories = ['prePCE', 'preDI'];
    const colorMap = [
      {
        name: '人均可支配收入',
        value: 'preDI',
        color: '#009db2',
      },
      {
        name: '人均消费支出',
        value: 'prePCE',
        color: '#f9e264',
      },
    ];
    // 获取年份
    const year = d3.map(preIAE, (item) => item.year);
    // 获取收入最大值
    const maxValue = d3.max(preIAE, (d) => +d.preDI);
    // 获取div的宽高
    const iaeDiv = d3.select('#iaeDiv');
    // 检查并获取现有的 SVG 元素，如果没有则创建新的
    const svg = iaeDiv.select('svg').empty() ? iaeDiv.append('svg').attr('width', width).attr('height', height) : iaeDiv.select('svg');
    // 创建标题
    creatTitle(svg, width, margin);
    // 创建图例
    createLegend(svg, categories, colorMap, margin.left + 10, 30);
    // 更新图表
    updateChart(svg, margin, height, barWeight, year, maxValue, categories, preIAE, colorMap);
  }

  function updateChart(svg, margin, height, barWeight, year, maxValue, categories, preIAE, colorMap) {
    // 创建一个g元素，用于容纳所有的图形元素
    const g = svg.append('g').attr('transform', `translate(${margin.left}, ${margin.top})`);
    // 定义x轴和y轴
    const xScale = d3.scaleBand().domain(year).range([0, barWeight]).padding(0.05);
    const yScale = d3.scaleLinear().domain([0, maxValue]).range([380, 0]);
    // 创建一个子band，用于在每个柱状图上绘制多个条
    const subXScale = d3.scaleBand().domain(categories).range([0, xScale.bandwidth()]).padding(0.1);
    // 创建坐标轴
    const xAxis = d3.axisBottom(xScale).tickSize(0).tickPadding(10);
    const yAxis = d3.axisLeft(yScale).tickSize(0);
    // 添加坐标轴
    g.append('g')
      .attr('class', 'x-axis')
      .attr('transform', `translate(0, ${height - margin.top - margin.bottom})`)
      .call(xAxis)
      .attr('font-size', '14px')
      .append('text')
      .attr('class', 'x-axis-unit')
      .attr('x', barWeight + 5) // 放置在轴的末端
      .attr('y', 18) // 调整单位文本与轴线的间距
      .attr('fill', 'black') // 单位颜色
      .attr('font-size', '12px')
      .text('时间/年');
    g.append('g')
      .attr('class', 'y-axis')
      .call(yAxis)
      .attr('font-size', '14px')
      .append('text')
      .attr('class', 'x-axis-unit')
      .attr('x', -5) // 放置在轴的末端
      .attr('y', 16) // 调整单位文本与轴线的间距
      .attr('fill', 'black') // 单位颜色
      .attr('font-size', '12px')
      .text('金额/元')
      .selectAll('text'); // 选择所有刻度文本
    // 创建柱状图
    g.selectAll()
      .data(preIAE)
      .join('g')
      .attr('transform', (d) => `translate(${xScale(d.year)}, 0)`) // 按年份定位
      .selectAll('rect')
      .data((d) =>
        categories.map((category) => ({
          category,
          value: d[category], // 根据类别取值
        }))
      ) // 为每个类别创建数据
      .join('rect')
      .attr('x', (d) => subXScale(d.category))
      .attr('y', (d) => yScale(d.value))
      .attr('width', subXScale.bandwidth())
      .attr('height', (d) => yScale(0) - yScale(d.value))
      .attr('fill', (d) => colorMap.find((item) => item.value === d.category)?.color)
      .on('dbclick', function (d) {
        console.log(d);
      });
  }
  /**
   * 创建图例
   * @param {SVGAElement} svg
   * @param {Array} categories
   * @param {Object} colorMap
   * @param {Number} x
   * @param {Number} y
   * @returns
   */
  function createLegend(svg, categories, colorMap, x, y) {
    const legend = svg.append('g').attr('transform', `translate(${x}, ${y})`);
    categories.forEach((category, i) => {
      const legendRow = legend.append('g').attr('transform', `translate(0, ${i * 20})`);
      legendRow
        .append('rect')
        .attr('width', 10)
        .attr('height', 10)
        .attr('fill', colorMap.find((item) => item.value === category)?.color);
      legendRow
        .append('text')
        .attr('x', 15)
        .attr('y', 10)
        .attr('text-anchor', 'start')
        .style('font-size', '12px')
        .text(colorMap.find((item) => item.value === category)?.name);
    });
  }
  /**
   * 创建图例
   * @param {SVGAElement} svg
   * @param {Number} width
   * @param {Object} margin
   */
  function creatTitle(svg, width, margin) {
    // 添加标题
    svg
      .append('text')
      .attr('x', width / 2) // 居中对齐
      .attr('y', margin.top / 2) // 放置在顶部边距内
      .attr('text-anchor', 'middle') // 文本居中
      .attr('font-size', '18px') // 设置字体大小
      .attr('font-weight', 'bold') // 设置加粗字体
      .text('近七年全国人均可支配收入和消费支出'); // 设置标题内容
  }
</script>

<style></style>
