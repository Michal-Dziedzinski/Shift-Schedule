<template>
  <div class="calendar">
    <div class="days">
      <BaseDay v-for="day in days" :key="day.id" :day="day">
        <v-select
          v-model="getDoctorSelect(day.id)[0]"
          :items="doctorsNames"
          hint="Wybierz lekarza"
          label="Lekarze"
          multiple
          persistent-hint
        ></v-select>
      </BaseDay>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, watch } from "vue";
import BaseDay from "./BaseDay.vue";
import doctorsList from "../data/doctors.json";

interface Day {
  id: number;
  date: Date;
}

interface Doctor {
  name: string;
  surname: string;
  id: number;
  unavailable: number[];
  available: number[];
}

const props = defineProps({
  date: {
    type: Object as () => Date,
    required: true,
  },
});

const { doctors }: { doctors: Doctor[] } = doctorsList;

const doctorsNames = doctors.map(
  (doctor) => `${doctor.name} ${doctor.surname}`
);

const doctorsSelects = ref<Record<number, any>>({});

const days = ref<Day[]>([]);

const updateDays = () => {
  const month = props.date.getMonth();
  const daysInMonth = new Date(
    props.date.getFullYear(),
    month + 1,
    0
  ).getDate();
  days.value = [];
  for (let day = 1; day <= daysInMonth; day++) {
    days.value.push({
      id: day,
      date: new Date(props.date.getFullYear(), month, day),
    });
  }
};

const getDoctorSelect = (dayId: number) => {
  if (!doctorsSelects.value[dayId]) {
    doctorsSelects.value[dayId] = ref([]);
  }
  return doctorsSelects.value[dayId];
};

onMounted(() => {
  updateDays();
});

watch(
  () => props.date,
  () => {
    updateDays();
  }
);
</script>

<style scoped>
.calendar {
  display: flex;
  flex-direction: column;
  width: 100%;
}

.days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
}
</style>
