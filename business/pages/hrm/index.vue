<template>
  <div class="pt-24 p-6 bg-gray-50 min-h-screen">
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      <el-card>
        <div class="flex items-center">
          <div class="w-12 h-12 bg-blue-100 rounded-lg flex items-center justify-center">
            <el-icon :size="24" class="text-blue-600">
              <User />
            </el-icon>
          </div>
          <div class="ml-4">
            <p class="text-sm font-medium text-gray-600">{{ t('Total_employees') }}</p>
            <p class="text-2xl font-bold text-gray-900">{{ stats.total }}</p>
          </div>
        </div>
      </el-card>

      <el-card>
        <div class="flex items-center">
          <div class="w-12 h-12 bg-green-100 rounded-lg flex items-center justify-center">
            <el-icon :size="24" class="text-green-600">
              <Document />
            </el-icon>
          </div>
          <div class="ml-4">
            <p class="text-sm font-medium text-gray-600">{{ t('Active_employees') }}</p>
            <p class="text-2xl font-bold text-gray-900">{{ stats.active }}</p>
          </div>
        </div>
      </el-card>

      <el-card>
        <div class="flex items-center">
          <div class="w-12 h-12 bg-red-100 rounded-lg flex items-center justify-center">
            <el-icon :size="24" class="text-red-600">
              <Folder />
            </el-icon>
          </div>
          <div class="ml-4">
            <p class="text-sm font-medium text-gray-600">{{ t('Inactive_employees') }}</p>
            <p class="text-2xl font-bold text-gray-900">{{ stats.inactive }}</p>
          </div>
        </div>
      </el-card>
    </div>

    <!-- Headcount Trend -->
    <el-card v-if="headcountTrend">
      <Line :data="headcountTrend" :options="{ responsive: true, maintainAspectRatio: false }" />
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useI18n } from 'vue-i18n';
import { User, Document, Folder } from '@element-plus/icons-vue';
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
  ArcElement
} from 'chart.js';
import { Line, Doughnut } from 'vue-chartjs';

import EmployeeService from '@/services/hrm/employees';

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend, ArcElement);

const { t } = useI18n();

const employees = ref<any[]>([]);
const stats = ref({ total: 0, active: 0, inactive: 0 });

const headcountTrend = computed(() => {
  if (!employees.value.length) return null;
  const grouped: Record<string, number> = {};
  employees.value.forEach(emp => {
    const month = new Date(emp.join_date).toISOString().slice(0, 7); // YYYY-MM
    grouped[month] = (grouped[month] || 0) + 1;
  });
  const labels = Object.keys(grouped).sort();
  const data = labels.map(l => grouped[l]);
  return {
    labels,
    datasets: [{ label: t('Headcount_over_time'), data, borderColor: '#4f46e5', backgroundColor: '#a5b4fc' }]
  };
});

onMounted(async () => {
  try {
    await EmployeeService.fetch(true);
    const store = useEmployeesStore();
    employees.value = store.employees.data || [];

    stats.value.total = employees.value.length;
    stats.value.active = employees.value.filter(e => e.status === 1).length;
    stats.value.inactive = employees.value.filter(e => e.status !== 1).length;
  } catch (e) {
    console.error(e);
  }
});

definePageMeta({
    layout: 'hrm'
})
</script>
