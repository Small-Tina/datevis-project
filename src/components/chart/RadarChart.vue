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

  const margin = { top: 30, left: 80, bottom: 20 };
  const width = 565;
  const height = 500;
  const radius = Math.min(width, height) / 2 - 50;
  function createdChart() {
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

    updateChart(svg, dimensions, angleSlice, nationalData, cityData, villageData, maxValue);
    creatTitle(svg);
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
  function updateChart(svg, dimensions, angleSlice, nationalData, cityData, villageData, maxValue) {
    // 创建 g 元素并移动到合适的位置
    const g = svg.append('g').attr('transform', `translate(${width / 2 + margin.left / 2}, ${height / 2 + margin.top / 2})`);
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
      });
    }
    drawDataSet(g, nationalData, '#3498db', '全国');
    drawDataSet(g, cityData, '#e74c3c', '城市');
    drawDataSet(g, villageData, '#2ecc71', '农村');
    createLegend(svg);
  }
  function creatTitle(svg) {
    svg
      .append('text')
      .attr('x', width / 2 + margin.left / 2)
      .attr('y', margin.top / 2)
      .attr('text-anchor', 'middle')
      .attr('font-size', '18px')
      .attr('font-weight', 'bold')
      .text(`城乡${props.selectYear}收入来源`);
  }
  function createLegend(svg) {
    const legendData = [
      { label: '全国', color: '#3498db' },
      { label: '城市', color: '#e74c3c' },
      { label: '农村', color: '#2ecc71' },
    ];

    const legend = svg.append('g').attr('transform', `translate(${margin.left}, ${margin.top})`);

    legendData.forEach((d, i) => {
      const legendItem = legend.append('g').attr('transform', `translate(0, ${i * 20})`);

      legendItem.append('rect').attr('width', 15).attr('height', 15).style('fill', d.color);

      legendItem.append('text').attr('x', 20).attr('y', 12).attr('text-anchor', 'start').text(d.label).style('font-size', '12px');
    });
  }
</script>

<style></style>
