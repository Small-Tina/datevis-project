<template>
  <div
    id="StackedAreaChart"
    class="h-full"
  ></div>
</template>

<script setup>
  import { watch, onMounted } from 'vue';
  import * as d3 from 'd3';
  /**
   * 配置组件
   */
  defineOptions({
    name: 'StackedAreaChart',
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
  // 定义 margin
  const margin = { top: 30, left: 60, bottom: 50, right: 20 };
  const width = 600;
  const height = 430;
  function createdChart() {
    // 清空容器
    d3.select('#StackedAreaChart').selectAll('*').remove();

    // 创建 SVG 容器
    const svg = d3
      .select('#StackedAreaChart')
      .append('svg')
      .attr('width', width + margin.left + margin.right)
      .attr('height', height + margin.top + margin.bottom);

    const g = svg.append('g').attr('transform', `translate(${margin.left}, ${margin.top})`);
    // 提取堆叠的字段
    const keys = Object.keys(props.nationalData[0]).filter((k) => k !== '年');

    // 堆叠数据
    const stack = d3.stack().keys(keys).offset(d3.stackOffsetExpand);
    const stackedData = stack(props.nationalData);

    // 确保 '年' 是数值型
    props.nationalData.forEach((d) => {
      d.年 = +d.年; // 如果 '年' 是字符串，这里会将其转换为数值
    });

    // 定义 X 轴比例尺（线性比例，从最小年份开始）
    const x = d3
      .scaleLinear()
      .domain(d3.extent(props.nationalData, (d) => d.年)) // 从最小年份到最大年份
      .range([0, width]);

    // 定义 Y 轴比例尺
    const y = d3
      .scaleLinear()
      .domain([d3.min(stackedData, (layer) => d3.min(layer, (d) => d[0])), d3.max(stackedData, (layer) => d3.max(layer, (d) => d[1]))]) // 所有层的最小值和最大值
      .range([height, 0]);

    // 定义颜色比例尺
    const color = d3.scaleOrdinal().domain(keys).range(d3.schemeTableau10);

    // 定义面积生成器
    const area = d3
      .area()
      .x((d) => x(d.data.年))
      .y0((d) => y(d[0]))
      .y1((d) => y(d[1]));

    // 绘制堆叠面积图
    g.selectAll('path')
      .data(stackedData)
      .join('path')
      .attr('fill', (d) => color(d.key))
      .attr('d', area);

    // 添加 X 轴
    g.append('g')
      .attr('transform', `translate(0, ${height})`)
      .call(
        d3
          .axisBottom(x)
          .ticks(width / 80)
          .tickFormat(d3.format('d'))
      ); // 动态计算刻度数量

    // 添加 Y 轴
    g.append('g').call(d3.axisLeft(y));
    creatTitle(svg);
  }
  function creatTitle(svg) {
    // 添加标题
    svg
      .append('text')
      .attr('x', width / 2) // 居中对齐
      .attr('y', margin.top / 2) // 放置在顶部边距内
      .attr('text-anchor', 'middle') // 文本居中
      .attr('font-size', '18px') // 设置字体大小
      .attr('font-weight', 'bold') // 设置加粗字体
      .text('近七年全国人均消费支出比例'); // 设置标题内容
  }
</script>

<style></style>
