<template>
  <div class="container">
    <h1>Order Adjustment Analysis</h1>
    <div class="filters">
      <label>
        Select Store:
        <select v-model="selectedStore">
          <option
            v-for="store in stores"
            :key="store.id_store"
            :value="store.id_store"
          >
            {{ store.store_label }}
          </option>
        </select>
      </label>
      <label>
        Select Product:
        <select v-model="selectedProduct">
          <option
            v-for="product in products"
            :key="product.id_product"
            :value="product.id_product"
          >
            {{ product.name_product }}
          </option>
        </select>
      </label>
    </div>

    <div
      v-if="selectedStore && selectedProduct && chartData.length"
      class="chart-container"
    >
      <h2>Data Analysis for Selected Store and Product</h2>
      <ChartComponent :data="chartData" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";

import storesData from "./data/Stores.json";
import productsData from "./data/Products.json";
import salesData from "./data/Sales.json";
import recommendationsData from "./data/Recommendations.json";
import deliveriesData from "./data/Deliveries.json";
import ChartComponent from "./components/ChartComponent.vue";

const stores = ref([]);
const products = ref([]);

const selectedStore = ref(null);
const selectedProduct = ref(null);

// Load data into Vue components
onMounted(() => {
  try {
    stores.value = storesData.stores || [];
    products.value = productsData.products || [];
  } catch (error) {
    console.error("Error loading data:", error);
  }
});

// Computed property for filtering data
const filteredData = computed(() => {
  if (!selectedStore.value || !selectedProduct.value) {
    return {
      sales: [],
      recommendations: [],
      deliveries: [],
    };
  }

  try {
    const filteredSales = salesData.filter(
      (sale) =>
        sale.id_store === selectedStore.value &&
        sale.id_product === selectedProduct.value
    );

    const filteredRecommendations = recommendationsData.recommendations.filter(
      (rec) =>
        rec.id_store === selectedStore.value &&
        rec.id_product === selectedProduct.value
    );

    const filteredDeliveries = deliveriesData.deliveries.filter(
      (del) =>
        del.id_store === selectedStore.value &&
        del.id_product === selectedProduct.value
    );

    return {
      sales: filteredSales,
      recommendations: filteredRecommendations,
      deliveries: filteredDeliveries,
    };
  } catch (error) {
    console.error("Error filtering data:", error);
    return {
      sales: [],
      recommendations: [],
      deliveries: [],
    };
  }
});

// Computed property for chart data
const chartData = computed(() => {
  const { sales, recommendations, deliveries } = filteredData.value;

  if (
    recommendations.length === 0 ||
    deliveries.length === 0 ||
    sales.length === 0
  ) {
    return [];
  }

  // Merge and sort data by date
  return recommendations
    .map((rec) => {
      const correspondingDelivery = deliveries.find(
        (del) => del.target_date === rec.target_date
      );
      const correspondingSale = sales.find(
        (sale) => sale.target_date === rec.target_date
      );

      return {
        date: rec.target_date,
        recommended: rec.recommendation,
        delivered: correspondingDelivery
          ? correspondingDelivery.delivery_qty
          : null,
        sales: correspondingSale ? correspondingSale.sales_qty : null,
      };
    })
    .sort((a, b) => new Date(a.date) - new Date(b.date));
});
</script>

<style>
.container {
  max-width: 800px;
  margin: auto;
  padding: 20px;
  text-align: center;
}

.filters {
  display: flex;
  justify-content: space-around;
  margin-bottom: 20px;
}

select {
  margin-left: 10px;
}

.chart-container {
  margin-top: 20px;
}
</style>
