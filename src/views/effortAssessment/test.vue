<template>
  <div>
    <div v-for="(factor, index) in factors" :key="index" class="factor-container">
      <label :for="factor.name">{{ factor.label }}:</label>
      <el-select
        v-model="factor.selectedValue"
        :placeholder="`请选择${factor.label}`"
        @change="handleSelectionChange(index)"
        class="factor-select"
        clearable
      >
        <el-option v-for="option in factor.options" :key="option.value" :label="option.label" :value="option.value" />
      </el-select>
      <span class="value-display">{{ factor.selectedValue }}</span>
      <el-button type="primary" size="small" @click="showEditDialog(index)">编辑选项</el-button>

      <!-- 编辑选项对话框 -->
      <el-dialog v-model="editDialogVisible[index]" :title="`编辑 ${factor.label} 选项`" width="30%">
        <el-form>
          <el-form-item v-for="(option, optionIndex) in factor.options" :key="optionIndex" :label="`选项 ${optionIndex + 1}`">
            <el-input v-model="option.label" placeholder="请输入选项标签"></el-input>
            <el-input v-model.number="option.value" placeholder="请输入选项值" style="margin-top: 10px;" />
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="cancelEdit(index)">取 消</el-button>
            <el-button type="primary" @click="saveOptions(index)">确 定</el-button>
          </span>
        </template>
      </el-dialog>
    </div>

    <!-- ECharts 条形图 -->
    <div ref="chartContainer" class="chart-container"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import axios from 'axios';
import * as echarts from 'echarts';

// 定义因素列表
const factors = ref([
  {
    name: 'SF',
    label: 'SF',
    selectedValue: '',
    options: []
  },
  {
    name: 'BD',
    label: 'BD',
    selectedValue: '',
    options: []
  },
  {
    name: 'QR',
    label: 'QR',
    selectedValue: '',
    options: []
  },
  {
    name: 'AT',
    label: 'AT',
    selectedValue: '',
    options: []
  },
  {
    name: 'SL',
    label: 'SL',
    selectedValue: '',
    options: []
  },
  {
    name: 'DT',
    label: 'DT',
    selectedValue: '',
    options: []
  }
]);

// 对话框可见性状态
const editDialogVisible = ref(factors.value.map(() => false));

// ECharts 图表容器
const chartContainer = ref(null);

// 模拟 stdId 和 factor_type
const stdId = '123';  // 示例 stdId
const factorTypes = ['SF', 'BD', 'QR', 'AT', 'SL', 'DT'];  // 对应的 factor_type 类型

// 获取因子选项
const fetchFactorOptions = async (stdId: string, factorType: string) => {
  try {
    const response = await axios.get(`http://localhost:9000/Factor/find`, {
      params: { factor_type: factorType, stdId: stdId }  // 使用查询参数
    });
    const factorData = response.data.Factors;
    const factor = factors.value.find(factor => factor.name === factorType);
    if (factor) {
      factor.options = factorData.map(item => ({
        label: item.factor_name,
        value: item.factor_value
      }));
    }
  } catch (error) {
    console.error('获取因子选项失败:', error);
  }
};


// 初始化时根据 stdId 和 factor_type 获取因子数据
onMounted(() => {
  factorTypes.forEach(factorType => {
    fetchFactorOptions(stdId, factorType);
  });
});

// 显示编辑对话框
const showEditDialog = (index: number) => {
  editDialogVisible.value[index] = true;
};

// 保存编辑的选项
const saveOptions = (index: number) => {
  editDialogVisible.value[index] = false;
};

// 取消编辑
const cancelEdit = (index: number) => {
  editDialogVisible.value[index] = false;
};

// 选择变化处理函数
const handleSelectionChange = (index: number) => {
  console.log(`${factors.value[index].label} 选择的值: ${factors.value[index].selectedValue}`);
  updateChart();  // 更新图表
};

// 初始化 ECharts 图表
const initChart = () => {
  const chart = echarts.init(chartContainer.value);

  const chartOptions = {
    tooltip: {
      trigger: 'axis',
      axisPointer: { type: 'shadow' }
    },
    xAxis: {
      type: 'category',
      data: ['SF', 'BD', 'QR', 'AT', 'SL', 'DT'],  // x 轴标签
      axisLabel: { interval: 0 }
    },
    yAxis: { type: 'value' },
    series: [{
      name: '选项值',
      type: 'bar',
      barWidth: 60,  // 设置柱宽度
      data: factors.value.map(factor => factor.selectedValue || 0),
      emphasis: { focus: 'series' }
    }]
  };

  chart.setOption(chartOptions);
};

// 更新图表数据
const updateChart = () => {
  const chart = echarts.getInstanceByDom(chartContainer.value);
  const newData = factors.value.map(factor => factor.selectedValue || 0);

  chart.setOption({
    series: [{ data: newData }]
  });
};

</script>

<style lang="scss" scoped>
.factor-container {
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  width: 100%;
  color: #333;

  label {
    margin-right: 10px;
    margin-left: 30;
    width: 100px;
    text-align: right;
  }

  .factor-select {
    width: 150px;
    min-width: 150px;
    max-width: 150px;
  }

  .value-display {
    font-weight: bold;
    color: #333;
    width: 100px;
    text-align: left;
    margin-left: 40px;
  }

  .el-button {
    margin-left: 10px;
  }
}

.chart-container {
  width: 100%;
  height: 400px;
  margin: 0 auto;  /* 水平居中 */
}
</style>
