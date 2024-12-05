<script setup>
  import { ref, onMounted } from 'vue';
  import { read, utils } from 'xlsx';
  import IAEbarChart from '@/components/chart/IAEbarChart.vue';
  import UrbanAndRuralPieChart from '@/components/chart/UrbanAndRuralPieChart.vue';
  import RadarChart from '@/components/chart/RadarChart.vue';
  import LineChart from '@/components/chart/LineChart.vue';
  import StackedAreaChart from '@/components/chart/StackedAreaChart.vue';

  let nationalData = ref([]);
  let cityData = ref([]);
  let villageData = ref([]);
  let selectYear = ref('2023');

  /**
   *
   * @param {Array} dataUrl
   * 获取数据
   */
  function getData() {
    const dataUrl = ['/assets/城镇.xlsx', '/assets/农村.xlsx', '/assets/全国.xlsx'];
    let dataArray = [];

    /**
     *
     * @param {Array} dataUrl
     * 创建fetch数组，后面批量调用获取指定数据
     */
    const fetchPromises = dataUrl.map((url) => {
      return fetch(url)
        .then((response) => response.arrayBuffer())
        .then((statisticalExcel) => {
          const statisticalSheet = read(statisticalExcel, { type: 'array' }).Sheets['Sheet1'];
          const statisticalData = utils.sheet_to_json(statisticalSheet);
          dataArray.push(statisticalData);
        });
    });
    /**
     *
     * @param {Array} dataArray
     * 批量获取数据，并赋值给对应的data
     */
    Promise.all(fetchPromises)
      .then(() => {
        cityData.value = dataArray[0];
        villageData.value = dataArray[1];
        nationalData.value = dataArray[2];
      })
      .then(() => {});
  }

  onMounted(() => {
    getData();
  });
</script>

<template>
  <div class="h-screen w-screen">
    <div class="flex flex-col h-full">
      <div class="basis-1/2 flex flex-row">
        <div class="basis-1/2"><IAEbarChart :nationalData="nationalData" /></div>
        <div class="basis-1/2">
          <div class="basis-1/2">
            <LineChart :nationalData="nationalData" />
          </div>
        </div>
      </div>
      <div class="basis-1/2 flex flex-row">
        <div class="basis-1/3">
          <StackedAreaChart :nationalData="nationalData" />
        </div>
        <div class="basis-1/3">
          <RadarChart
            :nationalData="nationalData"
            :cityData="cityData"
            :villageData="villageData"
            :selectYear="selectYear"
          />
        </div>
        <div
          class="basis-1/3"
          v-if="nationalData.length > 0"
        >
          <UrbanAndRuralPieChart
            :cityData="cityData"
            :villageData="villageData"
            :selectYear="selectYear"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style></style>
