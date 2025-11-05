<script lang="ts">
  // Demo values – wire to your data later
  let asrsId = 'ASRS-2025-001';
  let runtime = '00:08:20';
  let timestamp = '11/03/2025, 06:38:08';
  let language: 'en' | 'ar' = 'en';
  let brightness = 60; // percent

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
    <section class="card">
      <div class="card-head">
        <img class="icon" src="/assets/frame7.svg" alt="Language" width="24" height="24" />
        <h2 class="h2">Language</h2>
      </div>

      <div class="row gap">
        <button class="pill pressable {language==='en' ? 'pill--active' : ''}" on:click={() => language='en'}>
          <span class="pill-dot">
            <img src="/assets/check.svg" alt="" width="14" height="14" />
          </span>
          English
        </button>

        <button class="pill pressable" on:click={() => language='ar'}>عربي / Arabic</button>
      </div>
    </section>

    <!-- Brightness -->
    <section class="card">
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
    <section class="card">
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

    <!-- Logs -->
    <section class="card">
      <div class="card-head">
        <h2 class="h2">Machine Logs</h2>
        <img class="icon ml8" src="/assets/frame6.svg" alt="Locked" width="24" height="24" />
      </div>

      <div class="row gap">
        <button class="pill pressable pill--active">All logs</button>
        <button class="pill pressable">Error logs</button>
        <button class="pill pressable">Operation logs</button>
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

    <!-- ===== BACKUP (new, right column) ===== -->
    <section class="card">
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
  </div>
</div>

<style>
  /* Responsive canvas: up to 1024px, padding scales on mobile */
  .page{
    width: min(1024px, 100%);
    margin: clamp(12px, 3vw, 24px) auto;
    padding: clamp(12px, 3vw, 24px);
    background:#fff;
  }

  /* Header */
  .topbar{ display:flex; align-items:flex-start; gap:16px; }
  .iconbtn{
    width:42px; height:42px; display:grid; place-items:center;
    border:1px solid var(--muted); border-radius:8px; background:#fff;
  }
  .titleblock{ flex:1; }
  .meta{ margin-top:8px; font-size:14px; }

  .rule{
    margin: clamp(16px, 3vw, 24px) 0 clamp(8px, 2vw, 16px);
    height:1px; border:none; background:var(--border);
  }

  /* Grid: 1 column on small, 2 columns on >= 900px */
  .grid{
    display:grid;
    grid-template-columns: 1fr;
    gap: clamp(20px, 3vw, 32px);
  }
  @media (min-width: 900px){
    .grid{ grid-template-columns: 1fr 1fr; column-gap: 48px; row-gap: 32px; }
  }

  /* Cards */
  .card{}
  .card-head{
    display:flex; align-items:center; gap:8px; margin-bottom:16px;
  }
  .ml8{ margin-left:8px; }
  .small{ font-size:12.5px; margin: 0 0 12px; }

  /* Pills */
  .row{ display:flex; align-items:center; flex-wrap: wrap; }
  .gap{ gap:12px; }
  .mt12{ margin-top:12px; }

  .pill{
    height:40px; padding:0 19px;
    border-radius:24px;
    border:1px solid var(--muted);
    color:var(--muted); background:#fff;
    font-size:14px; font-weight:400;
    display:inline-flex; align-items:center; gap:9px;
  }
  .pill--active{
    border-color:var(--brand);
    color:var(--brand);
    font-weight:500;
  }
  .pill-dot{
    width:24px; height:24px; border-radius:999px;
    display:grid; place-items:center;
    background:var(--brand);
    border:1px solid #fff;
  }

  /* Slider area */
  .slider-wrap{ width: 100%; max-width: 309px; height: 34px; display:flex; align-items:center; }

  /* Track via background gradient on the input element itself */
  .slider{
    -webkit-appearance:none; appearance:none;
    width: 100%;
    height: 10px;
    border-radius: 140px;
    outline: none;
  }

  /* === Custom thumb (Figma-accurate) ===
     - size: 34×28 (oval), peach fill, 1px dark border
     - inner “C” arcs using an inline SVG as background image
  */
  .slider::-webkit-slider-thumb{
    -webkit-appearance:none; appearance:none;
    width: 34px; height: 28px;
    border-radius: 140px;
    border: 1px solid var(--muted);
    background:
      url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cg fill='none' stroke='rgba(0,0,0,0.68)' stroke-width='2' stroke-linecap='round'%3E%3Cpath d='M9 8a5 5 0 0 0 0 8'/%3E%3Cpath d='M15 8a5 5 0 0 1 0 8'/%3E%3Cpath d='M11 9a3 3 0 0 0 0 6'/%3E%3Cpath d='M13 9a3 3 0 0 1 0 6'/%3E%3C/g%3E%3C/svg%3E")
      center/16px 16px no-repeat,
      var(--thumb);
    box-shadow: 0 1px 0 rgba(0,0,0,.06), 0 2px 6px rgba(0,0,0,.12);
    transition: transform .12s ease, box-shadow .12s ease, border-color .12s ease;
  }
  .slider:active::-webkit-slider-thumb{
    transform: scale(0.96);
    box-shadow: 0 1px 0 rgba(0,0,0,.06), 0 3px 10px rgba(0,0,0,.18);
  }

  /* Firefox */
  .slider::-moz-range-thumb{
    width: 34px; height: 28px;
    border-radius: 140px;
    border: 1px solid var(--muted);
    background:
      url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'%3E%3Cg fill='none' stroke='rgba(0,0,0,0.68)' stroke-width='2' stroke-linecap='round'%3E%3Cpath d='M9 8a5 5 0 0 0 0 8'/%3E%3Cpath d='M15 8a5 5 0 0 1 0 8'/%3E%3Cpath d='M11 9a3 3 0 0 0 0 6'/%3E%3Cpath d='M13 9a3 3 0 0 1 0 6'/%3E%3C/g%3E%3C/svg%3E")
      center/16px 16px no-repeat,
      var(--thumb);
    box-shadow: 0 1px 0 rgba(0,0,0,.06), 0 2px 6px rgba(0,0,0,.12);
    transition: transform .12s ease, box-shadow .12s ease, border-color .12s ease;
  }
  .slider:active::-moz-range-thumb{
    transform: scale(0.96);
    box-shadow: 0 1px 0 rgba(0,0,0,.06), 0 3px 10px rgba(0,0,0,.18);
  }

  /* Labels & inputs */
  .label{ font-size:16px; font-weight:500; line-height:24px; margin:8px 0 10px; }
  .iprow{ display:flex; align-items:center; gap:8px; margin-bottom:12px; }
  .ip{
    flex:1; height:40px; border-radius:6px;
    border:1px solid var(--border-strong); background:#fff;
    color:var(--muted); padding:0 12px;
  }

  /* Buttons */
  .btn{
    height:48px; padding:0 24px;
    border-radius:8px; border:1px solid var(--border-strong);
    background:#fff; display:inline-flex; align-items:center; gap:10px;
    font-size:15.65px; font-weight:500;
  }

  /* ===== BACKUP styles ===== */
  .backup-row{
    display:flex; gap:12px; align-items:center; margin-top:10px; flex-wrap:wrap;
  }
  .holdbtn{
    position:relative;
    height:56px; padding:0 16px;
    border:1px solid var(--border-strong); border-radius:8px;
    background:#fff;
    display:inline-flex; align-items:center; gap:10px;
  }
  /* small circular progress indicator */
  .holdbtn .ring{
    width:18px; height:18px; border-radius:999px; flex:0 0 18px;
    background: conic-gradient(var(--brand) calc(var(--p,0)*1*1turn), #eee 0);
    transition: background .05s linear;
  }
  .stamp{
    flex:1 1 280px;
    height:44px;
    border-radius:8px;
    border:1px solid var(--border-strong);
    padding:0 12px;
    color:var(--muted);
    background:#fff;
  }
</style>
