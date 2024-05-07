<template>
  <div v-if="session">
    <v-date-picker v-model="date"></v-date-picker>
    <h1 class="heading">{{ monthName }}</h1>
    <Suspense>
      <template #default>
        <BaseMonth :date="date" />
      </template>
      <template #fallback>
        <v-progress-circular indeterminate></v-progress-circular>
      </template>
    </Suspense>
  </div>
  <AuthForm v-else />
</template>

<script lang="ts" setup>
import BaseMonth from "../components/BaseMonth.vue";
import AuthForm from "../components/AuthForm.vue";
import { supabase } from "../supabase";

import { ref, computed, onMounted } from "vue";

const session = ref();

onMounted(() => {
  supabase.auth.getSession().then(({ data }) => {
    session.value = data.session;
  });

  supabase.auth.onAuthStateChange((_, _session) => {
    session.value = _session;
  });
});

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
