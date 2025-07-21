<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Tabletop Dice Roller</title>
  <style>
    :root {
      --bg-color: #f4f4f4;
      --text-color: #000;
      --card-bg: #fff;
      --border-color: #ccc;
      --highlight-crit: green;
      --highlight-fail: red;
    }

    .dark-mode {
      --bg-color: #121212;
      --text-color: #eee;
      --card-bg: #1f1f1f;
      --border-color: #444;
      --highlight-crit: #00e676;
      --highlight-fail: #ff5252;
    }

    body {
      font-family: sans-serif;
      text-align: center;
      padding: 2rem;
      background-color: var(--bg-color);
      color: var(--text-color);
      transition: background 0.3s, color 0.3s;
    }

    input, select, button {
      margin: 0.5rem;
      padding: 0.5rem;
    }

    #results, #stats {
      margin-top: 1rem;
      font-weight: bold;
    }

    #stats, #log {
      max-width: 600px;
      margin: 1rem auto;
      background: var(--card-bg);
      padding: 1rem;
      border: 1px solid var(--border-color);
      border-radius: 5px;
    }

    #log {
      text-align: left;
      max-height: 300px;
      overflow-y: auto;
    }

    .log-entry {
      border-bottom: 1px solid var(--border-color);
      padding: 0.5rem 0;
    }

    .crit {
      color: var(--highlight-crit);
      font-weight: bold;
    }

    .fail {
      color: var(--highlight-fail);
      font-weight: bold;
    }

    .toggle-btn {
      position: absolute;
      top: 1rem;
      right: 1rem;
      padding: 0.5rem 1rem;
    }

    .controls {
      margin-top: 1rem;
    }
  </style>
</head>
<body>
  <button class="toggle-btn" onclick="toggleDarkMode()">(Grim)Dark Mode</button>

  <h1>Tabletop Dice Roller</h1>

  <label>Number of Dice: 
    <input type="number" id="numDice" value="5" min="1" max="100"/>
  </label>

  <label>Dice Type:
    <select id="diceType">
      <option value="3">d3</option>
      <option value="4">d4</option>
      <option value="6" selected>d6</option>
      <option value="8">d8</option>
      <option value="10">d10</option>
      <option value="12">d12</option>
      <option value="20">d20</option>
      <option value="100">d100</option>
    </select>
  </label>

  <label>Modifier: 
    <input type="number" id="modifier" value="0"/>
  </label>

  <label>Target Value (≥): 
    <input type="number" id="target" value="4" min="1"/>
  </label>

  <button onclick="rollDice()">Roll!</button>
  <button onclick="toggleStats()"> Show/Hide Stats</button>

  <div id="results"></div>

  <div id="stats-container" style="display: none;">
    <div id="stats">
      <h2>Session Stats</h2>
      <p>Total Dice Rolled: <span id="totalRolls">0</span></p>
      <p>Total Successes (≥ target): <span id="totalSuccesses">0</span></p>
      <p>Total Crits (Max Rolls): <span id="totalCrits">0</span></p>
      <p>Total Fails (Rolls of 1): <span id="totalFails">0</span></p>
    </div>
  </div>

  <div class="controls">
    <button onclick="resetStats()">Reset Stats</button>
    <button onclick="exportText()">Export as Text</button>
    <button onclick="exportCSV()">Export as CSV</button>
  </div>

  <h2>Roll History</h2>
  <div id="log"></div>

  <footer style="margin-top: 2rem; font-size: 0.9rem; color: gray;">
    Created by Nathan Hopwood-Curry – <a href="mailto:nathan.f.hopwood@gmail.com" style="color: inherit; text-decoration: underline;">nathan.f.hopwood@gmail.com</a>
  </footer>

  <script>
    let sessionTotalRolls = 0;
    let sessionSuccesses = 0;
    let sessionCrits = 0;
    let sessionFails = 0;
    let rollHistory = [];

    function rollDice() {
      const numDice = parseInt(document.getElementById('numDice').value);
      const diceSides = parseInt(document.getElementById('diceType').value);
      const modifier = parseInt(document.getElementById('modifier').value);
      const target = parseInt(document.getElementById('target').value);

      let rolls = [];
      let displayRolls = [];
      let total = 0;
      let successCount = 0;
      let critCount = 0;
      let failCount = 0;

      for (let i = 0; i < numDice; i++) {
        const roll = Math.floor(Math.random() * diceSides) + 1;
        rolls.push(roll);
        total += roll;

        if (roll === diceSides) {
          displayRolls.push(`<span class="crit">${roll}</span>`);
          critCount++;
        } else if (roll === 1) {
          displayRolls.push(`<span class="fail">${roll}</span>`);
          failCount++;
        } else {
          displayRolls.push(roll);
        }

        if (roll >= target) successCount++;
      }

      total += modifier;

      sessionTotalRolls += numDice;
      sessionCrits += critCount;
      sessionFails += failCount;
      sessionSuccesses += successCount;
      updateStatsDisplay();

      rollHistory.push({
        type: `${numDice}d${diceSides} ${modifier >= 0 ? '+' : ''}${modifier}`,
        rolls: rolls.join(', '),
        total,
        successes: successCount,
        crits: critCount,
        fails: failCount,
        target
      });

      document.getElementById('results').innerHTML = `
        Rolls: ${displayRolls.join(', ')}<br/>
        Modifier: ${modifier >= 0 ? '+' : ''}${modifier}<br/>
        Total: ${total}<br/>
        Rolls ≥ ${target}: ${successCount}<br/>
        Crits (Max Rolls): ${critCount}<br/>
        Fails (Rolls of 1): ${failCount}
      `;

      const logEntry = document.createElement('div');
      logEntry.classList.add('log-entry');
      logEntry.innerHTML = `
        <strong>${numDice}d${diceSides} ${modifier >= 0 ? '+' : ''}${modifier} (Target ≥ ${target})</strong><br/>
        Rolls: ${displayRolls.join(', ')} | Total: ${total} | Successes: ${successCount} | Crits: ${critCount} | Fails: ${failCount}
      `;
      document.getElementById('log').prepend(logEntry);
    }

    function updateStatsDisplay() {
      document.getElementById('totalRolls').textContent = sessionTotalRolls;
      document.getElementById('totalSuccesses').textContent = sessionSuccesses;
      document.getElementById('totalCrits').textContent = sessionCrits;
      document.getElementById('totalFails').textContent = sessionFails;
    }

    function resetStats() {
      sessionTotalRolls = 0;
      sessionSuccesses = 0;
      sessionCrits = 0;
      sessionFails = 0;
      rollHistory = [];
      updateStatsDisplay();
      document.getElementById('log').innerHTML = '';
      document.getElementById('results').innerHTML = '';
    }

    function exportText() {
      let text = `--- Dice Roller Session ---\n`;
      text += `Total Rolls: ${sessionTotalRolls}\nSuccesses: ${sessionSuccesses}\nCrits: ${sessionCrits}\nFails: ${sessionFails}\n\nRoll Log:\n`;

      rollHistory.forEach((entry, i) => {
        text += `${i + 1}) ${entry.type} (Target ≥ ${entry.target})\n`;
        text += `   Rolls: ${entry.rolls} | Total: ${entry.total} | Successes: ${entry.successes} | Crits: ${entry.crits} | Fails: ${entry.fails}\n`;
      });

      const blob = new Blob([text], { type: "text/plain" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "dice_rolls.txt";
      link.click();
    }

    function exportCSV() {
      let csv = `Roll,Type,Target,Rolls,Total,Successes,Crits,Fails\n`;
      rollHistory.forEach((entry, i) => {
        csv += `${i + 1},"${entry.type}",${entry.target},"${entry.rolls}",${entry.total},${entry.successes},${entry.crits},${entry.fails}\n`;
      });

      const blob = new Blob([csv], { type: "text/csv" });
      const link = document.createElement("a");
      link.href = URL.createObjectURL(blob);
      link.download = "dice_rolls.csv";
      link.click();
    }

    function toggleDarkMode() {
      document.body.classList.toggle("dark-mode");
      const isDark = document.body.classList.contains("dark-mode");
      localStorage.setItem("darkMode", isDark ? "true" : "false");
    }

    function toggleStats() {
      const statsContainer = document.getElementById("stats-container");
      if (statsContainer.style.display === "none") {
        statsContainer.style.display = "block";
      } else {
        statsContainer.style.display = "none";
      }
    }

    window.onload = () => {
      if (localStorage.getItem("darkMode") === "true") {
        document.body.classList.add("dark-mode");
      }
    };
  </script>
</body>
</html>
