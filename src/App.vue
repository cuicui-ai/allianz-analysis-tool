<script setup lang="ts">
import { ref, watch, onMounted, provide } from 'vue';
import { 
  ConfigProvider, 
  Layout, 
  Menu, 
  Select, 
  DatePicker, 
  Radio, 
  Space, 
  Typography, 
  Row, 
  Col,
  Card
} from 'ant-design-vue';
import { 
  FilterOutlined,
  DashboardOutlined, 
  PieChartOutlined, 
  BarChartOutlined, 
  SafetyCertificateOutlined, 
  FileTextOutlined,
  DatabaseOutlined
} from '@ant-design/icons-vue';
import dayjs from 'dayjs';
import zhCN from 'ant-design-vue/es/locale/zh_CN';
import 'dayjs/locale/zh-cn';

import AssetAllocation from './components/AssetAllocation.vue';
import PerformanceAttribution from './components/PerformanceAttribution.vue';
import ValuationMaintenance from './components/ValuationMaintenance.vue';
import { Unit, GlobalFilters } from './types.ts';

const { Header, Sider, Content } = Layout;
const { RangePicker } = DatePicker;
const { Title, Text } = Typography;

const filters = ref<GlobalFilters>({
  product: '安联测试产品1',
  dateRange: [dayjs().subtract(3, 'month').format('YYYY-MM-DD'), dayjs().format('YYYY-MM-DD')],
  benchmark: '沪深300',
  unit: 'CNY',
});

provide('filters', filters);

const collapsed = ref(false);
const currentPath = ref(window.location.hash.replace('#', '') || 'valuation');

watch(collapsed, () => {
  setTimeout(() => {
    window.dispatchEvent(new Event('resize'));
  }, 250);
});

const menuItems = [
  { key: 'valuation', icon: () => h(DatabaseOutlined), label: '估值表维护' },
  { 
    key: 'analysis', 
    icon: () => h(DashboardOutlined), 
    label: '投后分析',
    children: [
      { key: 'attribution', label: '业绩归因' },
      { key: 'allocation', label: '资产配置' },
    ]
  },
];

import { h, computed } from 'vue';

const selectedKeys = ref<string[]>([currentPath.value]);

watch(currentPath, (newVal) => {
  selectedKeys.value = [newVal];
});

const handleMenuClick = ({ key }: { key: string }) => {
  window.location.hash = key;
  currentPath.value = key;
  selectedKeys.value = [key];
};

const getPageTitle = () => {
  switch (currentPath.value) {
    case 'valuation': return '估值表维护';
    case 'attribution': return '业绩归因';
    case 'allocation': return '资产配置';
    default: return '投后分析';
  }
};

const isAnalysisPage = computed(() => ['allocation', 'attribution'].includes(currentPath.value));

onMounted(() => {
  window.addEventListener('hashchange', () => {
    currentPath.value = window.location.hash.replace('#', '') || 'valuation';
  });
});
</script>

<template>
  <ConfigProvider 
    :locale="zhCN"
    :theme="{
      token: {
        colorPrimary: '#023D7F',
        borderRadius: 8,
        colorBgContainer: '#fff',
        colorBgLayout: '#f0f2f5',
      },
      components: {
        Card: {
          boxShadowTertiary: '0 4px 12px rgba(0,0,0,0.05)',
        }
      }
    }"
  >
    <Layout class="optimized-theme" style="min-height: 100vh">
      <Sider 
        v-model:collapsed="collapsed"
        collapsible 
        collapsed-width="80"
        theme="light"
        :style="{
          overflow: 'auto',
          height: '100vh',
          position: 'fixed',
          left: 0,
          top: 0,
          bottom: 0,
          zIndex: 100,
          backgroundColor: '#fff',
          borderRight: '1px solid #f0f0f0',
          boxShadow: '2px 0 8px rgba(0,0,0,0.02)'
        }"
      >
        <div style="height: 64px; margin: 16px; display: flex; align-items: center; justify-content: center; border-bottom: 1px solid #f0f2f5">
          <Title :level="4" :style="{ color: '#023D7F', margin: 0, whiteSpace: 'nowrap', fontSize: collapsed ? '16px' : '18px' }">
            {{ collapsed ? 'AL' : '安联投后分析工具' }}
          </Title>
        </div>
        <Menu 
          theme="light" 
          v-model:selectedKeys="selectedKeys" 
          mode="inline" 
          :items="menuItems"
          @click="handleMenuClick"
          style="border-right: 0"
        />
      </Sider>
      <Layout :style="{ 
        marginLeft: collapsed ? '80px' : '200px', 
        width: `calc(100% - ${collapsed ? '80px' : '200px'})`,
        transition: 'all 0.2s cubic-bezier(0.2, 0, 0, 1)',
        minWidth: 0,
        overflowX: 'hidden'
      }">
        <Header :style="{ 
          padding: '0 24px', 
          background: '#fff', 
          display: 'flex', 
          alignItems: 'center', 
          justifyContent: 'space-between',
          position: 'sticky',
          top: 0,
          zIndex: 99,
          boxShadow: '0 2px 8px rgba(0,0,0,0.06)',
          height: 'auto',
          minHeight: '64px',
          paddingTop: '8px',
          paddingBottom: '8px',
          width: '100%',
          overflowX: 'auto'
        }">
          <div class="flex items-center space-x-4">
            <Space>
              <DatabaseOutlined v-if="currentPath === 'valuation'" style="color: #023D7F; font-size: 18px" />
              <DashboardOutlined v-else style="color: #023D7F; font-size: 18px" />
              <Title :level="5" style="margin: 0">{{ getPageTitle() }}</Title>
            </Space>
            
            <div v-if="isAnalysisPage" class="ml-8 flex items-center space-x-6 border-l pl-8 border-gray-200">
              <Space>
                <FilterOutlined style="color: #023D7F" />
                <Text strong>全局筛选</Text>
              </Space>
              <Space>
                <Text type="secondary">产品：</Text>
                <Select 
                  v-model:value="filters.product"
                  style="width: 140px"
                  :options="[
                    { value: '安联测试产品1', label: '安联测试产品1' },
                    { value: '测试资管产品B', label: '测试资管产品B' },
                  ]"
                  size="small"
                />
              </Space>
              <Space>
                <Text type="secondary">时间：</Text>
                <RangePicker 
                  v-model:value="filters.dateRange"
                  :value-format="'YYYY-MM-DD'"
                  size="small"
                  style="width: 220px"
                />
              </Space>
              <Radio.Group 
                v-model:value="filters.unit"
                button-style="solid"
                size="small"
              >
                <Radio.Button value="CNY">元</Radio.Button>
                <Radio.Button value="10K">万元</Radio.Button>
                <Radio.Button value="100M">亿元</Radio.Button>
              </Radio.Group>
            </div>
          </div>
          
          <Space>
            <Text type="secondary" style="font-size: 12px">更新：{{ dayjs().format('MM-DD HH:mm') }}</Text>
          </Space>
        </Header>
        <Content style="margin: 24px; padding: 0; background: transparent; min-height: 280px">
          <ValuationMaintenance v-if="currentPath === 'valuation'" />
          <PerformanceAttribution v-else-if="currentPath === 'attribution'" />
          <AssetAllocation v-else-if="currentPath === 'allocation'" />
        </Content>
      </Layout>
    </Layout>
  </ConfigProvider>
</template>

<style>
.optimized-theme {
  background: #f0f2f5 !important;
}
</style>
