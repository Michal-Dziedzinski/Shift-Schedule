<template>
  <div class="day">
    {{ day.date.getDate() }}
    <v-select
      v-model="doctorsSelects"
      :items="doctorsNames"
      hint="Wybierz lekarza"
      label="Select"
      multiple
      persistent-hint
    ></v-select>
  </div>
</template>

<script setup lang="ts">
import { defineProps, ref } from "vue";

import doctorsList from "../data/doctors.json";

interface Day {
  id: number;
  date: Date;
}

interface Doctor {
  name: string;
  surname: string;
}

defineProps({
  day: {
    type: Object as () => Day,
    required: true,
  },
});

const { doctors }: { doctors: Doctor[] } = doctorsList;

const doctorsNames = doctors.map(
  (doctor) => `${doctor.name} ${doctor.surname}`
);

const doctorsSelects = ref([]);
</script>

<style scoped>
.day {
  flex: 1;
  border: 1px solid #ccc;
  padding: 10px;
  text-align: center;
}
</style>
