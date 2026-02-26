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
        <button type="submit" class="btn btn-primary">Let's Study</button>
      </form>
    </div>

    <div v-else="">
      <h1>Studying: {{ subject }}</h1>
      <div class="timer">
        {{ formatTime(remainingTime) }}
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
          alert("Time's up!"); // Or you can trigger another event here.
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

input[type="text"],
input[type="number"] {
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
</style>
