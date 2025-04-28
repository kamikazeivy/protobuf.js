protobuf.js version: <please fill in>

<please describe the expected and actual behavior>

```js
<please provide a code snippet for reproduction>
```

```
<please paste the stack trace of the error if applicable>
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Stream Boss</title>
  <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: 'Press Start 2P', monospace;
      background: transparent;
      color: white;
      display: flex;
      align-items: center;
      gap: 16px;
      padding: 20px;
      position: relative;
    }

    .stream-buddy {
      width: 120px;
      height: 120px;
      border-radius: 16px;
      background: red;
      animation: pulse 3s infinite;
    }

    @keyframes pulse {
      0% { box-shadow: 0 0 5px red; }
      50% { box-shadow: 0 0 20px red; }
      100% { box-shadow: 0 0 5px red; }
    }

    .hp-container {
      flex-grow: 1;
    }

    .username {
      font-size: 12px;
      color: red;
      margin-bottom: 8px;
    }

    .hp-bar-bg {
      background: black;
      border: 2px solid white;
      height: 24px;
      width: 100%;
      border-radius: 6px;
      overflow: hidden;
      position: relative;
    }

    .hp-bar-fill {
      height: 100%;
      width: 100%;
      background: green;
      transition: width 0.3s ease;
    }

    .hit-effect {
      animation: hitFlash 0.2s;
    }

    @keyframes hitFlash {
      0% { filter: brightness(2); }
      100% { filter: brightness(1); }
    }

    .leaderboard {
      position: absolute;
      top: 20px;
      right: 20px;
      background: rgba(0, 0, 0, 0.8);
      border: 2px solid white;
      padding: 10px;
      font-size: 10px;
    }
  </style>
</head>
<body>
  <div class="stream-buddy" id="streamBuddy"></div>
  <div class="hp-container">
    <div class="username" id="bossName">???</div>
    <div class="hp-bar-bg">
      <div class="hp-bar-fill" id="hpBar"></div>
    </div>
  </div>
  <div class="leaderboard" id="leaderboard">
    <div>Top Damage:</div>
    <ol id="topUsers"></ol>
  </div>

  <audio id="hitSound" src="hit.mp3"></audio>
  <audio id="killSound" src="kill.mp3"></audio>

  <script>
    let boss = {
      name: "???",
      hp: 2000,
      maxHp: 2000
    };

    const damageBy = { coin: 1, like: 1, share: 10 };
    const healBy = { coin: 2, like: 2, share: 20 };
    const hpBar = document.getElementById("hpBar");
    const bossName = document.getElementById("bossName");
    const leaderboard = document.getElementById("topUsers");
    const hitSound = document.getElementById("hitSound");
    const killSound = document.getElementById("killSound");
    const streamBuddy = document.getElementById("streamBuddy");

    let topDamage = {};
    const buddyColors = ['red', 'blue', 'purple', 'orange', 'cyan', 'lime'];

    function updateHPBar() {
      let pct = (boss.hp / boss.maxHp) * 100;
      hpBar.style.width = pct + "%";
      if (pct < 30) hpBar.style.background = "red";
      else if (pct < 60) hpBar.style.background = "orange";
      else hpBar.style.background = "green";
    }

    function updateLeaderboard() {
      let sorted = Object.entries(topDamage).sort((a,b) => b[1] - a[1]).slice(0, 3);
      leaderboard.innerHTML = '<div>Top Damage:</div>' + sorted.map(([user, dmg]) => `<li>${user}: ${dmg}</li>`).join("");
    }

    function setBuddyColor() {
      const color = buddyColors[Math.floor(Math.random() * buddyColors.length)];
      streamBuddy.style.background = color;
    }

    function handleEvent({ user, type, amount, isBoss }) {
      let effect = isBoss ? healBy[type] * amount : damageBy[type] * amount;
      if (!isBoss) {
        boss.hp -= effect;
        if (!topDamage[user]) topDamage[user] = 0;
        topDamage[user] += effect;
        hitSound.play();
      } else {
        boss.hp += effect;
      }

      if (boss.hp > boss.maxHp) boss.hp = boss.maxHp;

      hpBar.classList.remove("hit-effect");
      void hpBar.offsetWidth;
      hpBar.classList.add("hit-effect");

      if (boss.hp <= 0) {
        killSound.play();
        boss.name = user;
        boss.hp = boss.maxHp;
        topDamage = {};
        setBuddyColor();
      }

      bossName.textContent = boss.name;
      updateHPBar();
      updateLeaderboard();
    }

    // WebSocket for TikFinity or Streamer.bot
    const socket = new WebSocket('ws://localhost:8080'); // Replace with your WebSocket endpoint

    socket.onmessage = function (event) {
      try {
        const data = JSON.parse(event.data);
        if (data.type === 'coin' || data.type === 'like' || data.type === 'share') {
          handleEvent({
            user: data.user || 'Viewer',
            type: data.type,
            amount: data.amount || 1,
            isBoss: data.user === boss.name
          });
        }
      } catch (err) {
        console.error('Invalid event data', err);
      }
    };

    window.onload = () => {
      setBuddyColor();
      updateHPBar();
    };
  </script>
</body>
</html>

