<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking Planner</h2>
      <p>
        Set your budget and get recommendations for items to restock based on
        demand forecasts.
      </p>
    </div>

    <div class="card budget-card">
      <div class="card-header">
        <h3 class="card-title">Budget</h3>
      </div>
      <div class="budget-controls">
        <div class="slider-row">
          <input
            type="range"
            class="budget-slider"
            min="1000"
            max="50000"
            step="1000"
            v-model.number="budget"
          />
          <span class="budget-display">${{ budget.toLocaleString() }}</span>
        </div>
        <button
          class="btn-primary"
          @click="loadRecommendations"
          :disabled="loading"
        >
          {{ loading ? "Loading..." : "Get Recommendations" }}
        </button>
      </div>
    </div>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="successMessage" class="success-banner">
      {{ successMessage }}
    </div>

    <div
      v-if="recommendations && recommendations.items.length > 0"
      class="card"
    >
      <div class="card-header">
        <h3 class="card-title">
          Recommended Items ({{ recommendations.items.length }})
        </h3>
        <div class="budget-summary">
          <span
            >Total:
            <strong
              >${{ recommendations.total_cost.toLocaleString() }}</strong
            ></span
          >
          <span class="budget-remaining">
            Remaining: ${{
              (
                recommendations.budget - recommendations.total_cost
              ).toLocaleString()
            }}
          </span>
        </div>
      </div>
      <div class="table-container">
        <table class="restock-table">
          <thead>
            <tr>
              <th>SKU</th>
              <th>Item Name</th>
              <th>Forecasted Demand</th>
              <th>Current Stock</th>
              <th>Demand Gap</th>
              <th>Unit Cost</th>
              <th>Restock Qty</th>
              <th>Line Total</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in recommendations.items" :key="item.item_sku">
              <td>
                <code>{{ item.item_sku }}</code>
              </td>
              <td>{{ item.item_name }}</td>
              <td>{{ item.forecasted_demand.toLocaleString() }}</td>
              <td>{{ item.current_stock.toLocaleString() }}</td>
              <td>
                <span class="badge danger">{{
                  item.demand_gap.toLocaleString()
                }}</span>
              </td>
              <td>${{ item.unit_cost.toFixed(2) }}</td>
              <td>
                <strong>{{ item.restock_quantity.toLocaleString() }}</strong>
              </td>
              <td>
                <strong>${{ item.line_total.toLocaleString() }}</strong>
              </td>
            </tr>
          </tbody>
          <tfoot>
            <tr>
              <td colspan="7" class="total-label">Total Cost</td>
              <td>
                <strong
                  >${{ recommendations.total_cost.toLocaleString() }}</strong
                >
              </td>
            </tr>
          </tfoot>
        </table>
      </div>
      <div class="card-footer">
        <button class="btn-submit" @click="placeOrder" :disabled="submitting">
          {{ submitting ? "Placing Order..." : "Place Order" }}
        </button>
      </div>
    </div>

    <div
      v-if="recommendations && recommendations.items.length === 0"
      class="card empty-state"
    >
      <p>
        No items need restocking within this budget, or all demand is met by
        current stock.
      </p>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import { api } from "../api";

export default {
  name: "Restocking",
  setup() {
    const budget = ref(10000);
    const loading = ref(false);
    const error = ref(null);
    const recommendations = ref(null);
    const submitting = ref(false);
    const successMessage = ref("");

    const loadRecommendations = async () => {
      try {
        loading.value = true;
        error.value = null;
        successMessage.value = "";
        recommendations.value = await api.getRestockingRecommendations(
          budget.value,
        );
      } catch (err) {
        error.value = "Failed to load recommendations: " + err.message;
      } finally {
        loading.value = false;
      }
    };

    const placeOrder = async () => {
      if (!recommendations.value || !recommendations.value.items.length) return;
      try {
        submitting.value = true;
        error.value = null;
        const orderData = {
          budget: recommendations.value.budget,
          items: recommendations.value.items.map((item) => ({
            item_sku: item.item_sku,
            item_name: item.item_name,
            restock_quantity: item.restock_quantity,
            unit_cost: item.unit_cost,
          })),
        };
        await api.submitRestockingOrder(orderData);
        successMessage.value =
          "Restocking order placed successfully! View it in the Orders tab.";
        recommendations.value = null;
      } catch (err) {
        error.value = "Failed to place order: " + err.message;
      } finally {
        submitting.value = false;
      }
    };

    return {
      budget,
      loading,
      error,
      recommendations,
      submitting,
      successMessage,
      loadRecommendations,
      placeOrder,
    };
  },
};
</script>

<style scoped>
.budget-card {
  margin-bottom: 1.5rem;
}

.budget-controls {
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.slider-row {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.budget-slider {
  flex: 1;
  height: 6px;
  -webkit-appearance: none;
  appearance: none;
  background: #e2e8f0;
  border-radius: 3px;
  outline: none;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #2563eb;
  cursor: pointer;
  border: 2px solid #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.budget-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #2563eb;
  cursor: pointer;
  border: 2px solid #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);
}

.budget-display {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
  min-width: 120px;
  text-align: right;
}

.btn-primary {
  align-self: flex-start;
  padding: 0.625rem 1.5rem;
  background: #2563eb;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-primary:hover {
  background: #1d4ed8;
}
.btn-primary:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.budget-summary {
  display: flex;
  gap: 1.5rem;
  font-size: 0.9rem;
  color: #334155;
}

.budget-remaining {
  color: #059669;
}

.restock-table {
  width: 100%;
  border-collapse: collapse;
}

.restock-table th {
  text-align: left;
  padding: 0.75rem 1rem;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
  background: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
}

.restock-table td {
  padding: 0.75rem 1rem;
  font-size: 0.875rem;
  border-bottom: 1px solid #f1f5f9;
}

.restock-table tbody tr:hover {
  background: #f8fafc;
}

.restock-table tfoot td {
  padding: 0.875rem 1rem;
  font-size: 0.9rem;
  background: #f8fafc;
  border-top: 2px solid #e2e8f0;
}

.total-label {
  text-align: right;
  font-weight: 600;
  color: #334155;
}

code {
  font-family: "SF Mono", "Fira Code", "Consolas", monospace;
  font-size: 0.8rem;
  background: #f1f5f9;
  padding: 2px 6px;
  border-radius: 4px;
}

.card-footer {
  padding: 1rem 1.5rem;
  border-top: 1px solid #e2e8f0;
  display: flex;
  justify-content: flex-end;
}

.btn-submit {
  padding: 0.75rem 2rem;
  background: #059669;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-submit:hover {
  background: #047857;
}
.btn-submit:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.success-banner {
  background: #d1fae5;
  color: #065f46;
  padding: 1rem 1.5rem;
  border-radius: 8px;
  font-weight: 500;
  margin-bottom: 1.5rem;
  border: 1px solid #6ee7b7;
}

.empty-state {
  padding: 2rem;
  text-align: center;
  color: #64748b;
}
</style>
