<template>
  <div
    id="radarChart"
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
    nationalData: {
      type: Array,
      required: true,
    },
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
    () => [props.nationalData, props.cityData, props.villageData, props.selectYear],
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

  function createdChart() {
    const margin = { top: 30, left: 60, bottom: 20 };
    const width = 565;
    const height = 430;
    const radius = Math.min(width, height) / 2 - 50;

    const chart = d3.select('#radarChart');
    const dimensions = ['工资性收入', '经营净收入', '财产净收入', '转移净收入'];
    const angleSlice = (Math.PI * 2) / dimensions.length;

    const nationalData = dataProcess(props.nationalData);
    const cityData = dataProcess(props.cityData);
    const villageData = dataProcess(props.villageData);

    // 获取数据的最大值
    const allData = [nationalData, cityData, villageData];
    const maxValue = Math.max(...allData.map((d) => Math.max(...Object.values(d))));

    // 创建或更新 SVG
    const svg = chart.select('svg').empty()
      ? chart
          .append('svg')
          .attr('width', width + margin.left)
          .attr('height', height)
      : chart.select('svg');

    updateChart(svg, radius, dimensions, angleSlice, nationalData, cityData, villageData, margin, maxValue, width, height);
    creatTitle(svg, width, margin);
  }
  function dataProcess(data) {
    // 转换数据
    const processData = data.map((item) => ({
      年: item.年,
      工资性收入: item.工资性收入,
      经营净收入: item.经营净收入,
      财产净收入: item.财产净收入,
      转移净收入: item.转移净收入,
    }));
    console.log(processData);

    // 获取选择年份的数据
    const processData_now = processData.find((item) => item.年 == props.selectYear);
    return processData_now;
  }
  function updateChart(svg, radius, dimensions, angleSlice, nationalData, cityData, villageData, margin, maxValue, width, height) {
    // 创建 g 元素并移动到合适的位置
    const g = svg.append('g').attr('transform', `translate(${width / 2}, ${height / 2 + margin.top / 2})`);
    // 绘制网格线（圆圈）
    g.selectAll('.gridCircle')
      .data(d3.range(1, 6)) // 可调整圈数
      .enter()
      .append('circle')
      .attr('class', 'gridCircle')
      .attr('r', (d) => (radius / 5) * d)
      .style('fill', '#ccc')
      .style('stroke', '#ccc')
      .style('fill-opacity', 0.1);

    // 绘制轴线
    const axis = g.selectAll('.axis').data(dimensions).enter().append('g').attr('class', 'axis');

    axis
      .append('line')
      .attr('x1', 0)
      .attr('y1', 0)
      .attr('x2', (d, i) => radius * Math.cos(angleSlice * i - Math.PI / 2))
      .attr('y2', (d, i) => radius * Math.sin(angleSlice * i - Math.PI / 2))
      .attr('class', 'line')
      .style('stroke', '#ccc')
      .style('stroke-width', 2);
    // 添加轴标签
    axis
      .append('text')
      .attr('transform', (d, i) => `translate(${radius * 1.1 * Math.cos(angleSlice * i - Math.PI / 2)}, ${radius * 1.1 * Math.sin(angleSlice * i - Math.PI / 2)})`)
      .attr('text-anchor', (d, i) => (i % 2 === 0 ? 'start' : 'end'))
      .attr('dy', '.35em')
      .text((d) => d);
    function drawDataSet(g, data, color, label) {
      // 计算数据的坐标
      const dataValues = dimensions.map((d, j) => {
        const x = radius * (data[d] / maxValue) * Math.cos(angleSlice * j - Math.PI / 2);
        const y = radius * (data[d] / maxValue) * Math.sin(angleSlice * j - Math.PI / 2);
        return [x, y];
      });
      dataValues.push(dataValues[0]); // 连接到起始点

      // 绘制雷达图区域
      g.append('polygon')
        .attr('class', 'radar-chart-series')
        .attr('points', dataValues.map((d) => d.join(',')).join(' '))
        .style('fill', color)
        .style('stroke', color)
        .style('stroke-width', 2)
        .style('fill-opacity', 0.5);

      // 绘制每个数据点及数据值标签
      dimensions.forEach((d, j) => {
        const x = radius * (data[d] / maxValue) * Math.cos(angleSlice * j - Math.PI / 2);
        const y = radius * (data[d] / maxValue) * Math.sin(angleSlice * j - Math.PI / 2);

        g.append('circle').attr('class', 'radar-chart-point').attr('cx', x).attr('cy', y).attr('r', 4).style('fill', color).style('stroke', '#fff').style('stroke-width', 2);

        // 添加数据值标签
        g.append('text')
          .attr('x', x)
          .attr('y', y)
          .attr('dx', j % 2 === 0 ? 8 : -8) // 调整文本与圆点的距离
          .attr('dy', '.35em')
          .attr('text-anchor', j % 2 === 0 ? 'start' : 'end')
          .text(data[d].toFixed(2)); // 显示数据点的值
      });

      // 添加图例
      g.append('rect')
        .attr('x', -radius - 30)
        .attr('y', -radius + 30)
        .attr('width', 10)
        .attr('height', 10)
        .style('fill', color);

      g.append('text')
        .attr('x', -radius - 20)
        .attr('y', -radius + 40)
        .attr('text-anchor', 'start')
        .attr('font-size', '12px')
        .text(label);
    }
    drawDataSet(g, nationalData, '#3498db', '全国');
    drawDataSet(g, cityData, '#e74c3c', '城市');
    drawDataSet(g, villageData, '#2ecc71', '农村');
  }
  function creatTitle(svg, width, margin) {
    svg
      .append('text')
      .attr('x', width / 2)
      .attr('y', margin.top / 2)
      .attr('text-anchor', 'middle')
      .attr('font-size', '18px')
      .attr('font-weight', 'bold')
      .text(`城乡${props.selectYear}收入来源`);
  }
</script>

<style></style>
