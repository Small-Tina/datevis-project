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
  let selectYear = ref(2023);

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
     * 批量获取数据，并赋值给对应的data
     * @param {Array} dataArray
     * @param {Array} dataUrl
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
  function barClick(year) {
    selectYear.value = year;
  }
</script>

<template>
  <div class="h-screen w-screen">
    <div class="flex items-center justify-center">
      <span class="text-4xl">近年我国国民收支情况</span>
    </div>
    <div class="flex flex-col">
      <div class="basis-1/2 flex flex-row mb-5">
        <div class="basis-1/2">
          <IAEbarChart
            v-if="nationalData.length > 0"
            :nationalData="nationalData"
            @barClick="barClick"
          />
        </div>
        <div class="basis-1/2">
          <div class="basis-1/2">
            <LineChart
              v-if="nationalData.length > 0"
              :nationalData="nationalData"
            />
          </div>
        </div>
      </div>
      <div class="basis-1/2 flex flex-row">
        <div class="basis-1/3">
          <StackedAreaChart
            v-if="nationalData.length > 0"
            :nationalData="nationalData"
          />
        </div>
        <div class="basis-1/3">
          <RadarChart
            v-if="nationalData.length > 0 && cityData.length > 0 && villageData.length > 0"
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
            v-if="nationalData.length > 0 && cityData.length > 0 && villageData.length > 0"
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
