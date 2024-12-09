<template>
  <div
    id="UARdiv"
    class="h-full"
  ></div>
</template>

<script setup>
  import { watch, onMounted } from 'vue';
  import * as d3 from 'd3';

  // 接收来自父组件的数据
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
      type: Number,
      required: true,
    },
  });

  // 配置组件
  defineOptions({
    name: 'RadarChart',
  });

  // 监听cityData数据变化
  watch(
    () => [props.selectYear],
    (newYear) => {
      createdChart();
    }
  );

  // 组件挂载时执行
  onMounted(() => {
    createdChart();
  });

  // 创建图表
  function createdChart() {
    const cityData = props.cityData.find((item) => item.年 == props.selectYear)?.人均可支配收入;
    const villageData = props.villageData.find((item) => item.年 == props.selectYear)?.人均可支配收入;
    const data = [
      { name: '城市', data: cityData, color: '#3498db' }, // 蓝色
      { name: '农村', data: villageData, color: '#e74c3c' }, // 红色
    ];
    const margin = { top: 30, left: 60, bottom: 20 };
    const width = 565;
    const height = 500;
    const radius = Math.min(width, height) / 2 - 50;

    // 获取 div 的宽高
    const uarDiv = d3.select('#UARdiv');
    uarDiv.select('svg').remove(); // 清除旧的图表

    const svg = uarDiv
      .append('svg')
      .attr('width', width + margin.left)
      .attr('height', height);

    creatTitle(svg, width, margin); // 创建标题
    updateChart(svg, radius, width, height, margin, data); // 更新图表
  }

  // 更新图表
  function updateChart(svg, radius, width, height, margin, data) {
    // 创建饼图布局
    const pie = d3.pie();

    // 设置圆环的内半径
    const arc = d3
      .arc()
      .innerRadius(radius * 0.5)
      .outerRadius(radius); // 设置内半径来创建圆环
    const arcHover = d3
      .arc()
      .innerRadius(radius * 0.5)
      .outerRadius(radius * 1.1); // 放大效果

    // 创建一个 g 元素，用于容纳所有的图形元素
    const g = svg.append('g').attr('transform', `translate(${width / 2 + margin.left / 2}, ${height / 2 + margin.top / 2})`);

    // 绘制饼图
    const arcs = g
      .selectAll('.slice')
      .data(pie(data.map((item) => item.data)))
      .enter()
      .append('g')
      .attr('class', 'slice');

    // 绘制圆环图的弧路径，并添加从无到有的动画
    arcs
      .append('path')
      .attr('d', arc) // 初始路径为空
      .attr('class', 'slice')
      .attr('data-color', (d, i) => data[i].color) // 绑定颜色到属性
      .style('fill', (d, i) => data[i].color) // 设置初始颜色
      .style('opacity', 0) // 初始时设置透明度为 0
      .transition() // 添加动画
      .duration(1000)
      .style('opacity', 1) // 动画结束后设置透明度为 1
      .attrTween('d', function (d) {
        // 使用 `attrTween` 来进行路径动画
        const interpolateStart = d3.interpolate(
          { startAngle: 0, endAngle: 0 }, // 初始值为空路径
          d // 最终目标路径（d 已经包含了最终的 startAngle 和 endAngle）
        );
        return function (t) {
          const interpolated = interpolateStart(t); // 插值计算出的角度
          return arc(interpolated); // 使用插值计算出的角度生成路径
        };
      });

    // 放大效果：鼠标悬停时的动画
    arcs
      .append('path')
      .attr('d', arc)
      .attr('class', 'slice')
      .attr('data-color', (d, i) => data[i].color) // 绑定颜色到属性
      .style('fill', (d, i) => data[i].color)
      .on('mouseover', function (event, d) {
        const color = d3.select(this).attr('data-color'); // 获取绑定的颜色
        d3.select(this)
          .transition()
          .duration(200)
          .attr('d', arcHover) // 放大区域
          .style('fill', d3.rgb(color).brighter(1)); // 高亮颜色
      })
      .on('mouseout', function (event, d) {
        const color = d3.select(this).attr('data-color'); // 获取绑定的颜色
        d3.select(this)
          .transition()
          .duration(200)
          .attr('d', arc) // 恢复原大小
          .style('fill', color); // 恢复原颜色
      });

    // 添加文本标签并加上动画
    arcs
      .append('text')
      .attr('transform', (d) => `translate(${arc.centroid(d)})`) // 根据弧的中心计算文本位置
      .attr('dy', '.35em') // 垂直方向调整
      .attr('text-anchor', 'middle') // 文本居中对齐
      .text((d, i) => {
        const labels = ['城镇', '乡村']; // 定义对应标签
        const percentage = ((d.endAngle - d.startAngle) / (2 * Math.PI)) * 100; // 计算百分比
        return `${labels[i]}: ${d.value.toFixed(2)} (${percentage.toFixed(1)}%)`; // 显示收入和百分比
      })
      .transition() // 文本淡入动画
      .duration(500);
  }

  // 创建标题
  function creatTitle(svg, width, margin) {
    svg
      .append('text')
      .attr('x', width / 2 + margin.left / 2)
      .attr('y', margin.top / 2)
      .attr('text-anchor', 'middle')
      .attr('font-size', '18px')
      .attr('font-weight', 'bold')
      .text(`${props.selectYear}城乡收入对比`);
  }
</script>
