<template>
  <div>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUpdated } from "vue";
import { Chart } from "chart.js/auto";

const props = defineProps({
  data: {
    type: Array,
    default: () => [],
  },
});

const canvas = ref(null);
let chartInstance = null;

// Function to calculate if the delivery was better compared to the recommendation
function calculateImprovementStatus(data) {
  return data.map((point) => {
    if (point.delivered !== null && point.recommended !== null) {
      const recommendedDiff = Math.abs(point.recommended - point.sales);
      const deliveredDiff = Math.abs(point.delivered - point.sales);

      if (deliveredDiff < recommendedDiff) {
        // Delivered quantity is closer to sales, so it's "improved"

        point.status = "improved";
        point.recommendedStatus = "deteriorated";
      } else if (deliveredDiff > recommendedDiff) {
        // Recommended quantity is closer to sales, so it's "improved"
        point.status = "deteriorated";
        point.recommendedStatus = "improved";
      } else {
        // Both are equally close to sales
        point.status = "equal";
        point.recommendedStatus = "equal";
      }
    } else {
      point.status = "unknown";
      point.recommendedStatus = "unknown";
    }
    return point;
  });
}

// Helper function to get point color based on status
function getPointColor(status) {
  switch (status) {
    case "improved":
      return "green";
    case "deteriorated":
      return "red";
    case "equal":
      return "white";
    default:
      return "gray";
  }
}

function renderChart(data) {
  const processedData = calculateImprovementStatus(data);

  if (!chartInstance) {
    chartInstance = new Chart(canvas.value, {
      type: "line",
      data: {
        labels: processedData.map((point) => point.date),
        datasets: [
          {
            label: "Recommended Quantity",
            data: processedData.map((point) => point.recommended),
            borderColor: "yellow",
            borderWidth: 1,
            pointBackgroundColor: processedData.map((point) =>
              getPointColor(point.recommendedStatus)
            ),
          },
          {
            label: "Delivered Quantity",
            data: processedData.map((point) => point.delivered),
            borderColor: "green",
            borderWidth: 1,
            pointBackgroundColor: processedData.map((point) =>
              getPointColor(point.status)
            ),
          },
        ],
      },
      options: {
        plugins: {
          tooltip: {
            callbacks: {
              label: function (context) {
                const point = processedData[context.dataIndex];
                const statusText =
                  context.dataset.label === "Delivered Quantity"
                    ? point.status
                    : point.recommendedStatus;
                return `${context.dataset.label}: ${context.raw} (${
                  statusText.charAt(0).toUpperCase() + statusText.slice(1)
                })`;
              },
            },
          },
        },
      },
    });
  } else {
    chartInstance.data.labels = processedData.map((point) => point.date);
    chartInstance.data.datasets[0].data = processedData.map(
      (point) => point.recommended
    );
    chartInstance.data.datasets[0].pointBackgroundColor = processedData.map(
      (point) => getPointColor(point.recommendedStatus)
    );
    chartInstance.data.datasets[1].data = processedData.map(
      (point) => point.delivered
    );
    chartInstance.data.datasets[1].pointBackgroundColor = processedData.map(
      (point) => getPointColor(point.status)
    );
    chartInstance.update();
  }
}

onMounted(() => {
  if (props.data.length > 0) {
    renderChart(props.data);
  }
});

onUpdated(() => {
  if (props.data.length > 0) {
    renderChart(props.data);
  }
});
</script>

<style scoped>
canvas {
  width: 100%;
  max-width: 800px;
  margin: auto;
}
</style>
