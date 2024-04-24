<template>
  <v-date-picker v-model="date"></v-date-picker>
  <h1 class="heading">{{ monthName }}</h1>
  <BaseMonth :date="date" />
</template>

<script lang="ts" setup>
import BaseMonth from "../components/BaseMonth.vue";

import { ref, computed } from "vue";

const currentDate = new Date();
const nextMonthDate = new Date(
  currentDate.getFullYear(),
  currentDate.getMonth() + 1,
  currentDate.getDate()
);
const date = ref(nextMonthDate);

const monthName = computed(() => {
  return date.value.toLocaleString("pl-PL", { month: "long" });
});
</script>

<style scoped>
.heading {
  text-transform: capitalize;
}
</style>
