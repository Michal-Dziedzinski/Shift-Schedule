<template>
  <div class="day">
    {{ day.date.getDate() }}
    <slot />
  </div>
</template>

<script setup lang="ts">
interface Day {
  id: number;
  date: Date;
}

defineProps({
  day: {
    type: Object as () => Day,
    required: true,
  },
});
</script>

<style scoped>
.day {
  flex: 1;
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
}
</style>
