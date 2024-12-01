<template>
  <div
    id="wordCloud"
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
    name: 'WordCloud',
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
  async function createdChart() {
    if (!props.nationalData) return;
    let nationalData = [
      { text: '食品烟酒', size: 0 },
      { text: '衣着', size: 0 },
      { text: '居住', size: 0 },
      { text: '生活用品及服务', size: 0 },
      { text: '交通通信', size: 0 },
      { text: '教育文化娱乐', size: 0 },
      { text: '医疗保健', size: 0 },
      { text: '其他用品及服务', size: 0 },
    ];
    await props.nationalData.forEach((item) => {
      nationalData.forEach((element) => {
        element.size += item[element.text] || 0;
      });
    });

    const wordcloud = d3.layout
      .cloud()
      .size([600, 400])
      .words(nationalData)
      .padding(5)
      .font('Impact')
      .fontSize((d) => d.size)
      .rotate(0)
      .on('end', draw);

    const color = d3.scaleOrdinal(d3.schemeCategory10);

    wordcloud.start();
    function draw(words) {
      console.log(words);

      d3.select('#wordCloud')
        .append('svg') //根据id插入svg
        .attr('width', 600) //宽度
        .attr('height', 400) //高度
        .attr('viewBox', '0 0 600 350') //可见区域
        .attr('preserveAspectRatio', 'xMaxYMax meet')
        .attr('class', 'wordcloud')
        .append('g')
        .attr('transform', 'translate(200,100)')
        .selectAll('text')
        .data(words)
        .enter()
        .append('text')
        .style('font-size', function (d) {
          return d.size + 'px';
        })
        .style('fill', function (d, i) {
          return color(i);
        }) //颜色
        .attr('transform', function (d) {
          //每个词条的偏移量
          return 'translate(' + [d.x, d.y] + ')rotate(' + d.rotate + ')';
        })
        .text(function (d) {
          return d.text;
        }); //内容
    }
  }
</script>

<style></style>
