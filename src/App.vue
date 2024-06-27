<script setup>
import { ref, shallowRef, computed, watch, nextTick, onMounted } from "vue";
import Chart from "chart.js/auto";

const weights = ref([]);
const weightsCharte1 = ref(null);
const weightsChart = shallowRef(null);
const weightInput = ref(0);
const errorMessage = ref("");

// Ambil data dari local storage saat aplikasi dimuat
onMounted(() => {
  const savedWeights = localStorage.getItem("weights");
  if (savedWeights) {
    weights.value = JSON.parse(savedWeights);
  }
});

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  if (weightInput.value <= 0) {
    errorMessage.value = "Invalid data. Weight must be greater than zero.";
    return;
  }

  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime(),
  });
  localStorage.setItem("weights", JSON.stringify(weights.value));
  weightInput.value = 0;
};

const deleteWeight = (index) => {
  weights.value.splice(index, 1);
  localStorage.setItem("weights", JSON.stringify(weights.value));
};

watch(
  weights,
  async (newWeights) => {
    const ws = [...newWeights];
    if (weightsChart.value) {
      weightsChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => new Date(weight.date).toLocaleDateString())
        .slice(-7);

      weightsChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((weight) => weight.weight)
        .slice(-7);

      weightsChart.value.update();
      return;
    }

    await nextTick();
    if (weightsCharte1.value) {
      weightsChart.value = new Chart(weightsCharte1.value.getContext("2d"), {
        type: "line",
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: "Weight",
              data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
              backgroundColor: "#5AB2FF",
              borderColor: "#A0DEFF",
              borderWidth: 1,
              fill: true,
            },
          ],
        },
        options: { responsive: true, maintainAspectRatio: false },
      });
    }
  },
  { deep: true }
);
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>
    <div class="current">
      <span>{{ currentWeight.weight }}</span>
      <small>Current Weight (KG)</small>
    </div>
    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" />
      <input type="submit" value="Add Weight" />
    </form>
    <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    <div v-if="weights && weights.length > 0">
      <h2>Last 7 Days</h2>
      <div class="canvas-box">
        <canvas ref="weightsCharte1"></canvas>
      </div>
      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="(weight, index) in weights" :key="weight.date">
            <span>{{ weight.weight }} Kg </span>
            <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
            <button @click="deleteWeight(index)" class="dlt">Delete</button>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>
