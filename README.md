<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Hourly Productivity & Break Dashboard</title>
  
  <!-- CDNs for Papaparse, jsPDF, and AutoTable -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.3.2/papaparse.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>

  <style>
    /* Dark Theme Variables */
    [data-theme="dark"] {
      --bg: #0f172a;
      --card-bg: #1e293b;
      --table-header: #0f172a;
      --header-text: #f8fafc;
      --border: #334155;
      --text: #f8fafc;
      --text-muted: #cbd5e1;
      --accent: #38bdf8;
      --accent-hover: #0284c7;
      --green-bg: #064e3b;
      --green-text: #6ee7b7;
      --red-bg: #7f1d1d;
      --red-text: #fca5a5;
      --yellow-bg: #78350f;
      --yellow-text: #fde047;
      --blue-bg: #1e3a8a;
      --blue-text: #93c5fd;
      --hover-bg: rgba(255, 255, 255, 0.04);
    }

    /* Light Theme Variables */
    [data-theme="light"] {
      --bg: #f1f5f9;
      --card-bg: #ffffff;
      --table-header: #e2e8f0;
      --header-text: #0f172a;
      --border: #cbd5e1;
      --text: #0f172a;
      --text-muted: #475569;
      --accent: #0284c7;
      --accent-hover: #0369a1;
      --green-bg: #dcfce7;
      --green-text: #15803d;
      --red-bg: #fee2e2;
      --red-text: #b91c1c;
      --yellow-bg: #fef3c7;
      --yellow-text: #b45309;
      --blue-bg: #dbeafe;
      --blue-text: #1d4ed8;
      --hover-bg: #f8fafc;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background-color: var(--bg); color: var(--text); padding: 12px; font-size: 13px; transition: background-color 0.2s, color 0.2s; }

    header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; flex-wrap: wrap; gap: 8px; }
    h1 { font-size: 1.3rem; color: var(--text); font-weight: 700; }
    
    .clock-card {
      font-size: 0.95rem;
      font-weight: 800;
      color: #38bdf8;
      background: var(--card-bg);
      padding: 6px 12px;
      border-radius: 6px;
      border: 1px solid var(--border);
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .window-info { font-size: 0.85rem; font-weight: 600; color: var(--accent); background: rgba(56, 189, 248, 0.1); padding: 5px 10px; border-radius: 6px; border: 1px solid var(--accent); }

    .kpis { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 10px; margin-bottom: 12px; }
    .kpi { background: var(--card-bg); border: 1px solid var(--border); border-radius: 6px; padding: 10px; }
    .kpi .label { font-size: 0.7rem; font-weight: 700; color: var(--text-muted); text-transform: uppercase; margin-bottom: 2px; }
    .kpi .value { font-size: 1.3rem; font-weight: 800; }
    .kpi .value.small { font-size: 1.05rem; }

    .controls { display: flex; flex-wrap: wrap; gap: 8px; align-items: center; justify-content: space-between; margin-bottom: 12px; background: var(--card-bg); padding: 8px 12px; border-radius: 6px; border: 1px solid var(--border); }
    .control-group { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
    
    input[type="text"], input[type="number"], select, input[type="color"] { background: var(--bg); border: 1px solid var(--border); color: var(--text); padding: 6px 10px; border-radius: 4px; outline: none; font-size: 0.85rem; font-weight: 500; }
    input[type="color"] { padding: 2px 4px; cursor: pointer; height: 32px; width: 40px; }
    input[type="text"]:focus, input[type="number"]:focus, select:focus { border-color: var(--accent); }
    
    .btn { background: var(--accent); color: #fff; font-weight: 700; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; transition: 0.2s; font-size: 0.85rem; }
    [data-theme="dark"] .btn { color: #000; }
    .btn:hover { background: var(--accent-hover); color: #fff; }
    .btn-secondary { background: var(--border); color: var(--text); font-weight: 600; }
    .btn-secondary:hover { opacity: 0.85; }

    .target-box {
      display: flex;
      gap: 12px;
      align-items: center;
      background: var(--card-bg);
      padding: 8px 14px;
      border-radius: 6px;
      border: 1px solid var(--border);
      margin-bottom: 12px;
      flex-wrap: wrap;
    }
    .target-input-item { display: flex; align-items: center; gap: 6px; font-weight: 600; font-size: 0.85rem; }
    .target-input-item input { width: 75px; text-align: center; font-weight: 700; }

    .tabs { display: flex; gap: 4px; border-bottom: 1px solid var(--border); margin-bottom: 12px; }
    .tab-btn { background: none; border: none; color: var(--text-muted); padding: 6px 14px; cursor: pointer; font-weight: 700; border-bottom: 2px solid transparent; font-size: 0.85rem; }
    .tab-btn.active { color: var(--accent); border-bottom-color: var(--accent); }
    .tab-btn.defaulter-tab.active { color: var(--red-text); border-bottom-color: var(--red-text); }

    .dropzone { border: 2px dashed var(--border); background: var(--card-bg); padding: 14px; text-align: center; border-radius: 6px; cursor: pointer; margin-bottom: 12px; font-weight: 500; }
    .dropzone.dragover { border-color: var(--accent); background: rgba(56, 189, 248, 0.05); }

    .table-wrapper { overflow-x: auto; background: var(--card-bg); border-radius: 6px; border: 1px solid var(--border); }
    table { width: 100%; border-collapse: collapse; text-align: left; white-space: nowrap; font-size: 0.88rem; }
    th, td { padding: 8px 12px; border-bottom: 1px solid var(--border); border-right: 1px solid var(--border); vertical-align: middle; }
    th:last-child, td:last-child { border-right: none; }
    
    th {
      background: var(--custom-header-bg, var(--table-header));
      color: var(--custom-header-text, var(--header-text));
      font-weight: 800;
      cursor: grab;
      user-select: none;
      text-transform: uppercase;
      font-size: 0.78rem;
      letter-spacing: 0.4px;
      transition: background 0.2s;
    }
    th:active { cursor: grabbing; }
    th.dragging { opacity: 0.4; background: var(--accent); }
    th.drag-over { border-left: 3px solid var(--accent) !important; }

    tr:hover { background: var(--hover-bg); }

    /* Badge Style for Colored Numbers and Highlights */
    .badge { 
      display: inline-block; 
      padding: 5px 12px; 
      border-radius: 6px; 
      font-size: 0.95rem; 
      font-weight: 800; 
      text-align: center; 
      letter-spacing: 0.3px;
    }
    .badge.green { background: var(--green-bg); color: var(--green-text); }
    .badge.red { background: var(--red-bg); color: var(--red-text); }
    .badge.yellow { background: var(--yellow-bg); color: var(--yellow-text); }
    .badge.blue { background: var(--blue-bg); color: var(--blue-text); }

    .badge.remark-tag {
      font-size: 0.85rem;
      font-weight: 700;
      padding: 4px 10px;
    }

    .name-cell { display: flex; flex-direction: column; }
    .name-cell .name { font-weight: 800; color: var(--text); font-size: 0.92rem; }
    .name-cell .mail { font-size: 0.75rem; color: var(--text-muted); font-weight: 500; }

    .bold-data { font-weight: 800; color: var(--text); font-size: 0.92rem; }
    .muted-data { font-weight: 600; color: var(--text-muted); }

    .dropdown { position: relative; }
    .dropdown-menu { display: none; position: absolute; right: 0; top: 100%; background: var(--card-bg); border: 1px solid var(--border); border-radius: 6px; padding: 8px; z-index: 100; max-height: 250px; overflow-y: auto; width: 180px; box-shadow: 0 4px 12px rgba(0,0,0,0.3); }
    .dropdown-menu.show { display: block; }
    .col-dropdown-item { display: flex; align-items: center; gap: 8px; margin-bottom: 6px; cursor: pointer; font-size: 0.8rem; font-weight: 600; color: var(--text); }

    .overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 200; }
    .overlay.open { display: block; }
    .modal { display: none; position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background: var(--card-bg); border: 1px solid var(--border); padding: 16px; border-radius: 8px; z-index: 201; width: 90%; max-width: 500px; color: var(--text); }
    .modal.open { display: block; }
    .modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
    .team-list { max-height: 150px; overflow-y: auto; margin-bottom: 12px; border: 1px solid var(--border); padding: 8px; border-radius: 4px; }
    .team-item { display: flex; justify-content: space-between; align-items: center; padding: 4px 0; border-bottom: 1px solid var(--border); font-weight: 600; }
  </style>
</head>
<body>

  <header>
    <h1 id="teamHeaderTitle">Productivity & Break Dashboard</h1>
    <div style="display: flex; gap: 10px; align-items: center; flex-wrap: wrap;">
      <div class="clock-card" id="liveClock">🕒 --:--:-- --</div>
      <div id="currentWindowInfo" class="window-info">⏱️ Loading schedule window...</div>
      <button class="btn btn-secondary" id="themeToggleBtn">☀️ Light Mode</button>
    </div>
  </header>

  <div class="dropzone" id="dropzone">
    📁 <strong>Drag & Drop your Call CSV file here</strong> or click to browse
    <input type="file" id="fileInput" accept=".csv" style="display:none;" />
    <div id="fileNote" style="font-size: 0.75rem; color: var(--text-muted); margin-top: 4px;"></div>
  </div>

  <!-- Dynamic Threshold Settings Bar -->
  <div class="target-box">
    <span style="font-size:0.9rem; font-weight:800; color:var(--accent);">⚙️ Set Defaulter Limits:</span>
    
    <div class="target-input-item">
      <label>Target Eff. Calls (>240s):</label>
      <input type="number" id="limitEffCalls" value="25" min="1" />
    </div>

    <div class="target-input-item">
      <label>Target Manual TT (mins):</label>
      <input type="number" id="limitManualTT" value="60" min="1" title="60 mins = 1 Hour" />
    </div>

    <div class="target-input-item">
      <label>Target Total TT (mins):</label>
      <input type="number" id="limitTotalTT" value="225" min="1" title="225 mins = 3 Hours 45 Mins" />
    </div>
  </div>

  <div class="kpis" id="kpis"></div>

  <div class="tabs">
    <button class="tab-btn active" id="tabProductivityBtn" onclick="switchTab('productivity')">Productivity Summary</button>
    <button class="tab-btn" id="tabBreaksBtn" onclick="switchTab('breaks')">Detailed Break Logs</button>
    <button class="tab-btn defaulter-tab" id="tabDefaultersBtn" onclick="switchTab('defaulters')">⚠️ Defaulters List</button>
  </div>

  <div class="controls">
    <div class="control-group">
      <input type="text" id="searchBox" placeholder="Search Counsellor..." />
      
      <div id="statusFilterWrapper">
        <select id="fStatus">
          <option value="all">All Statuses</option>
          <option value="On Track">On Track</option>
          <option value="Off Track">Off Track</option>
          <option value="Defaulter">Defaulter</option>
        </select>
      </div>

      <select id="fTeamMode">
        <option value="filtered">Filtered Team</option>
        <option value="all">Show All CSV Users</option>
      </select>

      <button class="btn btn-secondary" id="resetBtn">Reset Filters</button>
    </div>

    <div class="control-group">
      <label style="font-size:0.75rem; font-weight:700;">Header BG:</label>
      <input type="color" id="headerBgPicker" value="#0f172a" title="Header Background Color" />
      <label style="font-size:0.75rem; font-weight:700;">Header Text:</label>
      <input type="color" id="headerTextPicker" value="#f8fafc" title="Header Text Color" />
      
      <div class="dropdown" id="colToggleWrapper">
        <button class="btn btn-secondary" id="colToggleBtn">Columns ⚙️</button>
        <div class="dropdown-menu" id="colDropdown"></div>
      </div>
      <button class="btn btn-secondary" id="teamSettingsBtn">Manage Team 👥</button>
      <button class="btn btn-secondary" id="printBtn">Print 🖨️</button>
      <button class="btn" id="exportBtn">Export PDF 📄</button>
    </div>
  </div>

  <div id="productivitySection" class="table-wrapper">
    <table>
      <thead>
        <tr id="theadRow"></tr>
      </thead>
      <tbody id="tbody"></tbody>
    </table>
  </div>

  <div id="breakSection" class="table-wrapper" style="display: none;">
    <table>
      <thead>
        <tr>
          <th>#</th>
          <th>Counsellor</th>
          <th>Break Start</th>
          <th>Break End</th>
          <th>Duration</th>
          <th>Category</th>
        </tr>
      </thead>
      <tbody id="breakTbody"></tbody>
    </table>
  </div>

  <!-- Defaulters Detailed Table Section -->
  <div id="defaulterSection" class="table-wrapper" style="display: none;">
    <table>
      <thead>
        <tr>
          <th>#</th>
          <th>Counsellor</th>
          <th id="defThTotalTT">Total TT</th>
          <th id="defThManualTT">Manual TT</th>
          <th id="defThEff">Effective Connect</th>
          <th>Failed Reason / Remarks</th>
        </tr>
      </thead>
      <tbody id="defaulterTbody"></tbody>
    </table>
  </div>

  <!-- Team Management Modal -->
  <div class="overlay" id="overlay"></div>
  <div class="modal" id="teamModal">
    <div class="modal-header">
      <h3 style="font-weight: 700;">Manage Team Members</h3>
      <button class="btn btn-secondary" id="closeTeamModal">✕</button>
    </div>
    <div style="margin-bottom: 12px;">
      <label style="display:block; margin-bottom: 4px; font-weight: 600;">Team Name:</label>
      <input type="text" id="teamNameInput" style="width: 100%;" />
    </div>
    <div>
      <label style="display:block; margin-bottom: 4px; font-weight: 600;">Team Members (<span id="teamMemberCount">0</span>):</label>
      <div class="team-list" id="teamList"></div>
    </div>
    <div style="display:flex; gap: 8px; margin-bottom: 12px;">
      <input type="text" id="addMemberEmail" placeholder="user@pw.live" style="flex:1;" />
      <button class="btn" id="addMemberBtn">Add</button>
    </div>
    <div style="margin-bottom: 12px;">
      <textarea id="bulkImportText" placeholder="Paste emails separated by commas or newlines..." style="width:100%; height:60px; background:var(--bg); color:var(--text); border:1px solid var(--border); border-radius:4px; padding:6px; font-size:0.8rem; font-weight: 500;"></textarea>
      <button class="btn btn-secondary" id="bulkImportBtn" style="width:100%; margin-top:4px;">Bulk Add Emails</button>
    </div>
    <div style="display: flex; justify-content: space-between;">
      <button class="btn btn-secondary" id="clearTeamBtn" style="background:var(--red-bg); color:var(--red-text);">Clear All</button>
      <button class="btn" id="saveTeamBtn">Save Changes</button>
    </div>
  </div>

  <script>
    function startClock() {
      const clockEl = document.getElementById('liveClock');
      function update() {
        const now = new Date();
        clockEl.innerHTML = `🕒 ${now.toLocaleTimeString('en-US', { hour12: true })}`;
      }
      update();
      setInterval(update, 1000);
    }
    startClock();

    let currentTheme = localStorage.getItem('theme') || 'dark';
    document.documentElement.setAttribute('data-theme', currentTheme);

    const themeToggleBtn = document.getElementById('themeToggleBtn');
    updateThemeButtonText();

    themeToggleBtn.addEventListener('click', () => {
      currentTheme = currentTheme === 'dark' ? 'light' : 'dark';
      document.documentElement.setAttribute('data-theme', currentTheme);
      localStorage.setItem('theme', currentTheme);
      updateThemeButtonText();
    });

    function updateThemeButtonText() {
      themeToggleBtn.textContent = currentTheme === 'dark' ? '☀️ Light Mode' : '🌙 Dark Mode';
    }

    const inputLimitEffCalls = document.getElementById('limitEffCalls');
    const inputLimitManualTT = document.getElementById('limitManualTT');
    const inputLimitTotalTT = document.getElementById('limitTotalTT');

    if (localStorage.getItem('targetEffCalls')) inputLimitEffCalls.value = localStorage.getItem('targetEffCalls');
    if (localStorage.getItem('targetManualTT')) inputLimitManualTT.value = localStorage.getItem('targetManualTT');
    if (localStorage.getItem('targetTotalTT')) inputLimitTotalTT.value = localStorage.getItem('targetTotalTT');

    [inputLimitEffCalls, inputLimitManualTT, inputLimitTotalTT].forEach(input => {
      input.addEventListener('input', () => {
        localStorage.setItem('targetEffCalls', inputLimitEffCalls.value);
        localStorage.setItem('targetManualTT', inputLimitManualTT.value);
        localStorage.setItem('targetTotalTT', inputLimitTotalTT.value);
        render();
      });
    });

    function getTargetLimits() {
      const eff = parseInt(inputLimitEffCalls.value, 10) || 0;
      const manualMins = parseInt(inputLimitManualTT.value, 10) || 0;
      const totalMins = parseInt(inputLimitTotalTT.value, 10) || 0;

      return {
        eff,
        manualSecs: manualMins * 60,
        totalSecs: totalMins * 60,
        manualMins,
        totalMins
      };
    }

    const DEFAULT_COLUMNS = [
      { id: 'rank', label: '#', default: true },
      { id: 'name', label: 'Counsellor', default: true },
      { id: 'status', label: 'Status', default: true },
      { id: 'totalTT', label: 'Total TT', default: true },
      { id: 'autoTT', label: 'Auto TT', default: true },
      { id: 'manualTT', label: 'Manual TT', default: true },
      { id: 'idle', label: 'Idle Time', default: true },
      { id: 'break10', label: '10m Breaks', default: true },
      { id: 'break20', label: '20m Breaks', default: true },
      { id: 'break30', label: '30m Breaks', default: true },
      { id: 'break40', label: '40m Breaks', default: true },
      { id: 'break50', label: '50m+ Breaks', default: true },
      { id: 'totalDialed', label: 'Total Dialed', default: true },
      { id: 'totalConnected', label: 'Total Connected', default: true },
      { id: 'effConnect', label: 'Effective Connect >240', default: true },
      { id: 'autoDials', label: 'Auto Dials', default: true },
      { id: 'manualDials', label: 'Manual Dials', default: true },
      { id: 'firstCall', label: '1st Call Time', default: true },
      { id: 'lastCall', label: 'Last Call Time', default: true },
      { id: 'span', label: 'Working Span', default: true }
    ];

    let ALL_COLUMNS = JSON.parse(localStorage.getItem('customColumnOrder')) || DEFAULT_COLUMNS;
    let columnVisibility = JSON.parse(localStorage.getItem('columnVisibility')) || {};
    ALL_COLUMNS.forEach(col => {
      if (columnVisibility[col.id] === undefined) columnVisibility[col.id] = col.default;
    });

    const headerBgPicker = document.getElementById('headerBgPicker');
    const headerTextPicker = document.getElementById('headerTextPicker');

    headerBgPicker.addEventListener('input', (e) => {
      document.documentElement.style.setProperty('--custom-header-bg', e.target.value);
      localStorage.setItem('customHeaderBg', e.target.value);
    });

    headerTextPicker.addEventListener('input', (e) => {
      document.documentElement.style.setProperty('--custom-header-text', e.target.value);
      localStorage.setItem('customHeaderText', e.target.value);
    });

    if (localStorage.getItem('customHeaderBg')) {
      headerBgPicker.value = localStorage.getItem('customHeaderBg');
      document.documentElement.style.setProperty('--custom-header-bg', headerBgPicker.value);
    }
    if (localStorage.getItem('customHeaderText')) {
      headerTextPicker.value = localStorage.getItem('customHeaderText');
      document.documentElement.style.setProperty('--custom-header-text', headerTextPicker.value);
    }

    const DEFAULT_TEAM = [
      { name: "Vipin Sharma", email: "vipin.sharma1@pw.live" },
      { name: "Supriya Bharti", email: "supriya.bharti@pw.live" },
      { name: "Sudhanshu Verma", email: "sudhanshu.verma1@pw.live" },
      { name: "Priya Gupta", email: "priya.gupta5@pw.live" }
    ];

    let teamMembers = JSON.parse(localStorage.getItem('teamMembers')) || DEFAULT_TEAM;
    let teamName = localStorage.getItem('teamName') || "Jaipur Team";
    let rawDump = JSON.parse(localStorage.getItem('productivity_dataset')) || [];
    let rawBreakLogs = JSON.parse(localStorage.getItem('break_logs')) || [];

    let currentTab = 'productivity';
    let sortKey = 'totalTT';
    let sortDir = -1;

    function parseDate(dateStr) {
      if (!dateStr) return null;
      let str = String(dateStr).trim();
      const m = str.match(/^(\d{1,2})[\/\-](\d{1,2})[\/\-](\d{4})(?:\s+(\d{1,2}):(\d{2})(?::(\d{2}))?\s*(AM|PM)?)?/i);
      if (m) {
        const day = parseInt(m[1], 10);
        const month = parseInt(m[2], 10) - 1;
        const year = parseInt(m[3], 10);
        let hrs = m[4] ? parseInt(m[4], 10) : 0;
        const mins = m[5] ? parseInt(m[5], 10) : 0;
        const secs = m[6] ? parseInt(m[6], 10) : 0;
        const ampm = m[7] ? m[7].toUpperCase() : null;

        if (ampm === 'PM' && hrs < 12) hrs += 12;
        if (ampm === 'AM' && hrs === 12) hrs = 0;

        return new Date(year, month, day, hrs, mins, secs);
      }
      let d = new Date(str);
      return isNaN(d.getTime()) ? null : d;
    }

    function toSeconds(hms) {
      if (!hms) return 0;
      const p = String(hms).trim().split(':').map(Number);
      if (p.length === 3) return (p[0] * 3600) + (p[1] * 60) + (p[2] || 0);
      if (p.length === 2) return (p[0] * 60) + (p[1] || 0);
      return 0;
    }

    function fmtHMS(sec) {
      sec = Math.max(0, Math.round(sec));
      const h = Math.floor(sec / 3600);
      const m = Math.floor((sec % 3600) / 60);
      const s = sec % 60;
      return `${h}h ${m}m ${s}s`;
    }

    function fmtClock(dObj) {
      if (!dObj) return '—';
      const d = (dObj instanceof Date) ? dObj : new Date(dObj);
      if (isNaN(d.getTime())) return '—';
      return d.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit', hour12: true });
    }

    function getCurrentHourlyTarget() {
      const now = new Date();
      const hrs = now.getHours() + now.getMinutes() / 60;
      if (hrs < 12) return { targetSecs: 3600, label: "10:00 AM - 12:00 PM Window (Target: 1 Hour TT)" };
      if (hrs < 14) return { targetSecs: 7200, label: "12:00 PM - 02:00 PM Window (Target: 2 Hours TT)" };
      if (hrs < 17) return { targetSecs: 10800, label: "02:00 PM - 05:00 PM Window (Target: 3 Hours TT)" };
      return { targetSecs: 14400, label: "05:00 PM - 08:00 PM Window (Target: 4 Hours TT)" };
    }

    function calculateMetrics(a) {
      const limits = getTargetLimits();
      const firstCall = a.firstCall ? new Date(a.firstCall) : null;
      const lastCall = a.lastCall ? new Date(a.lastCall) : null;
      let span = (firstCall && lastCall && lastCall >= firstCall) ? Math.max(0, (lastCall.getTime() - firstCall.getTime()) / 1000) : 0;
      const idle = Math.max(0, span - a.totalTT);

      let status = 'Defaulter', statusColor = 'red';
      if (a.totalTT >= limits.totalSecs && (a.manualTT || 0) >= limits.manualSecs && a.effConnect >= limits.eff) {
        status = 'On Track'; statusColor = 'green';
      } else if (a.totalTT >= (limits.totalSecs * 0.70)) {
        status = 'Off Track'; statusColor = 'yellow';
      }

      return { ...a, firstCall, lastCall, span, idle, status, statusColor };
    }

    function switchTab(tab) {
      currentTab = tab;
      document.getElementById('tabProductivityBtn').classList.toggle('active', tab === 'productivity');
      document.getElementById('tabBreaksBtn').classList.toggle('active', tab === 'breaks');
      document.getElementById('tabDefaultersBtn').classList.toggle('active', tab === 'defaulters');
      
      document.getElementById('productivitySection').style.display = tab === 'productivity' ? 'block' : 'none';
      document.getElementById('breakSection').style.display = tab === 'breaks' ? 'block' : 'none';
      document.getElementById('defaulterSection').style.display = tab === 'defaulters' ? 'block' : 'none';

      document.getElementById('statusFilterWrapper').style.display = tab === 'productivity' ? 'flex' : 'none';
      document.getElementById('colToggleWrapper').style.display = tab === 'productivity' ? 'flex' : 'none';
      render();
    }

    function getProcessedData() {
      const teamMode = document.getElementById('fTeamMode').value;
      let dataset = rawDump;
      if (teamMode === 'filtered' && teamMembers.length > 0) {
        const teamEmailsSet = new Set(teamMembers.map(m => m.email.toLowerCase().trim()));
        dataset = dataset.filter(a => teamEmailsSet.has(a.email.toLowerCase().trim()));
      }
      return dataset.map(calculateMetrics);
    }

    function render() {
      const limits = getTargetLimits();
      document.getElementById('teamHeaderTitle').textContent = `${teamName} Hourly Productivity & Break Analysis`;
      document.getElementById('currentWindowInfo').innerHTML = `⏱️ <strong>Current Schedule Window:</strong> ${getCurrentHourlyTarget().label}`;

      document.getElementById('tabDefaultersBtn').textContent = `⚠️ Defaulters List (<${limits.eff} Eff / <${limits.manualMins}m Manual / <${limits.totalMins}m Total TT)`;
      document.getElementById('defThTotalTT').textContent = `Total TT (Target: ${limits.totalMins}m)`;
      document.getElementById('defThManualTT').textContent = `Manual TT (Target: ${limits.manualMins}m)`;
      document.getElementById('defThEff').textContent = `Effective Connect (Target: ${limits.eff}+)`;

      const data = getProcessedData();
      const search = document.getElementById('searchBox').value.toLowerCase().trim();

      const totalCalls = data.reduce((s, a) => s + a.totalDialed, 0);
      const totalTT = data.reduce((s, a) => s + a.totalTT, 0);
      const totalBreaks = data.reduce((s, a) => s + (a.break10+a.break20+a.break30+a.break40+a.break50), 0);

      document.getElementById('kpis').innerHTML = `
        <div class="kpi"><div class="label">Total Members</div><div class="value">${data.length}</div></div>
        <div class="kpi"><div class="label">On Track</div><div class="value" style="color:var(--green-text);">${data.filter(a=>a.status==='On Track').length}</div></div>
        <div class="kpi"><div class="label">Defaulters</div><div class="value" style="color:var(--red-text);">${data.filter(a=>a.status==='Defaulter').length}</div></div>
        <div class="kpi"><div class="label">Total Dialed</div><div class="value">${totalCalls.toLocaleString()}</div></div>
        <div class="kpi"><div class="label">Total Talk Time</div><div class="value small">${fmtHMS(totalTT)}</div></div>
        <div class="kpi"><div class="label">Total Breaks Logged</div><div class="value" style="color:var(--accent);">${totalBreaks}</div></div>
      `;

      if (currentTab === 'productivity') {
        renderProductivityTable(data, search);
      } else if (currentTab === 'breaks') {
        renderBreakTable(search);
      } else if (currentTab === 'defaulters') {
        renderDefaulterTabTable(data, search);
      }
    }

    function renderProductivityTable(data, search) {
      renderHeader();
      const limits = getTargetLimits();
      const fStatus = document.getElementById('fStatus').value;
      let filtered = data.filter(a => {
        const nameMatch = (a.name || '').toLowerCase().includes(search) || a.email.toLowerCase().includes(search);
        const statusMatch = (fStatus === 'all') || (a.status === fStatus);
        return nameMatch && statusMatch;
      });

      filtered.sort((a, b) => {
        let va = a[sortKey], vb = b[sortKey];
        if (typeof va === 'string') { va = va.toLowerCase(); vb = vb.toLowerCase(); }
        if (va < vb) return -1 * sortDir;
        if (va > vb) return 1 * sortDir;
        return 0;
      });

      const tbody = document.getElementById('tbody');
      const visibleCols = ALL_COLUMNS.filter(col => columnVisibility[col.id]);

      if (filtered.length === 0) {
        tbody.innerHTML = `<tr><td colspan="${visibleCols.length}" style="text-align:center; padding:20px; font-weight:600;">No matching records found.</td></tr>`;
        return;
      }

      tbody.innerHTML = filtered.map((a, i) => {
        let cellsHTML = '';
        visibleCols.forEach(col => {
          switch (col.id) {
            case 'rank': cellsHTML += `<td class="bold-data">${i + 1}</td>`; break;
            case 'name': cellsHTML += `<td class="name-cell"><span class="name">${a.name || a.email}</span><span class="mail">${a.email}</span></td>`; break;
            case 'status': cellsHTML += `<td><span class="badge ${a.statusColor}">${a.status}</span></td>`; break;
            
            /* Total TT Color Formatting Fixed Here */
            case 'totalTT': {
              const tTT = a.totalTT || 0;
              let tClass = tTT >= limits.totalSecs ? 'badge green' : 'badge red';
              cellsHTML += `<td><span class="${tClass}">${fmtHMS(tTT)}</span></td>`;
              break;
            }

            case 'autoTT': cellsHTML += `<td class="muted-data">${fmtHMS(a.autoTT || 0)}</td>`; break;
            
            case 'manualTT': {
              const mTT = a.manualTT || 0;
              let mClass = mTT >= limits.manualSecs ? 'badge green' : 'badge red';
              cellsHTML += `<td><span class="${mClass}">${fmtHMS(mTT)}</span></td>`;
              break;
            }

            case 'idle': cellsHTML += `<td class="muted-data">${fmtHMS(a.idle)}</td>`; break;
            case 'break10': cellsHTML += `<td><span class="badge ${a.break10 > 0 ? 'blue' : ''}">${a.break10 || 0}</span></td>`; break;
            case 'break20': cellsHTML += `<td><span class="badge ${a.break20 > 0 ? 'yellow' : ''}">${a.break20 || 0}</span></td>`; break;
            case 'break30': cellsHTML += `<td><span class="badge ${a.break30 > 0 ? 'yellow' : ''}">${a.break30 || 0}</span></td>`; break;
            case 'break40': cellsHTML += `<td><span class="badge ${a.break40 > 0 ? 'red' : ''}">${a.break40 || 0}</span></td>`; break;
            case 'break50': cellsHTML += `<td><span class="badge ${a.break50 > 0 ? 'red' : ''}">${a.break50 || 0}</span></td>`; break;
            case 'totalDialed': cellsHTML += `<td class="bold-data">${a.totalDialed}</td>`; break;
            case 'totalConnected': cellsHTML += `<td class="bold-data">${a.totalConnected}</td>`; break;
            case 'effConnect': cellsHTML += `<td><span class="badge ${a.effConnect >= limits.eff ? 'green' : 'red'}">${a.effConnect}</span></td>`; break;
            case 'autoDials': cellsHTML += `<td class="muted-data">${a.autoDials || 0}</td>`; break;
            case 'manualDials': cellsHTML += `<td class="bold-data">${a.manualDials || 0}</td>`; break;
            case 'firstCall': cellsHTML += `<td class="muted-data">${fmtClock(a.firstCall)}</td>`; break;
            case 'lastCall': cellsHTML += `<td class="muted-data">${fmtClock(a.lastCall)}</td>`; break;
            case 'span': cellsHTML += `<td class="muted-data">${fmtHMS(a.span)}</td>`; break;
          }
        });
        return `<tr>${cellsHTML}</tr>`;
      }).join('');
    }

    function renderBreakTable(search) {
      const teamMode = document.getElementById('fTeamMode').value;
      let logs = rawBreakLogs;

      if (teamMode === 'filtered' && teamMembers.length > 0) {
        const teamEmailsSet = new Set(teamMembers.map(m => m.email.toLowerCase().trim()));
        logs = logs.filter(b => teamEmailsSet.has(b.email.toLowerCase().trim()));
      }

      if (search) {
        logs = logs.filter(b => b.name.toLowerCase().includes(search) || b.email.toLowerCase().includes(search));
      }

      const tbody = document.getElementById('breakTbody');
      if (logs.length === 0) {
        tbody.innerHTML = `<tr><td colspan="6" style="text-align:center; padding:20px; font-weight:600;">No break logs recorded.</td></tr>`;
        return;
      }

      tbody.innerHTML = logs.map((b, i) => {
        let badgeClass = 'blue';
        if (b.duration >= 45) badgeClass = 'red';
        else if (b.duration >= 25) badgeClass = 'yellow';

        return `
          <tr>
            <td class="bold-data">${i + 1}</td>
            <td class="name-cell"><span class="name">${b.name}</span><span class="mail">${b.email}</span></td>
            <td class="muted-data">${b.start}</td>
            <td class="muted-data">${b.end}</td>
            <td class="bold-data">${b.duration} mins</td>
            <td><span class="badge ${badgeClass}">${b.category}</span></td>
          </tr>
        `;
      }).join('');
    }

    function renderDefaulterTabTable(data, search) {
      const limits = getTargetLimits();
      
      let defaultersList = data.filter(a => {
        const isEffDef = a.effConnect < limits.eff;
        const isManualDef = (a.manualTT || 0) < limits.manualSecs;
        const isTotalTTDef = a.totalTT < limits.totalSecs;
        
        return isEffDef || isManualDef || isTotalTTDef;
      });

      if (search) {
        defaultersList = defaultersList.filter(a => (a.name || '').toLowerCase().includes(search) || a.email.toLowerCase().includes(search));
      }

      const tbody = document.getElementById('defaulterTbody');
      if (defaultersList.length === 0) {
        tbody.innerHTML = `<tr><td colspan="6" style="text-align:center; padding:20px; font-weight:600; color:var(--green-text);">🎉 Great Job! Sabhi logon ne targets poore kiye hain.</td></tr>`;
        return;
      }

      tbody.innerHTML = defaultersList.map((a, i) => {
        let reasons = [];
        if (a.totalTT < limits.totalSecs) reasons.push(`Total TT < ${limits.totalMins}m`);
        if ((a.manualTT || 0) < limits.manualSecs) reasons.push(`Manual TT < ${limits.manualMins}m`);
        if (a.effConnect < limits.eff) reasons.push(`Effective Connect < ${limits.eff}`);

        const reasonText = reasons.join(" | ");

        return `
          <tr>
            <td class="bold-data">${i + 1}</td>
            <td class="name-cell"><span class="name">${a.name || a.email}</span><span class="mail">${a.email}</span></td>
            <td><span class="badge ${a.totalTT < limits.totalSecs ? 'red' : 'green'}">${fmtHMS(a.totalTT)}</span></td>
            <td><span class="badge ${(a.manualTT || 0) < limits.manualSecs ? 'red' : 'green'}">${fmtHMS(a.manualTT || 0)}</span></td>
            <td><span class="badge ${a.effConnect < limits.eff ? 'red' : 'green'}">${a.effConnect}</span></td>
            <td><span class="badge red remark-tag">⚠️ ${reasonText}</span></td>
          </tr>
        `;
      }).join('');
    }

    function handleCSV(text) {
      Papa.parse(text, {
        header: true, skipEmptyLines: true,
        complete: (res) => {
          const rows = res.data;
          const userCallsMap = new Map();

          rows.forEach(r => {
            const email = (r['User ID'] || r['User Name'] || r['Email'] || '').trim();
            if (!email) return;

            const rawTime = r['Call Time'] || r['Time'] || r['Login Time'];
            const parsedDt = parseDate(rawTime);
            if (!parsedDt) return;

            const ttSec = toSeconds(r['Customer Talk Time'] || r['User Talk Time'] || r['Talk Time'] || '0:00:00');
            const acwSec = toSeconds(r['ACW Duration'] || '0:00:00');
            const ringSec = toSeconds(r['User Ringing Time'] || r['Customer Ringing Time'] || '0:00:00');

            const sysDisp = (r['System Disposition'] || r['Disposition'] || '').toUpperCase();
            const dialType = (r['Dial Type'] || r['Call Type'] || '').toUpperCase();

            if (!userCallsMap.has(email)) userCallsMap.set(email, []);
            userCallsMap.get(email).push({
              dt: parsedDt, ttSec, acwSec, ringSec, sysDisp, dialType,
              rawUser: r['User Name'] || email
            });
          });

          const processedSummary = [];
          const allBreakLogs = [];

          userCallsMap.forEach((calls, email) => {
            calls.sort((a, b) => a.dt - b.dt);
            const matchedMember = teamMembers.find(m => m.email.toLowerCase() === email.toLowerCase());
            const name = matchedMember ? matchedMember.name : calls[0].rawUser;

            let totalTT = 0, autoTT = 0, manualTT = 0;
            let totalDialed = calls.length, totalConnected = 0, effConnect = 0;
            let autoDials = 0, manualDials = 0;
            let break10 = 0, break20 = 0, break30 = 0, break40 = 0, break50 = 0;

            let prevCallEndTime = null;

            calls.forEach(c => {
              totalTT += c.ttSec;
              if (c.dialType.includes('AUTO')) {
                autoDials++;
                autoTT += c.ttSec;
              } else {
                manualDials++;
                manualTT += c.ttSec;
              }

              if (c.sysDisp === 'CONNECTED') totalConnected++;
              if (c.ttSec >= 240) effConnect++;

              if (prevCallEndTime) {
                const gapMinutes = (c.dt.getTime() - prevCallEndTime.getTime()) / (1000 * 60);
                if (gapMinutes >= 5) {
                  const dur = Math.round(gapMinutes);
                  let category = '10m Break';

                  if (dur >= 5 && dur < 15) { break10++; category = '10m Break'; }
                  else if (dur >= 15 && dur < 25) { break20++; category = '20m Break'; }
                  else if (dur >= 25 && dur < 35) { break30++; category = '30m Break'; }
                  else if (dur >= 35 && dur < 45) { break40++; category = '40m Break'; }
                  else if (dur >= 45) { break50++; category = '50m+ Break'; }

                  allBreakLogs.push({
                    email, name,
                    start: fmtClock(prevCallEndTime),
                    end: fmtClock(c.dt),
                    duration: dur, category
                  });
                }
              }

              const callTotalDurationSec = c.ttSec + c.acwSec + c.ringSec;
              prevCallEndTime = new Date(c.dt.getTime() + callTotalDurationSec * 1000);
            });

            processedSummary.push({
              email, name, totalTT, autoTT, manualTT, totalDialed, totalConnected, effConnect,
              autoDials, manualDials,
              firstCall: calls[0].dt.toISOString(),
              lastCall: calls[calls.length - 1].dt.toISOString(),
              break10, break20, break30, break40, break50
            });
          });

          rawDump = processedSummary;
          rawBreakLogs = allBreakLogs;
          localStorage.setItem('productivity_dataset', JSON.stringify(rawDump));
          localStorage.setItem('break_logs', JSON.stringify(rawBreakLogs));
          document.getElementById('fileNote').textContent = `Loaded ${rows.length} call logs successfully!`;
          render();
        }
      });
    }

    function initColumns() {
      const dropdown = document.getElementById('colDropdown');
      dropdown.innerHTML = ALL_COLUMNS.map(col => `
        <label class="col-dropdown-item">
          <input type="checkbox" data-col="${col.id}" ${columnVisibility[col.id] ? 'checked' : ''}>
          ${col.label}
        </label>
      `).join('');

      dropdown.querySelectorAll('input').forEach(chk => {
        chk.addEventListener('change', (e) => {
          columnVisibility[e.target.dataset.col] = e.target.checked;
          localStorage.setItem('columnVisibility', JSON.stringify(columnVisibility));
          render();
        });
      });
    }

    let draggedColId = null;

    function renderHeader() {
      const tr = document.getElementById('theadRow');
      const visibleCols = ALL_COLUMNS.filter(col => columnVisibility[col.id]);

      tr.innerHTML = visibleCols.map(col => `
        <th draggable="true" data-key="${col.id}" title="Drag to swipe position | Double-click to edit label">
          <span class="header-text">${col.label}</span>
        </th>
      `).join('');

      tr.querySelectorAll('th').forEach(th => {
        th.addEventListener('click', (e) => {
          if (e.target.tagName === 'INPUT') return;
          const key = th.dataset.key;
          if (key === 'rank') return;
          if (sortKey === key) sortDir *= -1;
          else { sortKey = key; sortDir = -1; }
          render();
        });

        th.addEventListener('dblclick', (e) => {
          e.stopPropagation();
          const key = th.dataset.key;
          const colObj = ALL_COLUMNS.find(c => c.id === key);
          const currentLabel = colObj.label;

          const input = document.createElement('input');
          input.type = 'text';
          input.value = currentLabel;
          input.style.width = '90%';
          input.style.fontSize = '0.75rem';

          th.innerHTML = '';
          th.appendChild(input);
          input.focus();

          function saveHeaderLabel() {
            const newLabel = input.value.trim() || currentLabel;
            colObj.label = newLabel;
            localStorage.setItem('customColumnOrder', JSON.stringify(ALL_COLUMNS));
            initColumns();
            render();
          }

          input.addEventListener('blur', saveHeaderLabel);
          input.addEventListener('keydown', (evt) => {
            if (evt.key === 'Enter') saveHeaderLabel();
          });
        });

        th.addEventListener('dragstart', (e) => {
          draggedColId = th.dataset.key;
          th.classList.add('dragging');
          e.dataTransfer.effectAllowed = 'move';
        });

        th.addEventListener('dragover', (e) => {
          e.preventDefault();
          e.dataTransfer.dropEffect = 'move';
          th.classList.add('drag-over');
        });

        th.addEventListener('dragleave', () => {
          th.classList.remove('drag-over');
        });

        th.addEventListener('dragend', () => {
          th.classList.remove('dragging');
          document.querySelectorAll('th').forEach(el => el.classList.remove('drag-over'));
        });

        th.addEventListener('drop', (e) => {
          e.preventDefault();
          th.classList.remove('drag-over');
          const targetColId = th.dataset.key;

          if (draggedColId && targetColId && draggedColId !== targetColId) {
            const dragIdx = ALL_COLUMNS.findIndex(c => c.id === draggedColId);
            const targetIdx = ALL_COLUMNS.findIndex(c => c.id === targetColId);

            const [movedCol] = ALL_COLUMNS.splice(dragIdx, 1);
            ALL_COLUMNS.splice(targetIdx, 0, movedCol);

            localStorage.setItem('customColumnOrder', JSON.stringify(ALL_COLUMNS));
            initColumns();
            render();
          }
        });
      });
    }

    document.getElementById('searchBox').addEventListener('input', render);
    document.getElementById('fStatus').addEventListener('change', render);
    document.getElementById('fTeamMode').addEventListener('change', render);

    const colToggleBtn = document.getElementById('colToggleBtn');
    const colDropdown = document.getElementById('colDropdown');

    colToggleBtn.addEventListener('click', (e) => {
      e.stopPropagation();
      colDropdown.classList.toggle('show');
    });

    document.addEventListener('click', (e) => {
      if (!colDropdown.contains(e.target) && e.target !== colToggleBtn) colDropdown.classList.remove('show');
    });

    document.getElementById('resetBtn').addEventListener('click', () => {
      document.getElementById('searchBox').value = '';
      document.getElementById('fStatus').value = 'all';
      document.getElementById('fTeamMode').value = 'filtered';
      inputLimitEffCalls.value = 25;
      inputLimitManualTT.value = 60;
      inputLimitTotalTT.value = 225;
      localStorage.removeItem('targetEffCalls');
      localStorage.removeItem('targetManualTT');
      localStorage.removeItem('targetTotalTT');
      ALL_COLUMNS = JSON.parse(JSON.stringify(DEFAULT_COLUMNS));
      localStorage.removeItem('customColumnOrder');
      initColumns();
      render();
    });

    const dropzone = document.getElementById('dropzone');
    const fileInput = document.getElementById('fileInput');

    dropzone.addEventListener('click', () => fileInput.click());
    ['dragenter', 'dragover'].forEach(e => dropzone.addEventListener(e, (evt) => { evt.preventDefault(); dropzone.classList.add('dragover'); }));
    ['dragleave', 'drop'].forEach(e => dropzone.addEventListener(e, (evt) => { evt.preventDefault(); dropzone.classList.remove('dragover'); }));

    dropzone.addEventListener('drop', (e) => {
      if (e.dataTransfer.files.length > 0) {
        const reader = new FileReader();
        reader.onload = ev => handleCSV(ev.target.result);
        reader.readAsText(e.dataTransfer.files[0]);
      }
    });

    fileInput.addEventListener('change', e => {
      if (e.target.files[0]) {
        const reader = new FileReader();
        reader.onload = ev => handleCSV(ev.target.result);
        reader.readAsText(e.target.files[0]);
      }
    });

    document.getElementById('teamSettingsBtn').addEventListener('click', () => {
      document.getElementById('teamNameInput').value = teamName;
      renderTeamList();
      document.getElementById('overlay').classList.add('open');
      document.getElementById('teamModal').classList.add('open');
    });

    document.getElementById('closeTeamModal').addEventListener('click', () => {
      document.getElementById('overlay').classList.remove('open');
      document.getElementById('teamModal').classList.remove('open');
    });

    function renderTeamList() {
      document.getElementById('teamMemberCount').textContent = teamMembers.length;
      document.getElementById('teamList').innerHTML = teamMembers.map((m, i) => `
        <div class="team-item">
          <span>${m.email}</span>
          <button class="btn" style="padding:2px 6px; font-size:10px;" onclick="removeMember(${i})">Remove</button>
        </div>
      `).join('');
    }

    function removeMember(i) {
      teamMembers.splice(i, 1);
      renderTeamList();
    }

    document.getElementById('addMemberBtn').addEventListener('click', () => {
      const email = document.getElementById('addMemberEmail').value.trim();
      if (email) {
        teamMembers.push({ name: email.split('@')[0], email });
        document.getElementById('addMemberEmail').value = '';
        renderTeamList();
      }
    });

    document.getElementById('bulkImportBtn').addEventListener('click', () => {
      const raw = document.getElementById('bulkImportText').value;
      const emails = raw.split(/[\n,]/).map(e => e.trim()).filter(e => e);
      emails.forEach(email => {
        if (!teamMembers.some(m => m.email.toLowerCase() === email.toLowerCase())) {
          teamMembers.push({ name: email.split('@')[0], email });
        }
      });
      document.getElementById('bulkImportText').value = '';
      renderTeamList();
    });

    document.getElementById('clearTeamBtn').addEventListener('click', () => {
      teamMembers = [];
      renderTeamList();
    });

    document.getElementById('saveTeamBtn').addEventListener('click', () => {
      teamName = document.getElementById('teamNameInput').value.trim() || "My Team";
      localStorage.setItem('teamMembers', JSON.stringify(teamMembers));
      localStorage.setItem('teamName', teamName);
      document.getElementById('overlay').classList.remove('open');
      document.getElementById('teamModal').classList.remove('open');
      render();
    });

    document.getElementById('printBtn').addEventListener('click', () => window.print());
    document.getElementById('exportBtn').addEventListener('click', () => {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF('l', 'mm', 'a4');
      const limits = getTargetLimits();

      if (currentTab === 'productivity') {
        const data = getProcessedData();
        const visibleCols = ALL_COLUMNS.filter(col => columnVisibility[col.id]);
        doc.text(`${teamName} - Productivity & Summary Report`, 14, 15);
        doc.autoTable({
          head: [visibleCols.map(c => c.label)],
          body: data.map((a, i) => visibleCols.map(col => {
            switch (col.id) {
              case 'rank': return i + 1;
              case 'name': return a.name || a.email;
              case 'status': return a.status;
              case 'totalTT': return fmtHMS(a.totalTT);
              case 'autoTT': return fmtHMS(a.autoTT || 0);
              case 'manualTT': return fmtHMS(a.manualTT || 0);
              case 'idle': return fmtHMS(a.idle);
              case 'break10': return a.break10 || 0;
              case 'break20': return a.break20 || 0;
              case 'break30': return a.break30 || 0;
              case 'break40': return a.break40 || 0;
              case 'break50': return a.break50 || 0;
              case 'totalDialed': return a.totalDialed;
              case 'totalConnected': return a.totalConnected;
              case 'effConnect': return a.effConnect;
              case 'autoDials': return a.autoDials || 0;
              case 'manualDials': return a.manualDials || 0;
              case 'firstCall': return fmtClock(a.firstCall);
              case 'lastCall': return fmtClock(a.lastCall);
              case 'span': return fmtHMS(a.span);
              default: return '';
            }
          })), startY: 20
        });
      } else if (currentTab === 'breaks') {
        doc.text(`${teamName} - Detailed Break Time Logs`, 14, 15);
        doc.autoTable({
          head: [['#', 'Counsellor', 'Start Time', 'End Time', 'Duration', 'Category']],
          body: rawBreakLogs.map((b, i) => [i + 1, b.name, b.start, b.end, `${b.duration} mins`, b.category]),
          startY: 20
        });
      } else if (currentTab === 'defaulters') {
        const data = getProcessedData();
        let defaultersList = data.filter(a => a.effConnect < limits.eff || (a.manualTT || 0) < limits.manualSecs || a.totalTT < limits.totalSecs);
        doc.text(`${teamName} - Defaulters List Report`, 14, 15);
        doc.autoTable({
          head: [['#', 'Counsellor', 'Total TT', 'Manual TT', 'Effective Connect', 'Remarks']],
          body: defaultersList.map((a, i) => {
            let reasons = [];
            if (a.totalTT < limits.totalSecs) reasons.push(`Total TT < ${limits.totalMins}m`);
            if ((a.manualTT || 0) < limits.manualSecs) reasons.push(`Manual TT < ${limits.manualMins}m`);
            if (a.effConnect < limits.eff) reasons.push(`Eff Connect < ${limits.eff}`);
            return [i + 1, a.name || a.email, fmtHMS(a.totalTT), fmtHMS(a.manualTT || 0), a.effConnect, reasons.join(" | ")];
          }),
          startY: 20
        });
      }
      doc.save('dashboard_report.pdf');
    });

    initColumns();
    render();
  </script>
</body>
</html>
