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

<div class="screen">
  <h1>🎲 Shake to Roll!</h1>
  
  <div class="dice {isRolling ? 'shake' : ''}">
    {diceValue}
  </div>

  <button on:click={rollDice} class="btn">
    {isRolling ? 'Rolling...' : 'Tap to Roll'}
  </button>

  <button on:click={startShake} id="enableBtn" class="btn enable">
    Enable Shake
  </button>

  <p class="tip">Shake your phone to roll the dice</p>
</div>

<style>
  .screen {
    height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    background: #4f46e5;
    color: white;
    font-family: Arial, sans-serif;
    padding: 20px;
  }

  h1 {
    font-size: 2rem;
    margin-bottom: 40px;
  }

  .dice {
    width: 100px;
    height: 100px;
    background: white;
    border-radius: 15px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2.5rem;
    font-weight: bold;
    color: #333;
    margin-bottom: 40px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.3);
  }

  .shake {
    animation: shake 0.5s;
  }

  @keyframes shake {
    0% { transform: rotate(0deg); }
    25% { transform: rotate(-10deg); }
    50% { transform: rotate(10deg); }
    75% { transform: rotate(-10deg); }
    100% { transform: rotate(0deg); }
  }

  .btn {
    background: white;
    color: #4f46e5;
    border: none;
    padding: 12px 25px;
    font-size: 1.1rem;
    border-radius: 20px;
    cursor: pointer;
    margin: 5px;
    font-weight: bold;
  }

  .enable {
    background: #ffd700;
    color: #000;
  }

  .tip {
    margin-top: 30px;
    opacity: 0.8;
  }
</style>