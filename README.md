<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>DRONE DESTROYER — ППО України</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; user-select: none; }
        body { background: #050505; font-family: 'Courier New', monospace; overflow: hidden; height: 100vh; width: 100vw; }
        #map { height: 100vh; width: 100vw; position: absolute; top: 0; left: 0; z-index: 1; }
        
        /* UI поверх карти */
        .game-ui { position: absolute; top: 0; left: 0; right: 0; z-index: 100; pointer-events: none; }
        .game-ui * { pointer-events: auto; }
        
        /* Верхня панель */
        .top-bar {
            background: rgba(0, 0, 0, 0.85);
            border-bottom: 2px solid #00ff44;
            padding: 12px 24px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 15px;
            backdrop-filter: blur(8px);
        }
        .logo { color: #00ff44; font-size: 20px; font-weight: bold; letter-spacing: 2px; text-shadow: 0 0 10px #00ff44; }
        .logo span { color: #ff4444; }
        .menu { display: flex; gap: 20px; }
        .menu-btn { color: #00aa55; cursor: pointer; font-size: 13px; transition: 0.2s; }
        .menu-btn:hover { color: #00ff44; text-shadow: 0 0 5px #00ff44; }
        .balance { color: #ffaa44; font-weight: bold; font-size: 16px; }
        .time { color: #5588aa; font-family: monospace; font-size: 14px; }
        
        /* Панель статистики зліва */
        .stats-panel {
            position: fixed;
            top: 80px;
            left: 20px;
            background: rgba(0, 0, 0, 0.85);
            border: 1px solid #00ff44;
            padding: 12px 18px;
            z-index: 100;
            backdrop-filter: blur(8px);
            min-width: 160px;
        }
        .stats-title { color: #00ff44; font-size: 11px; border-bottom: 1px solid #00aa44; padding-bottom: 5px; margin-bottom: 8px; }
        .stat-row { display: flex; justify-content: space-between; margin-bottom: 5px; color: #88ffaa; font-size: 12px; }
        .stat-value { color: #ffaa44; font-weight: bold; }
        .threat-critical { color: #ff4444; animation: blink 0.5s infinite; }
        
        /* Магазин (термінальне вікно) */
        .shop-window {
            position: fixed;
            bottom: 20px;
            left: 20px;
            right: 20px;
            background: rgba(0, 0, 0, 0.95);
            border: 2px solid #00ff44;
            z-index: 1000;
            display: none;
            backdrop-filter: blur(10px);
        }
        .shop-window.open { display: flex; flex-direction: column; }
        .shop-header {
            background: #00aa44;
            color: #000;
            padding: 10px 20px;
            font-weight: bold;
            display: flex;
            justify-content: space-between;
        }
        .shop-body { padding: 20px; max-height: 50vh; overflow-y: auto; }
        .shop-grid { display: flex; gap: 20px; justify-content: center; flex-wrap: wrap; }
        .shop-item {
            border: 1px solid #00aa44;
            padding: 15px 20px;
            cursor: pointer;
            text-align: center;
            min-width: 130px;
            background: #0a150a;
            transition: 0.2s;
        }
        .shop-item:hover, .shop-item.active { background: #00aa44; color: #000; transform: scale(1.02); }
        .shop-icon { font-size: 48px; margin-bottom: 8px; }
        .shop-price { color: #ffaa44; font-size: 11px; margin-top: 8px; }
        .shop-desc { font-size: 10px; color: #88aa88; margin-top: 4px; }
        
        /* Радар */
        .radar {
            position: fixed;
            bottom: 20px;
            right: 20px;
            width: 140px;
            background: rgba(0, 0, 0, 0.85);
            border: 1px solid #00ff44;
            padding: 8px;
            z-index: 100;
        }
        .radar-title { font-size: 9px; color: #00aa44; text-align: center; margin-bottom: 5px; }
        canvas#radarCanvas { width: 100%; background: #0a1a0a; border-radius: 50%; }
        
        /* Статус бар */
        .status-bar {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: rgba(0, 0, 0, 0.85);
            border-top: 1px solid #00aa44;
            padding: 5px 20px;
            display: flex;
            justify-content: space-between;
            font-size: 10px;
            color: #668866;
            z-index: 100;
        }
        
        @keyframes blink { 0% { opacity: 1; } 50% { opacity: 0.5; } 100% { opacity: 1; } }
        .drone-marker { filter: drop-shadow(0 0 3px #ff4444); }
        .unit-marker { filter: drop-shadow(0 0 3px #00ff44); }
        
        .leaflet-control-attribution { background: rgba(0,0,0,0.6) !important; color: #448866 !important; font-size: 8px !important; z-index: 50 !important; }
        ::-webkit-scrollbar { width: 5px; background: #0a0a0a; }
        ::-webkit-scrollbar-thumb { background: #00aa44; }
        
        @media (max-width: 700px) {
            .stats-panel { display: none; }
            .radar { width: 100px; bottom: 40px; }
            .top-bar { flex-direction: column; align-items: stretch; text-align: center; }
            .menu { justify-content: center; }
        }
    </style>
</head>
<body>
<div id="map"></div>

<div class="game-ui">
    <div class="top-bar">
        <div class="logo">🔫 DRONE <span>DESTROYER</span> 🔫</div>
        <div class="menu">
            <span class="menu-btn" id="shopBtn">🏪 МАГАЗИН</span>
            <span class="menu-btn" id="statsBtn">📊 СТАТИСТИКА</span>
            <span class="menu-btn" id="helpBtn">❓ ДОВІДКА</span>
        </div>
        <div class="balance">💰 <span id="balanceDisplay">15000</span> ₴</div>
        <div class="time" id="timeDisplay">ДЕНЬ 1 | 09:41</div>
    </div>
</div>

<div class="stats-panel">
    <div class="stats-title">⚡ БОЙОВА СТАТИСТИКА</div>
    <div class="stat-row">🎯 ЗБИТО ДРОНІВ: <span class="stat-value" id="killsDisplay">0</span></div>
    <div class="stat-row">🚁 АКТИВНІ ЦІЛІ: <span class="stat-value" id="threatCount">0</span></div>
    <div class="stat-row">🏭 УСТАНОВОК: <span class="stat-value" id="unitsCount">0</span></div>
    <div class="stat-row" id="threatStatus">🟢 СТАТУС: НОРМАЛЬНО</div>
</div>

<div class="radar">
    <div class="radar-title">📡 РАДАР</div>
    <canvas id="radarCanvas" width="130" height="130"></canvas>
</div>

<div class="shop-window" id="shopWindow">
    <div class="shop-header">
        <span>🏪 ТЕРМІНАЛ ЗАКУПІВЛІ ЗБРОЇ</span>
        <span style="cursor:pointer" id="closeShop">[X]</span>
    </div>
    <div class="shop-body">
        <div class="shop-grid">
            <div class="shop-item" data-unit="pickup">
                <div class="shop-icon">🔫</div>
                <div><strong>ПІКАП 12.7мм</strong></div>
                <div class="shop-desc">Кулемет, дальність 80км</div>
                <div class="shop-price">💰 5 500 ₴</div>
            </div>
            <div class="shop-item" data-unit="shilka">
                <div class="shop-icon">⚡</div>
                <div><strong>ЗСУ-23-4 "ШИЛКА"</strong></div>
                <div class="shop-desc">Автомат, дальність 120км</div>
                <div class="shop-price">💰 12 000 ₴</div>
            </div>
            <div class="shop-item" data-unit="reb">
                <div class="shop-icon">📡</div>
                <div><strong>РЕБ (ГЛУШІННЯ)</strong></div>
                <div class="shop-desc">Зупиняє дрони, радіус 45км</div>
                <div class="shop-price">💰 8 000 ₴</div>
            </div>
        </div>
        <div style="margin-top:15px; font-size:10px; color:#448866; text-align:center;">
            💡 ОБЕРІТЬ ЗБРОЮ → КЛІКНІТЬ НА КАРТІ УКРАЇНИ
        </div>
    </div>
</div>

<div class="status-bar">
    <span>🔫 DRONE DESTROYER</span>
    <span>⚡ ЗАХИСТИ НЕБО УКРАЇНИ</span>
    <span id="clockStatus">19:39 | 28.04.2026</span>
</div>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script>
(function(){
    // ========== МІСТА УКРАЇНИ ==========
    const CITIES = [
        { name: "КИЇВ", lat: 50.45, lng: 30.52 },
        { name: "ХАРКІВ", lat: 49.99, lng: 36.23 },
        { name: "ДНІПРО", lat: 48.46, lng: 35.04 },
        { name: "ОДЕСА", lat: 46.48, lng: 30.73 },
        { name: "ЛЬВІВ", lat: 49.84, lng: 24.03 },
        { name: "ЗАПОРІЖЖЯ", lat: 47.84, lng: 35.14 },
        { name: "ВІННИЦЯ", lat: 49.23, lng: 28.48 },
        { name: "ЧЕРКАСИ", lat: 49.44, lng: 32.06 },
        { name: "ПОЛТАВА", lat: 49.59, lng: 34.55 },
        { name: "СУМИ", lat: 50.91, lng: 34.80 },
        { name: "ЧЕРНІГІВ", lat: 51.50, lng: 31.30 },
        { name: "МИКОЛАЇВ", lat: 46.97, lng: 31.99 }
    ];
    
    // ========== ТОЧКИ ЗАПУСКУ З РОСІЇ ==========
    const LAUNCH_POINTS = [
        { name: "БЄЛГОРОД", lat: 50.60, lng: 36.60 },
        { name: "КУРСЬК", lat: 51.73, lng: 36.18 },
        { name: "ВОРОНЄЖ", lat: 51.67, lng: 39.18 },
        { name: "РОСТОВ", lat: 47.23, lng: 39.70 },
        { name: "КРИМ", lat: 44.61, lng: 33.52 },
        { name: "БРЯНСЬК", lat: 53.24, lng: 34.36 },
        { name: "МІЛЛЄРОВО", lat: 48.92, lng: 40.39 }
    ];
    
    // ========== ЗБРОЯ ==========
    const UNITS = {
        pickup: { name: "ПІКАП 12.7мм", price: 5500, range: 80, cooldown: 1600, damage: 45, color: "#ffaa44" },
        shilka: { name: "ШИЛКА 23мм", price: 12000, range: 120, cooldown: 800, damage: 85, color: "#ff6644" },
        reb: { name: "РЕБ", price: 8000, range: 45, cooldown: 2000, damage: 0, isJammer: true, jamRadius: 50, color: "#aa88ff" }
    };
    
    // ========== МЕЖІ УКРАЇНИ ==========
    const UKRAINE_POLYGON = [
        [52.3, 22.1], [52.2, 33.5], [50.2, 40.0], [46.0, 40.0], 
        [44.3, 33.9], [45.2, 29.0], [48.0, 22.1]
    ];
    
    function pointInPolygon(lat, lng, polygon) {
        let inside = false;
        for (let i = 0, j = polygon.length-1; i < polygon.length; j = i++) {
            const xi = polygon[i][0], yi = polygon[i][1];
            const xj = polygon[j][0], yj = polygon[j][1];
            const intersect = ((yi > lng) != (yj > lng)) &&
                (lat < (xj - xi) * (lng - yi) / (yj - yi) + xi);
            if (intersect) inside = !inside;
        }
        return inside;
    }
    
    function isInsideUkraine(lat, lng) {
        return pointInPolygon(lat, lng, UKRAINE_POLYGON);
    }
    
    // ========== ЗМІННІ ГРИ ==========
    let playerBalance = 15000;
    let totalKills = 0;
    let selectedUnit = null;
    let deployedUnits = [];
    let drones = [];
    let map, radarCtx, lastFrameTime = 0;
    let lastSpawnTime = 0;
    let gameTime = { day: 1, hour: 9, minute: 41 };
    
    function updateUI() {
        document.getElementById('balanceDisplay').innerText = Math.floor(playerBalance).toLocaleString();
        document.getElementById('killsDisplay').innerText = totalKills;
        document.getElementById('unitsCount').innerText = deployedUnits.length;
        
        const activeThreats = drones.filter(d => d.isAlive && !d.isJammed).length;
        document.getElementById('threatCount').innerText = activeThreats;
        
        const threatStatus = document.getElementById('threatStatus');
        if(drones.length > 8) {
            threatStatus.innerHTML = "🔴 СТАТУС: КРИТИЧНА НЕБЕЗПЕКА!";
            threatStatus.className = "stat-row threat-critical";
        } else if(drones.length > 0) {
            threatStatus.innerHTML = "🟡 СТАТУС: УВАГА! ШАХЕДИ В НЕБІ";
            threatStatus.className = "stat-row";
        } else {
            threatStatus.innerHTML = "🟢 СТАТУС: НОРМАЛЬНО";
            threatStatus.className = "stat-row";
        }
        
        document.getElementById('timeDisplay').innerHTML = `ДЕНЬ ${gameTime.day} | ${String(gameTime.hour).padStart(2,'0')}:${String(gameTime.minute).padStart(2,'0')}`;
        
        const now = new Date();
        document.getElementById('clockStatus').innerHTML = `${String(gameTime.hour).padStart(2,'0')}:${String(gameTime.minute).padStart(2,'0')} | ${String(now.getDate()).padStart(2,'0')}.${String(now.getMonth()+1).padStart(2,'0')}.2026`;
    }
    
    function addMoney(amount) {
        playerBalance += amount;
        updateUI();
        showFloatingText(`+${amount}₴`, '#00ff88');
    }
    
    function showFloatingText(text, color, latLng = null) {
        if(!latLng && map) latLng = map.getCenter();
        if(!latLng) return;
        const point = map.latLngToContainerPoint(latLng);
        const div = document.createElement('div');
        div.textContent = text;
        div.style.cssText = `position:absolute; left:${point.x}px; top:${point.y}px; color:${color}; font-weight:bold; font-size:14px; text-shadow:0 0 5px black; pointer-events:none; z-index:2000; transition:all 1s ease-out; white-space:nowrap; font-family:monospace;`;
        document.body.appendChild(div);
        setTimeout(() => { div.style.transform = 'translateY(-30px)'; div.style.opacity = '0'; setTimeout(() => div.remove(), 1000); }, 10);
    }
    
    function getDistanceKm(lat1, lng1, lat2, lng2) {
        const R = 6371;
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lng2 - lng1) * Math.PI / 180;
        const a = Math.sin(dLat/2)**2 + Math.cos(lat1*Math.PI/180) * Math.cos(lat2*Math.PI/180) * Math.sin(dLon/2)**2;
        return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    }
    
    // ========== КЛАС ДРОНА ==========
    class Drone {
        constructor(start, target) {
            this.startLat = start.lat;
            this.startLng = start.lng;
            this.target = target;
            this.currentLat = start.lat;
            this.currentLng = start.lng;
            this.progress = 0;
            this.hp = 100;
            this.isAlive = true;
            this.speed = 0.014;
            this.isJammed = false;
            this.jamEndTime = 0;
            this.marker = null;
        }
        
        update(deltaTime) {
            if(!this.isAlive) return;
            if(this.isJammed && Date.now() < this.jamEndTime) return;
            else this.isJammed = false;
            
            this.progress += this.speed * deltaTime * 0.03;
            if(this.progress >= 1) {
                this.isAlive = false;
                playerBalance = Math.max(0, playerBalance - 1500);
                showFloatingText(`-1500₴ (${this.target.name} атаковано)`, '#ff4444', [this.currentLat, this.currentLng]);
                if(this.marker) map.removeLayer(this.marker);
                updateUI();
                return;
            }
            this.currentLat = this.startLat + (this.target.lat - this.startLat) * this.progress;
            this.currentLng = this.startLng + (this.target.lng - this.startLng) * this.progress;
            if(this.marker) this.marker.setLatLng([this.currentLat, this.currentLng]);
        }
        
        render() {
            if(!this.isAlive) return;
            if(!this.marker) {
                const icon = L.divIcon({ html: '✈', iconSize: [28,28], className: 'drone-marker', iconAnchor: [14,14] });
                this.marker = L.marker([this.currentLat, this.currentLng], { icon: icon }).addTo(map);
            }
            if(this.marker._icon) {
                this.marker._icon.style.filter = this.isJammed ? 'hue-rotate(180deg)' : '';
                this.marker._icon.style.opacity = this.isJammed ? '0.6' : '1';
            }
        }
        
        destroy() {
            if(!this.isAlive) return;
            this.isAlive = false;
            if(this.marker) map.removeLayer(this.marker);
            totalKills++;
            addMoney(1200);
            showFloatingText('💥 ЗБИТО! +1200₴', '#ffaa44', [this.currentLat, this.currentLng]);
        }
        
        applyJam(duration = 3500) {
            this.isJammed = true;
            this.jamEndTime = Date.now() + duration;
        }
    }
    
    // ========== КЛАС УСТАНОВКИ ==========
    class DefenseUnit {
        constructor(type, lat, lng) {
            this.type = type;
            this.lat = lat;
            this.lng = lng;
            this.data = UNITS[type];
            this.lastShotTime = 0;
            this.marker = null;
            this.rangeCircle = null;
            this.ammo = (type === 'pickup') ? 300 : (type === 'shilka') ? 700 : 1;
        }
        
        canShoot() {
            return Date.now() - this.lastShotTime >= this.data.cooldown && this.ammo > 0 && this.data.damage > 0;
        }
        
        shootAt(drone) {
            if(!this.canShoot()) return false;
            const dist = getDistanceKm(this.lat, this.lng, drone.currentLat, drone.currentLng);
            if(dist <= this.data.range) {
                this.lastShotTime = Date.now();
                this.ammo--;
                drone.hp -= this.data.damage;
                this.showEffect(drone.currentLat, drone.currentLng);
                if(drone.hp <= 0) drone.destroy();
                return true;
            }
            return false;
        }
        
        jamDrones(dronesList) {
            if(this.type !== 'reb') return;
            for(let drone of dronesList) {
                const dist = getDistanceKm(this.lat, this.lng, drone.currentLat, drone.currentLng);
                if(dist <= this.data.jamRadius && drone.isAlive && !drone.isJammed) {
                    drone.applyJam(4000);
                }
            }
        }
        
        showEffect(lat, lng) {
            const point = map.latLngToContainerPoint([lat, lng]);
            const div = document.createElement('div');
            div.innerHTML = '💥';
            div.style.cssText = `position:absolute; left:${point.x}px; top:${point.y}px; font-size:18px; pointer-events:none; z-index:2000; transition:all 0.2s ease-out;`;
            document.body.appendChild(div);
            setTimeout(() => { div.style.transform = 'scale(1.5)'; div.style.opacity = '0'; setTimeout(() => div.remove(), 300); }, 10);
        }
        
        render() {
            if(!this.marker) {
                const icon = L.divIcon({ 
                    html: this.type === 'pickup' ? '🔫' : (this.type === 'shilka' ? '⚡' : '📡'),
                    iconSize: [32,32], 
                    className: 'unit-marker', 
                    iconAnchor: [16,16] 
                });
                this.marker = L.marker([this.lat, this.lng], { icon: icon }).addTo(map);
                this.rangeCircle = L.circle([this.lat, this.lng], {
                    radius: this.data.range * 1000,
                    color: this.data.color,
                    fillColor: this.data.color,
                    fillOpacity: 0.1,
                    weight: 1
                }).addTo(map);
            }
            this.marker.bindTooltip(`${this.data.name} | 🎯${this.ammo}`, { sticky: true });
        }
    }
    
    // ========== РАДАР ==========
    function drawRadar() {
        if(!radarCtx || !map) return;
        const canvas = document.getElementById('radarCanvas');
        canvas.width = 130;
        canvas.height = 130;
        const w = 130, h = 130;
        radarCtx.clearRect(0, 0, w, h);
        radarCtx.fillStyle = '#0a1a0a';
        radarCtx.fillRect(0, 0, w, h);
        radarCtx.strokeStyle = '#00ff44';
        radarCtx.beginPath();
        radarCtx.arc(w/2, h/2, w/2 - 2, 0, 2*Math.PI);
        radarCtx.stroke();
        
        const mapSize = map.getSize();
        for(let drone of drones) {
            if(!drone.isAlive) continue;
            const point = map.latLngToContainerPoint([drone.currentLat, drone.currentLng]);
            const rx = (point.x / mapSize.x) * w;
            const ry = (point.y / mapSize.y) * h;
            if(rx >= 0 && rx <= w && ry >= 0 && ry <= h) {
                radarCtx.fillStyle = drone.isJammed ? '#ffaa44' : '#ff4444';
                radarCtx.beginPath();
                radarCtx.arc(rx, ry, 3, 0, 2*Math.PI);
                radarCtx.fill();
            }
        }
        for(let unit of deployedUnits) {
            const point = map.latLngToContainerPoint([unit.lat, unit.lng]);
            const rx = (point.x / mapSize.x) * w;
            const ry = (point.y / mapSize.y) * h;
            if(rx >= 0 && rx <= w && ry >= 0 && ry <= h) {
                radarCtx.fillStyle = unit.data.color;
                radarCtx.fillRect(rx-2, ry-2, 4, 4);
            }
        }
    }
    
    // ========== ІГРОВИЙ ЦИКЛ ==========
    function gameUpdate() {
        const now = Date.now();
        let dt = Math.min(100, now - lastFrameTime);
        if(dt < 0) dt = 16;
        lastFrameTime = now;
        
        if(Math.random() < 0.02) {
            gameTime.minute += 1;
            if(gameTime.minute >= 60) {
                gameTime.minute = 0;
                gameTime.hour += 1;
                if(gameTime.hour >= 24) {
                    gameTime.hour = 0;
                    gameTime.day += 1;
                }
            }
            updateUI();
        }
        
        for(let i=0; i<drones.length; i++) {
            drones[i].update(dt / 1000);
            if(!drones[i].isAlive) {
                drones.splice(i,1);
                i--;
            }
        }
        
        for(let unit of deployedUnits) {
            if(unit.type === 'reb') unit.jamDrones(drones);
        }
        
        for(let unit of deployedUnits) {
            if(unit.data.damage > 0) {
                for(let drone of drones) {
                    unit.shootAt(drone);
                    if(!drone.isAlive) break;
                }
            }
        }
        
        for(let unit of deployedUnits) unit.render();
        for(let drone of drones) drone.render();
        drawRadar();
        updateUI();
    }
    
    // ========== СПАВН ДРОНІВ ==========
    function spawnDrone() {
        const now = Date.now();
        const interval = Math.max(3000, 5000 - Math.floor(totalKills / 30) * 100);
        if(now - lastSpawnTime < interval) return;
        if(drones.length >= 25) return;
        
        const start = LAUNCH_POINTS[Math.floor(Math.random() * LAUNCH_POINTS.length)];
        const target = CITIES[Math.floor(Math.random() * CITIES.length)];
        drones.push(new Drone(start, target));
        lastSpawnTime = now;
        updateUI();
    }
    
    // ========== ІНІЦІАЛІЗАЦІЯ ==========
    function init() {
        map = L.map('map').setView([48.9, 31.5], 6.5);
        L.tileLayer('https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png', {
            attribution: '© OpenStreetMap'
        }).addTo(map);
        
        L.polygon(UKRAINE_POLYGON, { color: '#00ff44', weight: 1.5, fillOpacity: 0.03 }).addTo(map);
        
        radarCtx = document.getElementById('radarCanvas').getContext('2d');
        
        // Клік по карті для розміщення установки
        map.on('click', (e) => {
            if(!selectedUnit) {
                alert("⚡ Оберіть зброю в МАГАЗИНІ!");
                return;
            }
            if(!isInsideUkraine(e.latlng.lat, e.latlng.lng)) {
                alert("❌ Розміщення дозволене тільки на території України!");
                return;
            }
            if(playerBalance < UNITS[selectedUnit].price) {
                alert("💰 Недостатньо коштів!");
                return;
            }
            playerBalance -= UNITS[selectedUnit].price;
            const unit = new DefenseUnit(selectedUnit, e.latlng.lat, e.latlng.lng);
            deployedUnits.push(unit);
            unit.render();
            updateUI();
            selectedUnit = null;
            document.querySelectorAll('.shop-item').forEach(c => c.classList.remove('active'));
        });
        
        // Магазин
        document.querySelectorAll('.shop-item').forEach(card => {
            card.addEventListener('click', (e) => {
                e.stopPropagation();
                const unitType = card.dataset.unit;
                if(selectedUnit === unitType) {
                    selectedUnit = null;
                    card.classList.remove('active');
                } else {
                    document.querySelectorAll('.shop-item').forEach(c => c.classList.remove('active'));
                    selectedUnit = unitType;
                    card.classList.add('active');
                }
            });
        });
        
        // Кнопки меню
        document.getElementById('closeShop').addEventListener('click', () => {
            document.getElementById('shopWindow').classList.remove('open');
        });
        
        document.getElementById('shopBtn').addEventListener('click', () => {
            document.getElementById('shopWindow').classList.toggle('open');
        });
        
        document.getElementById('statsBtn').addEventListener('click', () => {
            alert(`📊 СТАТИСТИКА\n━━━━━━━━━━━━━━━━━━━\n🎯 Збито дронів: ${totalKills}\n🏭 Активних установок: ${deployedUnits.length}\n🚁 Дронів в небі: ${drones.length}\n💰 Баланс: ${Math.floor(playerBalance)} ₴`);
        });
        
        document.getElementById('helpBtn').addEventListener('click', () => {
            alert(`📖 ДОВІДКА\n━━━━━━━━━━━━━━━━━━━\n1️⃣ Відкрийте МАГАЗИН\n2️⃣ Оберіть зброю\n3️⃣ Клікніть на карті України\n4️⃣ Установки стріляють автоматично\n5️⃣ РЕБ глушить дрони в радіусі\n\n🚁 Шахеди летять з території РФ!\n💥 Кожен збитий дрон +1200₴`);
        });
        
        // Демо-установка для зручності
        setTimeout(() => {
            if(deployedUnits.length === 0) {
                const demo = new DefenseUnit('pickup', 49.59, 34.55);
                deployedUnits.push(demo);
                demo.render();
                updateUI();
            }
        }, 500);
        
        lastFrameTime = Date.now();
        setInterval(gameUpdate, 50);
        setInterval(spawnDrone, 800);
        updateUI();
    }
    
    window.onload = init;
})();
</script>
</body>
</html>
