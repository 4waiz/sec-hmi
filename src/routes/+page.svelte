<script lang="ts">
  // Demo values – wire to your data later
  let asrsId = 'ASRS-2025-001';
  let runtime = '00:08:20';
  let timestamp = '11/03/2025, 06:38:08';
  let language: 'en' | 'ar' = 'en';
  let brightness = 60; // percent
  let activeLogTab: 'all' | 'error' | 'operation' = 'all';

  // Backup timestamps (simulate persistence)
  let storeTimestamp = '— —';
  let restoreTimestamp = '— —';

  // Hold-to-activate logic (2 seconds)
  const HOLD_MS = 2000;
  let storeTimer: any = null;
  let restoreTimer: any = null;
  let storeProgress = 0;    // 0..1
  let restoreProgress = 0;  // 0..1

  function nowString() {
    return new Date().toLocaleString();
  }

  function startHold(kind: 'store'|'restore') {
    cancelHold(kind); // reset
    const t0 = Date.now();
    const tick = () => {
      const p = Math.min(1, (Date.now() - t0) / HOLD_MS);
      if (kind === 'store') storeProgress = p; else restoreProgress = p;
      if (p >= 1) {
        cancelHold(kind);
        if (kind === 'store') {
          storeTimestamp = nowString();
        } else {
          restoreTimestamp = nowString();
        }
      }
    };
    const id = setInterval(tick, 16);
    if (kind === 'store') storeTimer = id; else restoreTimer = id;
  }

  function cancelHold(kind: 'store'|'restore') {
    if (kind === 'store') {
      if (storeTimer) clearInterval(storeTimer);
      storeTimer = null;
      storeProgress = 0;
    } else {
      if (restoreTimer) clearInterval(restoreTimer);
      restoreTimer = null;
      restoreProgress = 0;
    }
  }

  // Build a colored track like your Figma rectangles
  $: sliderStyle = `
    background: linear-gradient(90deg,
      var(--brand) 0%,
      var(--brand) ${brightness}%,
      var(--track-rest) ${brightness}%,
      var(--track-rest) 100%);
  `;
</script>

<!-- Responsive wrapper: centers up to 1024px, breathes on small screens -->
<div class="page">
  <!-- Header -->
  <div class="topbar">
    <button class="iconbtn pressable" aria-label="Back">
      <img class="icon" src="/assets/arrow-1.svg" alt="Back" width="24" height="24" />
    </button>

    <div class="titleblock">
      <h1 class="h1">System Setting</h1>
      <div class="meta muted">
        <span class="strong">ASRS ID:</span>&nbsp;{asrsId}&nbsp;&nbsp;·&nbsp;&nbsp;
        <span class="strong">Runtime:</span>&nbsp;{runtime}&nbsp;&nbsp;·&nbsp;&nbsp;
        <span class="strong">Timestamp:</span>&nbsp;{timestamp}
      </div>
    </div>
  </div>

  <hr class="rule" />

  <!-- Main grid -->
  <div class="grid">
    <!-- Language -->
    <section class="card card--language">
      <div class="card-head">
        <img class="icon" src="/assets/frame7.svg" alt="Language" width="24" height="24" />
        <h2 class="h2">Language</h2>
      </div>

      <div class="row gap lang-toggle" class:lang-en={language==='en'} class:lang-ar={language==='ar'}>
        <button type="button" class="pill pressable {language==='en' ? 'pill--active' : ''}" aria-pressed={language==='en'} on:click={() => language='en'}>
          <span class="pill-dot">
            <img src="/assets/check.svg" alt="" width="14" height="14" />
          </span>
          English
        </button>

        <button
          type="button"
          class="pill pressable {language==='ar' ? 'pill--active' : ''}"
          aria-pressed={language==='ar'}
          on:click={() => language='ar'}
        >
          <span class="pill-dot">
            <img src="/assets/check.svg" alt="" width="14" height="14" />
          </span>
          عربي / Arabic
        </button>
      </div>
    </section>

    <!-- Brightness -->
    <section class="card card--brightness">
      <div class="card-head">
        <img class="icon" src="/assets/frame3.svg" alt="Brightness" width="24" height="24" />
        <h2 class="h2">Display Brightness</h2>
      </div>

      <div class="slider-wrap">
        <!-- Track color via inline style, knob styled in CSS to match Figma -->
        <input
          class="slider"
          type="range"
          min="0"
          max="100"
          bind:value={brightness}
          style={sliderStyle}
          aria-label="Brightness"
        />
      </div>
    </section>

    <!-- Network -->
    <section class="card card--network">
      <div class="card-head">
        <img class="icon" src="/assets/frame.svg" alt="Network" width="24" height="24" />
        <h2 class="h2">Network Configuration</h2>
        <img class="icon ml8" src="/assets/frame6.svg" alt="Locked" width="24" height="24" />
      </div>

      <div class="label">Machine IPs:</div>

      <div class="iprow">
        <input class="ip" value="*** ***** *" readonly />
        <button class="iconbtn pressable" title="Reveal">
          <img class="icon" src="/assets/frame1.svg" alt="Reveal" width="24" height="24" />
        </button>
      </div>

      <div class="iprow">
        <input class="ip" value="*** ***** *" readonly />
        <button class="iconbtn pressable" title="Reveal">
          <img class="icon" src="/assets/frame1.svg" alt="Reveal" width="24" height="24" />
        </button>
      </div>
    </section>

    <!-- ===== BACKUP (new, right column) ===== -->
    <section class="card card--backup">
      <div class="card-head">
        <!-- Save / backup icon -->
        <svg class="icon" viewBox="0 0 24 24" aria-hidden="true">
          <!-- floppy disk -->
          <path d="M5 3h11l5 5v13a0 0 0 0 1 0 0H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2Z" fill="none" stroke="black" stroke-width="2"/>
          <rect x="7" y="3" width="8" height="5" fill="none" stroke="black" stroke-width="2"/>
          <rect x="7" y="13" width="10" height="6" rx="1" fill="none" stroke="black" stroke-width="2"/>
        </svg>
        <h2 class="h2">Backup</h2>
      </div>

      <p class="muted small">Press and hold for at least 2 seconds to save/restore parameters.</p>

      <!-- Store parameters -->
      <div class="backup-row">
        <button
          class="holdbtn pressable"
          style={`--p:${storeProgress}`}
          on:pointerdown={() => startHold('store')}
          on:pointerup={() => cancelHold('store')}
          on:pointerleave={() => cancelHold('store')}
          on:pointercancel={() => cancelHold('store')}
        >
          <span class="ring" aria-hidden="true"></span>
          <svg width="20" height="20" viewBox="0 0 24 24" aria-hidden="true">
            <!-- save -->
            <path d="M5 3h11l5 5v13H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2Z" fill="none" stroke="black" stroke-width="2"/>
            <rect x="7" y="3" width="8" height="5" fill="none" stroke="black" stroke-width="2"/>
          </svg>
          <span>Store parameters</span>
        </button>

        <input class="stamp" value={storeTimestamp} readonly />
      </div>

      <!-- Restore parameters -->
      <div class="backup-row">
        <button
          class="holdbtn pressable"
          style={`--p:${restoreProgress}`}
          on:pointerdown={() => startHold('restore')}
          on:pointerup={() => cancelHold('restore')}
          on:pointerleave={() => cancelHold('restore')}
          on:pointercancel={() => cancelHold('restore')}
        >
          <span class="ring" aria-hidden="true"></span>
          <svg width="20" height="20" viewBox="0 0 24 24" aria-hidden="true">
            <!-- restore (arrow into tray) -->
            <path d="M12 3v10M7 8l5 5 5-5" fill="none" stroke="black" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M5 21h14" fill="none" stroke="black" stroke-width="2" stroke-linecap="round"/>
          </svg>
          <span>Restore parameters</span>
        </button>

        <input class="stamp" value={restoreTimestamp} readonly />
      </div>
    </section>

    <!-- Logs -->
    <section class="card card--logs">
      <div class="card-head">
        <h2 class="h2">Machine Logs</h2>
        <img class="icon ml8" src="/assets/frame6.svg" alt="Locked" width="24" height="24" />
      </div>

      <div class="row gap">
        <button
          type="button"
          class="pill pressable {activeLogTab === 'all' ? 'pill--active' : ''}"
          aria-pressed={activeLogTab === 'all'}
          on:click={() => (activeLogTab = 'all')}
        >
          All logs
        </button>
        <button
          type="button"
          class="pill pressable {activeLogTab === 'error' ? 'pill--active' : ''}"
          aria-pressed={activeLogTab === 'error'}
          on:click={() => (activeLogTab = 'error')}
        >
          Error logs
        </button>
        <button
          type="button"
          class="pill pressable {activeLogTab === 'operation' ? 'pill--active' : ''}"
          aria-pressed={activeLogTab === 'operation'}
          on:click={() => (activeLogTab = 'operation')}
        >
          Operation logs
        </button>
      </div>

      <div class="row gap mt12">
        <button class="btn pressable">
          <img class="icon" src="/assets/frame2.svg" alt="Export" width="24" height="24" />
          <span>Export</span>
        </button>

        <button class="btn pressable">
          <span>View</span>
          <img class="icon" src="/assets/arrow-6.svg" alt="View" width="24" height="24" />
        </button>
      </div>
    </section>
  </div>
</div>

<style>
  /* Responsive canvas */
  .page{
    width: 100%;
    max-width: 1080px;
    margin: 0 auto;
    padding: 16px 28px 18px;
    background:#fff;
    box-sizing:border-box;
    height: 100vh;
    display:flex;
    flex-direction:column;
  }
  @media (max-width: 900px){
    .page{
      height:auto;
      min-height:100vh;
      padding: clamp(16px, 4vw, 24px);
    }
  }

  /* Header */
  .topbar{ display:flex; align-items:flex-start; gap:16px; }
  .iconbtn{
    width:46px; height:46px; display:grid; place-items:center;
    border:1px solid var(--muted); border-radius:8px; background:#fff;
  }
  .titleblock{ flex:1; }
  .meta{ margin-top:10px; font-size:15px; }
  .h1{ font-size: clamp(28px, 2.8vw, 34px); line-height:1.1; margin:0; }
  .h2{ font-size: clamp(18px, 2vw, 24px); margin:0; }

  .rule{
    margin: 14px 0 10px;
    height:1px; border:none; background:var(--border);
  }

  /* Grid: 1 column on small, 2 columns on >= 900px */
  .grid{
    display:grid;
    grid-template-columns: 1fr;
    gap: 20px;
    flex:1 1 auto;
    min-height:0;
  }
  @media (min-width: 900px){
    .grid{
      grid-template-columns: repeat(2, minmax(0, 1fr));
      column-gap: 40px;
      row-gap: 22px;
      grid-template-rows: auto auto 1fr;
      grid-template-areas:
        "language brightness"
        "network backup"
        "logs backup";
      align-content:stretch;
    }
    .grid > .card{ height:100%; }
    .card--language{ grid-area: language; }
    .card--brightness{ grid-area: brightness; }
    .card--network{ grid-area: network; }
    .card--backup{ grid-area: backup; }
    .card--logs{ grid-area: logs; }
  }

  /* Cards */
  .card{
    display:flex;
    flex-direction:column;
  }
  .card-head{
    display:flex; align-items:center; gap:10px; margin-bottom:14px;
  }
  .ml8{ margin-left:8px; }
  .small{ font-size:12.5px; margin: 0 0 12px; }

  /* Pills */
  .row{ display:flex; align-items:center; flex-wrap: wrap; }
  .gap{ gap:12px; }
  .mt12{ margin-top:12px; }

  .pill{
    height:42px; padding:0 20px;
    border-radius:24px;
    border:1px solid var(--muted);
    color:var(--muted); background:#fff;
    font-size:15px; font-weight:500;
    display:inline-flex; align-items:center; gap:9px;
  }
  .pill--active{
    border-color:var(--brand);
    color:var(--brand);
    font-weight:500;
  }
  .lang-toggle .pill-dot{ display:none; }
  .lang-toggle .pill.pill--active .pill-dot{ display:grid; }
  .lang-toggle.lang-en .pill:first-child,
  .lang-toggle.lang-ar .pill:last-child{
    border-color:var(--brand);
    color:var(--brand);
    font-weight:500;
  }
  .lang-toggle.lang-en .pill:first-child .pill-dot,
  .lang-toggle.lang-ar .pill:last-child .pill-dot{ display:grid; }
  .pill-dot{
    width:24px; height:24px; border-radius:999px;
    display:grid; place-items:center;
    background:var(--brand);
    border:1px solid #fff;
  }

  /* Slider area */
  .slider-wrap{ width: 100%; max-width: 330px; height: 34px; display:flex; align-items:center; }

  /* Track via background gradient on the input element itself */
  .slider{
    -webkit-appearance:none; appearance:none;
    width: 100%;
    height: 12px;
    border-radius: 140px;
    outline: none;
  }

/* WebKit (Chrome / Edge / Safari) */
.slider::-webkit-slider-thumb{
  -webkit-appearance:none; appearance:none;

  /* same physical size as your SVG artboard */
  width:34px; 
  height:29px;

  /* center the 28px thumb on a 10px track: (28-10)/2 = 9 */
  margin-top:-0px;

  /* the SVG IS the thumb */
  background: url('/assets/slider.svg') center/100% 100% no-repeat transparent !important;
  border:0; 
  border-radius:0; 
  box-shadow:none;                 /* optional: keep it clean since SVG has stroke */
  /* optional subtle shadow if you still want depth:
     filter: drop-shadow(0 1px 0 rgba(0,0,0,.10)) drop-shadow(0 2px 6px rgba(0,0,0,.12));
  */

  transition: transform .12s ease;
}
.slider:active::-webkit-slider-thumb{ transform: scale(0.96); }

/* Firefox */
.slider::-moz-range-thumb{
  width:34px; 
  height:28px;
  background: url('/assets/slider.svg') center/100% 100% no-repeat transparent !important;
  border:0; 
  border-radius:0; 
  box-shadow:none;
  /* optional: filter: drop-shadow(...) like above */
  transition: transform .12s ease;
}
.slider:active::-moz-range-thumb{ transform: scale(0.96); }
  .label{ font-size:17px; font-weight:600; line-height:24px; margin:6px 0 10px; }
  .iprow{ display:flex; align-items:center; gap:10px; margin-bottom:12px; }
  .ip{
    flex:1; height:44px; border-radius:8px;
    border:1px solid var(--border-strong); background:#fff;
    color:var(--muted); padding:0 14px;
    font-size:15px;
  }

  /* Buttons */
  .btn{
    height:48px; padding:0 24px;
    border-radius:8px; border:1px solid var(--border-strong);
    background:#fff; display:inline-flex; align-items:center; gap:10px;
    font-size:15.5px; font-weight:600;
  }

  /* ===== BACKUP styles ===== */
  .backup-row{
    display:flex; gap:12px; align-items:center; margin-top:10px; flex-wrap:wrap;
  }
  .holdbtn{
    position:relative;
    height:54px; padding:0 18px;
    border:1px solid var(--border-strong); border-radius:8px;
    background:#fff;
    display:inline-flex; align-items:center; gap:10px;
    font-size:16px;
  }
  /* small circular progress indicator */
  .holdbtn .ring{
    width:20px; height:20px; border-radius:999px; flex:0 0 20px;
    background: conic-gradient(var(--brand) calc(var(--p,0)*1*1turn), #eee 0);
    transition: background .05s linear;
  }
  .stamp{
    flex:1 1 280px;
    height:44px;
    border-radius:10px;
    border:1px solid var(--border-strong);
    padding:0 14px;
    color:var(--muted);
    font-size:15px;
    background:#fff;
  }

  @media (min-width: 900px){
    .card--backup{
      justify-content:space-between;
      gap:18px;
    }
    .card--logs{
      justify-content:space-between;
    }
  }
</style>
