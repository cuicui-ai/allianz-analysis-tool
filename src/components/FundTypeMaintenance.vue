<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { 
  Card, 
  Table, 
  Button, 
  Input, 
  Select, 
  Space, 
  Typography, 
  Alert, 
  Modal, 
  Form, 
  Popconfirm,
  Badge,
  message 
} from 'ant-design-vue';
import { 
  PlusOutlined, 
  SearchOutlined, 
  DeleteOutlined, 
  EditOutlined,
  SaveOutlined,
  CheckCircleOutlined
} from '@ant-design/icons-vue';
import dayjs from 'dayjs';

const { Title, Text, Link } = Typography;

interface FundTypeItem {
  id: string;
  code: string;
  name: string;
  type: '权益类' | '固收类' | '未维护';
  lastModified: string;
}

const defaultFunds: FundTypeItem[] = [
  { id: '1', code: '001475', name: '易方达国防军工混合A', type: '权益类', lastModified: '2026-06-02 10:24' },
  { id: '2', code: '002969', name: '广发多因子混合', type: '权益类', lastModified: '2026-06-03 09:15' },
  { id: '3', code: '110020', name: '易方达沪深300ETF联接A', type: '权益类', lastModified: '2026-06-01 14:20' },
  { id: '4', code: '001882', name: '中欧医疗健康混合A', type: '权益类', lastModified: '2026-05-28 11:45' },
  { id: '5', code: '000147', name: '易方达高等级信用债A', type: '固收类', lastModified: '2026-06-02 15:30' },
  { id: '6', code: '003382', name: '工银瑞信瑞景定期开放债券', type: '固收类', lastModified: '2026-06-03 08:00' },
  { id: '7', code: '270044', name: '广发双债添利债券A', type: '固收类', lastModified: '2026-05-30 16:12' },
  { id: '8', code: '001925', name: '汇添富双利债券A', type: '固收类', lastModified: '2026-06-01 17:05' },
  { id: '9', code: '005827', name: '易方达蓝筹精选混合A (样例：待分类维护)', type: '未维护', lastModified: '未配置' },
  { id: '10', code: '003003', name: '华夏纯债债券A (样例：待分类维护)', type: '未维护', lastModified: '未配置' }
];

const funds = ref<FundTypeItem[]>([]);
const searchText = ref('');
const filterType = ref<string>('all');

// Modals and Forms
const isModalVisible = ref(false);
const submitting = ref(false);
const formRef = ref();
const formState = ref({
  code: '',
  name: '',
  type: '权益类' as '权益类' | '固收类' | '未维护'
});

const loadFunds = () => {
  const stored = localStorage.getItem('hc_fund_types_list');
  if (stored) {
    try {
      const parsed = JSON.parse(stored) as FundTypeItem[];
      // Migrate / ensure the unmaintained specimens exist in user's UI
      if (!parsed.some(f => f.code === '005827')) {
        parsed.push({ id: '9', code: '005827', name: '易方达蓝筹精选混合A (样例：待分类维护)', type: '未维护', lastModified: '未配置' });
      }
      if (!parsed.some(f => f.code === '003003')) {
        parsed.push({ id: '10', code: '003003', name: '华夏纯债债券A (样例：待分类维护)', type: '未维护', lastModified: '未配置' });
      }
      funds.value = parsed;
    } catch (e) {
      funds.value = [...defaultFunds];
    }
  } else {
    funds.value = [...defaultFunds];
    saveToStorage();
  }
};

const saveToStorage = () => {
  localStorage.setItem('hc_fund_types_list', JSON.stringify(funds.value));
};

onMounted(() => {
  loadFunds();
});

const handleSearch = () => {
  // Driven automatically by computed filteredFunds below
};

const handleReset = () => {
  searchText.value = '';
  filterType.value = 'all';
};

// Default restoration function removed to avoid unnecessary options.

const handleTypeChange = (record: FundTypeItem, newType: '权益类' | '固收类' | '未维护') => {
  record.type = newType;
  record.lastModified = dayjs().format('YYYY-MM-DD HH:mm');
  saveToStorage();
  message.success(`已更新 [${record.name}] 类型为 ${newType}`);
};

const showAddModal = () => {
  formState.value = {
    code: '',
    name: '',
    type: '权益类'
  };
  isModalVisible.value = true;
};

const handleAddFund = async () => {
  if (!formState.value.code) {
    message.error('请填写完整代码！');
    return;
  }
  
  if (funds.value.some(f => f.code === formState.value.code)) {
    message.error('该基金代码已存在！');
    return;
  }

  const newItem: FundTypeItem = {
    id: Date.now().toString(),
    code: formState.value.code,
    name: formState.value.name || formState.value.code,
    type: formState.value.type,
    lastModified: dayjs().format('YYYY-MM-DD HH:mm')
  };

  funds.value.unshift(newItem);
  saveToStorage();
  isModalVisible.value = false;
  message.success('手动维护添加基金类型成功');
};

const handleDelete = (id: string, name: string) => {
  funds.value = funds.value.filter(item => item.id !== id);
  saveToStorage();
  message.success(`已删除基金 [${name}] 关系`);
};

// Computed list
const getFilteredFunds = () => {
  const filtered = funds.value.filter(fund => {
    const matchSearch = 
      fund.code.includes(searchText.value) || 
      fund.name.toLowerCase().includes(searchText.value.toLowerCase());
    
    const matchType = filterType.value === 'all' || fund.type === filterType.value;
    return matchSearch && matchType;
  });

  // Sort '未维护' (unmaintained) to the very top, preserving order for others
  return [...filtered].sort((a, b) => {
    if (a.type === '未维护' && b.type !== '未维护') return -1;
    if (a.type !== '未维护' && b.type === '未维护') return 1;
    return 0;
  });
};

const columns = [
  { title: '基金代码', dataIndex: 'code', key: 'code', width: '150px' },
  { title: '基金名称', dataIndex: 'name', key: 'name' },
  { title: '手动维护基金分类', dataIndex: 'type', key: 'type', width: '220px' },
  { title: '最后维护时间', dataIndex: 'lastModified', key: 'lastModified', width: '180px' },
  { title: '操作', key: 'action', width: '120px', align: 'center' }
];

import { h } from 'vue';
</script>

<template>
  <div class="fund-type-maintenance-container space-y-6">
    <Card :bordered="false" class="shadow-sm">
      <template #title>
        <Space direction="vertical" :size="2">
          <div class="flex items-center space-x-2">
            <span class="text-lg font-semibold text-gray-800">基金类型手动维护</span>
          </div>
        </Space>
      </template>

      <!-- Alert explaining connection -->
      <Alert
        type="warning"
        show-icon
        class="mb-6"
        style="background-color: #fffbe6; border: 1px solid #ffe58f;"
      >
        <template #description>
          <div class="text-amber-800 text-xs leading-relaxed">
            <b>穿透规则说明：</b>在估值表维护解析完成后，系统会根据下方对每一只基金代码维护的类别映射表，对大类下的“基金”进行精细化拆分类收益贡献。如含有未进行手动配置类型的基金将无法细分基金的收益贡献。
          </div>
        </template>
      </Alert>

      <!-- Filter bar and toolbar -->
      <div class="bg-gray-50 p-4 rounded-lg flex flex-wrap gap-4 items-center justify-between mb-4 border border-gray-100">
        <div class="flex flex-wrap gap-4 items-center">
          <Space>
            <Text strong>条件筛选：</Text>
            <Input 
              v-model:value="searchText" 
              placeholder="输入基金名称/代码" 
              style="width: 200px" 
              allow-clear
            >
              <template #suffix><SearchOutlined class="text-gray-400" /></template>
            </Input>
          </Space>
          
          <Space>
            <Text type="secondary">基金类别：</Text>
            <Select v-model:value="filterType" style="width: 140px">
              <Select.Option value="all">全部类型</Select.Option>
              <Select.Option value="权益类">权益类基金</Select.Option>
              <Select.Option value="固收类">固收类基金</Select.Option>
              <Select.Option value="未维护">未维护/待处理</Select.Option>
            </Select>
          </Space>

          <Button @click="handleReset" type="text" class="text-gray-500 hover:text-gray-700">
            重置
          </Button>
        </div>

        <div class="flex gap-2">
          <Button type="primary" @click="showAddModal" style="background-color: #023D7F; border-color: #023D7F">
            <template #icon><PlusOutlined /></template>
            新增基金维护
          </Button>
        </div>
      </div>

      <!-- Main Fund Config List -->
      <Table 
        :columns="columns" 
        :data-source="getFilteredFunds()" 
        row-key="id"
        size="middle"
        :pagination="{ pageSize: 10, showSizeChanger: true, showTotal: (total) => `共 ${total} 条基金数据` }"
        class="fund-type-table"
      >
        <template #bodyCell="{ column, record }">
          <template v-if="column.key === 'code'">
            <Text code class="font-mono">{{ record.code }}</Text>
          </template>

          <template v-if="column.key === 'name'">
            <Text strong class="text-gray-700">{{ record.name }}</Text>
          </template>

          <template v-if="column.key === 'type'">
            <!-- Inline fully functional select selector supporting '未维护' -->
            <div class="flex items-center space-x-1">
              <Select
                v-model:value="record.type"
                @change="(val) => handleTypeChange(record, val as any)"
                style="width: 150px"
                size="small"
                class="type-select-dropdown"
                :class="{ 'border border-amber-300 rounded': record.type === '未维护' }"
              >
                <Select.Option v-if="record.type === '未维护'" value="未维护">
                  <span class="text-amber-500 font-medium">未维护</span>
                </Select.Option>
                <Select.Option value="固收类">
                  <Badge color="#4ECBEE" text="固收类" />
                </Select.Option>
                <Select.Option value="权益类">
                  <Badge color="#BC4736" text="权益类" />
                </Select.Option>
              </Select>
            </div>
          </template>

          <template v-if="column.key === 'lastModified'">
            <span class="text-xs text-gray-400 font-mono">{{ record.lastModified }}</span>
          </template>

          <template v-if="column.key === 'action'">
            <Popconfirm
              title="确定删除此基金的分类维护关系吗？"
              ok-text="是"
              cancel-text="否"
              @confirm="handleDelete(record.id, record.name)"
            >
              <Button type="text" danger size="small">
                <template #icon><DeleteOutlined /></template>
                删除
              </Button>
            </Popconfirm>
          </template>
        </template>
      </Table>
    </Card>

    <!-- Modal for adding fund maintenance -->
    <Modal
      v-model:open="isModalVisible"
      title="新增基金类型维护"
      ok-text="保存维护"
      cancel-text="取消"
      @ok="handleAddFund"
      :destroyOnClose="true"
    >
      <div class="py-4">
        <Form layout="vertical">
          <Form.Item label="基金代码" required>
            <Input 
              v-model:value="formState.code" 
              placeholder="请输入6位公募基金代码 (如 000147)" 
              maxlength="12"
            />
          </Form.Item>
          <Form.Item label="基金简称">
            <Input 
              v-model:value="formState.name" 
              placeholder="请输入基金简称" 
            />
          </Form.Item>
          <Form.Item label="收益分配二级类型" required>
            <Select v-model:value="formState.type" style="width: 100%">
              <Select.Option value="未维护">未维护 (先加入列表，后续再指定)</Select.Option>
              <Select.Option value="权益类">权益类基金 (包括股票型、强股混合型、指数等)</Select.Option>
              <Select.Option value="固收类">固收类基金 (包括纯债型、一级二级债基、货币型等)</Select.Option>
            </Select>
          </Form.Item>
        </Form>
      </div>
    </Modal>
  </div>
</template>

<style>
.fund-type-table .ant-table-thead > tr > th {
  background-color: #f8fafc !important;
  color: #1e293b !important;
  font-weight: 600 !important;
}

.type-select-dropdown .ant-select-selector {
  border-radius: 4px !important;
}

.fund-type-maintenance-container .ant-btn-primary {
  background-color: #023D7F !important;
  border-color: #023D7F !important;
}
</style>
