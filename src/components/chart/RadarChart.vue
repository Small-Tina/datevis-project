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
    const height = 450;
    const radius = Math.min(width, height) / 2 - 50;

    const chart = d3.select('#radarChart');
    const dimensions = ['food', 'clothes', 'reside', 'goods', 'transportation', 'educational', 'healthcare', 'other'];
    const angleSlice = (Math.PI * 2) / dimensions.length;

    const nationalData = dataProcess(props.nationalData);
    const cityData = dataProcess(props.cityData);
    const villageData = dataProcess(props.villageData);
    // 如果有任意数据无效则退出
    if (!nationalData || !cityData || !villageData) {
      return;
    }
    // 获取数据的最大值
    const maxValue = Math.max(...Object.values(nationalData));

    // 创建或更新 SVG
    const svg = chart.select('svg').empty() ? chart.append('svg').attr('width', width).attr('height', height) : chart.select('svg');

    updateChart(svg, radius, dimensions, angleSlice, nationalData, margin, maxValue, width, height);
    creatTitle(svg, width, margin);
  }
  function dataProcess(data) {
    // 转换数据
    const processData = data.map((item) => ({
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

    // 获取选择年份的数据
    const processData_now = processData.find((item) => item.year == props.selectYear);
    return processData_now;
  }
  function updateChart(svg, radius, dimensions, angleSlice, nationalData, margin, maxValue, width, height) {
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

    // 计算数据的坐标并绘制雷达图区域
    const dataValues = dimensions.map((d, j) => {
      const x = radius * (nationalData[d] / maxValue) * Math.cos(angleSlice * j - Math.PI / 2);
      const y = radius * (nationalData[d] / maxValue) * Math.sin(angleSlice * j - Math.PI / 2);
      return [x, y];
    });
    dataValues.push(dataValues[0]); // 连接到起始点

    g.append('polygon')
      .attr('class', 'radar-chart-series')
      .attr('points', dataValues.map((d) => d.join(',')).join(' '))
      .style('fill', '#ccc')
      .style('stroke', 'red')
      .style('stroke-width', 2)
      .style('fill-opacity', 0.5);

    // 绘制每个数据点
    dimensions.forEach((d, j) => {
      const x = radius * (nationalData[d] / maxValue) * Math.cos(angleSlice * j - Math.PI / 2);
      const y = radius * (nationalData[d] / maxValue) * Math.sin(angleSlice * j - Math.PI / 2);

      g.append('circle').attr('class', 'radar-chart-point').attr('cx', x).attr('cy', y).attr('r', 4).style('fill', 'red').style('stroke', '#fff').style('stroke-width', 2);
    });
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
