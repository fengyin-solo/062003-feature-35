<template>
  <div class="funds-panel card">
    <h3 class="panel-title">💰 资金面板</h3>

    <div class="balance-section">
      <div class="balance-info">
        <span class="label">当前余额</span>
        <span class="value" :class="{ danger: money < 10000 }">
          ¥{{ money.toLocaleString() }}
        </span>
      </div>
      <div class="profit-info">
        <span class="label">累计盈利</span>
        <span class="value" :class="profit >= 0 ? 'success' : 'danger'">
          {{ profit >= 0 ? '+' : '' }}¥{{ profit.toLocaleString() }}
        </span>
      </div>
    </div>

    <div class="forecast-section">
      <h4 class="section-title">📊 日成本预测</h4>
      <div class="forecast-details">
        <div class="forecast-item">
          <span class="forecast-label">固定运营成本</span>
          <span class="forecast-value">¥{{ dailyCostForecast.toLocaleString() }}/天</span>
        </div>
        <div class="forecast-item">
          <span class="forecast-label">今日日程成本</span>
          <span class="forecast-value">¥{{ scheduledActivityCost.toLocaleString() }}</span>
        </div>
        <div class="forecast-item total">
          <span class="forecast-label">预计今日总支出</span>
          <span class="forecast-value danger">¥{{ totalDailyCost.toLocaleString() }}</span>
        </div>
      </div>
      <div class="forecast-warning" v-if="totalDailyCost > money">
        ⚠️ 今日预计支出超过当前余额，请注意资金链风险！
      </div>
      <div class="forecast-warning" v-else-if="totalDailyCost * 7 > money">
        ⚠️ 按当前支出速度，资金仅够维持不到一周！
      </div>
      <div class="runway-info">
        <span class="runway-label">资金可维持</span>
        <span class="runway-value">{{ runwayDays }} 天</span>
      </div>
    </div>

    <div class="composition-section">
      <h4 class="section-title">📈 收支构成</h4>
      <div v-if="composition" class="composition-grid">
        <div class="composition-column">
          <div class="column-title success">
            收入 <span class="total">¥{{ composition.revenue.total.toLocaleString() }}</span>
          </div>
          <div class="composition-bars">
            <div
              v-for="(value, key) in composition.revenue.breakdown"
              :key="'rev-' + key"
              class="bar-row"
              v-if="value > 0"
            >
              <span class="bar-label">{{ revenueLabels[key] || key }}</span>
              <div class="bar-container">
                <div
                  class="bar success"
                  :style="{ width: getBarWidth(value, composition.revenue.total) }"
                ></div>
              </div>
              <span class="bar-value">¥{{ value.toLocaleString() }}</span>
            </div>
            <div v-if="composition.revenue.total === 0" class="empty-text">
              暂无收入
            </div>
          </div>
        </div>

        <div class="composition-column">
          <div class="column-title danger">
            支出 <span class="total">¥{{ composition.expenses.total.toLocaleString() }}</span>
          </div>
          <div class="composition-bars">
            <div
              v-for="(value, key) in composition.expenses.breakdown"
              :key="'exp-' + key"
              class="bar-row"
              v-if="value > 0"
            >
              <span class="bar-label">{{ expenseLabels[key] || key }}</span>
              <div class="bar-container">
                <div
                  class="bar danger"
                  :style="{ width: getBarWidth(value, composition.expenses.total) }"
                ></div>
              </div>
              <span class="bar-value">¥{{ value.toLocaleString() }}</span>
            </div>
            <div v-if="composition.expenses.total === 0" class="empty-text">
              暂无支出
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="large-expenses-section">
      <h4 class="section-title">🔔 大额支出提醒</h4>
      <div v-if="largeExpenses.length > 0" class="expense-list">
        <div
          v-for="expense in recentLargeExpenses"
          :key="expense.id"
          class="expense-item"
        >
          <div class="expense-icon">
            {{ getCategoryIcon(expense.category) }}
          </div>
          <div class="expense-info">
            <div class="expense-desc">{{ expense.description }}</div>
            <div class="expense-meta">
              第 {{ expense.day }} 天 · {{ expenseLabels[expense.category] || expense.category }}
            </div>
          </div>
          <div class="expense-amount danger">-¥{{ expense.amount.toLocaleString() }}</div>
        </div>
      </div>
      <div v-else class="empty-text">
        暂无大额支出记录
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  money: Number,
  profit: Number,
  dailyCostForecast: Number,
  scheduledActivityCost: Number,
  totalDailyCost: Number,
  composition: Object,
  largeExpenses: Array,
})

const expenseLabels = {
  training: '培训课程',
  pr: '公关活动',
  dailyOperating: '日常运营',
  singleCreation: '单曲制作',
  poaching: '挖角应对',
  other: '其他支出',
}

const revenueLabels = {
  singleSales: '单曲销售',
  other: '其他收入',
}

const runwayDays = computed(() => {
  if (props.totalDailyCost <= 0) return '∞'
  const days = Math.floor(props.money / props.totalDailyCost)
  return Math.max(0, days)
})

const recentLargeExpenses = computed(() => {
  return [...(props.largeExpenses || [])]
    .sort((a, b) => b.day - a.day)
    .slice(0, 5)
})

function getBarWidth(value, total) {
  if (total <= 0) return '0%'
  const pct = Math.round((value / total) * 100)
  return `${Math.max(5, pct)}%`
}

function getCategoryIcon(category) {
  const icons = {
    training: '🎓',
    pr: '📸',
    dailyOperating: '🏢',
    singleCreation: '💿',
    poaching: '⚔️',
    other: '📦',
  }
  return icons[category] || '💰'
}
</script>

<style scoped>
.funds-panel {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.panel-title {
  font-size: 1.1rem;
  font-weight: 700;
  margin-bottom: 0.25rem;
}

.section-title {
  font-size: 0.95rem;
  font-weight: 600;
  margin-bottom: 0.75rem;
  color: var(--text-primary);
}

.balance-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}

.balance-info,
.profit-info {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.balance-info .label,
.profit-info .label {
  font-size: 0.75rem;
  color: var(--text-muted);
}

.balance-info .value,
.profit-info .value {
  font-size: 1.35rem;
  font-weight: 700;
}

.forecast-section {
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}

.forecast-details {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.forecast-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem 0.75rem;
  background: var(--bg-secondary);
  border-radius: 8px;
}

.forecast-item.total {
  background: var(--accent-soft);
  font-weight: 600;
}

.forecast-label {
  font-size: 0.85rem;
  color: var(--text-secondary);
}

.forecast-value {
  font-weight: 600;
  font-size: 0.9rem;
}

.forecast-warning {
  margin-top: 0.75rem;
  padding: 0.5rem 0.75rem;
  background: var(--danger-soft);
  color: var(--danger);
  border-radius: 8px;
  font-size: 0.8rem;
  font-weight: 500;
}

.runway-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 0.75rem;
  padding: 0.5rem 0.75rem;
  background: var(--bg-secondary);
  border-radius: 8px;
}

.runway-label {
  font-size: 0.85rem;
  color: var(--text-secondary);
}

.runway-value {
  font-weight: 700;
  color: var(--accent);
  font-size: 1rem;
}

.composition-section {
  padding-bottom: 1rem;
  border-bottom: 1px solid var(--border);
}

.composition-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.composition-column {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.column-title {
  font-size: 0.85rem;
  font-weight: 600;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.column-title .total {
  font-size: 0.8rem;
  font-weight: 700;
}

.composition-bars {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.bar-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.75rem;
}

.bar-label {
  min-width: 60px;
  color: var(--text-secondary);
  font-size: 0.7rem;
}

.bar-container {
  flex: 1;
  height: 8px;
  background: var(--bg-secondary);
  border-radius: 4px;
  overflow: hidden;
}

.bar {
  height: 100%;
  border-radius: 4px;
  transition: width 0.3s;
}

.bar.success {
  background: var(--success);
}

.bar.danger {
  background: var(--danger);
}

.bar-value {
  min-width: 60px;
  text-align: right;
  font-weight: 500;
  font-size: 0.7rem;
}

.empty-text {
  padding: 0.75rem;
  text-align: center;
  color: var(--text-muted);
  font-size: 0.8rem;
  background: var(--bg-secondary);
  border-radius: 8px;
}

.large-expenses-section {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.expense-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.expense-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.75rem;
  background: var(--bg-secondary);
  border-radius: 8px;
}

.expense-icon {
  font-size: 1.25rem;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--bg-card);
  border-radius: 8px;
}

.expense-info {
  flex: 1;
  min-width: 0;
}

.expense-desc {
  font-size: 0.85rem;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.expense-meta {
  font-size: 0.7rem;
  color: var(--text-muted);
  margin-top: 0.15rem;
}

.expense-amount {
  font-weight: 700;
  font-size: 0.9rem;
  white-space: nowrap;
}

.success {
  color: var(--success);
}

.danger {
  color: var(--danger);
}

@media (max-width: 600px) {
  .composition-grid {
    grid-template-columns: 1fr;
  }

  .balance-section {
    grid-template-columns: 1fr;
  }
}
</style>
