<template>
  <div
    id="iaeDiv"
    class="h-full mt-2"
  ></div>
</template>

<script setup>
  import { watch, onMounted, ref } from 'vue';
  import * as d3 from 'd3';
  /**
   * 配置组件
   */
  defineOptions({
    name: 'IAEbarChart',
  });

  const emit = defineEmits(['barClick']);
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

  // 在组件的作用域内定义 selectedCategories 状态
  let selectedCategories = {
    人均可支配收入: true,
    人均消费支出: true,
  };
  // 定义margin，确定与上下边的距离
  const margin = { top: 30, left: 60, bottom: 20 };
  const width = 1000;
  const height = 430;
  // 定义柱状图宽度
  const barWeight = 900;
  const selectYear = ref();
  /**
   * 创建图表
   * @returns
   */
  function createdChart() {
    const preIAE = props.nationalData.map((item) => {
      return {
        year: item.年,
        人均可支配收入: item.人均可支配收入,
        人均消费支出: item.人均消费支出,
      };
    });
    const categories = ['人均消费支出', '人均可支配收入'];
    const colorMap = [
      {
        name: '人均可支配收入',
        value: '人均可支配收入',
        color: '#009db2',
      },
      {
        name: '人均消费支出',
        value: '人均消费支出',
        color: '#f9e264',
      },
    ];
    // 获取年份
    const year = d3.map(preIAE, (item) => item.year);
    // 获取收入最大值
    const maxValue = d3.max(preIAE, (d) => +d.人均可支配收入);
    // 获取div的宽高
    const iaeDiv = d3.select('#iaeDiv');
    // 检查并获取现有的 SVG 元素，如果没有则创建新的
    const svg = iaeDiv.select('svg').empty() ? iaeDiv.append('svg').attr('width', width).attr('height', height) : iaeDiv.select('svg');
    // 创建标题
    creatTitle(svg, width, margin);
    // 创建图例
    createLegend(svg, categories, colorMap, year, maxValue, preIAE);
    // 更新图表
    updateChart(svg, year, maxValue, categories, preIAE, colorMap);
  }

  function updateChart(svg, year, maxValue, categories, preIAE, colorMap) {
    const g = svg.append('g').attr('transform', `translate(${margin.left}, ${margin.top})`);
    const xScale = d3.scaleBand().domain(year).range([0, barWeight]).padding(0.05);
    const yScale = d3.scaleLinear().domain([0, maxValue]).range([380, 0]);
    const subXScale = d3.scaleBand().domain(categories).range([0, xScale.bandwidth()]).padding(0.1);

    const xAxis = d3.axisBottom(xScale).tickSize(0).tickPadding(10);
    const yAxis = d3.axisLeft(yScale).tickSize(0);

    // 在创建图表时添加 tooltip 容器
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
    g.append('g')
      .attr('class', 'x-axis')
      .attr('transform', `translate(0, ${height - margin.top - margin.bottom})`)
      .call(xAxis)
      .attr('font-size', '14px');

    g.append('g').attr('class', 'y-axis').call(yAxis).attr('font-size', '14px');

    const barGroups = g
      .selectAll()
      .data(preIAE)
      .join('g')
      .attr('transform', (d) => `translate(${xScale(d.year)}, 0)`);

    barGroups
      .selectAll('rect')
      .data((d) =>
        categories
          .map((category) => ({
            category,
            year: d.year,
            value: d[category],
            selected: selectedCategories[category],
          }))
          .filter((d) => d.selected)
      )
      .join('rect')
      .attr('class', (d) => `bar ${d.category}`) // 给每个柱子添加类别类名
      .attr('x', (d) => subXScale(d.category))
      .attr('y', yScale(0)) // 初始位置从 yScale(0) 开始（图表底部）
      .attr('width', subXScale.bandwidth())
      .attr('height', 0) // 初始高度为 0
      .attr('fill', (d) => colorMap.find((item) => item.value === d.category)?.color)
      .transition() // 第一阶段动画
      .duration(800) // 动画时长 800ms
      .ease(d3.easeCubicOut) // 平滑效果
      .attr('y', (d) => yScale(d.value) + (yScale(0) - yScale(d.value)) / 2) // 到中间高度
      .attr('height', (d) => (yScale(0) - yScale(d.value)) / 2) // 高度增长到一半
      .transition() // 第二阶段动画
      .duration(800) // 动画时长 800ms
      .ease(d3.easeCubicOut) // 第二阶段带回弹效果
      .attr('y', (d) => yScale(d.value)) // 最终位置
      .attr('height', (d) => yScale(0) - yScale(d.value)); // 最终高度
    // 添加交互
    barGroups
      .selectAll('rect')
      .on('mouseover', function (event, d) {
        // 高亮当前柱子
        d3.select(this)
          .transition()
          .duration(200)
          .attr('fill', d3.color(d3.select(this).attr('fill')).darker(1)); // 提高亮度
        // 显示提示信息
        tooltip.style('opacity', 1).text(`${d.year}年${d.category}: ${d.value}`);
      })
      .on('mousemove', function (event) {
        // 更新 tooltip 位置
        tooltip
          .style('left', `${event.pageX + 10}px`) // 在鼠标右侧显示
          .style('top', `${event.pageY + 10}px`); // 在鼠标下方显示
      })
      .on('click', function (event, d) {
        emit('barClick', d.year);
      })
      .on('mouseout', function () {
        // 恢复柱子原始颜色
        d3.select(this)
          .transition()
          .duration(200)
          .attr('fill', (d) => colorMap.find((item) => item.value === d.category)?.color);
        tooltip.style('opacity', 0);
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
  function createLegend(svg, categories, colorMap, year, maxValue, preIAE) {
    const legend = svg.append('g').attr('transform', `translate(${margin.left + 10}, ${30})`);

    categories.forEach((category, i) => {
      const legendRow = legend
        .append('g')
        .attr('transform', `translate(0, ${i * 30})`) // 调整每行之间的间距
        .style('cursor', 'pointer') // 鼠标变为手型，提示可交互
        .on('click', function () {
          // 切换当前类别的选中状态
          selectedCategories[category] = !selectedCategories[category];
          // 更新图表以反映新的选择
          updateChart(svg, year, maxValue, categories, preIAE, colorMap);
        })
        .on('mouseover', function () {
          // 高亮当前类别的柱状图
          svg.selectAll('.bar').transition().duration(200).style('opacity', 0.3); // 使所有柱子变透明
          svg.selectAll(`.bar.${category}`).transition().duration(200).style('opacity', 1); // 高亮当前类别
        })
        .on('mouseout', function () {
          // 恢复所有柱状图的透明度
          svg.selectAll('.bar').transition().duration(200).style('opacity', 1);
        });

      // 放大图例矩形
      legendRow
        .append('rect')
        .attr('width', 20) // 矩形宽度
        .attr('height', 20) // 矩形高度
        .attr('fill', colorMap.find((item) => item.value === category)?.color)
        .classed('selected', selectedCategories[category]); // 根据状态设置样式

      // 放大图例文字
      legendRow
        .append('text')
        .attr('x', 20) // 文字与矩形之间的间距
        .attr('y', 12) // 垂直居中
        .attr('text-anchor', 'start')
        .style('font-size', '14px') // 增大文字大小
        .style('fill', '#333') // 设置文字颜色
        .text(colorMap.find((item) => item.value === category)?.name);
    });
  }
  /**
   * 创建标题
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
