# SHaDOW-REaPER<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>SHADOW REAPERS — Esports Tournaments</title>
  <meta name="description" content="SHADOW REAPERS — Custom rooms & tournaments for BGMI, Free Fire, PUBG. Demo single-file app (client-side)."/>
  <link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 256 256'%3E%3Crect width='256' height='256' rx='48' fill='%2305050A'/%3E%3Cpath d='M128 44c-36 0-64 22-64 58 0 26 14 49 38 59l-10 33 36-24 36 24-10-33c24-10 38-33 38-59 0-36-28-58-64-58z' fill='%23d946ef'/%3E%3Cpath d='M96 122c0-19 15-34 32-34s32 15 32 34-15 34-32 34-32-15-32-34z' fill='%2300d1ff' opacity='.7'/%3E%3C/svg%3E" />
  <!-- Tailwind (runtime) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    :root { color-scheme: dark; }
    .neon { text-shadow: 0 0 8px rgba(217,70,239,.9), 0 0 18px rgba(129,140,248,.5); }
    /* small utility classes using Tailwind's @apply are not available here so use standard classes */
    body { font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; }
    .card { border-radius: 1rem; padding: 1.2rem; border: 1px solid rgba(255,255,255,0.06); background: linear-gradient(180deg, rgba(36,36,36,0.6), rgba(0,0,0,0.6)); box-shadow: 0 10px 30px rgba(0,0,0,0.6); }
    .btn { padding: .5rem 1rem; border-radius: .75rem; border: 1px solid rgba(255,255,255,0.06); background: rgba(255,255,255,0.03); backdrop-filter: blur(6px); cursor: pointer; transition: .18s; }
    .btn:hover { transform: translateY(-2px); background: rgba(255,255,255,0.05); }
    .badge { font-size: .75rem; padding: .2rem .6rem; border-radius: .5rem; border: 1px solid rgba(255,255,255,0.06); background: rgba(255,255,255,0.03); }
    .grid-bg { background-image: radial-gradient(transparent 1px, rgba(255,255,255,0.02) 1px); background-size: 20px 20px; }
    .muted { color: rgba(255,255,255,0.6); }
    .danger { color: #ef4444; }
    .success { color: #10b981; }
    .mono { font-family: "Roboto Mono", ui-monospace, SFMono-Regular, Menlo, Monaco, monospace; }
  </style>
</head>
<body class="min-h-screen bg-[#05050A] text-white grid-bg">

  <!-- Big background glows -->
  <div aria-hidden class="pointer-events-none fixed inset-0 -z-10">
    <div class="absolute -top-40 -left-40 w-[40rem] h-[40rem] bg-fuchsia-600/20 blur-3xl rounded-full"></div>
    <div class="absolute -bottom-40 -right-40 w-[40rem] h-[40rem] bg-indigo-600/18 blur-3xl rounded-full"></div>
  </div>

  <!-- Header -->
  <header class="sticky top-0 z-50 backdrop-blur bg-black/30 border-b border-white/6">
    <div class="max-w-6xl mx-auto px-4 py-3 flex items-center justify-between">
      <div class="flex items-center gap-3">
        <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-fuchsia-500 to-indigo-500 flex items-center justify-center shadow-lg">
          <svg width="22" height="22" viewBox="0 0 24 24" fill="none" class="text-black"><path d="M12 2c3 0 5.5 1.8 6.5 4.3C19.9 8 18 9.8 16 10.2 13.7 10.7 11.5 9.5 11 7.2 10.5 4.8 11.6 2 12 2z" fill="#000"/></svg>
        </div>
        <div class="font-black tracking-wide text-lg">SHADOW <span class="text-fuchsia-400 neon">REAPERS</span></div>
      </div>

      <nav class="flex items-center gap-2 text-sm">
        <button class="btn" data-view="home">Home</button>
        <button class="btn" data-view="tournaments">Tournaments</button>
        <button class="btn" data-view="leaderboard">Leaderboard</button>
        <button class="btn" data-view="admin">Admin</button>
      </nav>

      <div id="authBox" class="flex items-center gap-3">
        <!-- Auth UI injected here -->
      </div>
    </div>
  </header>

  <!-- Main -->
  <main id="app" class="max-w-6xl mx-auto px-4 py-10">
    <!-- Views injected by JS -->
  </main>

  <footer class="border-t border-white/6 mt-12">
    <div class="max-w-6xl mx-auto px-4 py-6 text-sm text-zinc-400 flex flex-col md:flex-row md:justify-between gap-2">
      <div>© <span id="year"></span> SHADOW REAPERS — Built for Gamers</div>
      <div>Non-transferable rooms • Variable entry fees • UPI payment (demo)</div>
    </div>
  </footer>

  <!-- App script (single-file app) -->
  <script>
  /*************************************************************************
   * SHADOW REAPERS — Single-file client-side prototype
   *
   * - IMPORTANT: This is a fully client-side demo using localStorage.
   * - Real payments (Razorpay/UPI) and secure order verification REQUIRE a backend.
   *   See notes at end of file explaining how to integrate Razorpay (server + webhook).
   *
   * Save this file as index.html and host on GitHub Pages or Netlify.
   *************************************************************************/

  // --- Tiny utilities ---
  const $ = s => document.querySelector(s);
  const $$ = s => Array.from(document.querySelectorAll(s));
  const now = () => Date.now();
  const uid = () => 'u_' + Math.random().toString(36).slice(2,9);
  const clamp = (v, a, b) => Math.max(a, Math.min(b, v));

  // --- Local DB wrapper (persistent to localStorage) ---
  const DB = {
    load(key, fallback) {
      try { return JSON.parse(localStorage.getItem(key)) ?? fallback; } catch(e) { return fallback; }
    },
    save(key, val) { localStorage.setItem(key, JSON.stringify(val)); }
  };

  // --- Application state persisted locally ---
  const state = {
    user: DB.load('sr_user', null),          // { uid, name, gamingId, createdAt }
    tournaments: DB.load('sr_tournaments', null), // array of tournament docs
    results: DB.load('sr_results', []),      // aggregated results
  };

  // --- default seed tournaments (if none exist) ---
  if (!state.tournaments) {
    const tnow = Date.now();
    // helper to make start times at next hours
    const inHours = h => { const d = new Date(); d.setMinutes(0,0,0); d.setHours(d.getHours() + h); return d.getTime(); };
    state.tournaments = [
      makeTournament('tf1','BGMI | Midnight Rush','BGMI',50,100, inHours(4), 60*60*1000),
      makeTournament('tf2','Free Fire | Morning Clash','Free Fire',30,80, inHours(1), 45*60*1000),
      makeTournament('tf3','PUBG | Noon Arena','PUBG',70,120, inHours(3), 75*60*1000),
    ];
    DB.save('sr_tournaments', state.tournaments);
  }

  // tournament structure:
  // { id, slug, name, title(game), fee (₹), maxPlayers, startAt(ms), matchDurationMs, status, entries: [{uid,name,gamingId,paidAt,paymentMeta}], rooms: { roomId -> { roomId, tournamentId, slotStartAt, gamingId, roomPass, createdAt, deleteAt } }, results: [] }
  function makeTournament(id, name, title, fee, maxPlayers, startAt, matchDurationMs=60*60*1000) {
    return {
      id, slug: id,
      name, title, fee, maxPlayers,
      startAt, matchDurationMs,
      status: 'upcoming',
      entries: [],
      rooms: {},    // dictionary keyed by roomId (but rooms are also indexable by gamingId)
      results: []
    };
  }

  // --- housekeeping: remove expired rooms (deleteAt <= now) on every render/load ---
  function cleanupExpiredRooms() {
    let changed = false;
    const ts = now();
    state.tournaments.forEach(t => {
      for (const [rId, r] of Object.entries(t.rooms || {})) {
        if (r.deleteAt && new Date(r.deleteAt).getTime() <= ts) {
          delete t.rooms[rId];
          changed = true;
        }
      }
    });
    if (changed) { DB.save('sr_tournaments', state.tournaments); }
  }

  // run cleanup immediately
  cleanupExpiredRooms();

  // --- helpers for room assignment ---
  function roomForTournamentAndGamingId(t, gamingId, slotStartAt) {
    // rooms are keyed by `${t.id}::${slotStartAt}::${gamingId}` to ensure one room per gamingId per time-slot
    const key = `${t.id}::${slotStartAt}::${gamingId}`;
    if (t.rooms && t.rooms[key]) return t.rooms[key];
    const room = {
      roomId: `${t.id.toUpperCase()}-${Math.random().toString(36).slice(2,6).toUpperCase()}`,
      tournamentId: t.id,
      slotStartAt,
      gamingId,
      roomPass: Math.random().toString(36).slice(2,8).toUpperCase(),
      createdAt: now(),
      deleteAt: slotStartAt + (t.matchDurationMs || 60*60*1000) + 24*3600*1000 // delete 24 hours after match end
    };
    t.rooms = t.rooms || {};
    t.rooms[key] = room;
    DB.save('sr_tournaments', state.tournaments);
    return room;
  }

  // check if user already joined a particular slot
  function userHasEntry(t, slotStartAt, uid) {
    return t.entries.some(e => e.uid === uid && e.slotStartAt === slotStartAt);
  }

  // --- UI rendering & navigation ---
  let currentView = 'home';
  const navButtons = () => $$('header [data-view]');
  navButtons().forEach(btn => btn.addEventListener('click', () => switchView(btn.dataset.view)));

  function switchView(v) {
    currentView = v;
    render();
    // highlight active nav
    navButtons().forEach(b => b.classList.remove('ring-2','ring-fuchsia-400'));
    const active = document.querySelector(`[data-view="${v}"]`);
    if (active) active.classList.add('ring-2','ring-fuchsia-400');
  }

  // Render auth box
  function renderAuth() {
    const box = $('#authBox');
    if (!state.user) {
      box.innerHTML = `
        <input id="nameIn" placeholder="Name" class="px-3 py-2 rounded-xl bg-black/40 border border-white/10 text-sm" />
        <input id="gidIn" placeholder="Gaming ID" class="px-3 py-2 rounded-xl bg-black/40 border border-white/10 text-sm" />
        <button id="loginBtn" class="btn flex items-center gap-2"><svg class="w-4 h-4" viewBox="0 0 24 24" fill="none"><path d="M15 3h4a2 2 0 0 1 2 2v14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg> Login</button>
      `;
      $('#loginBtn').onclick = () => {
        const name = $('#nameIn').value.trim() || 'Player';
        const gid = $('#gidIn').value.trim() || 'GUEST';
        const user = { uid: uid(), name, gamingId: gid, createdAt: now() };
        state.user = user; DB.save('sr_user', user);
        renderAuth(); render();
      };
    } else {
      box.innerHTML = `
        <div class="text-sm text-white/80">${escapeHtml(state.user.name)} • <span class="text-zinc-400">${escapeHtml(state.user.gamingId)}</span></div>
        <button id="logoutBtn" class="btn">Logout</button>
      `;
      $('#logoutBtn').onclick = () => {
        state.user = null; localStorage.removeItem('sr_user'); renderAuth(); render();
      };
    }
  }

  // escape helper to avoid injection in html strings
  function escapeHtml(str){ return String(str).replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;'); }

  // render main app views
  function render() {
    cleanupExpiredRooms();
    renderAuth();
    $('#year').textContent = new Date().getFullYear();

    const app = $('#app');
    if (currentView === 'home') {
      app.innerHTML = `
        <section class="grid md:grid-cols-2 gap-8 items-center">
          <div>
            <h1 class="text-4xl md:text-5xl font-extrabold leading-tight">Enter the <span class="text-fuchsia-400 neon">SHADOW REAPERS</span> Arena</h1>
            <p class="mt-4 text-zinc-400">Custom rooms. Real competition. Variable entry fees. Choose your game and time-slot. Rooms are non-transferable and deleted 24 hours after match end.</p>
            <div class="mt-6 flex gap-3">
              <button class="btn" onclick="switchView('tournaments')">Explore Tournaments</button>
              <button class="btn" onclick="switchView('leaderboard')">View Leaderboard</button>
            </div>
            <div class="mt-6 grid grid-cols-3 gap-3 text-xs text-zinc-400">
              <div class="flex items-center gap-2"><svg class="w-4 h-4" viewBox="0 0 24 24" fill="none"><path d="M12 2v6" stroke="currentColor" stroke-width="1.2"/></svg> Custom Rooms</div>
              <div class="flex items-center gap-2"><svg class="w-4 h-4" viewBox="0 0 24 24" fill="none"><path d="M12 2v6" stroke="currentColor" stroke-width="1.2"/></svg> Non-transferable</div>
              <div class="flex items-center gap-2"><svg class="w-4 h-4" viewBox="0 0 24 24" fill="none"><path d="M12 2v6" stroke="currentColor" stroke-width="1.2"/></svg> 24h room retention</div>
            </div>
          </div>
          <div>
            <div class="card rounded-2xl p-6">
              <div class="text-center">
                <svg class="mx-auto w-20 h-20 text-fuchsia-400" viewBox="0 0 24 24" fill="none"><path d="M12 2c3 0 5.5 1.8 6.5 4.3C19.9 8 18 9.8 16 10.2 13.7 10.7 11.5 9.5 11 7.2 10.5 4.8 11.6 2 12 2z" fill="#d946ef"/></svg>
                <div class="mt-3 text-xl tracking-wide font-semibold">ONE ID • ONE ROOM • NO TRANSFER</div>
                <div class="text-zinc-400 text-sm mt-2">Rooms per time-slot. Rooms appear after payment and show join links & match result when available.</div>
                <div class="mt-4"><button class="btn" onclick="switchView('tournaments')">Find Tournaments</button></div>
              </div>
            </div>
          </div>
        </section>
      `;
    } else if (currentView === 'tournaments') {
      // list tournaments grouped by game, and each tournament can show time slots (we treat each tournament entry as a slot for simplicity)
      const cards = state.tournaments.map(t => {
        const entriesCount = t.entries.length;
        const spotsLeft = Math.max(0, t.maxPlayers - entriesCount);
        const start = new Date(t.startAt);
        const startLabel = start.toLocaleString();
        return `
          <div class="card relative">
            <div class="flex items-start justify-between">
              <div>
                <div class="badge">${escapeHtml(t.title)}</div>
                <div class="text-xl font-bold mt-2">${escapeHtml(t.name)}</div>
                <div class="text-zinc-400 text-sm mt-1">Starts: <span class="mono">${startLabel}</span></div>
                <div class="text-zinc-400 text-sm">Duration: ${Math.round((t.matchDurationMs||3600000)/60000)} min</div>
              </div>
              <div class="text-right">
                <div class="text-zinc-400 text-sm">Fee</div>
                <div class="text-2xl font-extrabold text-amber-400">₹${t.fee}</div>
                <div class="text-zinc-400 text-sm mt-2">Spots: ${entriesCount}/${t.maxPlayers}</div>
              </div>
            </div>
            <div class="mt-4 flex items-center justify-between">
              <div class="text-sm text-zinc-400">Status: <span class="${t.status==='live'?'success':'muted'}">${t.status}</span></div>
              <div class="flex gap-2">
                <button class="btn" onclick="viewTournament('${t.id}')">Details</button>
                <button class="btn" onclick="openJoinModal('${t.id}')">Join (Pay)</button>
              </div>
            </div>
            ${t.entries.some(e => state.user && e.uid === state.user.uid) ? `<div class="mt-3 p-3 rounded-md bg-white/3 text-sm">You have joined this slot with ID <span class="mono">${escapeHtml(state.user.gamingId)}</span></div>` : ''}
          </div>
        `;
      }).join('');
      app.innerHTML = `
        <section>
          <h2 class="text-2xl font-bold mb-4">Active & Upcoming Tournaments</h2>
          <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">${cards}</div>
        </section>
      `;
    } else if (currentView === 'leaderboard') {
      // aggregate results
      const totals = {};
      (state.results || []).forEach(r => {
        const key = r.gamingId;
        if (!totals[key]) totals[key] = { gamingId: key, points: 0, kills: 0, events: 0 };
        totals[key].points += (r.points || 0);
        totals[key].kills += (r.kills || 0);
        totals[key].events += 1;
      });
      const rows = Object.values(totals).sort((a,b) => b.points - a.points).slice(0,100);
      const body = rows.map((r,i) => `<tr class="${i%2===0?'bg-white/2':''}"><td class="p-2">${i+1}</td><td class="p-2 mono">${escapeHtml(r.gamingId)}</td><td class="p-2 font-semibold">${r.points}</td><td class="p-2">${r.kills}</td><td class="p-2">${r.events}</td></tr>`).join('');
      app.innerHTML = `
        <section>
          <h2 class="text-2xl font-bold mb-4">Global Leaderboard</h2>
          ${rows.length ? `
            <div class="card overflow-x-auto">
              <table class="w-full text-sm">
                <thead class="text-zinc-400">
                  <tr><th class="text-left p-2">#</th><th class="text-left p-2">Gaming ID</th><th class="text-left p-2">Points</th><th class="text-left p-2">Kills</th><th class="text-left p-2">Events</th></tr>
                </thead>
                <tbody>${body}</tbody>
              </table>
            </div>
          ` : `<div class="card text-zinc-400">No results yet. Organize matches to generate results.</div>`}
        </section>
      `;
    } else if (currentView === 'admin') {
      // admin control to add new tournaments/time slots and to start/finish sim matches
      const opts = state.tournaments.map(t => `<option value="${t.id}">${escapeHtml(t.name)} — ${escapeHtml(t.title)} — ${new Date(t.startAt).toLocaleString()}</option>`).join('');
      app.innerHTML = `
        <section class="grid md:grid-cols-2 gap-6">
          <div class="card">
            <div class="text-sm text-zinc-400 mb-3">Create Tournament Time-slot</div>
            <div class="grid gap-3">
              <label>Game
                <select id="a_title" class="mt-1 p-2 rounded bg-black/40 border border-white/6">
                  <option>BGMI</option><option>Free Fire</option><option>PUBG</option>
                </select>
              </label>
              <label>Name
                <input id="a_name" placeholder="e.g. BGMI Night Rush" class="mt-1 p-2 rounded bg-black/40 border border-white/6" />
              </label>
              <div class="grid grid-cols-2 gap-2">
                <label>Entry Fee (₹)
                  <input id="a_fee" type="number" value="10" class="mt-1 p-2 rounded bg-black/40 border border-white/6" />
                </label>
                <label>Max Players
                  <input id="a_max" type="number" value="100" class="mt-1 p-2 rounded bg-black/40 border border-white/6" />
                </label>
              </div>
              <label>Start At (local)
                <input id="a_start" type="datetime-local" class="mt-1 p-2 rounded bg-black/40 border border-white/6" />
              </label>
              <label>Match Duration (minutes)
                <input id="a_duration" type="number" value="60" class="mt-1 p-2 rounded bg-black/40 border border-white/6" />
              </label>
              <div class="flex gap-2">
                <button id="a_publish" class="btn">Publish Slot</button>
              </div>
              <div class="text-xs text-zinc-500 mt-2">You can create multiple time-slots for same game (e.g., 8:00, 10:00). Rooms are created per slot.</div>
            </div>
          </div>

          <div class="card">
            <div class="text-sm text-zinc-400 mb-3">Control Room (simulate)</div>
            <select id="sel_t" class="w-full p-2 bg-black/40 border border-white/6 rounded mb-3">
              <option value="">Select Slot</option>
              ${opts}
            </select>
            <div class="flex gap-2">
              <button id="startBtn" class="btn">Start (simulate)</button>
              <button id="finishBtn" class="btn">Finish & Score</button>
            </div>
            <div class="text-xs text-zinc-500 mt-2">Finish will generate sample results and set rooms to be deleted 24 hours after match end.</div>
          </div>
        </section>
      `;

      // wire admin events
      $('#a_start').value = new Date(Date.now() + 60*60*1000).toISOString().slice(0,16);
      $('#a_publish').onclick = () => {
        const id = 't_' + Math.random().toString(36).slice(2,8);
        const t = makeTournament(id, $('#a_name').value || `${$('#a_title').value} Custom`, $('#a_title').value, Math.max(0,Number($('#a_fee').value)), Math.max(2,Number($('#a_max').value)), new Date($('#a_start').value).getTime(), Math.max(5,Number($('#a_duration').value))*60*1000);
        state.tournaments.push(t); DB.save('sr_tournaments', state.tournaments); alert('Slot published'); render();
      };
      $('#startBtn').onclick = () => {
        const id = $('#sel_t').value; if (!id) return alert('Select slot');
        const t = state.tournaments.find(x=>x.id===id); if (!t) return alert('Not found');
        t.status = 'live'; t.startAt = Date.now(); DB.save('sr_tournaments', state.tournaments); alert('Slot started'); render();
      };
      $('#finishBtn').onclick = () => {
        const id = $('#sel_t').value; if (!id) return alert('Select slot');
        const t = state.tournaments.find(x=>x.id===id); if (!t) return alert('Not found');
        // generate results for entries (simulate)
        t.results = t.entries.map(e => {
          const kills = Math.floor(Math.random()*12);
          const placement = Math.max(1, Math.floor(Math.random()*t.entries.length));
          const points = Math.max(0, 100 - placement) + kills*2;
          // push to global results store
          state.results.push({ tournamentId: t.id, gamingId: e.gamingId, kills, placement, points, ts: now() });
          return { gamingId: e.gamingId, kills, placement, points };
        });
        // compute deleteAt for rooms (24h after end)
        const matchEnd = (t.startAt || Date.now()) + (t.matchDurationMs || 3600000);
        for (const rKey of Object.keys(t.rooms || {})) {
          t.rooms[rKey].deleteAt = matchEnd + 24*3600*1000;
        }
        t.status = 'finished';
        DB.save('sr_tournaments', state.tournaments);
        DB.save('sr_results', state.results);
        alert('Finished & results generated. Rooms scheduled for deletion 24h after match end.');
        render();
      };
    } else if (currentView.startsWith('room:')) {
      // not used here
      app.innerHTML = `<div>Room view</div>`;
    }
  }

  // --- Tournament details + join modal + room panel ---
  function viewTournament(tId) {
    const t = state.tournaments.find(x=>x.id===tId);
    if (!t) return alert('Not found');
    // render details & available actions including specific time-slots (in this model each tournament is one slot)
    const entries = t.entries || [];
    const joined = state.user && entries.some(e => e.uid === state.user.uid);
    const startLabel = new Date(t.startAt).toLocaleString();
    $('#app').innerHTML = `
      <section>
        <div class="flex items-start gap-6">
          <div class="flex-1 card">
            <div class="flex justify-between items-start">
              <div>
                <div class="badge">${escapeHtml(t.title)}</div>
                <div class="text-2xl font-bold mt-2">${escapeHtml(t.name)}</div>
                <div class="text-zinc-400 mt-1">Start: <span class="mono">${startLabel}</span></div>
                <div class="text-zinc-400 text-sm">${entries.length}/${t.maxPlayers} joined</div>
              </div>
              <div class="text-right">
                <div class="text-zinc-400">Fee</div>
                <div class="text-3xl font-extrabold text-amber-400">₹${t.fee}</div>
                <div class="mt-3">
                  ${joined ? '<button class="btn" onclick="enterRoom(\''+t.id+'\')">Enter Room</button>' : '<button class="btn" onclick="openJoinModal(\\''+t.id+'\\')">Join (Pay)</button>'}
                </div>
              </div>
            </div>

            <div class="mt-6">
              <h3 class="font-semibold mb-2">Players</h3>
              <div class="grid gap-2">${entries.map(e => `<div class="p-2 rounded bg-white/3 flex justify-between"><div><b>${escapeHtml(e.name)}</b> • <span class="mono">${escapeHtml(e.gamingId)}</span></div><div class="text-zinc-400 text-xs">${new Date(e.paidAt).toLocaleString()}</div></div>`).join('')}</div>
            </div>
          </div>

          <div class="w-96">
            <div class="card">
              <div class="font-semibold mb-2">Slot Info</div>
              <div class="text-sm text-zinc-400">This slot starts at <b>${startLabel}</b> and lasts ${Math.round((t.matchDurationMs||3600000)/60000)} minutes.</div>
              <div class="mt-3 text-xs text-zinc-500">Rooms are created per-slot and locked to the participant's gaming ID. Rooms are retained 24 hours after match end.</div>
              ${t.status === 'finished' && t.results && t.results.length ? `<div class="mt-3"><h4 class="font-semibold">Match Results</h4>${t.results.map(r => `<div class="p-2 rounded bg-white/2 mt-2"><div class="mono">${escapeHtml(r.gamingId)}</div><div class="text-sm">Kills: ${r.kills} • Placement: ${r.placement} • Points: ${r.points}</div></div>`).join('')}</div>` : ''}
            </div>
          </div>
        </div>
      </section>
    `;
  }

  // --- Join / payment modal (simulated) ---
  function openJoinModal(tId) {
    const t = state.tournaments.find(x => x.id === tId);
    if (!t) return alert('Not found');
    if (!state.user) return alert('Please login first');

    // show an in-page modal (simple)
    const modal = document.createElement('div');
    modal.className = 'fixed inset-0 z-60 grid place-items-center';
    modal.style.background = 'linear-gradient(0deg, rgba(0,0,0,0.6), rgba(0,0,0,0.6))';
    modal.innerHTML = `
      <div class="w-full max-w-xl card">
        <div class="flex justify-between items-start">
          <div>
            <div class="text-xl font-bold">${escapeHtml(t.name)}</div>
            <div class="text-zinc-400 text-sm">${escapeHtml(t.title)} • Starts ${new Date(t.startAt).toLocaleString()}</div>
          </div>
          <button id="closeModal" class="btn">Close</button>
        </div>

        <div class="mt-4 grid gap-3">
          <div class="text-sm">Entry Fee: <span class="font-semibold text-amber-400">₹${t.fee}</span></div>
          <div>
            <label class="text-sm">Choose your gaming ID (locked to room)</label>
            <input id="pay_gamingId" class="mt-1 p-2 rounded bg-black/40 border border-white/6 mono" value="${escapeHtml(state.user.gamingId)}" />
            <div class="text-xs text-zinc-400 mt-1">This gaming ID will be locked to the room and cannot be transferred later.</div>
          </div>

          <div class="mt-2">
            <div class="text-sm font-semibold mb-2">Payment (Demo) - UPI</div>
            <div class="text-xs text-zinc-400">This demo simulates UPI payment. To accept real UPI payments you must integrate Razorpay (or other gateway) with a backend. See integration notes at bottom of this page.</div>

            <div class="mt-2 grid grid-cols-2 gap-2">
              <button id="payUpi" class="btn">Pay via UPI (Simulate)</button>
              <button id="payOffline" class="btn">Mark Paid (Admin / Offline)</button>
            </div>
          </div>
        </div>
      </div>
    `;
    document.body.appendChild(modal);
    modal.querySelector('#closeModal').onclick = () => modal.remove();

    // simulate payment flow
    modal.querySelector('#payUpi').onclick = async () => {
      // Simple simulated UPI flow:
      if (!confirm(`Simulate paying ₹${t.fee} via UPI? (This is a demo payment; to accept real UPI payments integrate a backend.)`)) return;
      const gamingId = modal.querySelector('#pay_gamingId').value.trim() || state.user.gamingId;
      // create entry & assign room
      t.entries.push({ uid: state.user.uid, name: state.user.name, gamingId, paidAt: now(), paymentMeta: { method: 'UPI-DEMO', ref: 'SIM-'+Math.random().toString(36).slice(2,8) } });
      // create room for that slot (slotStartAt = t.startAt here)
      const room = roomForTournamentAndGamingId(t, gamingId, t.startAt);
      DB.save('sr_tournaments', state.tournaments);
      modal.remove();
      alert(`Payment simulated success.\nRoom assigned:\nID: ${room.roomId}\nPass: ${room.roomPass}\n(This room is locked to ${gamingId} and will be kept until ${new Date(room.deleteAt).toLocaleString()})`);
      render();
    };

    // admin/offline mark paid (for local testing)
    modal.querySelector('#payOffline').onclick = () => {
      const gamingId = modal.querySelector('#pay_gamingId').value.trim() || state.user.gamingId;
      t.entries.push({ uid: state.user.uid, name: state.user.name, gamingId, paidAt: now(), paymentMeta: { method: 'OFFLINE', ref: 'OFF-'+Math.random().toString(36).slice(2,6) } });
      const room = roomForTournamentAndGamingId(t, gamingId, t.startAt);
      DB.save('sr_tournaments', state.tournaments);
      modal.remove();
      alert(`Marked paid (offline). Room ID: ${room.roomId}, Pass: ${room.roomPass}`);
      render();
    };
  }

  // --- Enter room (shows room details, join link placeholder, chat) ---
  function enterRoom(tId) {
    if (!state.user) return alert('Login first');
    const t = state.tournaments.find(x => x.id === tId);
    if (!t) return alert('Not found');
    // find user's entry for this slot
    const entry = (t.entries || []).find(e => e.uid === state.user.uid);
    if (!entry) return alert('You must pay to enter this room.');
    // find / create room for this gamingId and slotStartAt
    const room = roomForTournamentAndGamingId(t, entry.gamingId, t.startAt);

    // render room UI
    $('#app').innerHTML = `
      <section>
        <div class="grid md:grid-cols-2 gap-6">
          <div class="card">
            <div class="flex items-start justify-between">
              <div>
                <div class="text-sm text-zinc-400">Room (locked to gaming ID)</div>
                <div class="text-2xl font-bold mono">${escapeHtml(room.roomId)}</div>
                <div class="text-zinc-400 mt-1">Pass: <span class="mono">${escapeHtml(room.roomPass)}</span></div>
                <div class="mt-3 text-sm text-zinc-400">Slot: ${new Date(room.slotStartAt).toLocaleString()}</div>
                <div class="mt-4">
                  <div class="text-xs text-zinc-400">Join Link (deployed by admin/game organizer):</div>
                  <div class="mt-2 p-3 bg-white/3 rounded mono">https://play.shadowreapers.fake/join/${room.roomId}</div>
                  <div class="text-xs text-zinc-400 mt-2">This is a placeholder link for the match. Replace with actual deep-link / lobby link when integrating with game/organizer.</div>
                </div>
              </div>
              <div class="text-right">
                <div class="text-sm text-zinc-400">Participant</div>
                <div class="font-semibold">${escapeHtml(state.user.name)}</div>
                <div class="text-zinc-400 mono">${escapeHtml(state.user.gamingId)}</div>
              </div>
            </div>

            <div class="mt-4 text-xs text-zinc-400">Room will be deleted 24 hours after match end (<span class="mono">${new Date(room.deleteAt).toLocaleString()}</span>).</div>
          </div>

          <div class="card">
            <div class="font-semibold">Room Chat</div>
            <div id="chatBox" class="mt-3 p-3 bg-black/40 rounded h-56 overflow-auto"></div>
            <div class="mt-3 flex gap-2">
              <input id="chatIn" class="flex-1 p-2 rounded bg-black/40 mono" placeholder="Message..." />
              <button id="sendChat" class="btn">Send</button>
            </div>
          </div>
        </div>
      </section>
    `;

    // chat uses localStorage channel keyed by room.roomId
    const chatKey = `sr_chat_${room.roomId}`;
    const chatBox = $('#chatBox');

    function renderChat() {
      const msgs = DB.load(chatKey, []);
      chatBox.innerHTML = msgs.map(m => `<div class="mb-2"><div class="text-xs text-zinc-400">${new Date(m.ts).toLocaleTimeString()}</div><div><b>${escapeHtml(m.name)}</b>: ${escapeHtml(m.text)}</div></div>`).join('');
      chatBox.scrollTop = chatBox.scrollHeight;
    }
    renderChat();

    // send handler
    $('#sendChat').onclick = () => {
      const txt = $('#chatIn').value.trim(); if (!txt) return;
      const msgs = DB.load(chatKey, []);
      msgs.push({ uid: state.user.uid, name: state.user.name, text: txt, ts: now() });
      DB.save(chatKey, msgs);
      $('#chatIn').value = '';
      renderChat();
      // also trigger an event across other tabs by writing a timestamp key
      localStorage.setItem(chatKey + '_ping', Date.now());
    };

    // respond to storage events (so other tabs can see new chat)
    window.addEventListener('storage', (ev) => {
      if (ev.key === (chatKey + '_ping')) renderChat();
    });
  }

  // utility: open room if joined (used in tournament listing)
  function enterRoomOrAlert(tId) {
    const t = state.tournaments.find(x => x.id === tId);
    if (!t) return alert('not found');
    const e = t.entries.find(entry => entry.uid === (state.user && state.user.uid));
    if (!e) return alert('You need to pay to enter the room.');
    enterRoom(tId);
  }

  // --- small helpers for demo / dev ---
  // expose some functions to global scope for in-page buttons
  window.switchView = switchView;
  window.viewTournament = viewTournament;
  window.openJoinModal = openJoinModal;
  window.enterRoom = enterRoom;
  window.enterRoomOrAlert = enterRoomOrAlert;

  // --- initial view & render ---
  render();
  renderAuth();

  /*************************************************************************
   * Integration notes: How to replace demo payment with real Razorpay UPI
   *
   * 1) WHY A BACKEND? Razorpay requires your secret key to create orders & verify
   *    signatures. You MUST NOT put the secret in client-side JS (it would be public).
   *
   * 2) Backend endpoints (example):
   *    POST /create-order
   *      body: { tournamentId, userId, amountPaise, meta }
   *      server: calls razorpay.orders.create({ amount, currency: 'INR', receipt: ... })
   *      returns: { orderId, amount, razorpayKeyId }
   *
   *    POST /verify-payment
   *      body: { razorpay_payment_id, razorpay_order_id, razorpay_signature, tournamentId, userId, slotStartAt }
   *      server: verify HMAC(signature) using RP secret, then mark Firestore entries & create room for gamingId
   *
   *    Webhooks: configure Razorpay webhook for payment.captured to also mark payments server-side.
   *
   * 3) Frontend changes:
   *    - On Join, call /create-order to get orderId & keyId
   *    - Initialize Razorpay Checkout (client-side) with order_id and keyId. Razorpay UI includes UPI methods.
   *    - After successful checkout, call /verify-payment to validate the signature and finalize DB entries.
   *
   * 4) Room creation & deleteAt:
   *    - Your server should compute deleteAt = slotStartAt + matchDuration + 24h, store it in DB, and run a scheduled cleanup (cron or cloud function) to remove rooms after deleteAt OR use Firestore TTL / Cloud Tasks.
   *
   * 5) Chat & Realtime:
   *    - For production replace localStorage chat with Firebase Realtime DB or a socket server so chat works across users/devices.
   *
   * 6) Deployment hints:
   *    - Frontend: GitHub Pages / Netlify / Vercel (static)
   *    - Backend: Render / Railway / Heroku / Firebase Functions
   *    - Database: Firebase (Firestore + Realtime DB) or MongoDB + Redis for realtime
   *
   * This single-file demo lets you prototype UI, room logic, and flows. When you're ready I can:
   *  - provide the backend Express code for Razorpay order creation & verification
   *  - convert chat to Firebase example
   *  - produce a zip repo ready to deploy (frontend + backend).
   *************************************************************************/

  </script>
</body>
</html>
