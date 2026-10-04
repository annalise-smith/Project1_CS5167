<script>
  import { onMount } from 'svelte';

  // Demo clock: 1 "minute" of timer = 3 seconds. Set to 60000 for real minutes.
  const MINUTE_MS = 3000;
  const MIN_TEMP = 55, MAX_TEMP = 100;
  const MIN_WEIGHT = 2, MAX_WEIGHT = 20;
  const timers = [15, 30, 60, 120];
  const colors = ['#f6c9d8', '#bcd9f0', '#cfe8d2', '#fbeab0', '#dccff0', '#fbd5bd', '#dfe3e8'];
  const rainbow = 'conic-gradient(#ff5f6d, #ffc371, #f9f871, #5ee7a0, #4facfe, #b57bff, #ff5f6d)';

  let connected = true;
  let power = true;
  let menuOpen = false;

  let upper = 91, upperTarget = 95;
  let lower = 88, lowerTarget = 92;
  let weight = 6;
  let timer = 15, remaining = 15;
  let color = colors[1];
  let customColor = '#ff6b9d';
  let customPicked = false;
  let battery = 82;
  let emergency = false;
  let plugged = false;
  let colorMenuOpen = false;
  let stitchColor = '#4a5570';
  let matchStitch = false;

  const mode = (cur, target, on, link) =>
    !on || !link ? 'off' : cur < target ? 'warming' : cur > target ? 'cooling' : 'idle';

  $: upperMode = mode(upper, upperTarget, power, connected);
  $: lowerMode = mode(lower, lowerTarget, power, connected);
  $: zones = [
    { key: 'upper', mode: upperMode },
    { key: 'lower', mode: lowerMode }
  ];
  $: status = emergency ? 'Emergency off' : !connected ? 'Disconnected' : !power ? 'Off'
    : [upperMode, lowerMode].includes('warming') ? 'Warming'
    : [upperMode, lowerMode].includes('cooling') ? 'Cooling' : 'Maintaining current state';

  $: usable = connected && !emergency;
  $: stitchFill = matchStitch ? color : stitchColor;

  const waves = Array.from({ length: 6 }, (_, i) => ({ x: 8 + i * 16, delay: (i % 3) * 0.7 + i * 0.1, dur: 2.4 + (i % 2) * 0.6 }));
  const flakes = Array.from({ length: 10 }, (_, i) => ({
    x: 5 + i * 9.5, delay: (i * 0.37) % 3, dur: 3 + (i % 4) * 0.5,
    size: 12 + (i % 3) * 5, drift: (i % 2 ? 1 : -1) * (10 + (i % 3) * 8)
  }));

  const clamp = (v, lo, hi) => Math.min(hi, Math.max(lo, v));
  function setTimer(m) { timer = m; remaining = m; }
  function togglePower() { if (usable) { power = !power; if (power && remaining === 0) remaining = timer; } }
  function emergencyOff() { emergency = true; power = false; menuOpen = false; }
  function togglePlug() {
    plugged = !plugged;
    if (plugged && emergency) { emergency = false; power = true; remaining = timer; }
  }
  function pickStitch(e) { stitchColor = e.target.value; matchStitch = false; }
  function matchStitchToBlanket() { matchStitch = true; colorMenuOpen = false; }
  function pickCustom(e) { customColor = e.target.value; customPicked = true; color = customColor; }

  onMount(() => {
    const temps = setInterval(() => {
      if (!power || !connected) return;
      upper += Math.sign(upperTarget - upper);
      lower += Math.sign(lowerTarget - lower);
    }, 1500);
    const clock = setInterval(() => {
      if (!power || !connected) return;
      if (remaining > 0) remaining -= 1;
      if (remaining === 0) power = false;
    }, MINUTE_MS);
    // Battery drains 1% every 6 timer-minutes while running (demo speed).
    const drain = setInterval(() => {
      if (plugged || !power || !connected) return;
      battery = Math.max(0, battery - 1);
      if (battery === 0) power = false;
    }, MINUTE_MS * 6);
    const charge = setInterval(() => { if (plugged) battery = Math.min(100, battery + 1); }, 1200);
    return () => { clearInterval(temps); clearInterval(clock); clearInterval(drain); clearInterval(charge); };
  });
</script>

<div class="layout">
  <!-- Blanket display -->
  <div class="stage">
    <div class="blanket" style="--fill:{color}; --stitch:{stitchFill}">
      <svg viewBox="0 0 380 460" role="img" aria-label="Blanket, {status}">
        <defs>
          <linearGradient id="sheen" x1="0" y1="0" x2="1" y2="1">
            <stop offset="0" stop-color="#fff" stop-opacity=".25" />
            <stop offset="1" stop-color="#000" stop-opacity=".12" />
          </linearGradient>
        </defs>
        <path class="zone {upperMode}" d="M126 30H334Q350 30 350 46V205H110V46Q110 30 126 30Z" />
        <path class="zone {lowerMode}" d="M110 205H350V364Q350 380 334 380H126Q110 380 110 364Z" />
        <rect x="110" y="30" width="240" height="350" rx="16" fill="url(#sheen)" pointer-events="none" />
        <rect x="119" y="39" width="222" height="332" rx="9" class="hem" />
        <line x1="119" y1="205" x2="341" y2="205" class="stitch" />
        <path d="M328 349 320 361h5l-2 8 9-13h-5z" class="bolt-mark" class:on={plugged} />
        {#if plugged}<path d="M326 400C326 430 352 422 378 430" class="cable" />{/if}
        <g class="plug" class:armed={emergency && !plugged} class:on={plugged} role="button" tabindex="0"
          on:click={togglePlug} on:keydown={(e) => (e.key === 'Enter' || e.key === ' ') && togglePlug()}>
          <title>{plugged ? 'Charger plugged in - tap to unplug' : 'Charge port - tap to plug in the charger'}</title>
          <rect x="310" y="372" width="32" height="16" rx="8" class="shell" />
          <rect x="314" y="376" width="24" height="8" rx="4" class="slot" />
          <rect x="318" y="379" width="16" height="2" rx="1" class="tongue" />
          {#if plugged}<rect x="312" y="384" width="28" height="16" rx="5" class="head" />{/if}
          <circle cx="303" cy="380" r="3" class="led" />
          <text x="326" y={plugged ? 444 : 406} class="tag small" text-anchor="middle">{plugged ? (battery >= 100 ? 'Fully charged' : 'Charging') : emergency ? 'Tap to plug in' : 'Charge port'}</text>
        </g>
        <g class="estop" role="button" tabindex="0" aria-label="Emergency power off"
          on:click={emergencyOff} on:keydown={(e) => (e.key === 'Enter' || e.key === ' ') && emergencyOff()}>
          <circle cx="262" cy="380" r="11" />
          <rect x="257.5" y="375.5" width="9" height="9" rx="1.5" fill="#fff" />
          <text x="256" y="406" class="tag small" text-anchor="middle">Emergency off</text>
        </g>
        <path d="M100 42V198M100 212V374" class="bracket" />
        <text x="90" y="122" class="tag side" text-anchor="end">Upper body</text>
        <text x="90" y="297" class="tag side" text-anchor="end">Lower body</text>
      </svg>

      {#each zones as z}
        <div class="fx {z.key}">
          {#if z.mode === 'warming'}
            {#each waves as w}
              <svg class="wave" viewBox="0 0 20 44" style="left:{w.x}%; --delay:{w.delay}s; --dur:{w.dur}s" aria-hidden="true">
                <path d="M10 44 C0 34 20 24 10 14 S0 4 10 0" />
              </svg>
            {/each}
          {:else if z.mode === 'cooling'}
            {#each flakes as f}
              <span class="flake" aria-hidden="true"
                style="left:{f.x}%; --delay:{f.delay}s; --dur:{f.dur}s; --drift:{f.drift}px; font-size:{f.size}px">❄</span>
            {/each}
          {/if}
        </div>
      {/each}
    </div>
  </div>

  <!-- Phone -->
  <div class="phone">
    <header>
      <div class="menu-wrap">
        <button class="kebab" aria-label="Options" on:click={() => (menuOpen = !menuOpen)}>⋮</button>
        {#if menuOpen}
          <div class="menu">
            <button on:click={() => { connected = !connected; menuOpen = false; }}>
              {connected ? 'Disconnect blanket' : 'Connect to blanket'}
            </button>
          </div>
        {/if}
      </div>
      <div class="title">
        <h1>Smart Blanket</h1>
        <p class="conn" class:off={!connected}>
          {connected ? 'Connected' : 'Not connected'}
          {#if connected}
            <span class="batt" class:low={battery <= 20 && !plugged} class:charging={plugged} title="Blanket power level">
              <svg viewBox="0 0 20 20" aria-hidden="true">
                <circle cx="10" cy="10" r="7.5" class="trk" />
                <circle cx="10" cy="10" r="7.5" class="prg" transform="rotate(-90 10 10)"
                  style="stroke-dasharray:47.12; stroke-dashoffset:{47.12 * (1 - battery / 100)}" />
                <path d="M10.8 5.5 7.8 10.6h2l-.6 3.9 3-5.1h-2z" class="bolt" />
              </svg>
              {plugged ? (battery >= 100 ? 'Charged' : 'Charging') : 'Blanket'} {battery}%
            </span>
          {/if}
        </p>
      </div>
      <div class="pwr">
        <button class="power" class:on={power && usable} on:click={togglePower} disabled={!usable} aria-label="Blanket power" title="Turn the blanket on or off">⏻</button>
        <span class="pcap" class:on={power && usable}>Blanket {power && usable ? 'on' : 'off'}</span>
      </div>
    </header>

    <section class="readout">
      {#if emergency}
        <p class="emerg">Emergency button hit - plug blanket in to restart</p>
      {:else}
        <p class="status">Status: <b>{status}</b></p>
      {/if}
      <div class="stats">
        <div><span>Upper body</span><b>{upper}°F</b></div>
        <div><span>Lower body</span><b>{lower}°F</b></div>
        <div><span>Weight</span><b>{weight} lbs</b></div>
      </div>
      <p class="off-in">{power && connected ? `Shutting off in ${remaining}m` : 'Not running'}</p>
      {#if plugged}<p class="charge-note">⚡ {battery >= 100 ? 'Fully charged' : 'Charging blanket'} · {battery}%</p>{/if}
    </section>

    <div class="duo">
      <section class="card">
        <div class="row stack"><h2>Upper body temp</h2><small>Target {upperTarget}°F</small></div>
        <div class="pair">
          <button class="btn cool" disabled={!usable || upperTarget <= MIN_TEMP} on:click={() => (upperTarget = clamp(upperTarget - 1, MIN_TEMP, MAX_TEMP))}>− Cool</button>
          <button class="btn warm" disabled={!usable || upperTarget >= MAX_TEMP} on:click={() => (upperTarget = clamp(upperTarget + 1, MIN_TEMP, MAX_TEMP))}>+ Warm</button>
        </div>
      </section>
      <section class="card">
        <div class="row stack"><h2>Lower body temp</h2><small>Target {lowerTarget}°F</small></div>
        <div class="pair">
          <button class="btn cool" disabled={!usable || lowerTarget <= MIN_TEMP} on:click={() => (lowerTarget = clamp(lowerTarget - 1, MIN_TEMP, MAX_TEMP))}>− Cool</button>
          <button class="btn warm" disabled={!usable || lowerTarget >= MAX_TEMP} on:click={() => (lowerTarget = clamp(lowerTarget + 1, MIN_TEMP, MAX_TEMP))}>+ Warm</button>
        </div>
      </section>
    </div>

    <section class="card">
      <div class="row"><h2>Temp timer</h2><small>{timer} min</small></div>
      <div class="chips">
        {#each timers as m}
          <button class="btn chip" class:sel={timer === m} aria-pressed={timer === m} disabled={!usable} on:click={() => setTimer(m)}>{m}m</button>
        {/each}
      </div>
    </section>

    <section class="card">
      <div class="row"><h2>Weight</h2><small>{weight} lbs</small></div>
      <div class="pair">
        <button class="btn" aria-label="Lighter" disabled={!usable || weight <= MIN_WEIGHT} on:click={() => (weight = clamp(weight - 1, MIN_WEIGHT, MAX_WEIGHT))}>−</button>
        <button class="btn" aria-label="Heavier" disabled={!usable || weight >= MAX_WEIGHT} on:click={() => (weight = clamp(weight + 1, MIN_WEIGHT, MAX_WEIGHT))}>+</button>
      </div>
    </section>

    <section class="card">
      <div class="row"><h2>Color</h2>
        <div class="menu-wrap">
          <button class="kebab small" aria-label="Stitch options" on:click={() => (colorMenuOpen = !colorMenuOpen)}>⋮</button>
          {#if colorMenuOpen}
            <div class="menu right">
              <label class="item">
                <span class="dot" style="background:{stitchFill}"></span>Choose stitch color
                <input type="color" value={stitchColor} on:input={pickStitch} on:change={() => (colorMenuOpen = false)} aria-label="Stitch color" />
              </label>
              <button on:click={matchStitchToBlanket}>{matchStitch ? '✓ ' : ''}Match stitch color to blanket</button>
            </div>
          {/if}
        </div>
      </div>
      <div class="swatches">
        {#each colors as c}
          <button class="sw" class:sel={color === c} style="background:{c}" aria-label="Color {c}" on:click={() => (color = c)}></button>
        {/each}
        <label class="sw custom" class:sel={customPicked && color === customColor}
          style="background:{customPicked ? customColor : rainbow}" title="Pick a custom color">
          <svg viewBox="0 0 24 24" aria-hidden="true"><path d="M12 3C12 3 5 11 5 15a7 7 0 0 0 14 0C19 11 12 3 12 3Z" /></svg>
          <input type="color" value={customColor} on:input={pickCustom} aria-label="Custom color" />
        </label>
      </div>
    </section>
  </div>
</div>

<style>
  .layout { display: flex; flex-wrap: wrap; gap: 32px; justify-content: center; align-items: center; padding: 24px; font-family: system-ui, sans-serif; }

  /* Blanket */
  .stage { background: radial-gradient(circle at 50% 40%, #1d2a3a, #0c121b); border-radius: 24px; padding: 56px 48px; }
  .blanket { position: relative; width: min(380px, 80vw); aspect-ratio: 380 / 460; }
  .blanket svg { width: 100%; height: 100%; overflow: visible; }
  .zone { fill: var(--fill); stroke: rgba(255,255,255,.35); stroke-width: 2; transition: filter .6s, opacity .6s; }
  .zone.off { opacity: .55; }
  .zone.warming { filter: drop-shadow(0 0 16px rgba(255,120,40,.85)); animation: pulseW 2.4s ease-in-out infinite; }
  .zone.cooling { filter: drop-shadow(0 0 16px rgba(130,205,255,.85)); animation: pulseC 2.4s ease-in-out infinite; }
  .stitch { stroke: var(--stitch); filter: drop-shadow(0 1px 0 rgba(0,0,0,.25)); stroke-width: 2.5; stroke-dasharray: 7 6; stroke-linecap: round; }
  .hem { fill: none; stroke: var(--stitch); stroke-opacity: .35; stroke-width: 1.5; }
  .bracket { fill: none; stroke: rgba(255,255,255,.35); stroke-width: 2; stroke-linecap: round; }
  .shell { fill: #cfd6e4; stroke: #55607a; stroke-width: 1.5; }
  .slot { fill: #161b24; }
  .tongue { fill: #8b95a9; }
  .head { fill: #2b3345; stroke: #9aa6bd; stroke-width: 1.5; }
  .cable { fill: none; stroke: #cfd6e4; stroke-width: 4; stroke-linecap: round; }
  .led { fill: #3a4457; }
  .plug.on .led { fill: #4ade80; filter: drop-shadow(0 0 5px #4ade80); animation: ledBlink 1.4s ease-in-out infinite; }
  @keyframes ledBlink { 50% { opacity: .35; } }
  .bolt-mark { fill: rgba(40,50,70,.7); }
  .bolt-mark.on { fill: #f59e0b; filter: drop-shadow(0 0 4px #f59e0b); }
  .tag.small { font-size: 11px; }
  .plug, .estop { cursor: pointer; }
  .estop circle { fill: #e5322d; stroke: #ffb4b0; stroke-width: 2; filter: drop-shadow(0 0 6px rgba(229,50,45,.7)); }
  .estop:hover circle { fill: #ff4540; }
  .plug:focus-visible, .estop:focus-visible { outline: 2px solid #7cc4ff; }
  .plug.armed .shell { animation: plugPulse 1.2s ease-in-out infinite; }
  @keyframes plugPulse { 50% { fill: #7cc4ff; filter: drop-shadow(0 0 8px #7cc4ff); } }
  .tag.side { fill: rgba(255,255,255,.85); font-size: 13px; }
  .port { fill: #ddd; stroke: #888; }
  .tag { fill: rgba(255,255,255,.7); font-size: 12px; }
  .tag.mid { fill: rgba(255,255,255,.55); }

  .fx { position: absolute; left: 29%; right: 8%; pointer-events: none; }
  .fx.upper { top: 6.5%; height: 38%; }
  .fx.lower { top: 44.5%; height: 38%; }

  .wave { position: absolute; bottom: 45%; width: 18px; height: 44px; fill: none; stroke: #ffb066; stroke-width: 2.5; stroke-linecap: round; opacity: 0; animation: rise var(--dur) ease-out infinite; animation-delay: var(--delay); }
  .flake { position: absolute; top: 25%; color: #d6efff; text-shadow: 0 0 8px #9fd8ff; opacity: 0; animation: fall var(--dur) ease-in infinite; animation-delay: var(--delay); }

  @keyframes rise { 0% { transform: translateY(0) scaleY(.8); opacity: 0; } 20% { opacity: .9; } 100% { transform: translateY(-95px) scaleY(1.2); opacity: 0; } }
  @keyframes fall { 0% { transform: translate(0,0) rotate(0); opacity: 0; } 15% { opacity: 1; } 100% { transform: translate(var(--drift), 120px) rotate(200deg); opacity: 0; } }
  @keyframes pulseW { 50% { filter: drop-shadow(0 0 26px rgba(255,90,20,1)); } }
  @keyframes pulseC { 50% { filter: drop-shadow(0 0 26px rgba(150,220,255,1)); } }

  /* Phone (dark mode) */
  .phone {
    --bg: #0e1118; --card: #171c26; --line: #273042; --text: #e9edf5; --muted: #8b95a9;
    width: 380px; max-width: 100%; box-sizing: border-box;
    border: 8px solid #1c2230; border-radius: 44px; padding: 22px 16px 22px;
    background: var(--bg); color: var(--text);
    box-shadow: 0 24px 60px rgba(0,0,0,.45);
    display: flex; flex-direction: column; gap: 12px;
  }
  header { display: grid; grid-template-columns: 56px 1fr 56px; align-items: start; gap: 8px; padding: 0 4px; }
  .title { flex: 1; text-align: center; }
  .phone h1 { color: #fff; font-size: 20px; line-height: 1.2; margin: 0; letter-spacing: .2px; }
  .conn { margin: 3px 0 0; font-size: 12px; color: #4ade80; display: flex; justify-content: center; align-items: center; gap: 10px; }
  .conn.off { color: #f87171; }
  .batt { display: inline-flex; align-items: center; gap: 5px; color: var(--muted); }
    .batt svg { width: 18px; height: 18px; }
  .batt .trk { fill: none; stroke: rgba(255,255,255,.18); stroke-width: 2.5; }
  .batt .prg { fill: none; stroke: currentColor; stroke-width: 2.5; stroke-linecap: round; transition: stroke-dashoffset .6s; }
  .batt .bolt { fill: currentColor; }
  .batt.charging { color: #4ade80; }
  .batt.charging svg { animation: chg 1.4s ease-in-out infinite; }
  @keyframes chg { 50% { filter: drop-shadow(0 0 5px #4ade80); } }
  .charge-note { margin: 6px 0 0; font-size: 13px; color: #4ade80; }
  .pwr { display: flex; flex-direction: column; align-items: center; gap: 4px; }
  .pcap { font-size: 10px; color: var(--muted); white-space: nowrap; }
  .pcap.on { color: #4ade80; }
  .batt.low { color: #f87171; }
  .emerg { margin: 0 0 10px; font-size: 14px; font-weight: 600; color: #ff8a80; }
  .menu-wrap { position: relative; }
    .kebab.small { font-size: 20px; padding: 0 6px; }
  .menu.right { left: auto; right: 0; }
  .menu button, .menu .item { display: flex; align-items: center; gap: 8px; width: 100%; text-align: left; box-sizing: border-box; }
  .menu .item { position: relative; padding: 10px 14px; white-space: nowrap; cursor: pointer; }
  .menu .item input { position: absolute; inset: 0; width: 100%; height: 100%; opacity: 0; cursor: pointer; }
  .dot { width: 14px; height: 14px; border-radius: 50%; border: 1px solid rgba(255,255,255,.4); }
  .kebab { background: none; border: 0; color: var(--text); font-size: 24px; line-height: 1; cursor: pointer; padding: 2px 8px; }
  .menu { position: absolute; top: 32px; left: 0; z-index: 2; background: #1f2633; border: 1px solid var(--line); border-radius: 12px; box-shadow: 0 8px 20px rgba(0,0,0,.5); }
  .menu button { background: none; border: 0; color: var(--text); padding: 10px 14px; white-space: nowrap; cursor: pointer; font: inherit; }
  .power { width: 40px; height: 40px; border-radius: 50%; border: 2px solid #3a4457; background: var(--card); font-size: 18px; cursor: pointer; color: var(--muted); }
  .power.on { border-color: #4ade80; color: #4ade80; box-shadow: 0 0 14px rgba(74,222,128,.35); }

  /* Status square */
  .readout { border: 1px solid var(--line); border-radius: 18px; padding: 12px 14px; background: var(--card); transition: background .5s, border-color .5s, box-shadow .5s; }
  .status { margin: 0 0 10px; font-size: 14px; color: var(--muted); }
  .status b { color: var(--text); }
  .stats { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; }
  .stats div { display: flex; flex-direction: column; gap: 2px; font-size: 12px; color: var(--muted); }
  .stats b { font-size: 20px; color: var(--text); }
  .off-in { margin: 10px 0 0; font-size: 13px; color: var(--muted); }

  .card { background: var(--card); border: 1px solid var(--line); border-radius: 18px; padding: 12px 14px 14px; }
  .row { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 10px; }
  h2 { font-size: 13px; margin: 0; font-weight: 600; color: var(--text); }
  small { color: var(--muted); font-size: 12px; }

.duo { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
  .row.stack { flex-direction: column; gap: 2px; }
  .duo .pair { gap: 6px; }
  .duo .btn { padding: 10px 2px; font-size: 13px; }
  .pair { display: flex; gap: 10px; }
  .chips { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; }
  .btn { flex: 1; width: 100%; border: 1.5px solid #323c50; background: #1f2633; color: var(--text); border-radius: 14px; padding: 11px 8px; font: inherit; font-size: 14px; font-weight: 600; cursor: pointer; transition: transform .1s, background .2s, border-color .2s; }
  .btn:hover:not(:disabled) { background: #283142; }
  .btn:active:not(:disabled) { transform: scale(.97); }
  .btn.cool { color: #8fd0ff; border-color: rgba(120,190,255,.45); background: rgba(80,160,255,.10); }
  .btn.warm { color: #ffb27a; border-color: rgba(255,150,80,.5); background: rgba(255,120,40,.10); }
  .btn.chip.sel { background: #e9edf5; color: #0e1118; border-color: #e9edf5; box-shadow: 0 0 0 3px rgba(233,237,245,.25); }
  .phone button:disabled { opacity: .35; cursor: not-allowed; }
  .phone button:focus-visible, .sw:focus-within { outline: 3px solid #7cc4ff; outline-offset: 2px; }

  .swatches { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; }
  .sw { position: relative; display: block; aspect-ratio: 1.3; border-radius: 12px; border: 0; cursor: pointer; padding: 0; }
  .sw.sel { outline: 3px solid #e9edf5; outline-offset: 2px; }
  .custom svg { position: absolute; inset: 0; margin: auto; width: 26px; height: 26px; fill: rgba(255,255,255,.95); stroke: rgba(0,0,0,.35); stroke-width: 1; pointer-events: none; filter: drop-shadow(0 1px 2px rgba(0,0,0,.4)); }
  .custom input { position: absolute; inset: 0; width: 100%; height: 100%; opacity: 0; cursor: pointer; }

  @media (prefers-reduced-motion: reduce) {
    .wave, .flake, .zone, .plug.armed .shell, .plug.on .led, .batt.charging svg { animation: none !important; }
    .wave, .flake { opacity: .8; }
  }
</style>