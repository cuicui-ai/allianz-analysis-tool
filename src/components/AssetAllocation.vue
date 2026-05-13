<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { Row, Col, Card, Tabs, Space, Tooltip, Typography } from 'ant-design-vue';
import { QuestionCircleOutlined } from '@ant-design/icons-vue';
import VChart from 'vue-echarts';
import * as echarts from 'echarts/core';
import {
  CanvasRenderer
} from 'echarts/renderers';
import {
  LineChart,
  BarChart
} from 'echarts/charts';
import {
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
} from 'echarts/components';
import { mockStockIndustry } from '../mockData';

echarts.use([
  CanvasRenderer,
  LineChart,
  BarChart,
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent
]);

const { Title, Text } = Typography;

const activeKey = ref('1');

const getMajorAllocationOption = () => ({
  tooltip: { trigger: 'axis', axisPointer: { type: 'cross', label: { backgroundColor: '#6a7985' } } },
  legend: { data: ['现金', '股票', '债券', '基金'], top: 0 },
  color: ['#00BFA5', '#023D7F', '#0066CC', '#4DA1FF'],
  grid: { left: '3%', right: '4%', bottom: '10%', top: '15%', containLabel: true },
  xAxis: { type: 'category', boundaryGap: false, data: ['2025-12', '2026-01', '2026-02', '2026-03'] },
  yAxis: { type: 'value', axisLabel: { formatter: '{value}%' } },
  series: [
    { name: '现金', type: 'line', stack: 'Total', areaStyle: {}, emphasis: { focus: 'series' }, data: [10, 12, 15, 15] },
    { name: '股票', type: 'line', stack: 'Total', areaStyle: {}, emphasis: { focus: 'series' }, data: [40, 45, 42, 45] },
    { name: '债券', type: 'line', stack: 'Total', areaStyle: {}, emphasis: { focus: 'series' }, data: [40, 35, 33, 30] },
    { name: '基金', type: 'line', stack: 'Total', areaStyle: {}, emphasis: { focus: 'series' }, data: [10, 8, 10, 10] }
  ]
});

const getBondTypeOption = () => {
  const categories = [
    '政策性金融债', '商金债', '中期票据', '国债', '可转债', '企业债', 
    '资产支持证券', '银行存款和结算备付金', '企业短期融资券', '央行票据', 
    '地方政府债', '中小企业私募债券', '其他债券', '买入返售金融资产', 
    '货币市场工具', '同业存单'
  ];
  const quarters = ['2021Q1', '2021Q2', '2021Q3', '2021Q4', '2022Q1', '2022Q2', '2022Q3', '2022Q4', '2023Q1', '2023Q2', '2023Q3', '2023Q4', '2024Q1', '2024Q2', '2024Q3', '2024Q4', '2025Q1', '2025Q2', '2025Q3', '2025Q4'];
  
  const colors = [
    '#E6B652', '#74529E', '#3C6291', '#5A9BD5', '#4E6A94', '#C0504D', 
    '#9BBB59', '#8064A2', '#C0504D', '#4BACC6', '#F79646', '#92D050', 
    '#0070C0', '#C00000', '#FFC000', '#7030A0'
  ];

  const series = categories.map((cat, index) => ({
    name: cat,
    type: 'bar',
    stack: 'total',
    emphasis: { focus: 'series' },
    data: quarters.map(() => (Math.random() * 10 + 5).toFixed(2)),
    itemStyle: { color: colors[index % colors.length] },
    barWidth: '40%'
  }));

  series.push({
    name: '加权平均平底溢价率(右)',
    type: 'line',
    yAxisIndex: 1,
    data: quarters.map(() => (Math.random() * 40 + 60).toFixed(2)),
    itemStyle: { color: '#FF5A5A' },
    lineStyle: { width: 2 },
    symbol: 'circle',
    symbolSize: 6,
    emphasis: { focus: 'series' }
  } as any);

  return {
    tooltip: { 
      trigger: 'axis', 
      axisPointer: { type: 'shadow' },
      formatter: (params: any) => {
        let res = params[0].name + '<br/>';
        params.forEach((item: any) => {
          res += item.marker + item.seriesName + ': ' + item.value + (item.seriesType === 'bar' ? '%' : '') + '<br/>';
        });
        return res;
      }
    },
    legend: { 
      data: [...categories, '加权平均平底溢价率(右)'], 
      top: 0, 
      type: 'scroll',
      itemWidth: 12,
      itemHeight: 12,
      textStyle: { fontSize: 10 }
    },
    grid: { left: '3%', right: '4%', bottom: '10%', top: '15%', containLabel: true },
    xAxis: { 
      type: 'category', 
      data: quarters,
      axisLabel: { fontSize: 10 },
      axisTick: { alignWithLabel: true }
    },
    yAxis: [
      { 
        type: 'value', 
        name: '占比',
        max: 140,
        axisLabel: { formatter: '{value}%', fontSize: 10 },
        splitLine: { lineStyle: { type: 'dashed' } }
      },
      {
        type: 'value',
        name: '溢价率',
        axisLabel: { fontSize: 10 },
        splitLine: { show: false }
      }
    ],
    series: series
  };
};

const majorAllocationOption = ref(getMajorAllocationOption());
const bondTypeOption = ref(getBondTypeOption());

</script>

<template>
  <div class="space-y-6">
    <Card :bordered="false">
      <Tabs v-model:activeKey="activeKey">
        <Tabs.TabPane key="1" tab="资产配置">
          <div class="space-y-6">
            <Row :gutter="[16, 16]">
              <Col :span="24">
                <Card :bordered="false">
                  <template #title>
                    <Space :size="4">
                      <span>历史资产配置比例变化</span>
                      <Tooltip title="展示组合大类资产配置比例的历史演变过程。">
                        <QuestionCircleOutlined style="color: #bfbfbf; font-size: 14px; cursor: help" />
                      </Tooltip>
                    </Space>
                  </template>
                  <VChart :option="majorAllocationOption" style="height: 450px" autoresize />
                </Card>
              </Col>
            </Row>
            <Row :gutter="[16, 16]">
              <Col :span="24">
                <Card :bordered="false">
                  <template #title>
                    <Space :size="4">
                      <span>债券品种分布</span>
                      <Tooltip title="展示债券资产在国债、金融债、信用债等不同品种上的分布。">
                        <QuestionCircleOutlined style="color: #bfbfbf; font-size: 14px; cursor: help" />
                      </Tooltip>
                    </Space>
                  </template>
                  <VChart :option="bondTypeOption" style="height: 500px" autoresize />
                </Card>
              </Col>
            </Row>
          </div>
        </Tabs.TabPane>
      </Tabs>
    </Card>
  </div>
</template>

<style scoped>
.space-y-6 > * + * {
  margin-top: 1.5rem;
}
</style>
