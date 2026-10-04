<script>
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from './assets/vite.svg'
  import heroImg from './assets/hero.png'

  // Smart Blanket State
  let isPoweredOn = false;
  let temperature = 70; // 55°F to 95°F
  let weight = 10;        // 5 lbs to 20 lbs
  let selectedZone = "Both"; // 'Feet', 'Body', 'Both'
  let timerMinutes = 30;

  // Control Handlers
  function togglePower() {
    isPoweredOn = !isPoweredOn;
  }

  function adjustTemp(change) {
    if (!isPoweredOn) return;
    temperature = Math.min(Math.max(temperature + change, 55), 95);
  }

  function adjustWeight(change) {
    if (!isPoweredOn) return;
    weight = Math.min(Math.max(weight + change, 5), 20);
  }

  function setZone(zone) {
    if (!isPoweredOn) return;
    selectedZone = zone;
  }

  function setTimer(mins) {
    if (!isPoweredOn) return;
    timerMinutes = mins;
  }
</script>

<section id="center">
  <div>
    <h1>Project 1: Smart Blanket</h1>
    <p>Annalise Smith</p>
  </div>

  <!-- Smart Blanket Interactive UI -->
  <main class="smart-blanket-card">
    <div 
      class="status-panel" 
      class:active={isPoweredOn}
      class:cooling={isPoweredOn && temperature < 70}
      class:heating={isPoweredOn && temperature >= 70}
    >
      <p class="power-status">
        Status: 
        <strong>
          {#if !isPoweredOn}
            Off
          {:else if temperature < 70}
            Chilling Active ❄️
          {:else}
            Warming Active ☀️
          {/if}
        </strong>
      </p>

      {#if isPoweredOn}
        <div class="metrics-grid">
          <div class="metric-box">
            <span class="label">Temperature</span>
            <span class="value">{temperature}°F</span>
          </div>
          <div class="metric-box">
            <span class="label">Blanket Weight</span>
            <span class="value">{weight} lbs</span>
          </div>
        </div>

        <div class="sub-metrics">
          <p>Active Zone: <strong>{selectedZone}</strong></p>
          <p>Auto shut-off: <strong>{timerMinutes} mins</strong></p>
        </div>
      {/if}
    </div>

    <!-- Controls -->
    <div class="controls">
      <button class="power-btn" on:click={togglePower}>
        {isPoweredOn ? "Power Off" : "Power On"}
      </button>

      {#if isPoweredOn}
        <!-- Temperature Controls -->
        <div class="control-group">
          <label>Temperature (55°F – 95°F)</label>
          <div class="btn-row">
            <button on:click={() => adjustTemp(-1)} disabled={temperature <= 55}>− Cool</button>
            <button on:click={() => adjustTemp(1)} disabled={temperature >= 95}>+ Warm</button>
          </div>
        </div>

        <!-- Weight Controls -->
        <div class="control-group">
          <label>Pressure / Weight (5 – 20 lbs)</label>
          <div class="btn-row">
            <button on:click={() => adjustWeight(-1)} disabled={weight <= 5}>− Lighter</button>
            <button on:click={() => adjustWeight(1)} disabled={weight >= 20}>+ Heavier</button>
          </div>
        </div>

        <!-- Zone Controls -->
        <div class="control-group">
          <label>Heat/Chill Zone</label>
          <div class="btn-row">
            <button class:selected={selectedZone === 'Feet'} on:click={() => setZone('Feet')}>Feet</button>
            <button class:selected={selectedZone === 'Body'} on:click={() => setZone('Body')}>Body</button>
            <button class:selected={selectedZone === 'Both'} on:click={() => setZone('Both')}>Both</button>
          </div>
        </div>

        <!-- Timer Controls -->
        <div class="control-group">
          <label>Timer Preset</label>
          <div class="btn-row">
            {#each [15, 30, 60, 120] as mins}
              <button 
                class:selected={timerMinutes === mins} 
                on:click={() => setTimer(mins)}>
                {mins}m
              </button>
            {/each}
          </div>
        </div>
      {/if}
    </div>
  </main>
</section>

<div class="ticks"></div>

<section id="next-steps"></section>

<div class="ticks"></div>
<section id="spacer"></section>

<style>
  .smart-blanket-card {
    max-width: 380px;
    margin: 1.5rem auto;
    padding: 1.5rem;
    border-radius: 12px;
    background-color: #242424;
    color: #ffffff;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.4);
    font-family: system-ui, sans-serif;
    text-align: center;
  }

  .status-panel {
    padding: 1rem;
    border-radius: 8px;
    background-color: #333;
    margin-bottom: 1.2rem;
    transition: background-color 0.3s ease;
  }

  .status-panel.cooling {
    background-color: #1a3a4b;
  }

  .status-panel.heating {
    background-color: #4a2c1d;
  }

  .power-status {
    margin: 0;
    font-size: 1rem;
  }

  .metrics-grid {
    display: flex;
    justify-content: space-around;
    margin: 0.8rem 0;
  }

  .metric-box {
    display: flex;
    flex-direction: column;
  }

  .metric-box .label {
    font-size: 0.75rem;
    opacity: 0.8;
  }

  .metric-box .value {
    font-size: 1.8rem;
    font-weight: bold;
  }

  .sub-metrics {
    font-size: 0.85rem;
    margin-top: 0.5rem;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
    padding-top: 0.5rem;
  }

  .sub-metrics p {
    margin: 0.2rem 0;
  }

  .controls {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .control-group {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.3rem;
  }

  .control-group label {
    font-size: 0.8rem;
    color: #aaa;
  }

  .btn-row {
    display: flex;
    gap: 0.4rem;
    justify-content: center;
    width: 100%;
  }

  button {
    flex: 1;
    padding: 0.5rem 0.8rem;
    border: none;
    border-radius: 6px;
    font-weight: bold;
    cursor: pointer;
    background-color: #444;
    color: white;
    transition: background-color 0.2s ease;
  }

  button:hover:not(:disabled) {
    background-color: #666;
  }

  button:disabled {
    opacity: 0.3;
    cursor: not-allowed;
  }

  .power-btn {
    background-color: #ff3e00;
    font-size: 1rem;
    padding: 0.7rem;
  }

  .power-btn:hover {
    background-color: #e03700;
  }

  button.selected {
    background-color: #00adb5;
    color: #fff;
  }
</style>
