<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Shadow Reapers — Esports Arena</title>
<link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&display=swap" rel="stylesheet">
<style>
  body {
    margin: 0;
    font-family: 'Orbitron', sans-serif;
    background: #000;
    color: white;
    overflow-x: hidden;
  }

  /* === Intro Animation === */
  #intro {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background: radial-gradient(circle at center, #05050A, #000);
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    z-index: 9999;
  }

  .dynamic-logo {
    width: 140px;
    height: 140px;
    border-radius: 50%;
    border: 4px solid #00eaff;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2.5rem;
    font-weight: bold;
    color: #fff;
    text-shadow: 0 0 20px #00eaff, 0 0 40px #ff00ff;
    box-shadow: 0 0 25px #00eaff, 0 0 50px #ff00ff inset;
    opacity: 0;
    transform: scale(0);
    animation: popIn 1s ease forwards;
  }

  .game-title {
    margin-top: 20px;
    font-size: 2.5rem;
    color: #00eaff;
    text-shadow: 0 0 20px #00eaff, 0 0 40px #ff00ff;
    opacity: 0;
    transform: scale(0.5);
    animation: popIn 1s ease forwards;
    animation-delay: 0.5s;
  }

  @keyframes popIn {
    to { opacity: 1; transform: scale(1); }
  }

  @keyframes slideUp {
    to { opacity: 0; transform: translateY(-100%); }
  }

  #homepage {
    display: none;
    text-align: center;
    padding: 50px;
  }
</style>
</head>
<body>

<!-- ==== Intro Animation === -->
<div id="intro">
  <div class="dynamic-logo">SR</div>
  <h1 class="game-title">Shadow Reapers</h1>
  <audio id="bgMusic" loop autoplay>
    <source src="https://files.freemusicarchive.org/storage-freemusicarchive-org/music/no_curator/Komiku/Final_Boss/Komiku_-_06_-_Battle_of_Pogs.mp3" type="audio/mpeg">
  </audio>
</div>

<!-- ==== Homepage === -->
<div id="homepage">
  <h2>Welcome to Shadow Reapers Arena</h2>
  <p>Custom tournaments, live leaderboards, and exclusive rooms!</p>
</div>

<script>
  // Auto-start music on page load
  const music = document.getElementById('bgMusic');
  music.volume = 0.4; // set comfortable volume
  music.play().catch(e => console.log("Autoplay blocked: ", e));

  // Intro animation sequence
  setTimeout(() => {
    const intro = document.getElementById('intro');
    intro.style.animation = 'slideUp 1s ease forwards';
    setTimeout(() => {
      intro.style.display = 'none';
      document.getElementById('homepage').style.display = 'block';
    }, 1000);
  }, 3000); // 2s pop-in + 1s hold
</script>

</body>
</html>
