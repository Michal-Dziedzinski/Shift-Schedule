<template>
  <div class="calendar">
    <div class="week" v-for="week in weeks" :key="week.id">
      <BaseDay v-for="day in week.days" :key="day.id" :day="day"></BaseDay>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from "vue";
import BaseDay from "./BaseDay.vue";

interface Day {
  id: number;
  date: Date;
}

interface Week {
  id: number;
  days: Day[];
}

const weeks = ref<Week[]>([]);

onMounted(() => {
  const date = new Date();
  const month = date.getMonth();
  let day = 1;
  while (day <= new Date(date.getFullYear(), month + 1, 0).getDate()) {
    const week: Day[] = [];
    for (let i = 0; i < 7; i++) {
      week.push({
        id: day,
        date: new Date(date.getFullYear(), month, day++),
      });
    }
    weeks.value.push({ id: weeks.value.length + 1, days: week });
  }
});
</script>

<style scoped>
.calendar {
  display: flex;
  flex-direction: column;
  width: 100%;
}

.week {
  display: flex;
  justify-content: space-between;
}
</style>
