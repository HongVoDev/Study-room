<template>
  <div class="container">
    <div v-if="!isStudying">
      <h1>Let's Focus!</h1>
      <form @submit.prevent="startStudying">
        <div class="form-group">
          <label for="subject">What are you studying?</label>
          <input type="text" id="subject" v-model="subject" class="form-control" required="">
        </div>
        <div class="form-group">
          <label for="duration">Study Duration (minutes):</label>
          <input type="number" id="duration" v-model.number="duration" class="form-control" min="1" required="">
        </div>
        <!-- Checklist Input -->
        <div class="form-group">
          <label for="task">Add Task to Checklist:</label>
          <input type="text" id="task" v-model="newTask" @keyup.enter="addTask" class="form-control" placeholder="Enter task and press Enter">
        </div>

        <button type="submit" class="btn btn-primary">Let's Study</button>
      </form>

      <!-- Initial Checklist Display -->
      <div v-if="tasks.length > 0">
        <h3>Checklist:</h3>
        <ul>
          <li v-for="(task, index) in tasks" :key="index" @click="toggleTask(index)" :class="{ completed: task.completed }">
            {{ task.text }}
          </li>
        </ul>
      </div>
    </div>

    <div v-else="">
      <h1>Studying: {{ subject }}</h1>
      <div class="timer">
        {{ formatTime(remainingTime) }}
      </div>

      <!-- Checklist Display during study session -->
      <div v-if="tasks.length > 0">
        <h3>Checklist:</h3>
        <ul>
          <li v-for="(task, index) in tasks" :key="index" @click="toggleTask(index)" :class="{ completed: task.completed }">
            {{ task.text }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue';

export default {
  setup() {
    const subject = ref('');
    const duration = ref(25); // Default duration, you can change this
    const isStudying = ref(false);
    const remainingTime = ref(0);
    let timerInterval = null;

    // Checklist Data
    const tasks = ref([]);
    const newTask = ref('');

    const startStudying = () => {
      isStudying.value = true;
      remainingTime.value = duration.value * 60; // Convert minutes to seconds
      startTimer();
    };

    const startTimer = () => {
      timerInterval = setInterval(() => {
        if (remainingTime.value > 0) {
          remainingTime.value--;
        } else {
          stopTimer();
          alert('Time is up!'); 
          resetStudySession();
        }
      }, 1000);
    };

    const stopTimer = () => {
      clearInterval(timerInterval);
      timerInterval = null;
    };

    const resetStudySession = () => {
      isStudying.value = false;
      subject.value = '';
      duration.value = 25; // Reset to default.
    };

    const formatTime = (seconds) => {
      const minutes = Math.floor(seconds / 60);
      const remainingSeconds = seconds % 60;
      return `${String(minutes).padStart(2, '0')}:${String(remainingSeconds).padStart(2, '0')}`;
    };

    // Checklist Functionality
    const addTask = () => {
      if (newTask.value.trim() !== '') {
        tasks.value.push({ text: newTask.value.trim(), completed: false });
        newTask.value = '';
      }
    };

    const toggleTask = (index) => {
      tasks.value[index].completed = !tasks.value[index].completed;
    };

    onUnmounted(() => {
      stopTimer(); // Clear interval when component unmounts.
    });


    return {
      subject,
      duration,
      isStudying,
      remainingTime,
      startStudying,
      formatTime,
      resetStudySession,
      tasks, //Expose tasks
      newTask, // Expose newTask
      addTask, // Expose addTask method
      toggleTask, //Expose toggleTask

    };
  }
};
</script>

<style scoped="">
.container {
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  text-align: center;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input[type='text'],
input[type='number'] {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
  margin-top: 5px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-primary:hover {
  background-color: #0056b3;
}

.timer {
  font-size: 2em;
  margin-top: 20px;
}

/* Checklist styles */
ul {
  list-style-type: none;
  padding: 0;
}

li {
  padding: 8px 12px;
  margin-bottom: 5px;
  background-color: #f0f0f0;
  border-radius: 4px;
  cursor: pointer;
  text-align: left;
}

li:hover {
  background-color: #e0e0e0;
}

.completed {
  text-decoration: line-through;
  color: #888;
}
</style>
