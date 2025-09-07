<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Shadow Reapers</title>
  <style>
    /* ==== Global Dark Gamer Theme ==== */
    body, html {
      margin: 0;
      padding: 0;
      font-family: 'Orbitron', sans-serif;
      background: #0a0a0a;
      color: #fff;
      overflow-x: hidden;
    }
    a { color: cyan; text-decoration: none; }
    button {
      cursor: pointer;
      border: none;
      border-radius: 8px;
      padding: 10px 20px;
      background: linear-gradient(45deg, #ff004c, #6e00ff);
      color: #fff;
      font-weight: bold;
      transition: 0.3s;
    }
    button:hover { transform: scale(1.05); }
    header, nav {
      display: flex; justify-content: space-between; align-items: center;
      padding: 10px 20px;
      background: rgba(0,0,0,0.8);
      border-bottom: 1px solid #222;
    }
    nav a { margin: 0 10px; }
    .hidden { display: none; }

    /* ==== Intro Animation ==== */
    #intro {
      position: fixed; top:0; left:0; right:0; bottom:0;
      background: black;
      display:flex; flex-direction:column; align-items:center; justify-content:center;
      z-index: 9999;
      animation: fadeIn 2s ease;
    }
    #intro h1 {
      font-size: 3rem;
      color: #ff004c;
      text-shadow: 0 0 20px #ff004c, 0 0 40px #6e00ff;
      animation: glow 2s infinite alternate;
    }
    @keyframes glow {
      from { text-shadow: 0 0 10px #ff004c; }
      to { text-shadow: 0 0 30px #6e00ff; }
    }
    @keyframes fadeIn {
      from { opacity: 0; }
      to { opacity: 1; }
    }

    /* ==== Sections ==== */
    .section { padding: 20px; }
    #leaderboard img, #profile img { width:60px; height:60px; border-radius:50%; }

    /* ==== Chat ==== */
    #chatBox {
      border:1px solid #333;
      height:200px;
      overflow-y:scroll;
      padding:10px;
      margin-bottom:10px;
      background:#111;
    }
    #chatBox p { margin:5px 0; }
  </style>
</head>
<body>
  <!-- ==== Intro Animation ==== -->
  <div id="intro">
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/59/Skull_icon.svg/1024px-Skull_icon.svg.png" width="120" alt="Skull"/>
    <h1>Welcome to Shadow Reapers</h1>
    <button onclick="enterArena()">Enter Arena</button>
    <br>
    <audio id="bgMusic" loop>
      <source src="https://www.computerhope.com/jargon/m/example.mp3" type="audio/mpeg">
    </audio>
    <button onclick="toggleMusic()">🎵 Toggle Music</button>
  </div>

  <!-- ==== Main Website ==== -->
  <header id="mainHeader" class="hidden">
    <h2>☠️ Shadow Reapers</h2>
    <nav>
      <a href="#" onclick="showSection('home')">Home</a>
      <a href="#" onclick="showSection('profile')">My Profile</a>
      <a href="#" onclick="showSection('leaderboard')">Leaderboard</a>
      <a href="#" onclick="showSection('tournaments')">Tournaments</a>
      <a href="#" onclick="logout()">Logout</a>
    </nav>
  </header>

  <main>
    <!-- ==== Home ==== -->
    <div id="home" class="section hidden">
      <h2>Welcome Warrior!</h2>
      <p>Choose your path. Enter tournaments. Prove your skills. ☠️</p>
    </div>

    <!-- ==== Profile ==== -->
    <div id="profile" class="section hidden">
      <h2>My Profile</h2>
      <img id="avatarImg" src="" alt="Avatar"/>
      <p id="username"></p>
      <p>Free Avatar Changes Left: <span id="avatarFreeCount"></span></p>
      <button onclick="changeAvatar()">Change Avatar</button>
      <h3>Stats</h3>
      <p>Matches: <span id="statMatches">0</span></p>
      <p>Kills: <span id="statKills">0</span></p>
      <p>Wins: <span id="statWins">0</span></p>
      <p>Badges: <span id="badgeList">None</span></p>
    </div>

    <!-- ==== Leaderboard ==== -->
    <div id="leaderboard" class="section hidden">
      <h2>Global Top 10</h2>
      <div id="leaderboardList"></div>
    </div>

    <!-- ==== Tournaments ==== -->
    <div id="tournaments" class="section hidden">
      <h2>Available Tournaments</h2>
      <div id="tournamentList"></div>
    </div>

    <!-- ==== Room Chat ==== -->
    <div id="room" class="section hidden">
      <h2 id="roomTitle"></h2>
      <div id="chatBox"></div>
      <input type="text" id="chatInput" placeholder="Type message..."/>
      <button onclick="sendMessage()">Send</button>
    </div>
  </main>

  <!-- ==== Firebase + Razorpay ==== -->
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore.js"></script>
  <script src="https://checkout.razorpay.com/v1/checkout.js"></script>

  <script>
    /* ==== Firebase Setup ==== */
    // TODO: Paste your Firebase Config here
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT.firebaseapp.com",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT.appspot.com",
      messagingSenderId: "SENDER_ID",
      appId: "APP_ID"
    };
    const app = firebase.initializeApp(firebaseConfig);
    const auth = firebase.getAuth(app);
    const db = firebase.getFirestore(app);

    // TODO: Paste your Razorpay Key ID here
    const razorpayKey = "rzp_test_XXXX";

    /* ==== Intro ==== */
    function enterArena(){
      document.getElementById('intro').style.display="none";
      document.getElementById('mainHeader').classList.remove("hidden");
      showSection('home');
    }
    function toggleMusic(){
      let music = document.getElementById("bgMusic");
      if(music.paused){ music.play(); }
      else { music.pause(); }
    }

    /* ==== Navigation ==== */
    function showSection(id){
      document.querySelectorAll("main .section").forEach(sec => sec.classList.add("hidden"));
      document.getElementById(id).classList.remove("hidden");
    }

    /* ==== Avatar ==== */
    let avatarChanges = 0;
    function changeAvatar(){
      if(avatarChanges < 10){
        uploadAvatar();
        avatarChanges++;
      } else {
        // Require Razorpay Payment
        var options = {
          "key": razorpayKey,
          "amount": 1000, // Rs.10
          "currency": "INR",
          "name": "Shadow Reapers",
          "description": "Avatar Change",
          "handler": function (response){
            uploadAvatar();
          }
        };
        var rzp1 = new Razorpay(options);
        rzp1.open();
      }
      document.getElementById("avatarFreeCount").innerText = Math.max(0, 10 - avatarChanges);
    }
    function uploadAvatar(){
      let url = prompt("Enter Avatar Image URL:");
      if(url){
        document.getElementById("avatarImg").src = url;
      }
    }

    /* ==== Leaderboard (Demo Data) ==== */
    const leaderboardData = [
      {name:"Player1", kills:25, wins:5, avatar:"https://i.pravatar.cc/60?img=1"},
      {name:"Player2", kills:20, wins:3, avatar:"https://i.pravatar.cc/60?img=2"},
      {name:"Player3", kills:15, wins:2, avatar:"https://i.pravatar.cc/60?img=3"},
    ];
    function loadLeaderboard(){
      let list = document.getElementById("leaderboardList");
      list.innerHTML = "";
      leaderboardData.forEach(p=>{
        let div = document.createElement("div");
        div.innerHTML = `<img src="${p.avatar}"/> ${p.name} - Kills:${p.kills}, Wins:${p.wins}`;
        list.appendChild(div);
      });
    }
    loadLeaderboard();

    /* ==== Tournaments (Demo) ==== */
    const tournaments = [
      {id:1, game:"Free Fire", time:"8:00 AM"},
      {id:2, game:"BGMI", time:"10:00 AM"},
    ];
    function loadTournaments(){
      let list = document.getElementById("tournamentList");
      list.innerHTML="";
      tournaments.forEach(t=>{
        let div = document.createElement("div");
        div.innerHTML = `${t.game} @ ${t.time} <button onclick="joinTournament(${t.id})">Join</button>`;
        list.appendChild(div);
      });
    }
    loadTournaments();

    function joinTournament(id){
      var options = {
        "key": razorpayKey,
        "amount": 5000, // Rs.50 Example fee
        "currency": "INR",
        "name": "Shadow Reapers",
        "description": "Tournament Entry",
        "handler": function (response){
          enterRoom(id);
        }
      };
      var rzp1 = new Razorpay(options);
      rzp1.open();
    }

    /* ==== Room + Chat ==== */
    function enterRoom(id){
      showSection("room");
      document.getElementById("roomTitle").innerText = "Tournament Room #" + id;
    }
    function sendMessage(){
      let msg = document.getElementById("chatInput").value;
      if(!msg) return;
      let box = document.getElementById("chatBox");
      let p = document.createElement("p");
      p.innerText = "You: " + msg;
      box.appendChild(p);
      document.getElementById("chatInput").value="";
    }

    function logout(){
      alert("Logged out (demo). Integrate Firebase Auth here.");
    }
  </script>
</body>
</html>
