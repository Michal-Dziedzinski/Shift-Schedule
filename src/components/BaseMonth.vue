<template>
  <div id="element-to-convert" class="calendar">
    <div class="days">
      <BaseDay v-for="day in days" :key="day.id" :day="day">
        <v-select
          v-model="getDoctorSelect(day.id)[0]"
          :items="sortedDoctorsMaternityWard[day.id - 1]"
          item-title="fullName"
          hint="Wybierz lekarza porodówka"
          label="Lekarze"
          persistent-hint
        >
          <template v-slot:item="{ props, item }">
            <v-list-item
              v-bind="props"
              :base-color="
                item.raw.available ? 'green' : item.raw.unavailable ? 'red' : ''
              "
            >
            </v-list-item>
          </template>
        </v-select>
        <v-select
          v-model="getDoctorSelect(day.id)[1]"
          :items="sortedDoctorsOCP[day.id - 1]"
          item-title="fullName"
          hint="Wybierz lekarza OCP"
          label="Lekarze"
          multiple
          persistent-hint
        >
          <template v-slot:item="{ props, item }">
            <v-list-item
              v-bind="props"
              :base-color="
                item.raw.available ? 'green' : item.raw.unavailable ? 'red' : ''
              "
            >
            </v-list-item>
          </template>
        </v-select>
      </BaseDay>
    </div>
  </div>
  <v-btn
    @click="clearSelection"
    class="button"
    color="red-darken-1"
    append-icon="mdi-trash-can-outline"
  >
    Wyczyść
  </v-btn>
  <v-btn
    @click="generatePDFwithSelections"
    class="button"
    color="green-accent-4"
    append-icon="mdi-export-variant"
  >
    Generuj PDF
  </v-btn>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";
import html2pdf from "html2pdf.js";
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

const {
  doctorsOCP,
  doctorsMaternityWard,
}: { doctorsOCP: Doctor[]; doctorsMaternityWard: Doctor[] } = doctorsList;

const doctorsSelects = ref<Record<number, any>>({});

const days = computed(() => {
  const month = props.date.getMonth();
  const daysInMonth = new Date(
    props.date.getFullYear(),
    month + 1,
    0
  ).getDate();
  const daysArray: Day[] = [];
  for (let day = 1; day <= daysInMonth; day++) {
    daysArray.push({
      id: day,
      date: new Date(props.date.getFullYear(), month, day),
    });
  }
  return daysArray;
});

const mapDoctors = (doctors: Doctor[], id: number) => {
  return doctors
    .map((doctor) => {
      return {
        ...doctor,
        fullName: `${doctor.name} ${doctor.surname}`,
        available: doctor.available.includes(id),
        unavailable: doctor.unavailable.includes(id),
      };
    })
    .sort((a, b) => {
      if (a.available && !b.available) {
        return -1;
      } else if (!a.available && b.available) {
        return 1;
      } else if (a.unavailable && !b.unavailable) {
        return 1;
      } else if (!a.unavailable && b.unavailable) {
        return -1;
      } else {
        return 0;
      }
    });
};

const clearSelection = () => {
  doctorsSelects.value = {};
};

const sortedDoctorsOCP = computed(() => {
  return days.value.map(({ id }) => {
    return mapDoctors(doctorsOCP, id);
  });
});
const sortedDoctorsMaternityWard = computed(() => {
  return days.value.map(({ id }) => {
    return mapDoctors(doctorsMaternityWard, id);
  });
});

const getDoctorSelect = (dayId: number) => {
  if (!doctorsSelects.value[dayId]) {
    doctorsSelects.value[dayId] = ref([]);
  }
  return doctorsSelects.value[dayId];
};

const generatePDFwithSelections = () => {
  html2pdf(document.getElementById("element-to-convert"), {
    margin: 1,
    filename: "i-was-html.pdf",
    html2canvas: {
      scrollX: 0,
      scrollY: 0,
    },
  });
};
</script>

<style scoped>
.calendar {
  display: flex;
  flex-direction: column;
  width: 100%;
  margin-bottom: 32px;
}

.button {
  margin-right: 16px;
}

.days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
}
</style>
