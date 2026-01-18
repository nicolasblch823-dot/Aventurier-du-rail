[Aventuriers du rail.html](https://github.com/user-attachments/files/24696597/Aventuriers.du.rail.html)
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Les Aventuriers du Rail - Scores</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>
body {
    margin: 0;
    font-family: "Segoe UI", sans-serif;
    background: linear-gradient(135deg, #2c3e50, #4ca1af);
}

h1 {
    text-align: center;
    padding: 20px;
    color: white;
    font-size: 2.6em;
}

.container {
    display: flex;
    justify-content: center;
    gap: 20px;
    padding: 20px;
    align-items: flex-start;
}

.box {
    background: white;
    border-radius: 14px;
    padding: 15px;
    width: 280px;
    box-shadow: 0 8px 20px rgba(0,0,0,0.2);
}

h2 { text-align: center; margin-top: 0; }

input, select, button {
    width: 100%;
    margin-top: 8px;
    padding: 8px;
    border-radius: 6px;
    border: 1px solid #ccc;
}

button {
    background: #3498db;
    color: white;
    border: none;
    cursor: pointer;
}

button:hover { background: #2980b9; }

.success { background: #27ae60; }
.danger { background: #e74c3c; }

ul { list-style: none; padding: 0; }

.player-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 5px;
    margin-top: 8px;
}

.tag {
    background: #3498db;
    color: white;
    padding: 4px 8px;
    border-radius: 12px;
    font-size: 0.8em;
}

.tag span { margin-left: 6px; cursor: pointer; }

/* SCORES */
.groups-container {
    max-height: 420px;
    overflow-y: auto;
}

.group {
    padding: 10px;
    border-radius: 10px;
    margin-bottom: 10px;
}

.red { background: #ffb3b3; }
.yellow { background: #fff1a8; }
.light-green { background: #c8f7c5; }
.dark-green { background: #2ecc71; color: white; }

.history { font-size: 0.8em; }

/* CLASSEMENT */
.rank {
    padding: 8px;
    border-radius: 8px;
    margin-bottom: 6px;
    background: #ecf0f1;
    font-weight: bold;
}

footer {
    text-align: center;
    color: white;
    padding: 10px;
    opacity: 0.8;
}
</style>
</head>

<body>

<h1>🚆 Les Aventuriers du Rail</h1>

<div class="container">

<!-- JOUEURS -->
<div class="box">
    <h2>Joueurs</h2>
    <input id="playerName" placeholder="Nom du joueur">
    <button onclick="addPlayer()">Ajouter</button>
    <ul id="playerList"></ul>
</div>

<!-- GROUPES -->
<div class="box">
    <h2>Créer un groupe</h2>
    <input id="groupName" placeholder="Nom du groupe">
    <select id="playerSelect"></select>
    <button onclick="addPlayerToGroup()">Ajouter joueur</button>
    <div class="player-tags" id="selectedPlayers"></div>
    <button onclick="addGroup()">Créer groupe</button>
</div>

<!-- SCORES -->
<div class="box">
    <h2>Scores</h2>
    <div class="groups-container" id="groups"></div>
    <button class="success" onclick="exportPDF()">Exporter PDF</button>
    <button class="danger" onclick="resetAll()">Réinitialiser</button>
</div>

<!-- CLASSEMENT -->
<div class="box">
    <h2>Classement</h2>
    <div id="ranking"></div>
</div>

</div>

<footer>Suivi des scores – Les Aventuriers du Rail</footer>

<script>
const { jsPDF } = window.jspdf;

let players = [];
let groups = [];
let tempGroupPlayers = [];

/* JOUEURS */
function addPlayer() {
    const name = playerName.value.trim();
    if (name && !players.includes(name)) {
        players.push(name);
        playerName.value = "";
        renderPlayers();
        renderPlayerSelect();
    }
}

function renderPlayers() {
    playerList.innerHTML = "";
    players.forEach(p => {
        const li = document.createElement("li");
        li.textContent = "• " + p;
        playerList.appendChild(li);
    });
}

function renderPlayerSelect() {
    playerSelect.innerHTML = "";
    players.forEach(p => {
        const o = document.createElement("option");
        o.value = p;
        o.textContent = p;
        playerSelect.appendChild(o);
    });
}

/* GROUPES */
function addPlayerToGroup() {
    const p = playerSelect.value;
    if (p && !tempGroupPlayers.includes(p)) {
        tempGroupPlayers.push(p);
        renderSelectedPlayers();
    }
}

function renderSelectedPlayers() {
    selectedPlayers.innerHTML = "";
    tempGroupPlayers.forEach(p => {
        const t = document.createElement("div");
        t.className = "tag";
        t.innerHTML = `${p} <span onclick="removePlayer('${p}')">×</span>`;
        selectedPlayers.appendChild(t);
    });
}

function removePlayer(p) {
    tempGroupPlayers = tempGroupPlayers.filter(x => x !== p);
    renderSelectedPlayers();
}

function addGroup() {
    if (!groupName.value || tempGroupPlayers.length === 0) return;

    groups.push({
        name: groupName.value,
        members: [...tempGroupPlayers],
        score: 0,
        history: []
    });

    groupName.value = "";
    tempGroupPlayers = [];
    renderSelectedPlayers();
    renderGroups();
}

/* COULEURS */
function getColor(score) {
    if (score < 20) return "red";
    if (score < 30) return "yellow";
    if (score < 40) return "light-green";
    return "dark-green";
}

/* SCORES */
function addScore(i) {
    const input = document.getElementById(`score-${i}`);
    const value = parseInt(input.value);

    if (!isNaN(value)) {
        groups[i].score += value;
        groups[i].history.push(value);
        input.value = 0; // ✅ remise à 0
        renderGroups();
    }
}

function renderGroups() {
    const container = document.getElementById("groups");
    container.innerHTML = "";

    groups.forEach((g, i) => {
        const d = document.createElement("div");
        d.className = `group ${getColor(g.score)}`;
        d.innerHTML = `
            <strong>${g.name}</strong><br>
            ${g.members.join(", ")}<br>
            Score : <strong>${g.score}</strong>
            <input id="score-${i}" type="number" value="0">
            <button onclick="addScore(${i})">Ajouter</button>
            <div class="history">Historique : ${g.history.join(", ") || "Aucun"}</div>
        `;
        container.appendChild(d);
    });

    renderRanking();
}

/* CLASSEMENT */
function renderRanking() {
    ranking.innerHTML = "";
    [...groups]
        .sort((a,b) => b.score - a.score)
        .forEach((g,i) => {
            const d = document.createElement("div");
            d.className = "rank";
            d.textContent = `${i+1}. ${g.name} — ${g.score} pts`;
            ranking.appendChild(d);
        });
}

/* RESET */
function resetAll() {
    if (confirm("Tout réinitialiser ?")) {
        players = [];
        groups = [];
        tempGroupPlayers = [];
        renderPlayers();
        renderPlayerSelect();
        renderGroups();
        ranking.innerHTML = "";
    }
}

/* PDF */
function exportPDF() {
    const doc = new jsPDF();
    let y = 20;

    doc.setFontSize(18);
    doc.text("Les Aventuriers du Rail - Résultats", 10, y);
    y += 15;

    groups
        .sort((a,b) => b.score - a.score)
        .forEach(g => {
            if (y > 260) { doc.addPage(); y = 20; }
            doc.text(`${g.name} — ${g.score} pts`, 10, y);
            y += 8;
        });

    doc.save("aventuriers_du_rail_scores.pdf");
}
</script>

</body>
</html>
