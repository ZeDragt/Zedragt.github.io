<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>My Music</title>

<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

<style>
body {
    margin:0;
    font-family:-apple-system, BlinkMacSystemFont;
    background:linear-gradient(180deg,#0f0f0f,#1a1a1a);
    color:white;
}

header {
    padding:20px;
    font-size:24px;
    font-weight:bold;
}

.upload {
    padding:10px 20px;
}

input[type=file] {
    color:white;
}

.section {
    padding:10px 20px;
}

.song {
    background:#1e1e1e;
    border-radius:15px;
    padding:15px;
    margin-bottom:12px;
    box-shadow:0 4px 10px rgba(0,0,0,0.5);
    transition:0.2s;
}

.song:hover {
    transform:scale(1.02);
}

.song-title {
    font-weight:bold;
    margin-bottom:8px;
}

.controls button {
    margin:5px 5px 0 0;
    padding:6px 10px;
    border:none;
    border-radius:10px;
    background:#333;
    color:white;
    font-size:12px;
}

.controls button:hover {
    background:#555;
}

.favorite {
    color:#1DB954;
    font-weight:bold;
}

.playlist {
    background:#141414;
    padding:10px;
    border-radius:12px;
    margin-bottom:15px;
}
</style>
</head>

<body>

<header>🎧 My Music</header>

<div class="upload">
<input type="file" id="fileInput" accept="audio/*" multiple>
</div>

<div class="section">
<h3>🎵 Bibliothèque</h3>
<div id="songs"></div>
</div>

<div class="section">
<h3>⭐ Favoris</h3>
<div id="favorites"></div>
</div>

<div class="section">
<h3>📂 Playlists</h3>
<button onclick="createPlaylist()">Créer Playlist</button>
<div id="playlists"></div>
</div>

<script>
let db;
let favorites = JSON.parse(localStorage.getItem("favorites")) || [];
let playlists = JSON.parse(localStorage.getItem("playlists")) || {};

let request = indexedDB.open("MusicDB", 1);

request.onupgradeneeded = function(e) {
    db = e.target.result;
    db.createObjectStore("songs", { keyPath: "name" });
};

request.onsuccess = function(e) {
    db = e.target.result;
    loadSongs();
};

document.getElementById("fileInput").addEventListener("change", function(e) {
    let transaction = db.transaction(["songs"], "readwrite");
    let store = transaction.objectStore("songs");

    for (let file of e.target.files) {
        let reader = new FileReader();
        reader.onload = function() {
            store.put({ name: file.name, data: reader.result });
        };
        reader.readAsDataURL(file);
    }

    transaction.oncomplete = loadSongs;
});

function loadSongs() {
    let container = document.getElementById("songs");
    let favContainer = document.getElementById("favorites");
    container.innerHTML = "";
    favContainer.innerHTML = "";

    let transaction = db.transaction(["songs"], "readonly");
    let store = transaction.objectStore("songs");
    let request = store.getAll();

    request.onsuccess = function() {
        request.result.forEach(song => {

            let html = `
                <div class="song">
                    <div class="song-title">${song.name}</div>
                    <audio controls src="${song.data}"></audio>
                    <div class="controls">
                        <button onclick="toggleFav('${song.name}')">❤️</button>
                        <button onclick="addToPlaylist('${song.name}')">➕ Playlist</button>
                        <button onclick="deleteSong('${song.name}')">🗑</button>
                    </div>
                </div>
            `;

            container.innerHTML += html;

            if (favorites.includes(song.name)) {
                favContainer.innerHTML += `
                    <div class="song favorite">${song.name}</div>
                `;
            }
        });

        renderPlaylists();
    };
}

function toggleFav(name) {
    if (favorites.includes(name)) {
        favorites = favorites.filter(f => f !== name);
    } else {
        favorites.push(name);
    }
    localStorage.setItem("favorites", JSON.stringify(favorites));
    loadSongs();
}

function deleteSong(name) {
    let transaction = db.transaction(["songs"], "readwrite");
    let store = transaction.objectStore("songs");
    store.delete(name);
    transaction.oncomplete = loadSongs;
}

function createPlaylist() {
    let name = prompt("Nom de la playlist ?");
    if (!name) return;
    playlists[name] = [];
    localStorage.setItem("playlists", JSON.stringify(playlists));
    renderPlaylists();
}

function addToPlaylist(song) {
    let name = prompt("Ajouter à quelle playlist ?");
    if (!playlists[name]) return alert("Playlist inexistante");
    playlists[name].push(song);
    localStorage.setItem("playlists", JSON.stringify(playlists));
    renderPlaylists();
}

function renderPlaylists() {
    let container = document.getElementById("playlists");
    container.innerHTML = "";

    for (let name in playlists) {
        container.innerHTML += `
            <div class="playlist">
                <strong>${name}</strong><br>
                ${playlists[name].join("<br>")}
            </div>
        `;
    }
}
</script>

</body>
</html>
