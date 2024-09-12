<script setup>
  import { ref, computed } from 'vue';

  const breakSound = new Audio('/audio/acoustic-motivation-189747.mp3');

  //REVISAR RESPONSIVE DESIGN
  const minutes = ref(0);
  const seconds = ref(0);
  const result = ref(0);
  const timeInterval = ref(null);
  const isPressed = ref(false);
  const isBreak = ref(false);

  //FORMAT TIME WITH TWO DIGITS
    const addFormat = timeUnit => timeUnit < 10 ? '0' + timeUnit : timeUnit;
    const minutesWithFormat = computed(() => addFormat(minutes.value));
    const secondsWithFormat = computed(() => addFormat(seconds.value));
  
    //INITIALIZE THE WORK COUNTER
    const setCounter = () => {
      seconds.value++;
      if (seconds.value >= 60) {
        seconds.value = 0;
        minutes.value++;
      }
      if (minutes.value >= 1) {
        clearInterval(timeInterval.value);
        result.value++;
        startBreak();
        return;
      }
    };

    //START THE BREAK AFTER 25 MINUTES
    const startBreak = () => {
      playBreakSound();
      minutes.value = 1;
      seconds.value = 0;
      isBreak.value = true;
      timeInterval.value = setInterval(setBreakCounter, 1000);
    };
    //WHEN THE BREAK STARTS, THE SOUND PLAYS
    const playBreakSound = () => {
      breakSound.play();
    };
    
    //PAUSE SOUND
    const pauseBreakSound = () => {
        breakSound.pause();
        breakSound.currentTime = 0;
        //isBreakSoundPlaying.value = false;
    };

    //SET THE COUNTDOWN FOR THE BREAK
    const setBreakCounter = () => {
      seconds.value--;
      if (seconds.value < 0) {
        seconds.value = 59;
        minutes.value--;
      }
      if (minutes.value === 0 && seconds.value === 0) {
        clearInterval(timeInterval.value);
        isBreak.value = false;
        startWork();
      }
    };

    //RESTART WORK MODE AFTER THE BREAK 
    const startWork = () => {
      pauseBreakSound();
      minutes.value = 0;
      seconds.value = 0;
      isBreak.value = false;
      timeInterval.value = setInterval(setCounter, 1000);
    };

    //HANDLE TIME INTERVALS BETWEEN PLAY AND PAUSE BUTTONS 
    const toggleTimer = () => {
      if (isPressed.value) {
        clearInterval(timeInterval.value);
        isPressed.value = false;
        if (isBreak.value) {
          breakSound.pause();
        } 
      } else {
        if (isBreak.value) {
          timeInterval.value = setInterval(setBreakCounter, 1000);
          playBreakSound();
        } else {
          timeInterval.value = setInterval(setCounter, 1000);
        }
        isPressed.value = true;
      }
    };
    
    //RESET THE TIMER
    const resetTimer = () => {
      clearInterval(timeInterval.value);
      minutes.value = 0;
      seconds.value = 0;
      isPressed.value = false;
      isBreak.value = false;
      pauseBreakSound();
    };

    //CHANGE STATES BETWEEN BREAK MODE AND WORK MODE
    const modeText = computed(() => (isBreak.value ? 'Break Mode' : 'Work Mode'));
    const modeClass = computed(() => (isBreak.value ? 'break-mode' : 'work-mode'));
 
</script>

<template>
  <div class="container vh-100 d-flex flex-column justify-content-center align-items-center text-white">
    <div class="container border border-4 rounded-4 d-flex flex-column justify-content-center align-items-center">
        <div 
          class="container state" 
          :class="modeClass">{{ modeText.toUpperCase() }}
        </div>  
        <div 
          ref="counter" 
          id="counter">{{ minutesWithFormat }}:{{ secondsWithFormat }} 
        </div>
        <div class="button-container d-flex justify-content-center">
          <button 
            id="play-and-pause-button" 
            type="button" 
            @click="toggleTimer" 
            class="btn btn-lg"
            :class="isPressed? 'btn-outline-warning' : 'btn-outline-success'"
            >
            <i :class="isPressed? 'bi bi-pause-fill' : 'bi bi-play-fill'"></i>
          </button>
          <button 
            id="reset-button" 
            type="button"
            @click="resetTimer" 
            class="btn btn-outline-danger btn-lg"
            >
            <i class="bi bi-arrow-clockwise"></i>
          </button>
        </div>
        <h2 id="modules">Total Work Sessions: {{ result }}</h2>   
    </div> 
  </div>
</template>

