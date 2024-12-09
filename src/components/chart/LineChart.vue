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
   * @param {*} props
   * @param {*} emit
   * @returns
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
  // 定义margin，确定与上下边的距离
  const margin = { top: 30, left: 60, bottom: 20 };
  const width = 1000;
  const height = 430;
  const barWeight = 900;
  /**
   * 创建图表
   * @returns
   */
  function createdChart() {
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
    creatTitle(svg);
    updateChart(svg, maxValue, year, data);
  }
  function updateChart(svg, maxValue, year, data) {
    // 创建一个g元素，用于容纳所有的图形元素
    const g = svg.append('g').attr('transform', `translate(${margin.left}, ${margin.top})`);
    // 定义x轴和y轴
    const xScale = d3.scaleBand().domain(year).range([0, barWeight]).padding(0.05);
    const yScale = d3.scaleLinear().domain([0, maxValue]).range([380, 0]);
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
        .attr('d', lineGenerator)
        .on('mouseover', function () {
          // 高亮当前折线
          d3.selectAll('.line').style('opacity', 0.2); // 变暗其他折线
          d3.select(this).style('opacity', 1).style('stroke-width', 3); // 高亮当前折线
        })

        .on('mouseout', function () {
          // 恢复所有折线
          d3.selectAll('.line').style('opacity', 1).style('stroke-width', 2);
        }); // 使用线条生成器定义路径

      // 添加点以突出显示数据点
      g.selectAll(`.${category}-dot`)
        .data(data)
        .enter()
        .append('circle')
        .attr('class', `${category}-dot`)
        .attr('cx', (d) => xScale(d.year) + xScale.bandwidth() / 2)
        .attr('cy', (d) => yScale(d[category]))
        .attr('r', 4)
        .style('fill', colorScale(category))
        .on('mouseover', function (event, d) {
          // 显示 tooltip
          tooltip.style('opacity', 1).html(`
        <strong>${getCategoryName(category)}:</strong> ${d[category]} 元<br/>
        <strong>年份:</strong> ${d.year}
      `); // 设置内容
          // 高亮当前点
          d3.select(this).transition().duration(200).attr('r', 10); // 放大点
        })
        .on('mousemove', function (event) {
          // 更新 tooltip 位置
          tooltip
            .style('left', `${event.pageX + 10}px`) // 在鼠标右侧显示
            .style('top', `${event.pageY + 10}px`); // 在鼠标下方显示
        })
        .on('mouseout', function () {
          // 隐藏 tooltip
          tooltip.style('opacity', 0);
          // 恢复点大小
          d3.select(this).transition().duration(200).attr('r', 4);
        });
    });
    // 调用添加图例的方法
    createLegend(svg, categories, colorScale);
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
      .text('近七年全国人均消费支出'); // 设置标题内容
  }
  function createLegend(svg, categories, colorScale) {
    const legendGroup = svg.append('g').attr('class', 'legend-group');

    // 图例的起始位置
    const legendX = margin.left + 10;
    const legendY = margin.top;

    // 用于记录当前高亮的类别
    let activeCategory = null;

    categories.forEach((category, index) => {
      // 每一项图例的位置
      const legendItem = legendGroup
        .append('g')
        .attr('class', `legend-item ${category}`)
        .attr('transform', `translate(${legendX}, ${legendY + index * 25})`) // 每一项间隔 25px
        .style('cursor', 'pointer') // 鼠标悬停时变为手型

        // 添加鼠标悬停事件
        .on('mouseover', function () {
          if (activeCategory !== null) return; // 如果有激活的类别，不响应悬停
          d3.selectAll('.line').style('opacity', 0.2);
          d3.selectAll(`.line.${category}`).style('opacity', 1).style('stroke-width', 3); // 高亮对应折线
        })
        .on('mouseout', function () {
          if (activeCategory !== null) return; // 如果有激活的类别，不恢复样式
          d3.selectAll('.line').style('opacity', 1).style('stroke-width', 2); // 恢复所有折线
        })

        // 添加点击事件
        .on('click', function () {
          if (activeCategory === category) {
            // 如果当前点击的类别是激活的，则取消高亮
            activeCategory = null;
            d3.selectAll('.line').style('opacity', 1).style('stroke-width', 2); // 恢复所有折线
          } else {
            // 否则高亮点击的类别
            activeCategory = category;
            d3.selectAll('.line').style('opacity', 0.2);
            d3.selectAll(`.line.${category}`).style('opacity', 1).style('stroke-width', 3); // 高亮对应折线
          }
        });

      // 添加颜色标记
      legendItem.append('rect').attr('width', 20).attr('height', 20).attr('fill', colorScale(category));

      // 添加文字
      legendItem
        .append('text')
        .attr('x', 30) // 文字与标记间隔
        .attr('y', 15) // 文字垂直居中对齐
        .attr('font-size', '14px')
        .text(getCategoryName(category)); // 转换类别名称为可读形式
    });
  }

  // 转换类别名称为可读形式的辅助函数
  function getCategoryName(category) {
    const names = {
      food: '食品烟酒',
      clothes: '衣着',
      reside: '居住',
      goods: '生活用品及服务',
      transportation: '交通通信',
      educational: '教育文化娱乐',
      healthcare: '医疗保健',
      other: '其他用品及服务',
    };
    return names[category] || category;
  }
</script>
