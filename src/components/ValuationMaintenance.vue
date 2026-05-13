<script setup lang="ts">
import { ref } from 'vue';
import { 
  Card, 
  Tabs, 
  DatePicker, 
  Upload, 
  Button, 
  Input, 
  Table, 
  Alert, 
  Space, 
  Typography,
  message,
  Radio
} from 'ant-design-vue';
import { 
  UploadOutlined, 
  SearchOutlined, 
  InfoCircleFilled,
  FolderOpenOutlined
} from '@ant-design/icons-vue';

const { Text } = Typography;

const subTab = ref('success');
const uploadMode = ref<'manual' | 'path'>('manual');

const columns = [
  { title: '估值日期', dataIndex: 'date', key: 'date' },
  { title: '产品/策略名称', dataIndex: 'productName', key: 'productName' },
  { title: '单位净值', dataIndex: 'unitNetValue', key: 'unitNetValue' },
  { title: '市值(元)', dataIndex: 'marketValue', key: 'marketValue' },
];

const mockData = [
  {
    key: '1',
    date: '2025-02-21',
    productName: '中银证券中国红-汇农鑫享1号',
    unitNetValue: 1.1245,
    marketValue: 12540000,
  },
  {
    key: '2',
    date: '2025-02-21',
    productName: '中银证券中国红-汇农鑫享1号',
    unitNetValue: 1.1248,
    marketValue: 12560000,
  }
];

const handleOpenBrowser = () => {
  message.info('正在打开文件浏览器...');
};
</script>

<template>
  <div class="valuation-maintenance-container">
    <Card :bordered="false" :body-style="{ padding: '24px' }">
      <div class="space-y-6">
        <!-- Header Controls -->
        <div class="bg-white p-6 rounded-lg border border-gray-100 shadow-sm">
          <div class="flex flex-col space-y-4">
            <div class="flex items-center space-x-8">
              <Space>
                <Text strong>选择日期：</Text>
                <DatePicker placeholder="选择日期" style="width: 180px" />
              </Space>
              
              <Space>
                <Text strong>上传方式：</Text>
                <Radio.Group 
                  v-model:value="uploadMode" 
                  option-type="button"
                  button-style="solid"
                >
                  <Radio.Button value="manual">手动上传</Radio.Button>
                  <Radio.Button value="path">路径读取</Radio.Button>
                </Radio.Group>
              </Space>

              <template v-if="uploadMode === 'manual'">
                <Upload directory multiple :show-upload-list="false">
                  <Button>
                    <template #icon><UploadOutlined /></template>
                    上传文件或文件夹
                  </Button>
                </Upload>
              </template>
              <template v-else>
                <Input 
                  placeholder="请输入或选择电脑文件夹路径" 
                  style="width: 320px"
                >
                  <template #suffix>
                    <FolderOpenOutlined class="cursor-pointer" @click="handleOpenBrowser" />
                  </template>
                </Input>
              </template>

              <Button 
                type="primary" 
                style="background-color: #FAAD14; border-color: #FAAD14"
              >
                <template #icon><SearchOutlined /></template>
                查询
              </Button>
            </div>
            
            <Text type="secondary" style="font-size: 12px">
              <InfoCircleFilled style="margin-right: 4px; color: #023D7F" />
              提示：{{ uploadMode === 'manual' ? '支持拖拽或批量选择估值表文件' : '选择文件路径后，系统将自动扫描并解析该路径下的所有估值文件' }}
            </Text>
          </div>
        </div>

        <!-- Sub Tabs -->
        <Tabs 
          v-model:activeKey="subTab" 
          class="valuation-sub-tabs"
        >
          <Tabs.TabPane key="success" tab="解析成功数据" />
          <Tabs.TabPane key="error" tab="解析失败数据" />
        </Tabs>

        <div v-if="subTab === 'success'" class="space-y-4">
          <Alert
            type="info"
            :show-icon="false"
            style="background-color: #f0f7ff; border: 1px solid #adc6ff"
          >
            <template #message>
              <Space>
                <InfoCircleFilled style="color: #023D7F" />
                <span>共识别成功 <Text strong style="color: #023D7F">201</Text> 个估值表</span>
              </Space>
            </template>
          </Alert>
          <Table 
            :columns="columns" 
            :data-source="mockData" 
            :pagination="{ pageSize: 10 }"
            size="middle"
            :bordered="false"
            class="custom-valuation-table"
            :scroll="{ x: 'max-content' }"
          >
            <template #bodyCell="{ column, text }">
              <template v-if="column.key === 'unitNetValue'">
                {{ text?.toFixed(4) || '-' }}
              </template>
              <template v-if="column.key === 'marketValue'">
                {{ text?.toLocaleString() || '-' }}
              </template>
            </template>
          </Table>
        </div>
        
        <div v-else class="py-20 text-center text-gray-400">
          暂无解析失败数据
        </div>
      </div>
    </Card>
  </div>
</template>

<style>
.valuation-sub-tabs .ant-tabs-nav {
  margin-bottom: 0 !important;
}
.valuation-sub-tabs .ant-tabs-tab {
  font-size: 15px;
  padding: 12px 16px;
}
.custom-valuation-table .ant-table-thead > tr > th {
  background-color: #f5f7fa !important;
  color: #333 !important;
  font-weight: 600 !important;
}
.valuation-maintenance-container .ant-tabs-ink-bar {
  background-color: #023D7F !important;
}
.valuation-maintenance-container .ant-tabs-tab.ant-tabs-tab-active .ant-tabs-tab-btn {
  color: #023D7F !important;
}
.valuation-maintenance-container .ant-radio-button-wrapper-checked:not(.ant-radio-button-wrapper-disabled) {
  background: #023D7F !important;
  border-color: #023D7F !important;
  color: #fff !important;
}
.valuation-maintenance-container .ant-radio-button-wrapper-checked:not(.ant-radio-button-wrapper-disabled):hover {
  background: #002d5e !important;
  border-color: #002d5e !important;
  color: #fff !important;
}
.space-y-6 > * + * {
  margin-top: 1.5rem;
}
.space-y-4 > * + * {
  margin-top: 1rem;
}
</style>
