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

  const margin = { top: 30, left: 60, bottom: 20 };
  const width = 565;
  const height = 500;
  const radius = Math.min(width, height) / 2 - 50;
  // 创建图表
  function createdChart() {
    const cityData = props.cityData.find((item) => item.年 == props.selectYear)?.人均可支配收入;
    const villageData = props.villageData.find((item) => item.年 == props.selectYear)?.人均可支配收入;
    const data = [
      { name: '城市', data: cityData, color: '#3498db' }, // 蓝色
      { name: '农村', data: villageData, color: '#e74c3c' }, // 红色
    ];

    // 获取 div 的宽高
    const uarDiv = d3.select('#UARdiv');
    uarDiv.select('svg').remove(); // 清除旧的图表

    const svg = uarDiv
      .append('svg')
      .attr('width', width + margin.left)
      .attr('height', height);

    creatTitle(svg, width, margin); // 创建标题
    updateChart(svg, data); // 更新图表
  }

  // 更新图表
  function updateChart(svg, data) {
    // 创建饼图布局
    const pie = d3
      .pie()
      .value((d) => d.data)
      .padAngle(0.01);

    // 设置圆环的内半径
    const arc = d3
      .arc()
      .innerRadius(radius * 0.5)
      .outerRadius(radius);
    const arcHover = d3
      .arc()
      .innerRadius(radius * 0.5)
      .outerRadius(radius * 1.1);
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
    // 创建一个 g 元素，用于容纳所有的图形元素
    const g = svg.append('g').attr('transform', `translate(${width / 2 + margin.left / 2}, ${height / 2 + margin.top / 2})`);

    // 绘制饼图
    g.selectAll('.slice')
      .data(pie(data))
      .enter()
      .append('path')
      .attr('d', arc) // 初始路径为空
      .attr('class', 'slice')
      .style('fill', (d, i) => data[i].color) // 设置初始颜色
      .transition()
      .duration(2000)
      .attrTween('d', function (d) {
        const start = { startAngle: 0, endAngle: 0 };
        const interpolate = d3.interpolate(start, d);
        return function (t) {
          return arc(interpolate(t));
        };
      });

    g.selectAll('.slice')
      .on('mouseover', function (event, d) {
        const total = data[0].data + data[1].data;
        d3.select(this)
          .transition() // 平滑过渡
          .duration(300) // 动画时间
          .attr('d', arcHover); // 切换到放大的路径
        console.log(event);

        tooltip.style('opacity', 1).html(`
        <strong>${d.data.name}收入:</strong> ${d.value}<br/>
        <strong>占比:</strong> ${((d.data.data / total) * 100).toFixed(2)}%
        `);
      })
      .on('mousemove', function (event) {
        // 更新 tooltip 位置
        tooltip
          .style('left', `${event.pageX + 10}px`) // 在鼠标右侧显示
          .style('top', `${event.pageY + 10}px`); // 在鼠标下方显示
      })
      .on('mouseout', function () {
        d3.select(this)
          .transition() // 平滑过渡
          .duration(300)
          .attr('d', arc); // 恢复到原始路径
        tooltip.style('opacity', 0);
      });
    // 添加文本标签并加上动画
    g.selectAll('text')
      .data(pie(data))
      .enter()
      .append('text')
      .attr('text-anchor', 'middle') // 文本居中对齐
      .attr('fill', 'white')
      .attr('font-size', '18px')
      .text((d) => {
        const total = data[0].data + data[1].data;
        return `${d.data.name}: ${((d.data.data / total) * 100).toFixed(2)}%`;
      })
      .transition() // 文本淡入动画
      .duration(3000)
      .attr('transform', (d) => `translate(${arc.centroid(d)})`);
  }

  // 创建标题
  function creatTitle(svg) {
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
