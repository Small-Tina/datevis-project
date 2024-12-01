<template>
  <div
    id="lineChart"
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
    const height = 450;
    // 定义柱状图宽度
    const barWeight = 900;
    const data = props.nationalData.map((item) => ({
      year: item.年,
      food: item.食品烟酒,
      clothes: item.衣着,
      reside: item.居住,
      goods: item.生活用品及服务,
      transportation: item.交通通信,
      educational: item.教育文化娱乐,
      healthcare: item.医疗保健,
      other: item.其他用品及服务,
    }));
    if (!data.length) {
      return;
    }
    const maxValue = Math.max(
      ...data.map((item) => {
        return Math.max(...Object.values(item));
      })
    );
    const year = data.map((item) => item.year);

    // 获取div的宽高
    const lineChart = d3.select('#lineChart');
    // 检查并获取现有的 SVG 元素，如果没有则创建新的
    const svg = lineChart.select('svg').empty() ? lineChart.append('svg').attr('width', width).attr('height', height) : lineChart.select('svg');
    // 创建标题
    creatTitle(svg, width, margin);
    updateChart(svg, margin, height, barWeight, maxValue, year, data);
  }
  function updateChart(svg, margin, height, barWeight, maxValue, year, data) {
    // 创建一个g元素，用于容纳所有的图形元素
    const g = svg.append('g').attr('transform', `translate(${margin.left}, ${margin.top})`);
    // 定义x轴和y轴
    const xScale = d3.scaleBand().domain(year).range([0, barWeight]).padding(0.05);
    const yScale = d3.scaleLinear().domain([0, maxValue]).range([400, 0]);
    // 创建坐标轴
    const xAxis = d3.axisBottom(xScale).tickSize(0).tickPadding(10);
    const yAxis = d3.axisLeft(yScale).tickSize(0);
    // 添加坐标轴
    g.append('g')
      .attr('class', 'x-axis')
      .attr('transform', `translate(0, ${height - margin.top - margin.bottom})`)
      .call(xAxis)
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
      .text('金额/元');

    const categories = ['food', 'clothes', 'reside', 'goods', 'transportation', 'educational', 'healthcare', 'other'];
    // 定义颜色比例尺，根据类别返回不同的颜色
    const colorScale = d3.scaleOrdinal(d3.schemeCategory10);
    categories.forEach((category) => {
      // 定义线条生成器
      const lineGenerator = d3
        .line()
        .x((d) => xScale(d.year) + xScale.bandwidth() / 2) // 确保点位于柱状图中间
        .y((d) => yScale(d[category]));

      // 创建折线路径
      g.append('path')
        .datum(data) // 将数据绑定到路径
        .attr('class', `line ${category}`) // 为每个类别的线添加不同的样式类
        .attr('fill', 'none')
        .attr('stroke', colorScale(category)) // 使用颜色比例尺来给不同类别分配颜色
        .attr('stroke-width', 2)
        .attr('d', lineGenerator); // 使用线条生成器定义路径

      // 可选：添加点以突出显示数据点
      g.selectAll(`.${category}-dot`)
        .data(data)
        .enter()
        .append('circle')
        .attr('class', `${category}-dot`)
        .attr('cx', (d) => xScale(d.year) + xScale.bandwidth() / 2)
        .attr('cy', (d) => yScale(d[category]))
        .attr('r', 4)
        .style('fill', colorScale(category));
    });
  }
  function creatTitle(svg, width, margin) {
    // 添加标题
    svg
      .append('text')
      .attr('x', width / 2) // 居中对齐
      .attr('y', margin.top / 2) // 放置在顶部边距内
      .attr('text-anchor', 'middle') // 文本居中
      .attr('font-size', '18px') // 设置字体大小
      .attr('font-weight', 'bold') // 设置加粗字体
      .text('近七年全国人均消费支出'); // 设置标题内容
  }
</script>
