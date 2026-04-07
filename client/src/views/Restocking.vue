<template>
  <div class="restocking-container">
    <div v-if="loading" class="loading">Loading data...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <!-- Header -->
      <div class="page-header">
        <h1>Restocking Planner</h1>
        <p>Set your budget and receive intelligent restocking recommendations based on demand forecasts</p>
      </div>

      <!-- Success Banner -->
      <div v-if="orderSuccess" class="success-banner">
        <strong>✓ Order placed successfully!</strong>
        Order #{{ lastOrderNumber }} has been submitted with 14-day delivery. Check the Orders tab to view submitted orders.
      </div>

      <!-- Budget Card -->
      <div class="card budget-card">
        <div class="card-header">
          <h2>Available Budget</h2>
        </div>
        <div class="budget-controls">
          <div class="slider-wrapper">
            <input
              v-model.number="budget"
              type="range"
              min="0"
              max="250000"
              step="1000"
              class="budget-slider"
            />
          </div>
          <div class="budget-display">
            <div class="budget-value">${{ formatCurrency(budget) }}</div>
            <div class="budget-note">Drag to set your restocking budget</div>
          </div>
        </div>
      </div>

      <!-- Recommendations Card -->
      <div class="card recommendations-card">
        <div class="card-header">
          <div class="header-left">
            <h2>Recommended Items</h2>
            <span class="count-badge">{{ selectedItems.length }} items</span>
          </div>
          <div class="budget-status" :class="remainingBudget >= 0 ? 'within-budget' : 'over-budget'">
            {{ remainingBudget >= 0 ? 'Within Budget' : 'Over Budget' }}
          </div>
        </div>

        <div v-if="selectedItems.length === 0" class="empty-state">
          <p>No items can be recommended with the current budget. Increase your budget or check demand forecasts.</p>
        </div>

        <div v-else>
          <div class="table-container">
            <table class="recommendations-table">
              <thead>
                <tr>
                  <th>SKU</th>
                  <th>Item Name</th>
                  <th>Trend</th>
                  <th>Qty to Restock</th>
                  <th>Unit Cost</th>
                  <th>Line Total</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in selectedItems" :key="item.sku">
                  <td class="sku-cell"><strong>{{ item.sku }}</strong></td>
                  <td class="name-cell">{{ item.name }}</td>
                  <td class="trend-cell">
                    <span :class="['badge', 'trend', item.trend]">
                      {{ getTrendLabel(item.trend) }}
                    </span>
                  </td>
                  <td class="qty-cell">{{ item.qtyToRestock }}</td>
                  <td class="cost-cell">${{ formatCurrency(item.unitCost) }}</td>
                  <td class="total-cell"><strong>${{ formatCurrency(item.lineCost) }}</strong></td>
                </tr>
              </tbody>
            </table>
          </div>

          <div class="recommendations-footer">
            <div class="footer-row">
              <span>Total Cost:</span>
              <strong>${{ formatCurrency(totalCost) }}</strong>
            </div>
            <div class="footer-row" :class="remainingBudget < 0 ? 'over' : ''">
              <span>Remaining Budget:</span>
              <strong>${{ formatCurrency(Math.max(remainingBudget, 0)) }}</strong>
            </div>
          </div>

          <button
            @click="placeOrder"
            :disabled="orderSubmitting || selectedItems.length === 0"
            class="place-order-btn"
          >
            <span v-if="!orderSubmitting">Place Order (${{ formatCurrency(totalCost) }})</span>
            <span v-else>Submitting...</span>
          </button>
        </div>
      </div>

      <!-- Items with No Pricing Data -->
      <div v-if="unpricedItems.length > 0" class="card unpriced-items-card">
        <details class="collapsible-section">
          <summary class="collapsible-summary">
            Items without Pricing Data
            <span class="count-badge">{{ unpricedItems.length }}</span>
          </summary>
          <div class="collapsible-content">
            <div class="unpriced-note">
              These items are forecasted but don't have inventory matches, so unit costs are unavailable. Add them to inventory to include in recommendations.
            </div>
            <div class="table-container">
              <table class="unpriced-table">
                <thead>
                  <tr>
                    <th>SKU</th>
                    <th>Item Name</th>
                    <th>Trend</th>
                    <th>Qty Needed</th>
                    <th>Status</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="item in unpricedItems" :key="item.sku">
                    <td class="sku-cell"><strong>{{ item.sku }}</strong></td>
                    <td class="name-cell">{{ item.name }}</td>
                    <td class="trend-cell">
                      <span :class="['badge', 'trend', item.trend]">
                        {{ getTrendLabel(item.trend) }}
                      </span>
                    </td>
                    <td class="qty-cell">{{ item.qtyToRestock }}</td>
                    <td class="status-cell"><span class="badge muted">No unit cost</span></td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </details>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'

export default {
  name: 'Restocking',
  setup() {
    // State
    const loading = ref(true)
    const error = ref(null)
    const demandForecasts = ref([])
    const inventoryItems = ref([])
    const budget = ref(50000)
    const orderSubmitting = ref(false)
    const orderSuccess = ref(false)
    const lastOrderNumber = ref('')

    // Data loading
    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        const [forecasts, inventory] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory()
        ])
        demandForecasts.value = forecasts
        inventoryItems.value = inventory
      } catch (err) {
        error.value = 'Failed to load data: ' + err.message
        console.error('Load error:', err)
      } finally {
        loading.value = false
      }
    }

    onMounted(() => {
      loadData()
    })

    // Build SKU -> inventory item map for O(1) lookups
    const inventoryBySku = computed(() => {
      const map = {}
      for (const item of inventoryItems.value) {
        map[item.sku] = item
      }
      return map
    })

    // Enrich demand forecasts with cost data and restock quantities
    const candidates = computed(() => {
      return demandForecasts.value
        .filter(f => f.trend !== 'decreasing')
        .map(f => {
          const invItem = inventoryBySku.value[f.item_sku]
          const unitCost = invItem ? invItem.unit_cost : null
          const qtyToRestock = f.trend === 'increasing'
            ? f.forecasted_demand - f.current_demand
            : f.forecasted_demand
          const lineCost = unitCost !== null ? qtyToRestock * unitCost : null
          return {
            sku: f.item_sku,
            name: f.item_name,
            trend: f.trend,
            qtyToRestock,
            unitCost,
            lineCost,
            hasPricing: unitCost !== null
          }
        })
    })

    // Items without pricing data (shown separately)
    const unpricedItems = computed(() =>
      candidates.value.filter(c => !c.hasPricing)
    )

    // Greedy selection: maximize item count within budget
    // Priority: increasing-trend items first, then stable
    // Within each group: sort by ascending lineCost (fit more cheap items)
    const selectedItems = computed(() => {
      const priceable = candidates.value
        .filter(c => c.hasPricing)
        .sort((a, b) => {
          // Primary sort: trend priority (increasing=0, stable=1)
          const trendOrder = { increasing: 0, stable: 1 }
          const trendDiff = trendOrder[a.trend] - trendOrder[b.trend]
          if (trendDiff !== 0) return trendDiff
          // Secondary: ascending lineCost (cheaper items first = pack more in budget)
          return a.lineCost - b.lineCost
        })

      const result = []
      let remaining = budget.value
      for (const item of priceable) {
        if (item.lineCost <= remaining) {
          result.push(item)
          remaining -= item.lineCost
        }
        // Don't break — continue checking later items
      }
      return result
    })

    const totalCost = computed(() =>
      selectedItems.value.reduce((sum, item) => sum + item.lineCost, 0)
    )

    const remainingBudget = computed(() => budget.value - totalCost.value)

    // Place Order handler
    const placeOrder = async () => {
      if (selectedItems.value.length === 0) return
      try {
        orderSubmitting.value = true
        orderSuccess.value = false
        const payload = {
          items: selectedItems.value.map(item => ({
            sku: item.sku,
            name: item.name,
            quantity: item.qtyToRestock,
            unit_price: item.unitCost
          })),
          total_value: totalCost.value
        }
        const createdOrder = await api.createOrder(payload)
        lastOrderNumber.value = createdOrder.order_number
        orderSuccess.value = true
        // Reload data to refresh demand forecasts
        await loadData()
      } catch (err) {
        error.value = 'Failed to place order: ' + err.message
        console.error('Order error:', err)
      } finally {
        orderSubmitting.value = false
      }
    }

    // Helpers
    const formatCurrency = (value) => {
      return Math.round(value).toLocaleString('en-US')
    }

    const getTrendLabel = (trend) => {
      const labels = {
        increasing: '↑ Increasing',
        stable: '→ Stable',
        decreasing: '↓ Decreasing'
      }
      return labels[trend] || trend
    }

    return {
      loading,
      error,
      budget,
      selectedItems,
      unpricedItems,
      totalCost,
      remainingBudget,
      orderSubmitting,
      orderSuccess,
      lastOrderNumber,
      placeOrder,
      formatCurrency,
      getTrendLabel
    }
  }
}
</script>

<style scoped>
.restocking-container {
  padding: 20px;
  max-width: 1000px;
  margin: 0 auto;
}

.loading,
.error {
  text-align: center;
  padding: 40px 20px;
  font-size: 1.1em;
}

.error {
  color: #dc2626;
  background: #fee2e2;
  border-radius: 8px;
  border: 1px solid #fca5a5;
  padding: 20px;
}

.page-header {
  margin-bottom: 30px;
}

.page-header h1 {
  font-size: 2em;
  color: #1f2937;
  margin-bottom: 8px;
}

.page-header p {
  color: #6b7280;
  font-size: 1em;
}

.success-banner {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 20px;
  color: #065f46;
  font-size: 0.95em;
}

.card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 24px;
  margin-bottom: 20px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.card-header h2 {
  font-size: 1.3em;
  color: #1f2937;
  margin: 0;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.count-badge {
  background: #f3f4f6;
  color: #6b7280;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 0.85em;
  font-weight: 500;
}

.budget-status {
  padding: 6px 14px;
  border-radius: 6px;
  font-size: 0.9em;
  font-weight: 600;
}

.budget-status.within-budget {
  background: #d1fae5;
  color: #065f46;
}

.budget-status.over-budget {
  background: #fee2e2;
  color: #991b1b;
}

/* Budget Card */
.budget-card {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  border: 2px solid #e2e8f0;
}

.budget-controls {
  display: flex;
  gap: 30px;
  align-items: flex-start;
}

.slider-wrapper {
  flex: 1;
  min-width: 200px;
}

.budget-slider {
  width: 100%;
  height: 8px;
  border-radius: 5px;
  background: #e2e8f0;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
  accent-color: #3b82f6;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);
}

.budget-slider::-moz-range-thumb {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 8px rgba(59, 130, 246, 0.3);
}

.budget-display {
  min-width: 180px;
}

.budget-value {
  font-size: 1.8em;
  font-weight: 700;
  color: #1f2937;
  margin-bottom: 4px;
}

.budget-note {
  color: #6b7280;
  font-size: 0.85em;
}

/* Recommendations Card */
.recommendations-card {
  border: 2px solid #dbeafe;
  background: #f0f9ff;
}

.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: #6b7280;
}

.table-container {
  overflow-x: auto;
  margin-bottom: 20px;
}

.recommendations-table,
.unpriced-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95em;
}

.recommendations-table thead,
.unpriced-table thead {
  background: #f3f4f6;
  border-bottom: 2px solid #e5e7eb;
}

.recommendations-table th,
.unpriced-table th {
  padding: 12px;
  text-align: left;
  font-weight: 600;
  color: #374151;
}

.recommendations-table td,
.unpriced-table td {
  padding: 12px;
  border-bottom: 1px solid #e5e7eb;
  color: #1f2937;
}

.recommendations-table tbody tr:hover,
.unpriced-table tbody tr:hover {
  background: #f9fafb;
}

.sku-cell {
  font-family: 'Courier New', monospace;
  font-size: 0.9em;
  color: #0369a1;
}

.name-cell {
  max-width: 250px;
}

.trend-cell {
  text-align: center;
}

.qty-cell,
.cost-cell,
.total-cell {
  text-align: right;
}

.badge {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 0.85em;
  font-weight: 600;
  white-space: nowrap;
}

.badge.trend {
  font-size: 0.9em;
}

.badge.trend.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.trend.stable {
  background: #dbeafe;
  color: #0c4a6e;
}

.badge.trend.decreasing {
  background: #fee2e2;
  color: #991b1b;
}

.badge.muted {
  background: #f3f4f6;
  color: #6b7280;
}

.recommendations-footer {
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  padding: 16px;
  margin-bottom: 16px;
}

.footer-row {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  color: #1f2937;
}

.footer-row:first-child {
  border-bottom: 1px solid #e5e7eb;
  padding-bottom: 12px;
  margin-bottom: 12px;
}

.footer-row.over strong {
  color: #dc2626;
}

.place-order-btn {
  width: 100%;
  padding: 12px 24px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 6px;
  font-size: 1em;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.place-order-btn:hover:not(:disabled) {
  background: #2563eb;
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.3);
}

.place-order-btn:disabled {
  background: #d1d5db;
  cursor: not-allowed;
  opacity: 0.6;
}

/* Unpriced Items */
.unpriced-items-card {
  border: 1px solid #fcd34d;
  background: #fffbeb;
}

.collapsible-section {
  list-style: none;
  padding: 0;
}

.collapsible-summary {
  display: flex;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
  padding: 12px;
  background: #fef3c7;
  border-radius: 6px;
  font-weight: 600;
  color: #92400e;
  user-select: none;
  transition: background 0.2s ease;
}

.collapsible-summary:hover {
  background: #fde68a;
}

.collapsible-content {
  padding-top: 16px;
}

.unpriced-note {
  background: white;
  border: 1px solid #fcd34d;
  border-radius: 6px;
  padding: 12px;
  margin-bottom: 16px;
  font-size: 0.9em;
  color: #78350f;
}

.status-cell {
  text-align: center;
}

/* Responsive */
@media (max-width: 768px) {
  .budget-controls {
    flex-direction: column;
  }

  .table-container {
    font-size: 0.85em;
  }

  .card {
    padding: 16px;
  }

  .card-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }

  .recommendations-table th,
  .recommendations-table td,
  .unpriced-table th,
  .unpriced-table td {
    padding: 8px;
  }

  .page-header h1 {
    font-size: 1.5em;
  }
}
</style>
