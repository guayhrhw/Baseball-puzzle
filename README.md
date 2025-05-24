<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Baseball Lineup Logic with Bonus Level</title>
<style>
  body {
    font-family: Arial, sans-serif;
    background: #eef6ff url('https://upload.wikimedia.org/wikipedia/commons/8/8a/Baseball_Diamond_3d_animated.svg') no-repeat center top;
    background-size: 400px 400px;
    padding: 20px;
  }
  h1, h2 {
    text-align: center;
  }
  .meter {
    text-align: center;
    font-weight: bold;
    margin: 10px 0;
    font-size: 18px;
  }
  .clues {
    max-width: 600px;
    margin: 20px auto;
    background: #fff;
    padding: 15px;
    border-radius: 8px;
  }
  ul {
    list-style: none;
    padding: 0;
    max-width: 350px;
    margin: 10px auto;
    min-height: 50px;
    border: 2px dashed #aaa;
    border-radius: 6px;
    background: #f8f8f8;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
  }
  ul#bench, ul#injuredList {
    max-width: 350px;
    min-height: 70px;
    margin: 10px auto 30px auto;
  }
  li {
    padding: 8px 12px;
    margin: 5px;
    background: #fff;
    border: 2px solid #ccc;
    border-radius: 12px;
    cursor: grab;
    user-select: none;
    min-width: 90px;
    text-align: center;
    font-weight: 600;
  }
  li:active {
    cursor: grabbing;
  }
  button {
    display: block;
    margin: 20px auto;
    padding: 10px 24px;
    font-size: 16px;
    border-radius: 6px;
    background: #007bff;
    color: white;
    border: none;
    cursor: pointer;
  }
  button:hover {
    background: #0056b3;
  }
  .result {
    text-align: center;
    font-weight: bold;
    margin-top: 15px;
  }
  .section-title {
    text-align: center;
    font-weight: bold;
    margin-top: 20px;
  }
  .field {
    display: grid;
    grid-template-columns: repeat(3, 120px);
    justify-content: center;
    gap: 15px;
    margin-top: 20px;
  }
  .position {
    height: 50px;
    border: 2px dashed #555;
    border-radius: 12px;
    background: #fafafa;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    user-select: none;
  }
  #bench-title, #injured-title {
    max-width: 350px;
    margin: 30px auto 5px auto;
    font-weight: bold;
    text-align: center;
  }
  #levelSelector {
    margin: 0 auto 20px auto;
    display: block;
    font-size: 16px;
    padding: 6px 10px;
  }
</style>
</head>
<body>

<h1>Baseball Lineup Logic</h1>

<label for="levelSelector" style="display:block; text-align:center; font-weight:bold; margin-bottom:10px;">
  Select Level:
</label>
<select id="levelSelector">
  <option value="1">Level 1 - Seth Benched</option>
  <option value="bonus">Bonus Level - Seth Plays</option>
</select>

<div class="meter" id="sethMeter" style="display:none;">Seth Benched: <span id="sethCount">0</span> times</div>

<div class="clues">
  <h3>Clues:</h3>
  <ul id="clueList"></ul>
</div>

<div class="field" id="field"></div>

<div id="bench-title" class="section-title">🪑 Bench</div>
<ul id="bench"></ul>

<div id="injured-title" class="section-title" style="display:none;">🩹 Injured List</div>
<ul id="injuredList" style="display:none;"></ul>

<button id="checkBtn">Check Answer</button>
<p id="resultText" class="result"></p>

<script>
  // Common data for both levels
  const players = [
    "Logan", "Ava", "Mason", "Harper",
    "Lucas", "Isabella", "Elijah", "Mia", "Olivia",
    "Seth Rakestraw"
  ];

  const positions = ["P", "C", "1B", "2B", "SS", "3B", "LF", "CF", "RF"];

  // Level 1 Data
  const level1 = {
    correctAssignment: {
      "P": "Logan",
      "C": "Ava",
      "1B": "Mason",
      "2B": "Harper",
      "SS": "Lucas",
      "3B": "Isabella",
      "LF": "Elijah",
      "CF": "Mia",
      "RF": "Olivia"
    },
    clues: [
      "Ava is behind the plate.",
      "Harper plays next to Mason in the infield.",
      "Lucas anchors the shortstop position.",
      "Elijah plays deep in left field.",
      "Mia is positioned centrally in the outfield.",
      "Seth gets benched because he sucks."
    ],
    requiresSethBenched: true
  };

  // Bonus Level Data
  const bonusLevel = {
    correctAssignment: {
      "P": "Logan",
      "C": "Ava",
      "1B": "Mason",
      "2B": "Harper",
      "SS": "Seth Rakestraw", // Seth replaces injured Lucas
      "3B": "Isabella",
      "LF": "Elijah",
      "CF": "Mia",
      "RF": "Olivia"
    },
    clues: [
      "Ava is behind the plate.",
      "Harper plays next to Mason in the infield.",
      "The shortstop is injured and can't play.",
      "Seth must replace the injured shortstop.",
      "Elijah plays deep in left field.",
      "Mia is positioned centrally in the outfield."
    ],
    injuredPlayer: "Lucas",
    requiresSethBenched: false
  };

  let currentLevel = level1;
  let sethBenchCount = 0;

  const levelSelector = document.getElementById('levelSelector');
  const field = document.getElementById("field");
  const bench = document.getElementById("bench");
  const injuredList = document.getElementById("injuredList");
  const injuredTitle = document.getElementById("injured-title");
  const benchTitle = document.getElementById("bench-title");
  const clueList = document.getElementById("clueList");
  const sethMeter = document.getElementById("sethMeter");
  const sethCountSpan = document.getElementById("sethCount");
  const resultText = document.getElementById("resultText");
  const checkBtn = document.getElementById("checkBtn");

  let draggedItem = null;

  function updateSethCount() {
    sethCountSpan.textContent = sethBenchCount;
  }

  function allowDrop(ev) {
    ev.preventDefault();
  }

  function drag(ev) {
    draggedItem = ev.target;
    ev.dataTransfer.setData("text/plain", ev.target.id);
  }

  function dropOnPosition(ev) {
    ev.preventDefault();
    const posDiv = ev.currentTarget;

    if (!draggedItem) return;

    // If position already has a player, move them back to bench/injuredList
    let existingPlayer = posDiv.querySelector("li");
    if (existingPlayer) {
      if (currentLevel === bonusLevel && existingPlayer.textContent === currentLevel.injuredPlayer) {
        injuredList.appendChild(existingPlayer);
      } else {
        bench.appendChild(existingPlayer);
      }
    }

    posDiv.innerHTML = '';
    posDiv.appendChild(draggedItem);
    draggedItem = null;
  }

  function dropOnBench(ev) {
    ev.preventDefault();
    if (!draggedItem) return;

    // Remove dragged from current parent
    draggedItem.parentNode.removeChild(draggedItem);
    bench.appendChild(draggedItem);
    draggedItem = null;
  }

  function dropOnInjured(ev) {
    ev.preventDefault();
    if (!draggedItem) return;

    // Only allow injured player drop if bonus level
    if (currentLevel !== bonusLevel) return;

    // Remove dragged from current parent
    draggedItem.parentNode.removeChild(draggedItem);
    injuredList.appendChild(draggedItem);
    draggedItem = null;
  }

  // Initialize field positions with drag/drop
  function initField() {
    field.innerHTML = "";
    positions.forEach(pos => {
      let posDiv = document.createElement("div");
      posDiv.className = "position";
      posDiv.id = "pos-" + pos;
      posDiv.textContent = pos;

      posDiv.addEventListener("dragover", allowDrop);
      posDiv.addEventListener("drop", dropOnPosition);

      field.appendChild(posDiv);
    });
  }

  // Initialize bench with all players
  function initBench() {
    bench.innerHTML = "";
    players.forEach(name => {
      // Skip injured player if bonus level and player is injured
      if (currentLevel === bonusLevel && name === currentLevel.injuredPlayer) {
        return; // Injured player starts off in injured list
      }

      let li = document.createElement("li");
      li.textContent = name;
      li.id = "player-" + name.replace(/\s/g, "-");
      li.draggable = true;
      li.addEventListener("dragstart", drag);
      bench.appendChild(li);
    });
  }

  // Initialize injured list (only for bonus level)
  function initInjuredList() {
    injuredList.innerHTML = "";
    if (currentLevel === bonusLevel) {
      injuredTitle.style.display = "block";
      injuredList.style.display = "flex";
      injuredList.style.flexWrap = "wrap";
      injuredList.style.justifyContent = "center";
      injuredList.style.border = "2px dashed #aa0000";
      injuredList.style.background = "#fff0f0";
      injuredList.style.minHeight = "70px";

      // Add injured player li
      let injuredLi = document.createElement("li");
      injuredLi.textContent = currentLevel.injuredPlayer;
      injuredLi.id = "player-" + currentLevel.injuredPlayer.replace(/\s/g, "-");
      injuredLi.draggable = true;
      injuredLi.addEventListener("dragstart", drag);
      injuredList.appendChild(injuredLi);
    } else {
      injuredTitle.style.display = "none";
      injuredList.style.display = "none";
      injuredList.innerHTML = "";
    }
  }

  // Show clues for the current level
  function showClues() {
    clueList.innerHTML = "";
    currentLevel.clues.forEach(clue => {
      let li = document.createElement("li");
      li.textContent = clue;
      clueList.appendChild(li);
    });
  }

  // Check the current answer
  function checkAnswer() {
    // Build current assignment map
    let assignment = {};
    for (let pos of positions) {
      let posDiv = document.getElementById("pos-" + pos);
      let playerLi = posDiv.querySelector("li");
      assignment[pos] = playerLi ? playerLi.textContent : "";
    }

    if (currentLevel.requiresSethBenched) {
      // Seth must be on bench
      let sethOnBench = Array.from(bench.children).some(li => li.textContent === "Seth Rakestraw");
      if (!sethOnBench) {
        resultText.style.color = "red";
        resultText.textContent = "❌ Seth is not on the bench! He always gets benched because he sucks.";
        return;
      }
    } else {
      // Bonus level: Seth must be playing (assigned on field)
      let sethOnField = Object.values(assignment).includes("Seth Rakestraw");
      if (!sethOnField) {
        resultText.style.color = "red";
        resultText.textContent = "❌ Seth must be playing and replacing the injured player!";
        return;
      }
      // Injured player must be on injured list
      let injuredOnList = Array.from(injuredList.children).some(li => li.textContent === currentLevel.injuredPlayer);
      if (!injuredOnList) {
        resultText.style.color = "red";
        resultText.textContent = `❌ The injured player (${currentLevel.injuredPlayer}) must be on the injured list.`;
        return;
      }
    }

    // Check all positions assigned and match correct players
    for (let pos of positions) {
      if (!assignment[pos]) {
        resultText.style.color = "red";
        resultText.textContent = "❌ Some positions are still empty.";
        return;
      }
      if (assignment[pos] !== currentLevel.correctAssignment[pos]) {
        resultText.style.color = "red";
        resultText.textContent = `❌ Incorrect player at position ${pos}.`;
        return;
      }
    }

    // If here, puzzle solved!

    if (currentLevel === level1) {
      resultText.style.color = "green";
      resultText.textContent = "✅ Congratulations! You solved the puzzle correctly.";
      sethBenchCount++;
      sethMeter.style.display = "block";
      updateSethCount();
    } else if (currentLevel === bonusLevel) {
      resultText.style.color = "orange";
      resultText.innerHTML = "⚠️ Congratulations, puzzle completed — but you lost the game because Seth made a game-losing error and struck out every time.";
      sethMeter.style.display = "none";
    }
  }

  // Initialize the game with the selected level
  function initGame() {
    resultText.textContent = "";
    benchTitle.style.display = "block";
    sethMeter.style.display = currentLevel.requiresSethBenched ? "block" : "none";

    initField();
    initBench();
    initInjuredList();
    showClues();
    updateSethCount();
  }

  levelSelector.addEventListener("change", () => {
    if (levelSelector.value === "1") {
      currentLevel = level1;
    } else {
      currentLevel = bonusLevel;
    }
    initGame();
  });

  checkBtn.addEventListener("click", checkAnswer);

  window.onload = () => {
    initGame();
  };
</script>

</body>
</html>
