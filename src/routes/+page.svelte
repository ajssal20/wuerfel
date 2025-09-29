<script>
  import { onMount } from 'svelte';
  
  let diceValue = 1;
  let isRolling = false;

  function rollDice() {
    if (isRolling) return;
    
    isRolling = true;
    
    // Quick rolling animation
    let count = 0;
    const roll = setInterval(() => {
      diceValue = Math.floor(Math.random() * 6) + 1;
      count++;
      if (count > 8) {
        clearInterval(roll);
        isRolling = false;
      }
    }, 80);
  }

  // Handle device motion
  function handleMotion(event) {
    const acc = event.accelerationIncludingGravity;
    if (!acc) return;

    // Check if phone is being shaken
    const shake = Math.abs(acc.x) + Math.abs(acc.y) + Math.abs(acc.z);
    if (shake > 35 && !isRolling) {
      rollDice();
    }
  }

  // Enable motion detection
  async function startShake() {
    // iOS needs permission
    if (DeviceMotionEvent && typeof DeviceMotionEvent.requestPermission === 'function') {
      try {
        const response = await DeviceMotionEvent.requestPermission();
        if (response === 'granted') {
          window.addEventListener('devicemotion', handleMotion);
        }
      } catch (e) {
        alert('Need motion access to detect shaking');
      }
    } else {
      // Android and other browsers
      window.addEventListener('devicemotion', handleMotion);
    }
    
    // Hide the enable button
    document.getElementById('enableBtn').style.display = 'none';
  }

  onMount(() => {
    // Auto-start for browsers that don't need permission
    if (!DeviceMotionEvent || typeof DeviceMotionEvent.requestPermission !== 'function') {
      window.addEventListener('devicemotion', handleMotion);
      document.getElementById('enableBtn').style.display = 'none';
    }
  });
</script>

<div class="min-h-screen bg-gradient-to-br from-indigo-600 to-purple-600 flex flex-col items-center justify-center p-5 text-white text-center">
  <h1 class="text-3xl font-bold mb-10">🎲 Shake to Roll!</h1>
  
  <div class="w-24 h-24 bg-white rounded-xl flex items-center justify-center text-4xl font-bold text-gray-800 shadow-2xl mb-10 {isRolling ? 'animate-shake' : ''}">
    {diceValue}
  </div>

  <div class="flex flex-col gap-3">
    <button on:click={rollDice} class="bg-white text-indigo-600 border-none px-6 py-3 text-lg rounded-2xl cursor-pointer font-bold shadow-lg hover:bg-gray-100 transition-colors">
      {isRolling ? 'Rolling...' : 'Tap to Roll'}
    </button>

    <button on:click={startShake} id="enableBtn" class="bg-yellow-400 text-black border-none px-6 py-3 text-lg rounded-2xl cursor-pointer font-bold shadow-lg hover:bg-yellow-300 transition-colors">
      Enable Shake
    </button>
  </div>

  <p class="mt-8 opacity-80 text-lg">Shake your phone to roll the dice</p>
</div>

<style>
  @keyframes shake {
    0% { transform: rotate(0deg); }
    25% { transform: rotate(-10deg); }
    50% { transform: rotate(10deg); }
    75% { transform: rotate(-10deg); }
    100% { transform: rotate(0deg); }
  }
  
  .animate-shake {
    animation: shake 0.5s ease-in-out;
  }
</style>