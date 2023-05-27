<template>
  <div
    class="min-h-screen bg-gradient-to-tr from-gray-700 via-gray-900 to-black flex flex-col justify-between"
  >
    <div class="container mx-auto py-8">
      <div class="header mb-6">
        <h1 class="text-4xl font-bold text-center mb-2 text-white">Gym Bros</h1>
        <p class="text-xl text-center mb-4 text-white">
          The Calendar for Gym Enthusiasts &#x1F4AA;
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
    <footer
      class="bg-gradient-to-r from-gray-900 to-blue-900 text-gray-300 py-3 px-6 flex items-center justify-between sticky bottom-0 z-50"
    >
      <p class="text-white-600 text-sm">Made by Alejo with &#x2764;</p>
    </footer>
  </div>
</template>

<script>
 

import { createClient } from "@supabase/supabase-js";
const supabase = createClient(
  "https://fbnqzkjzrogkmnfekqlq.supabase.co",
  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImZibnF6a2p6cm9na21uZmVrcWxxIiwicm9sZSI6ImFub24iLCJpYXQiOjE2ODUxMzQwMTIsImV4cCI6MjAwMDcxMDAxMn0.OjdrpP6JO1pHjwh8l_g2KB8NCIz-Iwnd1XXy7XimrGY"
);

export default {
  data() {
    return {
      selectedProfile: 0,
      profiles: [
        { name: "AC", days: [] },
        { name: "AM", days: [] },
        { name: "JB", days: [] },
        { name: "JM", days: [] },
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
      const selectedProfile = this.profiles[this.selectedProfile];
      this.calendar.forEach((week) => {
        week.forEach((day) => {
          day.isSelected = selectedProfile.days.includes(day.date);
        });
      });
      this.updateScore();
    },

    async saveProfile() {
      const selectedProfile = this.profiles[this.selectedProfile];
      selectedProfile.days = this.calendar
        .flat()
        .filter((day) => day.isSelected)
        .map((day) => day.date);

      try {
        // Save the selected profile in the Supabase database
        const { data, error } = await supabase
          .from("profiles")
          .upsert([{ id: this.selectedProfile, days: selectedProfile.days }]);

        if (error) {
          console.error("Error saving days:", error);
          window.alert("Error!! Call Alejo!!");

          return;
        }

        console.log("Days saved successfully:", data);
        window.alert("Days saved successfully!");

      } catch (error) {
        console.error("Error saving days:", error);
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
