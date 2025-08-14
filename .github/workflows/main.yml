<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Flawless IT Solutions - Device Tracker</title>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
<style>
    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background-color: white;
        color: #333;
    }
    header {
        background: #004d4d;
        color: white;
        padding: 10px;
        text-align: center;
    }
    nav {
        background: #006666;
        display: flex;
        justify-content: center;
        padding: 10px;
        flex-wrap: wrap;
    }
    nav button {
        background: white;
        border: none;
        margin: 5px;
        padding: 10px 15px;
        cursor: pointer;
        font-weight: bold;
        border-radius: 5px;
    }
    nav button:hover {
        background: #ddd;
    }
    section {
        display: none;
        padding: 20px;
        max-width: 900px;
        margin: auto;
    }
    section.active {
        display: block;
    }
    #map {
        height: 500px;
        border-radius: 10px;
        overflow: hidden;
    }
    .card {
        background: white;
        border-radius: 10px;
        padding: 20px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        margin-bottom: 20px;
    }
    .input-group {
        display: flex;
        gap: 10px;
        margin-bottom: 15px;
    }
    .input-group input, .input-group select {
        flex: 1;
        padding: 10px;
        border-radius: 5px;
        border: 1px solid #ccc;
    }
    .input-group button {
        background: #004d4d;
        color: white;
        padding: 10px 15px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
    }
    .input-group button:hover {
        background: #003333;
    }
    .device-list {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
        gap: 10px;
    }
    .device-item {
        background: #f9f9f9;
        padding: 10px;
        border-radius: 8px;
        box-shadow: 0 1px 4px rgba(0,0,0,0.05);
        text-align: center;
        font-weight: bold;
        cursor: pointer;
        transition: background 0.2s;
    }
    .device-item:hover {
        background: #e0f7f7;
    }
    .device-note {
        font-size: 0.9em;
        font-weight: normal;
        color: #666;
        margin-top: 5px;
    }
</style>
</head>
<body>
<header>
    <h1>Flawless IT Solutions - Device Tracker</h1>
</header>

<nav>
    <button onclick="showSection('manage')">Manage Devices</button>
    <button onclick="showSection('assign')">Assign to Location</button>
    <button onclick="showSection('mapSection')">View Map</button>
</nav>

<section id="manage" class="active">
    <div class="card">
        <h2>Manage Devices</h2>
        <div class="input-group">
            <input type="text" id="deviceName" placeholder="Device name">
            <button onclick="addDevice()">Add Device</button>
        </div>
        <div class="device-list" id="deviceList"></div>
    </div>
</section>

<section id="assign">
    <div class="card">
        <h2>Assign Device to Location</h2>
        <div class="input-group">
            <select id="deviceSelect"></select>
            <select id="locationSelect"></select>
            <button onclick="assignDevice()">Assign</button>
        </div>
    </div>
</section>

<section id="mapSection">
    <div class="card">
        <h2>Device Map</h2>
        <div id="map"></div>
    </div>
</section>

<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
<script>
    const locations = {
        "Cape Town": [-33.9249, 18.4241],
        "Stellenbosch": [-33.9344, 18.8670],
        "Paarl": [-33.7342, 18.9621],
        "George": [-33.9630, 22.4617],
        "Knysna": [-34.0363, 23.0470]
    };

    let devices = JSON.parse(localStorage.getItem('devices')) || [];
    let assignments = JSON.parse(localStorage.getItem('assignments')) || {}; // { location: [deviceNames] }

    function saveData() {
        localStorage.setItem('devices', JSON.stringify(devices));
        localStorage.setItem('assignments', JSON.stringify(assignments));
    }

    function showSection(id) {
        document.querySelectorAll('section').forEach(sec => sec.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        if (id === 'mapSection') {
            setTimeout(() => map.invalidateSize(), 200);
        }
    }

    function updateDeviceList() {
        const list = document.getElementById('deviceList');
        list.innerHTML = '';
        devices.forEach((dev, index) => {
            const div = document.createElement('div');
            div.className = 'device-item';
            div.innerHTML = `<div>${dev.name}</div>` +
                            `<div class="device-note">${dev.note || ''}</div>`;
            div.onclick = () => editDevice(index);
            list.appendChild(div);
        });

        const select = document.getElementById('deviceSelect');
        select.innerHTML = '';
        devices.forEach(dev => {
            const option = document.createElement('option');
            option.value = dev.name;
            option.textContent = dev.name;
            select.appendChild(option);
        });
    }

    function updateLocationSelect() {
        const select = document.getElementById('locationSelect');
        select.innerHTML = '';
        for (let loc in locations) {
            const option = document.createElement('option');
            option.value = loc;
            option.textContent = loc;
            select.appendChild(option);
        }
    }

    function addDevice() {
        const name = document.getElementById('deviceName').value.trim();
        if (name && !devices.some(d => d.name === name)) {
            devices.push({ name: name, note: '' });
            saveData();
            updateDeviceList();
            document.getElementById('deviceName').value = '';
        }
    }

    function editDevice(index) {
        const currentDevice = devices[index];
        const newName = prompt("Edit device name:", currentDevice.name);
        if (newName === null) return;
        if (newName.trim() === "") {
            if (confirm(`Delete "${currentDevice.name}"?`)) {
                devices.splice(index, 1);
                for (let loc in assignments) {
                    assignments[loc] = assignments[loc].filter(d => d !== currentDevice.name);
                }
                saveData();
                updateDeviceList();
                updateMap();
            }
            return;
        }
        const newNote = prompt("Add or edit note:", currentDevice.note || "");
        devices[index] = { name: newName.trim(), note: newNote.trim() };
        for (let loc in assignments) {
            assignments[loc] = assignments[loc].map(d => d === currentDevice.name ? newName.trim() : d);
        }
        saveData();
        updateDeviceList();
        updateMap();
    }

    function assignDevice() {
        const dev = document.getElementById('deviceSelect').value;
        const loc = document.getElementById('locationSelect').value;
        if (!assignments[loc]) assignments[loc] = [];
        if (!assignments[loc].includes(dev)) {
            assignments[loc].push(dev);
            saveData();
            updateMap();
        } else {
            alert("This device is already assigned to that location.");
        }
    }

    const map = L.map('map').setView([-33.9249, 18.4241], 7);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        maxZoom: 19
    }).addTo(map);

    function updateMap() {
        map.eachLayer(layer => {
            if (layer instanceof L.Marker) map.removeLayer(layer);
        });
        for (let loc in assignments) {
            if (locations[loc] && assignments[loc].length > 0) {
                let popupContent = `<b>${loc}</b><br>`;
                assignments[loc].forEach(devName => {
                    let dev = devices.find(d => d.name === devName);
                    if (dev) popupContent += `${dev.name} - ${dev.note || ''}<br>`;
                });
                L.marker(locations[loc]).addTo(map).bindPopup(popupContent);
            }
        }
    }

    updateDeviceList();
    updateLocationSelect();
    updateMap();
</script>
</body>
</html>
