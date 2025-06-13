<template>
  <div id="element-to-convert" class="calendar">
    <div class="days">
      <BaseDay v-for="(day, index) in days" :key="day.date" :day="day">
        <v-select
          v-model="getDoctorSelect(day.id)[0]"
          :items="sortedDoctorsMaternityWard[index]"
          item-title="fullName"
          hint="Wybierz lekarza porodówka"
          label="Lekarze"
          multiple
          :disabled="day.disabled"
          persistent-hint
        >
          <template v-slot:item="{ props, item }">
            <v-list-item
              v-bind="props"
              :base-color="
                item.raw.isAvailable
                  ? 'green'
                  : item.raw.isUnavailable
                  ? 'red'
                  : ''
              "
            >
            </v-list-item>
          </template>
        </v-select>
        <v-select
          v-model="getDoctorSelect(day.id)[1]"
          :items="sortedDoctorsOCP[index]"
          item-title="fullName"
          hint="Wybierz lekarza OCP"
          label="Lekarze"
          multiple
          :disabled="day.disabled"
          persistent-hint
        >
          <template v-slot:item="{ props, item }">
            <v-list-item
              v-bind="props"
              :base-color="
                item.raw.isAvailable
                  ? 'green'
                  : item.raw.isUnavailable
                  ? 'red'
                  : ''
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
    @click="openInGoogleSheets"
    class="button"
    color="green-darken-1"
    append-icon="mdi-google-spreadsheet"
  >
    Otwórz w Google Sheets
  </v-btn>
  <SignOut />
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";
import { useStorage } from "@vueuse/core";
import BaseDay from "./BaseDay.vue";
import SignOut from "./SignOut.vue";
import { supabase } from "../supabase";

interface Day {
  id: number;
  date: Date;
  disabled?: boolean;
}

interface Doctor {
  name: string;
  surname: string;
  id: number;
  unavailable: number[] | null | undefined;
  available: number[] | null | undefined;
}

const fetchDoctorsList = async (table) => {
  try {
    const { data: doctorsList, error } = await supabase.from(table).select("*");
    if (error) {
      throw error;
    }
    return doctorsList;
  } catch (error) {
    console.error("Error fetching doctors list:", error.message);
    return [];
  }
};

const doctorsMaternityWard: Doctor[] = await fetchDoctorsList(
  "doctors_maternity_ward"
);
const doctorsOCP: Doctor[] = await fetchDoctorsList("doctors_ocp");

const props = defineProps({
  date: {
    type: Object as () => Date,
    required: true,
  },
});

const doctorsSelects = useStorage(
  "doctorsSelects",
  ref<Record<number, any>>({})
);

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
  const date = new Date(daysArray[0].date);
  const dayOfWeek = date.getDay();
  const index = dayOfWeek === 0 ? 7 : dayOfWeek;
  if (index > 1) {
    for (let i = 1; i < index; i++) {
      daysArray.unshift({
        id: i + 100,
        date: new Date(date.setDate(date.getDate() - 1)),
        disabled: true,
      });
    }
  }
  return daysArray;
});

const mapDoctors = (doctors: Doctor[], id: number) => {
  return doctors
    .map((doctor) => {
      return {
        ...doctor,
        fullName: `${doctor.name} ${doctor.surname}`,
        isAvailable: doctor.available?.includes(id) || false,
        isUnavailable: doctor.unavailable?.includes(id) || false,
      };
    })
    .sort((a, b) => {
      if (a.isAvailable && !b.isAvailable) {
        return -1;
      } else if (!a.isAvailable && b.isAvailable) {
        return 1;
      } else if (a.isUnavailable && !b.isUnavailable) {
        return 1;
      } else if (!a.isUnavailable && b.isUnavailable) {
        return -1;
      } else {
        return a.fullName.localeCompare(b.fullName);
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

const openInGoogleSheets = () => {
  const monthName = props.date.toLocaleString("pl-PL", { month: "long" });
  const year = props.date.getFullYear();

  let data = days.value
    .map((day) => {
      if (day.disabled) return null;

      const selections = doctorsSelects.value[day.id];

      const maternityDoctors =
        typeof selections?.[0] === "string"
          ? [selections[0]]
          : Array.isArray(selections?.[0])
          ? selections[0]
          : [];

      const ocpDoctors =
        typeof selections?.[1] === "string"
          ? [selections[1]]
          : Array.isArray(selections?.[1])
          ? selections[1]
          : [];

      return [
        day.date.getDate(),
        maternityDoctors.join(" | "),
        ocpDoctors.join(" | "),
      ];
    })
    .filter((row) => row !== null);

  const headers = [["Dzień", "Lekarze Porodówka", "Lekarze OCP"]];
  const title = `Harmonogram ${monthName} ${year}`;

  const sheet = [headers[0], ...data]
    .map((row) => row.map((cell) => cell || "").join("\t"))
    .join("\n");

  const blob = new Blob([sheet], {
    type: "text/tab-separated-values;charset=utf-8",
  });
  const url = URL.createObjectURL(blob);
  const link = document.createElement("a");
  link.href = url;
  link.download = `${title}.tsv`;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  URL.revokeObjectURL(url);

  alert(`Aby zaimportować dane do Google Sheets:
1. Kliknij "File" -> "Import"
2. Wybierz zakładkę "Upload"
3. Przeciągnij lub wybierz pobrany plik "${title}.tsv"
4. W opcjach importu wybierz:
   - Separator: "Tab"
   - "Replace current sheet"
5. Kliknij "Import data"`);

  window.open("https://docs.google.com/spreadsheets/create", "_blank");
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
