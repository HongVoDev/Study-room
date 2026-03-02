<template>
  <div class="container">
    <div v-if="!roomCode">
      <h1>Let's Focus!</h1>
      <div class="form-group">
        <label for="roomCode">Add room code:</label>
        <input type="text" id="roomCode" v-model="roomCodeInput" class="form-control">
        <button @click="joinRoom" class="btn btn-secondary">Join Room</button>
      </div>
      <div v-if="!isStudying">
        <h1>Start your own session</h1>
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
            <input
              type="text"
              id="task"
              v-model="newTask"
              @keydown.enter.prevent="addTask"
              class="form-control"
              placeholder="Enter task and press Enter"
            >
          </div>

          <button type="submit" class="btn btn-primary">Let's Study</button>
        </form>

        <!-- Initial Checklist Display -->
        <div v-if="tasks.length > 0">
          <h3>Checklist:</h3>
          <ul>
            <li
              v-for="(task, index) in tasks"
              :key="index"
              @click="toggleTask(index)"
              :class="{ completed: task.completed }"
            >
              {{ task.text }}
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div v-else="" class="split-screen">
      <div class="left-pane">
        <h1>Studying: {{ subject }}</h1>
        <div class="timer">{{ formatTime(remainingTime) }}</div>

        <!-- Checklist Display during study session -->
        <div v-if="tasks.length > 0">
          <h3>Checklist:</h3>
          <ul>
            <li
              v-for="(task, index) in tasks"
              :key="index"
              @click="toggleTask(index)"
              :class="{ completed: task.completed }"
            >
              {{ task.text }}
            </li>
          </ul>
        </div>
      </div>
      <div class="right-pane">
        <h1>Studying: Mock Friend's Session</h1>
        <div class="timer">{{ formatTime(friendRemainingTime) }}</div>
        <h3>Checklist:</h3>
        <ul>
          <li v-for="(task, index) in friendTasks" :key="index" :class="{ completed: task.completed }">
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

    // Room Code
    const roomCode = ref('');
    const roomCodeInput = ref('');

    // Mock Friend Data
    const friendRemainingTime = ref(600); // 10 minutes
    const friendTasks = ref([
      { text: 'Read documentation', completed: false },
      { text: 'Implement feature', completed: true },
    ]);
    let friendTimerInterval = null;

    const startStudying = () => {
      if (subject.value.trim() === '' || duration.value <= 0) {
        alert('Please enter a subject and a valid duration.');
        return;
      }
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
      remainingTime.value = 0; // Reset the timer to 0 when study ends.
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
      //Corrected this line to directly toggle completed state.
      tasks.value[index].completed = !tasks.value[index].completed;
    };

    // Room Code Functionality
    const joinRoom = () => {
      roomCode.value = roomCodeInput.value;
      startFriendTimer(); // Start friend's timer when joining room
    };

    //Mock Friend Timer
    const startFriendTimer = () => {
      friendTimerInterval = setInterval(() => {
        if (friendRemainingTime.value > 0) {
          friendRemainingTime.value--;
        } else {
          stopFriendTimer();
        }
      }, 1000);
    };

    const stopFriendTimer = () => {
      clearInterval(friendTimerInterval);
      friendTimerInterval = null;
    };

    onUnmounted(() => {
      stopTimer(); // Clear interval when component unmounts.
      stopFriendTimer(); // Clear friend's timer
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
      roomCode,
      roomCodeInput,
      joinRoom,
      friendRemainingTime,
      friendTasks,
    };
  },
};
</script>

<style scoped="">
.container {
  max-width: 800px; /* Increased width for split screen */
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
  margin-top: 10px; /* Added margin for better spacing */
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-primary:hover {
  background-color: #0056b3;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #545b62;
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
  color: black;
}

li:hover {
  background-color: #e0e0e0;
}

.completed {
  text-decoration: line-through;
  color: #888;
}

/* Split Screen Styles */
.split-screen {
  display: flex;
  height: 500px; /*Adjust as needed*/
}

.left-pane,
.right-pane {
  width: 50%;
  padding: 20px;
  border: 1px solid #ccc;
  box-sizing: border-box; /* Important to include padding and border in the element's total width and height */
  text-align: center; /* Keep content centered in each pane */
}

.left-pane {
  border-right: none; /*Remove border between panes*/
}

.right-pane {
  border-left: none; /*Remove border between panes*/
}
</style>
