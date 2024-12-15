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
    const data = props.nationalData.map((item) => {
      return { 年: +item.年, 食品烟酒: item.食品烟酒, 居住环境: item.居住, 医疗保健: item.医疗保健, 教育文化: item.教育文化娱乐, 交通通信: item.交通通信, 衣着用品: item.衣着, 其他: item.其他用品及服务 };
    });
    // 提取堆叠的字段
    const keys = Object.keys(data[0]).filter((k) => k !== '年');

    // 堆叠数据
    const stack = d3.stack().keys(keys).offset(d3.stackOffsetExpand);
    const stackedData = stack(data);

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

    // 创建 tooltip
    const tooltip = d3
      .select('body')
      .append('div')
      .attr('class', 'tooltip')
      .style('position', 'absolute')
      .style('background', 'rgba(0, 0, 0, 0.7)')
      .style('color', '#fff')
      .style('padding', '5px 10px')
      .style('border-radius', '4px')
      .style('pointer-events', 'none') // 禁止鼠标事件
      .style('opacity', 0); // 初始隐藏

    g.selectAll('path')
      .data(stackedData)
      .join('path')
      .attr('fill', (d) => color(d.key))
      .attr('d', area)
      // 添加鼠标事件处理
      .on('mouseover', function (event, d) {
        // 保存原始颜色
        const originalColor = d3.select(this).attr('fill');

        // 高亮当前区域
        d3.select(this).transition().duration(200).attr('fill', d3.rgb(originalColor).darker(1));

        // 更新并显示 tooltip
        tooltip.transition().duration(200).style('opacity', 1);
        tooltip
          .html(`${d.key}`)
          .style('left', `${event.pageX + 5}px`)
          .style('top', `${event.pageY - 28}px`);

        // 在 mouseout 中恢复颜色
        d3.select(this).on('mouseout', function () {
          // 恢复颜色
          d3.select(this).transition().duration(200).attr('fill', originalColor);

          // 隐藏 tooltip
          tooltip.transition().duration(200).style('opacity', 0);
        });
      })
      .on('mousemove', function (event) {
        // 更新 tooltip 位置
        tooltip
          .style('left', `${event.pageX + 10}px`) // 在鼠标右侧显示
          .style('top', `${event.pageY + 10}px`); // 在鼠标下方显示
      });

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
    // 创建图例并放置在底部
    const legend = svg.append('g').attr('transform', `translate(${margin.left}, ${height + margin.top + 20})`);

    // 为每个堆叠区域添加图例项
    legend
      .selectAll('.legend-item')
      .data(keys)
      .enter()
      .append('g')
      .attr('class', 'legend-item')
      .attr('transform', (d, i) => `translate(${i * 90}, 0)`) // 水平排列图例项
      .each(function (d) {
        const legendItem = d3.select(this);

        // 添加颜色块
        legendItem.append('rect').attr('width', 18).attr('height', 18).style('fill', color(d));

        // 添加文字
        legendItem.append('text').attr('x', 25).attr('y', 9).attr('dy', '.35em').style('text-anchor', 'start').text(d);
      });
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
