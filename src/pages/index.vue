<template>
  <v-date-picker v-model="date"></v-date-picker>
  <h1 class="heading">{{ monthName }}</h1>
  <BaseMonth :date="date" :monthName="monthName" />
</template>

<script lang="ts" setup>
import BaseMonth from "../components/BaseMonth.vue";

import { ref, computed } from "vue";

const date = ref(new Date());

const monthName = computed(() => {
  return date.value.toLocaleString("pl-PL", { month: "long" });
});
</script>

<style scoped>
.heading {
  text-transform: capitalize;
}
</style>
