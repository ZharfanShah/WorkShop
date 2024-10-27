<script setup>
import { ref, onMounted } from "vue";
import PocketBase from "pocketbase";

const db = new PocketBase('http://127.0.0.1:8090');

const selectedTime = ref("");
const selectedSubject = ref("");
const timetable = ref([]);
const timeSlots = ref([]);
const subjects = ref([]);

// Fetch unique time slots and subjects from timetable collection
async function fetchTimetableOptions() {
  try {
    const records = await db.collection("timetable").getFullList();
    
    // Get unique time slots and subjects, filtering out "N/A" values
    timeSlots.value = Array.from(new Set(
      records.filter(record => record.timeSlot !== "N/A").map(record => record.timeSlot)
    )).map(slot => ({ value: slot }));

    subjects.value = Array.from(new Set(
      records.filter(record => record.subject !== "N/A").map(record => record.subject)
    )).map(subject => ({ value: subject }));
    
  } catch (error) {
    console.error("Error fetching timetable options:", error);
  }
}

async function submit() {
  try {
    // Set the subject or timeSlot to "N/A" if one is missing
    const record = {
      timeSlot: selectedTime.value || "N/A",
      subject: selectedSubject.value || "N/A",
    };

    // Check if the slot is already occupied for that time
    const existingSlot = timetable.value.find(
      item => item.timeSlot === record.timeSlot && item.subject !== "N/A"
    );

    if (existingSlot) {
      alert(`Time slot ${selectedTime.value} already has ${existingSlot.subject}`);
      return;
    }

    // Add the record to the timetable
    timetable.value.push(record);

    // Reset the selections
    selectedSubject.value = "";
    selectedTime.value = "";
  } catch (error) {
    console.error("Error adding to timetable:", error);
  }
}

function deleteEntry(index) {
  timetable.value.splice(index, 1);
}

// Print function
function printTimetable() {
  const printSection = document.getElementById('timetable-section');
  const printWindow = window.open('', '_blank');

  printWindow.document.write(`
    <html>
      <head>
        <title>Print Timetable</title>
        <style>
          body {
            font-family: Arial, sans-serif;
          }
          table {
            width: 100%;
            border-collapse: collapse;
          }
          th, td {
            border: 1px solid #ddd;
            padding: 8px;
          }
          th {
            background-color: #f2f2f2;
          }
        </style>
      </head>
      <body>
        <h1>Timetable</h1>
        ${printSection.innerHTML}
      </body>
    </html>
  `);

  printWindow.document.close();
  printWindow.focus();
  printWindow.print();
  printWindow.close();
}

onMounted(() => {
  fetchTimetableOptions();
});
</script>

<template>
  <div class="container mx-auto p-4">
    <div class="bg-white rounded-lg shadow-md p-6 mb-6">
      <h1 class="text-2xl font-bold text-gray-800 mb-20 hover:text-green-500 transition-colors duration-200 relative group">Class Timetable
        <span class="tooltip absolute left-0 top-full mt-1 bg-gray-700 text-white text-sm rounded p-2 opacity-0 group-hover:opacity-100 transition-opacity duration-200">
          Create Your Own Timetable !
        </span>
      </h1>

      <div class="flex flex-col md:flex-row gap-4 mb-6">
        <!-- Time Slot Selection -->
        <select 
          v-model="selectedTime"
          class="p-2 border rounded-md w-full md:w-48 bg-white"
        >
          <option value="">Select Time</option>
          <option 
            v-for="slot in timeSlots" 
            :key="slot.value" 
            :value="slot.value"
          >
            {{ slot.value }}
          </option>
        </select>

        <!-- Subject Selection -->
        <select 
          v-model="selectedSubject"
          class="p-2 border rounded-md w-full md:w-48 bg-white"
        >
          <option value="">Select Subject</option>
          <option 
            v-for="subject in subjects" 
            :key="subject.value" 
            :value="subject.value"
          >
            {{ subject.value }}
          </option>
        </select>

        <button 
          @click="submit"
          class="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600 transition-colors w-full md:w-auto"
          :disabled="!selectedTime && !selectedSubject"
        >
          Add to Timetable
        </button>
      </div>

      <div class="overflow-x-auto" id="timetable-section">
        <table class="min-w-full divide-y divide-gray-200">
          <thead class="bg-gray-50">
            <tr>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Time
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Subject
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Actions
              </th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200">
            <tr v-for="(entry, index) in timetable" :key="index">
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                {{ entry.timeSlot }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                {{ entry.subject }}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm">
                <button 
                  @click="deleteEntry(index)"
                  class="text-red-600 hover:text-red-800 transition-colors"
                >
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Print Button -->
      <div class="mt-4">
        <button @click="printTimetable" class="bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600 transition-colors">
          Print Timetable
        </button>
      </div>
    </div>
  </div>
</template>

<style>
/* Styling for print */
@media print {
  body * {
    visibility: hidden;
  }

  #timetable-section, #timetable-section * {
    visibility: visible;
  }

  #timetable-section {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
  }
}
</style>
