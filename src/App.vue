<template>
  <div
    class="min-h-screen bg-gradient-to-tr from-gray-700 via-gray-900 to-black flex flex-col justify-between"
  >
    <div class="container mx-auto py-8">
      <div class="header mb-6">
        <h1 class="text-4xl font-bold text-center mb-2 text-white">Gym Bros</h1>
        <p class="text-xl text-center mb-4 text-white">
          The Tracking Calendar for Gym Enthusiasts &#x1F4AA;
        </p>
      </div>
      <div class="profile-selector mb-4">
        <label for="profile" class="mr-2 text-white">Select Profile:</label>
        <select
          v-model="selectedProfile"
          id="profile"
          @change="loadProfile"
          class="border rounded p-2"
        >
          <option
            v-for="(profile, index) in profiles"
            :key="index"
            :value="index"
          >
            {{ profile.name }}
          </option>
        </select>
      </div>
      <div class="month-selector mb-4">
        <label for="month" class="mr-2 text-white">Select Month:</label>
        <select
          v-model="selectedMonth"
          id="month"
          @change="updateCalendar"
          class="border rounded p-2"
        >
          <option v-for="(month, index) in months" :key="index" :value="index">
            {{ month }}
          </option>
        </select>
      </div>
      <div class="difficulty-selector mb-4">
        <label for="difficulty" class="mr-2 text-white"
          >Select Difficulty:</label
        >
        <select
          v-model="selectedDifficulty"
          id="difficulty"
          @change="updateScore"
          class="border rounded p-2"
        >
          <option
            v-for="(difficulty, index) in difficulties"
            :key="index"
            :value="difficulty.value"
          >
            {{ difficulty.label }}
          </option>
        </select>
      </div>
      <div class="score mb-6 flex items-center justify-center">
        <button
          class="text-xl score-button bg-yellow-200 text-blue-800 hover:text-blue-900 font-bold py-2 px-4 rounded"
        >
          Score: {{ score }}
        </button>
      </div>

      <table class="w-full">
        <thead>
          <tr>
            <th v-for="day in daysOfWeek" :key="day" class="py-2">{{ day }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(week, weekIndex) in calendar" :key="weekIndex">
            <td
              v-for="(day, dayIndex) in week"
              :key="day.date"
              :class="{
                'current-month': day.isCurrentMonth,
                selected: day.isSelected,
              }"
              @click="toggleDaySelection(weekIndex, dayIndex)"
              class="py-2 cursor-pointer"
            >
              {{ day.date }}
            </td>
          </tr>
        </tbody>
      </table>
      <button
        @click="saveProfile"
        class="save-button bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mt-4"
      >
        Save
      </button>
    </div>

    <div class="score mb-2 flex items-center justify-center">
      <button
        @click="showInformation"
        class="info-button bg-gray-400 hover:bg-blue-700 text-white text-3xl py-2 px-4 rounded mt-4"
      >
        &#128712;
      </button>
    </div>

    <div
      v-if="showModal"
      class="fixed inset-0 flex items-center justify-center z-50 bg-gray-900 bg-opacity-75"
    >
      <div class="bg-white rounded-lg p-6">
        <h2 class="text-2xl font-bold mb-4">
          &#x1F4AA; Site Information &#x1F4AA;
        </h2>
        <p>
          This site, Gym Bros, is a calendar for gym enthusiasts. It helps you
          plan and track your workouts. Here are some details:
        </p>
        <ul class="mt-4 mb-4">
          <li>
            <strong>Difficulties:</strong>
            <ul class="ml-4">
              <li><strong>Easy:</strong> Handicap 4</li>
              <li><strong>Normal:</strong> Handicap 3.33</li>
              <li><strong>Hard:</strong> Handicap 2</li>
            </ul>
          </li>
        </ul>
        <div>
          <strong>Scores:</strong>
          <p>
            You can select a difficulty level and score your monthly workouts.
            Please always press SAVE to save your progress.
          </p>
        </div>
        <button
          @click="closeModal"
          class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded mt-6"
        >
          Close
        </button>
      </div>
    </div>

    <footer
      class="bg-gradient-to-r from-gray-900 to-blue-900 text-gray-300 py-3 px-6 flex items-center justify-between sticky bottom-0 z-50"
    >
      <a
        href="https://github.com/alejocg/thegymbros"
        target="_blank"
        rel="noopener noreferrer"
      >
        <p class="text-white-600 text-sm">Github |</p>
      </a>
      <p class="text-white-600 text-sm">| Made by Alejo with &#x2764;</p>
      <div class="ml-auto">
        <p class="text-white-600 text-sm">Alpha 27523</p>
      </div>
    </footer>
  </div>
</template>

<script>
import { createClient } from "@supabase/supabase-js";
const supabase = createClient(
  process.env.VUE_APP_SUPABASE_URL,
  process.env.VUE_APP_SUPABASE_KEY
);

export default {
  data() {
    return {
      showModal: false,

      selectedProfile: 0,
      profiles: [
        { name: "AC", dates: [] },
        { name: "AM", dates: [] },
        { name: "JB", dates: [] },
        { name: "JM", dates: [] },
      ],
      selectedMonth: new Date().getMonth(),
      months: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ],
      daysOfWeek: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"],
      calendar: [],
      score: 0,
      selectedDifficulty: "normal",
      difficulties: [
        { label: "Easy", value: "easy", handicap: 4 },
        { label: "Normal", value: "normal", handicap: 3.33 },
        { label: "Hard", value: "hard", handicap: 2 },
      ],
    };
  },

  mounted() {
    this.loadProfile();
    this.updateCalendar();
  },

  methods: {
    showInformation() {
      this.showModal = true;
    },

    closeModal() {
      this.showModal = false;
    },
    updateCalendar() {
      const year = new Date().getFullYear();
      const month = this.selectedMonth;
      const firstDay = new Date(year, month, 1).getDay();
      const daysInMonth = new Date(year, month + 1, 0).getDate();

      const calendar = [];
      let week = [];
      for (let i = 0; i < firstDay; i++) {
        week.push({ date: "", isCurrentMonth: false, isSelected: false });
      }

      for (let day = 1; day <= daysInMonth; day++) {
        if (week.length === 7) {
          calendar.push(week);
          week = [];
        }
        week.push({ date: day, isCurrentMonth: true, isSelected: false });
      }

      calendar.push(week);
      this.calendar = calendar;
    },
    toggleDaySelection(weekIndex, dayIndex) {
      const day = this.calendar[weekIndex][dayIndex];
      day.isSelected = !day.isSelected;

      const monthIndex = this.selectedMonth;
      const selectedProfile = this.profiles[this.selectedProfile];
      const selectedDays = this.calendar
        .flat()
        .filter((day) => day.isSelected)
        .map((day) => `${monthIndex}-${day.date}`);

      selectedProfile.days = selectedDays;
      this.updateScore();
    },
    updateScore() {
      const selectedDifficulty = this.difficulties.find(
        (difficulty) => difficulty.value === this.selectedDifficulty
      );
      const handicap = selectedDifficulty.handicap;
      this.score = this.calculateScore(handicap);
    },
    calculateScore(handicap) {
      const selectedDays = this.calendar.flat().filter((day) => day.isSelected);
      const baseScore = selectedDays.length;
      const score = baseScore * handicap;
      return Math.round(score); // Round the score
    },
    loadProfile() {
      // Fetch the selected profile's data from the database
      supabase
        .from("profiles")
        .select("dates")
        .eq("id", this.selectedProfile)
        .single()
        .then(({ data, error }) => {
          if (error) {
            console.error("Error fetching profile data:", error);
            window.alert("Error!! Call Alejo!!");
            return;
          }

          const savedDates = data.dates || [];

          this.calendar.forEach((week) => {
            week.forEach((day) => {
              const isCurrentMonth = day.isCurrentMonth;
              const dayOfMonth = day.date;
              const monthIndex = this.selectedMonth;
              const date = `${monthIndex}-${dayOfMonth}`;

              // Check if the day belongs to the selected month and is saved
              day.isSelected = isCurrentMonth && savedDates.includes(date);
            });
          });

          this.updateScore();
        })
        .catch((error) => {
          console.error("Error fetching profile data:", error);
          window.alert("Error!! Call Alejo!!");
        });
    },
    async saveProfile() {
      const selectedProfile = this.profiles[this.selectedProfile];
      selectedProfile.dates = this.calendar
        .flat()
        .filter((day) => day.isSelected)
        .map((day) => `${this.selectedMonth}-${day.date}`);

      try {
        // Save the selected profile in the Supabase database
        const { data, error } = await supabase
          .from("profiles")
          .upsert([{ id: this.selectedProfile, dates: selectedProfile.dates }]);

        if (error) {
          console.error("Error saving dates:", error);
          window.alert("Error!! Call Alejo!!");
          return;
        }

        console.log("Dates saved successfully:", data);
        window.alert("Dates saved successfully!");
      } catch (error) {
        console.error("Error saving dates:", error);
        window.alert("Error!! Call Alejo!!");
      }
    },
  },
};
</script>

<style scoped>
.calendar {
  max-width: 500px;
  margin: 0 auto;
}

.month-selector {
  margin-bottom: 20px;
}

.difficulty-selector {
  margin-bottom: 20px;
}

.score {
  margin-bottom: 10px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 10px;
  text-align: center;
}

th {
  background-color: #eee;
}

td {
  border: 1px solid #ddd;
  cursor: pointer;
}

.current-month {
  background-color: #f0f0f0;
}

.selected {
  background-color: #aaf;
}
</style>
