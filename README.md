<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>Simulasi Kota 2D Sederhana</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            margin: 0;
            background-color: #f3f4f6;
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: flex-start;
        }
        .scroll-container {
            width: 100%;
            max-width: 900px;
            padding: 1rem;
            display: flex;
            flex-direction: column;
            align-items: center;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
        }
        canvas {
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            touch-action: none;
            width: 100%;
            height: auto;
            max-width: 800px;
            background: white;
        }
        .mode-active {
            box-shadow: 0 0 0 4px #60a5fa;
            transform: scale(1.05);
        }
        .control-button {
            background-color: rgba(209,213,219,0.7);
            color: #4b5563;
            border-radius: 0.5rem;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            box-shadow: inset 0 2px 4px rgba(0,0,0,0.1);
            transition: transform 0.1s ease-in-out;
            backdrop-filter: blur(2px);
            width: 50px;
            height: 50px;
        }
        .control-button:active { transform: scale(0.95); }
        .action-button {
            width: 100%;
            padding: 0.5rem 1rem;
            color: white;
            font-weight: bold;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition-property: background-color, transform, box-shadow;
            transition-duration: 0.15s;
            transition-timing-function: ease-in-out;
        }
        .modal {
            position: fixed;
            z-index: 100;
            left:0; top:0; width:100%; height:100%;
            background-color: rgba(0,0,0,0.6);
            display:flex; justify-content:center; align-items:center;
            visibility: hidden; opacity: 0;
            transition: visibility 0s, opacity 0.25s linear;
        }
        .modal-show { visibility: visible; opacity: 1; }
        .modal-content {
            background:#fff; padding:1.25rem; border-radius:.75rem; width: min(92%, 600px);
            box-shadow: 0 8px 16px rgba(0,0,0,0.2); max-height: 80vh; overflow:auto;
        }
        .modal-header { display:flex; justify-content:space-between; align-items:center; margin-bottom:0.75rem; }
        .modal-close { font-size:1.5rem; font-weight:bold; color:#9ca3af; cursor:pointer; border:none; background:none; }
        .modal-close:hover { color:#6b7280; }
        .popup-menu {
            position:absolute; bottom:100%; left:50%; transform:translateX(-50%); margin-bottom:1rem;
            background:#fff; padding:0.75rem; border-radius:0.75rem; box-shadow:0 4px 12px rgba(0,0,0,0.15);
            display:flex; flex-direction:column; gap:.5rem; z-index:50;
            visibility:hidden; opacity:0; transition: visibility 0s, opacity 0.2s linear;
        }
        .popup-menu.show { visibility:visible; opacity:1; }
        #landscape-controls { display:none; position:absolute; bottom:1rem; right:1rem; width:150px; height:150px; grid-template-columns:repeat(3,1fr); grid-template-rows:repeat(3,1fr); gap:0.5rem; z-index:20; }
        @media (max-width:768px) and (orientation: landscape) { #landscape-controls { display:grid; } }
        #portrait-controls { display:none; flex-direction:row; justify-content:center; gap:4px; margin-top:1rem; width:100%; z-index:20; }
        @media (max-width:768px) and (orientation: portrait) { #portrait-controls { display:flex; } }
        .message-box {
            position: fixed; top:50%; left:50%; transform:translate(-50%,-50%); background:#333; color:white; padding:1rem 2rem; border-radius:0.5rem; box-shadow:0 4px 12px rgba(0,0,0,0.3); z-index:101; opacity:0; visibility:hidden; transition: opacity .25s, visibility .25s;
        }
        .message-box.show { opacity:1; visibility:visible; }
    </style>
</head>
<body>
<div class="scroll-container">
    <div class="main-container flex flex-col items-center w-full max-w-full md:max-w-4xl">
        <div class="text-center mb-4">
            <h1 class="text-3xl font-bold mb-2">Simulasi Kota 2D Sederhana</h1>
            <p class="text-gray-600">Gunakan tombol di bawah dan panah keyboard untuk berinteraksi.</p>
        </div>

        <div class="relative w-full flex justify-center">
            <canvas id="gameCanvas" class="w-full h-auto max-w-full" ></canvas>

            <div id="landscape-controls">
                <div></div>
                <button id="landscape-up-btn" class="control-button text-2xl">▲</button>
                <div></div>
                <button id="landscape-left-btn" class="control-button text-2xl">◀</button>
                <div></div>
                <button id="landscape-right-btn" class="control-button text-2xl">►</button>
                <div></div>
                <button id="landscape-down-btn" class="control-button text-2xl">▼</button>
                <div></div>
            </div>
        </div>

        <div id="portrait-controls" class="flex flex-row justify-center gap-4 mt-4 w-full">
            <button id="portrait-up-btn" class="control-button text-2xl">▲</button>
            <button id="portrait-down-btn" class="control-button text-2xl">▼</button>
            <button id="portrait-left-btn" class="control-button text-2xl">◀</button>
            <button id="portrait-right-btn" class="control-button text-2xl">►</button>
        </div>

        <div class="w-full text-lg text-center font-bold my-4 p-2 bg-slate-200 rounded-lg shadow-inner flex flex-col md:flex-row justify-around">
            <div>Uang: <span id="moneyDisplay"></span></div>
            <div>Populasi: <span id="populationDisplay"></span></div>
            <div>Pekerja Tersedia: <span id="availableWorkersDisplay"></span></div>
            <div>Daya Tersedia: <span id="powerDisplay"></span></div>
            <div>Air Tersedia: <span id="waterDisplay"></span></div>
        </div>

        <div class="w-full p-2 bg-slate-200 rounded-lg shadow-inner mt-2">
            <label for="taxRateSlider" class="block text-center font-bold">Tingkat Pajak: <span id="taxRateDisplay"></span>%</label>
            <input type="range" id="taxRateSlider" min="0" max="50" value="5" class="w-full mt-1 accent-blue-500" />
        </div>

        <div class="mt-4 w-full flex flex-col items-center">
            <div class="relative w-full flex justify-center">
                <div id="popupMenu" class="popup-menu grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-2 w-full max-w-md">
                    <button id="houseButton" class="action-button" style="background-color: #fde047;">Bangun Rumah</button>
                    <button id="parkButton" class="action-button" style="background-color: #22c55e;">Bangun Taman</button>
                    <button id="storeButton" class="action-button" style="background-color: #f59e0b;">Bangun Toko</button>
                    <button id="industrialButton" class="action-button" style="background-color: #1f2937;">Bangun Industri</button>
                    <button id="roadButton" class="action-button" style="background-color: #64748b;">Bangun Jalan</button>
                    <button id="hospitalButton" class="action-button" style="background-color: #7b241c;">Bangun Rumah Sakit</button>
                    <button id="windTurbineButton" class="action-button" style="background-color: #3b82f6;">Bangun Kincir Angin</button>
                    <button id="waterTowerButton" class="action-button" style="background-color: #38bdf8;">Bangun Menara Air</button>
                </div>
            </div>
            <div class="w-full grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 gap-2 max-w-full mt-4">
                <button id="moveModeButton" class="action-button bg-blue-500 hover:bg-blue-600 transition-colors">Mode Pindah</button>
                <button id="destroyModeButton" class="action-button bg-red-500 hover:bg-red-600 transition-colors">Hancurkan</button>
                <button id="buildMenuButton" class="action-button bg-gray-600 hover:bg-gray-700 transition-colors">Bangun</button>
                <button id="guideButton" class="action-button bg-gray-400 hover:bg-gray-500">Panduan</button>
                <button id="restartButton" class="action-button bg-yellow-500 hover:bg-yellow-600">Mulai Ulang</button>
                <button id="reportButton" class="action-button bg-purple-500 hover:bg-purple-600 transition-colors">Laporan Kota</button>
            </div>
        </div>
    </div>
</div>

<!-- Modals -->
<div id="guideModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">
            <h2>Panduan Permainan</h2>
            <button id="guideModalCloseButton" class="modal-close">&times;</button>
        </div>
        <p>Selamat datang! Ini adalah simulasi pembangunan kota sederhana. Berikut panduan dasar untuk memulai:</p>
        <ul class="guide-list mt-4">
            <li><strong>Pergerakan (Tombol Panah):</strong> Gunakan tombol panah pada keyboard atau tombol di layar sentuh untuk menggerakkan peta.</li>
            <li><strong>Pintasan Keyboard:</strong>
                <ul>
                    <li>'M' = Mode Pindah</li>
                    <li>'H' = Bangun Rumah</li>
                    <li>'P' = Bangun Taman</li>
                    <li>'T' = Bangun Toko</li>
                    <li>'I' = Bangun Industri</li>
                    <li>'J' = Bangun Jalan</li>
                    <li>'A' = Bangun Menara Air</li>
                    <li>'O' = Bangun Rumah Sakit</li>
                    <li>'L' = Bangun Kincir Angin</li>
                    <li>'K' = Laporan Kota</li>
                    <li>'X' = Mode Hancurkan</li>
                    <li>'G' = Panduan</li>
                    <li>'R' = Mulai Ulang</li>
                </ul>
            <li><strong>Membangun Bangunan:</strong> Pilih salah satu tombol bangunan lalu klik di kanvas untuk membangunnya. Pastikan Anda memiliki cukup uang!</li>
            <li><strong>Mode Hancurkan:</strong> Pilih tombol Hancurkan, lalu klik di bangunan yang ingin Anda hancurkan. Anda akan mendapatkan setengah dari biaya bangunan kembali.</li>
            <li><strong>Tingkat Pajak:</strong> Sesuaikan tingkat pajak dengan penggeser di bawah kanvas. Tingkat pajak yang lebih tinggi akan meningkatkan uang Anda, tetapi bisa membuat populasi turun.</li>
            <li><strong>Uang dan Populasi:</strong> Perhatikan panel di atas kanvas untuk melihat uang dan populasi Anda saat ini. Bangun rumah untuk meningkatkan populasi, dan bangun toko atau industri untuk memberikan lapangan pekerjaan bagi populasi.</li>
            <li><strong>Pekerja:</strong> Bangunan bisnis seperti Toko dan Industri hanya akan menghasilkan uang jika Anda memiliki cukup populasi untuk mengisi semua posisi pekerjaan yang tersedia dan mendapat pasokan listrik yang cukup. Rumah sakit akan selalu beroperasi dan menghasilkan uang, tetapi membutuhkan pekerja (minimal 1/4 dari kapasitas pasiennya) untuk efisiensi maksimal.</li>
            <li><strong>Koneksi Jalan:</strong> Pastikan bangunan Anda terhubung ke jalan agar warga dan bisnis lebih bahagia dan menguntungkan. Koneksi ini juga penting untuk mendapatkan akses ke pasokan listrik dari Pembangkit Listrik (kecuali untuk rumah sakit).</li>
            <li><strong>Kebutuhan Listrik:</strong> Bangunan seperti Toko dan Industri membutuhkan listrik untuk beroperasi. Anda harus membangun <strong>Pembangkit Listrik</strong> untuk memenuhi kebutuhan ini. Pendapatan dari pembangkit listrik berasal dari penjualan listrik ke bangunan lain. Rumah sakit tidak membutuhkan listrik untuk beroperasi.</li>
            <li><strong>Mulai Ulang:</strong> Tombol ini akan mereset semua uang, populasi, dan bangunan ke awal permainan. Gunakan jika Anda ingin memulai dari nol.</li>
        </ul>
    </div>
</div>

<div id="reportModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">
            <h2>Laporan Kota</h2>
            <button id="reportModalCloseButton" class="modal-close">&times;</button>
        </div>
        <div id="reportModalContent" class="text-gray-800 space-y-4"></div>
    </div>
</div>

<div id="infoModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">
            <h2 id="infoModalTitle">Informasi Bangunan</h2>
            <button id="infoModalCloseButton" class="modal-close">&times;</button>
        </div>
        <div id="infoModalContent"></div>
    </div>
</div>

<div id="messageBox" class="message-box"></div>

<script>
/* -------------- Game state & constants -------------- */
let money = 1000.00;
let population = 0;
let buildings = [];
let mapOffset = { x: 0, y: 0 };
let speed = 2.0;
let activeMode = 'move';
let buildingType = null;
let taxRate = 5;
let isPopupMenuOpen = false;
let selectedBuilding = null;
let totalPowerOutput = 0;
let totalPowerUsage = 0;
let totalWaterOutput = 0;
let totalWaterUsage = 0;
let lastIncomeTime = Date.now();

const gridSize = 40;
const incomeInterval = 1000;
const incomePerPersonPerSecond = 10;
const pricePerUnitPower = 0.5;
const pricePerUnitWater = 0.2;

const keys = {};
const touchControls = { up:false, down:false, left:false, right:false };

// income per worker
const baseIncomePerWorker = { store: 17.00, industrial: 25.00 };

/* -------------- Building definitions -------------- */
const buildingStats = {
    house: { cost: 100, populationCapacity: 5, name:'Rumah', color:'#fde047' },
    park: { cost: 50, name:'Taman', color:'#22c55e', maintenance:10, influenceRadius:5 },
    store: { cost: 200, name:'Toko', color:'#f59e0b', workersRequired:3, powerRequired:10 },
    industrial: { cost: 300, name:'Industri', color:'#1f2937', workersRequired:8, powerRequired:25 },
    road: { cost: 10, name:'Jalan', color:'#64748b', maintenance:1.5 },
    hospital: { cost: 500, name:'Rumah Sakit', color:'#7b241c', maintenance:30, patientCapacity:100, treatmentCost:10, influenceRadius:10, workersRequired:25, powerRequired:20 },
    windTurbine: { cost: 250, name:'Kincir Angin', color:'#3b82f6', maintenance:10, powerOutput:40 },
    waterTower: { cost: 50, name:'Menara Air', color:'#38bdf8', maintenance:5, waterOutput:50 }
};

/* -------------- DOM elements -------------- */
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');

const moneyDisplay = document.getElementById('moneyDisplay');
const populationDisplay = document.getElementById('populationDisplay');
const availableWorkersDisplay = document.getElementById('availableWorkersDisplay');
const powerDisplay = document.getElementById('powerDisplay');
const waterDisplay = document.getElementById('waterDisplay');
const taxRateDisplay = document.getElementById('taxRateDisplay');
const taxRateSlider = document.getElementById('taxRateSlider');

const infoModal = document.getElementById('infoModal');
const infoModalCloseButton = document.getElementById('infoModalCloseButton');
const infoModalContent = document.getElementById('infoModalContent');

const guideModal = document.getElementById('guideModal');
const guideModalCloseButton = document.getElementById('guideModalCloseButton');

const popupMenu = document.getElementById('popupMenu');
const buildMenuButton = document.getElementById('buildMenuButton');
const moveModeButton = document.getElementById('moveModeButton');
const destroyModeButton = document.getElementById('destroyModeButton');
const messageBox = document.getElementById('messageBox');

const buildingButtons = {
    house: document.getElementById('houseButton'),
    park: document.getElementById('parkButton'),
    store: document.getElementById('storeButton'),
    industrial: document.getElementById('industrialButton'),
    road: document.getElementById('roadButton'),
    hospital: document.getElementById('hospitalButton'),
    windTurbine: document.getElementById('windTurbineButton'),
    waterTower: document.getElementById('waterTowerButton')
};

const guideButton = document.getElementById('guideButton');
const restartButton = document.getElementById('restartButton');

const landscapeControls = {
    up: document.getElementById('landscape-up-btn'),
    down: document.getElementById('landscape-down-btn'),
    left: document.getElementById('landscape-left-btn'),
    right: document.getElementById('landscape-right-btn')
};
const portraitControls = {
    up: document.getElementById('portrait-up-btn'),
    down: document.getElementById('portrait-down-btn'),
    left: document.getElementById('portrait-left-btn'),
    right: document.getElementById('portrait-right-btn')
};

const reportButton = document.getElementById('reportButton');
const reportModal = document.getElementById('reportModal');
const reportModalCloseButton = document.getElementById('reportModalCloseButton');
const reportModalContent = document.getElementById('reportModalContent');

/* -------------- Helpers -------------- */
function formatRupiah(amount) {
    return new Intl.NumberFormat('id-ID', { style:'currency', currency:'IDR', minimumFractionDigits:2, maximumFractionDigits:2 }).format(amount);
}
function showMessage(text) {
    messageBox.textContent = text;
    messageBox.classList.add('show');
    setTimeout(()=> messageBox.classList.remove('show'), 2500);
}
function findBuilding(tileX, tileY) {
    return buildings.find(b => Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY);
}

/* Cek apakah petak bersebelahan ada jalan */
function isConnectedToRoad(tileX, tileY) {
    const adjacent = [
        {x: tileX, y: tileY-1},
        {x: tileX, y: tileY+1},
        {x: tileX-1, y: tileY},
        {x: tileX+1, y: tileY}
    ];
    return adjacent.some(t => {
        const b = findBuilding(t.x, t.y);
        return b && b.type === 'road';
    });
}

/* Cek apakah bangunan terhubung ke layanan (misal waterTower) pada petak tetangga */
function isConnectedToService(building, serviceType) {
    const tileX = Math.floor(building.x / gridSize);
    const tileY = Math.floor(building.y / gridSize);
    const adjacent = [
        {x: tileX, y: tileY-1},
        {x: tileX, y: tileY+1},
        {x: tileX-1, y: tileY},
        {x: tileX+1, y: tileY}
    ];
    return adjacent.some(t => {
        const b = findBuilding(t.x, t.y);
        return b && b.type === serviceType;
    });
}

/* -------------- Report / modal functions -------------- */
function generateCityReport() {
    const report = {
        totalPopulation: 0,
        averageHappiness: 0,
        income: { total:0, house:0, store:0, industrial:0, hospital:0, windTurbine:0, waterTower:0 },
        expenditure: { total:0, road:0, park:0, hospital:0, windTurbine:0, waterTower:0 },
        power: { generated: totalPowerOutput, used: totalPowerUsage },
        water: { generated: totalWaterOutput, used: totalWaterUsage }
    };

    let totalHappiness = 0, houseCount = 0;

    buildings.forEach(b => {
        const stats = buildingStats[b.type];

        if (b.type === 'house') {
            report.totalPopulation += b.population || 0;
            totalHappiness += (b.needs && b.needs.happiness) ? b.needs.happiness : 0;
            houseCount++;
            const taxPerHouse = ( (b.population || 0) * incomePerPersonPerSecond ) * (taxRate/100);
            report.income.house += taxPerHouse;
            report.income.total += taxPerHouse;
        } else if (stats && stats.workersRequired) {
            if (b.type === 'hospital') {
                const taxGain = (b.currentPatients || 0) * stats.treatmentCost;
                report.income.hospital += taxGain;
                report.income.total += taxGain;
            } else if (b.isPowered && (b.workersAssigned || 0) > 0) {
                const taxGain = (baseIncomePerWorker[b.type] || 0) * (b.workersAssigned || 0) * (taxRate/100);
                report.income[b.type] += taxGain;
                report.income.total += taxGain;
            }
        } else if (b.hasWater) {
            if (b.type === 'store' || b.type === 'industrial') {
                const bonusIncome = (baseIncomePerWorker[b.type] || 0) * (b.workersAssigned || 0) * (taxRate/100) * 0.2;
                report.income[b.type] += bonusIncome;
                report.income.total += bonusIncome;
            } else if (b.type === 'park') {
                const discount = (stats.maintenance || 0) * 0.2;
                report.expenditure.park -= discount;
                report.expenditure.total -= discount;
            }
        }

        if (stats && stats.maintenance) {
            report.expenditure[b.type] += stats.maintenance;
            report.expenditure.total += stats.maintenance;
        }

        if (b.type === 'windTurbine') {
            report.income.windTurbine += (b.powerSold || 0) * pricePerUnitPower;
            report.income.total += (b.powerSold || 0) * pricePerUnitPower;
        }
        if (b.type === 'waterTower') {
            report.income.waterTower += (b.waterSold || 0) * pricePerUnitWater;
            report.income.total += (b.waterSold || 0) * pricePerUnitWater;
        }
    });

    report.averageHappiness = houseCount ? Math.floor(totalHappiness / houseCount) : 0;
    return report;
}

function showReport() {
    const r = generateCityReport();
    const incomeYearFactor = 365 * 24 * 60 * 60 / incomeInterval;
    reportModalContent.innerHTML = `
        <div class="p-4 border border-gray-300 rounded-lg bg-gray-100">
            <h3 class="font-bold text-lg mb-2">Ringkasan Kota</h3>
            <p>Populasi Total: <strong>${r.totalPopulation}</strong></p>
            <p>Tingkat Kebahagiaan Rata-Rata: <strong>${r.averageHappiness}%</strong></p>
        </div>
        <div class="p-4 border border-green-300 rounded-lg bg-green-50">
            <h3 class="font-bold text-lg mb-2 text-green-700">Pendapatan (proyeksi tahunan)</h3>
            <p>Pendapatan Total: <strong>${formatRupiah(r.income.total * incomeYearFactor)}</strong></p>
            <ul class="list-disc ml-6 mt-2">
                <li>Rumah: ${formatRupiah(r.income.house * incomeYearFactor)}</li>
                <li>Toko: ${formatRupiah(r.income.store * incomeYearFactor)}</li>
                <li>Industri: ${formatRupiah(r.income.industrial * incomeYearFactor)}</li>
                <li>Rumah Sakit: ${formatRupiah(r.income.hospital * incomeYearFactor)}</li>
                <li>Kincir Angin: ${formatRupiah(r.income.windTurbine * incomeYearFactor)}</li>
                <li>Menara Air: ${formatRupiah(r.income.waterTower * incomeYearFactor)}</li>
            </ul>
        </div>
        <div class="p-4 border border-red-300 rounded-lg bg-red-50">
            <h3 class="font-bold text-lg mb-2 text-red-700">Pengeluaran (proyeksi tahunan)</h3>
            <p>Pengeluaran Total: <strong>${formatRupiah(r.expenditure.total * incomeYearFactor)}</strong></p>
            <ul class="list-disc ml-6 mt-2">
                <li>Jalan: ${formatRupiah(r.expenditure.road * incomeYearFactor)}</li>
                <li>Taman: ${formatRupiah(r.expenditure.park * incomeYearFactor)}</li>
                <li>Rumah Sakit: ${formatRupiah(r.expenditure.hospital * incomeYearFactor)}</li>
                <li>Kincir Angin: ${formatRupiah(r.expenditure.windTurbine * incomeYearFactor)}</li>
                <li>Menara Air: ${formatRupiah(r.expenditure.waterTower * incomeYearFactor)}</li>
            </ul>
        </div>
        <div class="p-4 border border-blue-300 rounded-lg bg-blue-50">
            <h3 class="font-bold text-lg mb-2 text-blue-700">Statistik Listrik</h3>
            <p>Listrik yang Dihasilkan: <strong>${r.power.generated}</strong></p>
            <p>Listrik yang Digunakan: <strong>${r.power.used}</strong></p>
        </div>
        <div class="p-4 border border-blue-300 rounded-lg bg-blue-50">
            <h3 class="font-bold text-lg mb-2 text-blue-700">Statistik Air</h3>
            <p>Air yang Dihasilkan: <strong>${r.water.generated}</strong></p>
            <p>Air yang Digunakan: <strong>${r.water.used}</strong></p>
        </div>
    `;
    reportModal.classList.add('modal-show');
}
function hideReport() { reportModal.classList.remove('modal-show'); }

/* -------------- Info modal -------------- */
function updateInfoModal() {
    if (!selectedBuilding) return;
    const b = selectedBuilding;
    const stats = buildingStats[b.type] || {};
    let html = `<h3 class="font-bold text-lg mb-1">${stats.name || b.type}</h3>
                <p>Posisi: (${Math.floor(b.x / gridSize)}, ${Math.floor(b.y / gridSize)})</p>`;

    if (b.type === 'house') {
        const taxPerHouse = (b.population || 0) * incomePerPersonPerSecond * (taxRate/100);
        html += `<p>Populasi: ${b.population || 0} orang</p>
                 <p>Kapasitas Maks: ${stats.populationCapacity || 0} orang</p>
                 <p>Kebahagiaan: ${b.needs ? b.needs.happiness : 0}%</p>
                 <p>Pajak Bangunan: ${formatRupiah(taxPerHouse)}/detik</p>`;
    } else if (b.type === 'hospital') {
        const taxGain = (b.currentPatients || 0) * stats.treatmentCost;
        html += `<p>Kapasitas Pasien: ${stats.patientCapacity}</p>
                 <p>Pasien Saat Ini: ${b.currentPatients || 0}</p>
                 <p>Pekerja Dibutuhkan: ${stats.workersRequired}</p>
                 <p>Pekerja Ditugaskan: ${b.workersAssigned || 0}</p>
                 <p>Pendapatan Pengobatan: ${formatRupiah(taxGain)}/detik</p>
                 <p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
    } else if (b.type === 'waterTower') {
        html += `<p>Kapasitas Suplai Air: ${stats.waterOutput}</p>
                 <p>Air Terjual: ${b.waterSold || 0} unit</p>
                 <p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
    } else if (b.type === 'windTurbine') {
        html += `<p>Kapasitas Listrik: ${stats.powerOutput}</p>
                 <p>Listrik Terjual: ${b.powerSold || 0} unit</p>
                 <p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
    } else {
        // generic for store/industrial etc
        const taxGain = (baseIncomePerWorker[b.type] || 0) * (b.workersAssigned || 0) * (taxRate/100);
        html += `<p>Status Listrik: ${b.isPowered ? 'Tersedia' : 'Tidak Tersedia'}</p>
                 <p>Status Air: ${b.hasWater ? 'Tersedia' : 'Tidak Tersedia'}</p>
                 <p>Pekerja Dibutuhkan: ${stats.workersRequired || 0}</p>
                 <p>Pekerja Ditugaskan: ${b.workersAssigned || 0}</p>
                 <p>Pajak Bangunan: ${formatRupiah(taxGain)}/detik</p>`;
        if (stats.maintenance) html += `<p>Biaya Perawatan: ${formatRupiah(stats.maintenance)}/detik</p>`;
    }

    infoModalContent.innerHTML = html;
}
function showInfoModal(building) { selectedBuilding = building; updateInfoModal(); infoModal.classList.add('modal-show'); }
function hideInfoModal() { infoModal.classList.remove('modal-show'); selectedBuilding = null; }

/* -------------- Drawing helpers -------------- */
function drawHouse(x,y,w,h,color){
    ctx.fillStyle=color;
    ctx.beginPath();
    ctx.moveTo(x,y+h);
    ctx.lineTo(x, y + h*0.4);
    ctx.lineTo(x + w*0.5, y);
    ctx.lineTo(x + w, y + h*0.4);
    ctx.lineTo(x + w, y + h);
    ctx.closePath();
    ctx.fill();
    ctx.strokeStyle = '#334155';
    ctx.stroke();

    ctx.fillStyle = '#6b7280';
    ctx.fillRect(x + w*0.7, y + h*0.1, w*0.1, h*0.2);

    ctx.fillStyle = '#4a5568';
    ctx.fillRect(x + w*0.4, y + h*0.6, w*0.2, h*0.4);

    ctx.fillStyle = '#94a3b8';
    ctx.fillRect(x + w*0.15, y + h*0.5, w*0.2, h*0.2);
    ctx.fillRect(x + w*0.65, y + h*0.5, w*0.2, h*0.2);
}

function drawStore(x,y,w,h,color){
    ctx.fillStyle = color;
    ctx.fillRect(x, y + h*0.2, w, h*0.8);
    ctx.strokeStyle = '#334155';
    ctx.strokeRect(x, y + h*0.2, w, h*0.8);

    ctx.fillStyle = '#334155';
    ctx.fillRect(x, y, w, h*0.2);

    ctx.fillStyle = '#6b7280';
    ctx.fillRect(x + w*0.8, y, w*0.1, h*0.2);

    ctx.fillStyle = '#94a3b8';
    ctx.fillRect(x + w*0.2, y + h*0.4, w*0.2, h*0.2);
    ctx.fillRect(x + w*0.6, y + h*0.4, w*0.2, h*0.2);

    ctx.fillStyle = '#4a5568';
    ctx.fillRect(x + w*0.45, y + h*0.6, w*0.1, h*0.4);
}

function drawIndustrial(x,y,w,h,color){
    ctx.fillStyle = color;
    ctx.fillRect(x, y + h*0.5, w, h*0.5);

    ctx.fillStyle = '#4b5563';
    ctx.beginPath();
    ctx.moveTo(x, y + h*0.5);
    ctx.lineTo(x + w, y + h*0.2);
    ctx.lineTo(x + w, y + h*0.5);
    ctx.closePath();
    ctx.fill();
    ctx.strokeStyle = '#374151';
    ctx.stroke();

    ctx.fillStyle = '#9ca3af';
    ctx.fillRect(x + w*0.1, y + h*0.15, w*0.08, h*0.35);

    ctx.fillStyle = '#d1d5db';
    ctx.fillRect(x + w*0.2, y + h*0.65, w*0.2, h*0.15);
    ctx.fillRect(x + w*0.6, y + h*0.65, w*0.2, h*0.15);
}

function drawPark(x,y,w,h,flowers){
    const grassHeight = 2;
    ctx.fillStyle = '#4CAF50';
    ctx.fillRect(x, y + h - grassHeight, w, grassHeight);

    (flowers || []).forEach(f => {
        ctx.strokeStyle = f.stemColor;
        ctx.lineWidth = 1;
        ctx.beginPath();
        ctx.moveTo(x + f.x, y + f.y);
        ctx.lineTo(x + f.x, y + f.y - f.height);
        ctx.stroke();

        ctx.fillStyle = f.headColor;
        if (f.type === 'circle') {
            ctx.beginPath();
            ctx.arc(x + f.x, y + f.y - f.height, f.headSize, 0, Math.PI*2);
            ctx.fill();
        } else {
            ctx.fillRect(x + f.x - f.headSize/2, y + f.y - f.height - f.headSize/2, f.headSize, f.headSize);
        }
    });
}

function drawHospital(x,y,w,h,color){
    ctx.fillStyle = color;
    ctx.fillRect(x + w*0.1, y + h*0.2, w*0.8, h*0.8);
    ctx.fillRect(x + w*0.1, y + h*0.1, w*0.8, h*0.1);

    ctx.fillStyle = '#fff';
    const signW = w*0.4; const signH = h*0.2;
    const signX = x + (w - signW)/2; const signY = y;
    ctx.fillRect(signX, signY, signW, signH);

    ctx.fillStyle = '#a71c1c';
    const crossSize = Math.min(signW, signH)*0.7;
    const crossX = signX + (signW - crossSize)/2;
    const crossY = signY + (signH - crossSize)/2;
    ctx.fillRect(crossX + crossSize*0.4, crossY, crossSize*0.2, crossSize);
    ctx.fillRect(crossX, crossY + crossSize*0.4, crossSize, crossSize*0.2);

    ctx.fillStyle = '#fff';
    const doorW = w*0.2; const doorH = h*0.2;
    ctx.fillRect(x + (w - doorW)/2, y + h*0.8, doorW, doorH);

    const winW = w*0.2; const winH = h*0.15;
    ctx.fillRect(x + w*0.15, y + h*0.25, winW, winH);
    ctx.fillRect(x + w*0.65, y + h*0.25, winW, winH);
    ctx.fillRect(x + w*0.15, y + h*0.5, winW, winH);
    ctx.fillRect(x + w*0.65, y + h*0.5, winW, winH);
}

function drawWindTurbine(x,y,w,h){
    const baseX = x + w*0.4;
    const baseY = y + h*0.8;
    ctx.fillStyle = '#a0a0a0';
    ctx.beginPath();
    ctx.moveTo(baseX, baseY);
    ctx.lineTo(baseX + w*0.2, baseY);
    ctx.lineTo(baseX + w*0.15, y + h*0.6);
    ctx.lineTo(baseX + w*0.05, y + h*0.6);
    ctx.closePath();
    ctx.fill();

    const towerX = x + w*0.45, towerY = y + h*0.6, towerW = w*0.1, towerH = h*0.4;
    ctx.fillStyle = '#94a3b8';
    ctx.fillRect(towerX, towerY, towerW, towerH);

    const hubX = towerX + towerW/2, hubY = towerY - h*0.05;
    ctx.save();
    ctx.translate(hubX, hubY);
    const rotationAngle = (Date.now() / 500) % (2*Math.PI);
    ctx.rotate(rotationAngle);
    const bladeWidth = w*0.07, bladeLength = w*0.4;
    ctx.fillStyle = '#6b7280';
    ctx.fillRect(-bladeLength, -bladeWidth/2, bladeLength*2, bladeWidth);
    ctx.fillRect(-bladeWidth/2, -bladeLength, bladeWidth, bladeLength*2);
    ctx.restore();

    ctx.fillStyle = '#4b5563';
    ctx.beginPath();
    ctx.arc(hubX, hubY, w*0.05, 0, Math.PI*2);
    ctx.fill();
}

function drawWaterTower(x, y, w, h) {
    // --- Menggambar Kaki Menara ---
    // --- Tiang Penyangga ---
    ctx.fillStyle = '#0d85ba';
    ctx.strokeStyle = '#000000'; ctx.lineWidth = 0.5;
    ctx.fillRect(x + w * 0.3, y + h * 0.7, w * 0.1, h * 0.3);
    ctx.strokeRect(x + w * 0.3, y + h * 0.7, w * 0.1, h * 0.3);
    ctx.fillRect(x + w * 0.65, y + h * 0.7, w * 0.1, h * 0.3);
    ctx.strokeRect(x + w * 0.65, y + h * 0.7, w * 0.1, h * 0.3);

    // Fungsi untuk menggambar kotak dengan sudut melengkung.
    function drawRoundedRect(rectX, rectY, rectW, rectH, rectRadius) {
        ctx.beginPath();
        ctx.moveTo(rectX + rectRadius, rectY);
        ctx.lineTo(rectX + rectW - rectRadius, rectY);
        ctx.arcTo(rectX + rectW, rectY, rectX + rectW, rectY + rectRadius, rectRadius);
        ctx.lineTo(rectX + rectW, rectY + rectH - rectRadius);
        ctx.arcTo(rectX + rectW, rectY + rectH, rectX + rectW - rectRadius, rectY + rectH, rectRadius);
        ctx.lineTo(rectX + rectRadius, rectY + rectH);
        ctx.arcTo(rectX, rectY + rectH, rectX, rectY + rectH - rectRadius, rectRadius);
        ctx.lineTo(rectX, rectY + rectRadius);
        ctx.arcTo(rectX, rectY, rectX + rectRadius, rectY, rectRadius);
        ctx.closePath();
    }

    // --- Menggambar Atap ---
    // --- bagian 3 ---
    ctx.fillStyle = '#38bdf8';
    drawRoundedRect(x + w * 0.38, y + h * 0.17, w * 0.28, h * 0.15, 3);
    ctx.fill();
    ctx.stroke()

    // --- bagian 2 ---
    ctx.fillStyle = '#38bdf8';
    drawRoundedRect(x + w * 0.37, y + h * 0.12, w * 0.3, h * 0.1, 3);
    ctx.fill();
    ctx.stroke()

    // --- bangian 4 ---
    ctx.fillStyle = '#38bdf8';
    drawRoundedRect(x + w * 0.32, y + h * 0.22, w * 0.4, h * 0.15, 3);
    ctx.fill();
    ctx.stroke()

    // --- Menggambar Tabung ---
    ctx.fillStyle = '#38bdf8';
    drawRoundedRect(x + w * 0.3, y + h * 0.27, w * 0.45, h * 0.53, 3.5);
    ctx.fill();
    ctx.stroke()
}

/* drawRoad - dynamic dashed center */
function drawRoad(x,y,w,h,color){
    const drawX = x - mapOffset.x;
    const drawY = y - mapOffset.y;
    ctx.fillStyle = color;
    ctx.fillRect(drawX, drawY, w, h);

    const tileX = Math.floor(x / gridSize);
    const tileY = Math.floor(y / gridSize);

    const hasTop = findBuilding(tileX, tileY-1)?.type === 'road';
    const hasBottom = findBuilding(tileX, tileY+1)?.type === 'road';
    const hasLeft = findBuilding(tileX-1, tileY)?.type === 'road';
    const hasRight = findBuilding(tileX+1, tileY)?.type === 'road';

    const neighborCount = [hasTop,hasBottom,hasLeft,hasRight].filter(Boolean).length;
    if (neighborCount >= 3) return;

    function drawDashedLine(x1,y1,x2,y2){
        ctx.beginPath();
        ctx.setLineDash([5,5]);
        ctx.moveTo(x1,y1);
        ctx.lineTo(x2,y2);
        ctx.strokeStyle = '#f8fafc';
        ctx.lineWidth = 2;
        ctx.stroke();
        ctx.setLineDash([]);
    }

    const centerX = drawX + w/2;
    const centerY = drawY + h/2;

    if (hasTop && hasBottom) drawDashedLine(centerX, drawY, centerX, drawY + h);
    else if (hasLeft && hasRight) drawDashedLine(drawX, centerY, drawX + w, centerY);
    else {
        if (hasTop && hasRight) { drawDashedLine(centerX, drawY, centerX, centerY); drawDashedLine(centerX, centerY, drawX + w, centerY); }
        else if (hasRight && hasBottom) { drawDashedLine(centerX, centerY, drawX + w, centerY); drawDashedLine(centerX, centerY, centerX, drawY + h); }
        else if (hasBottom && hasLeft) { drawDashedLine(centerX, centerY, centerX, drawY + h); drawDashedLine(drawX, centerY, centerX, centerY); }
        else if (hasLeft && hasTop) { drawDashedLine(drawX, centerY, centerX, centerY); drawDashedLine(centerX, drawY, centerX, centerY); }
        else if (hasTop) drawDashedLine(centerX, centerY, centerX, drawY);
        else if (hasBottom) drawDashedLine(centerX, centerY, centerX, drawY + h);
        else if (hasLeft) drawDashedLine(drawX, centerY, centerX, centerY);
        else if (hasRight) drawDashedLine(centerX, centerY, drawX + w, centerY);
    }
}

/* -------------- Game logic -------------- */
function calculateNeeds(){
    const influential = buildings.filter(b => buildingStats[b.type] && buildingStats[b.type].influenceRadius);
    buildings.forEach(building => {
        if (building.type !== 'house') return;
        let happiness = 0;
        influential.forEach(inf => {
            const dx = Math.abs(building.x - inf.x);
            const dy = Math.abs(building.y - inf.y);
            const distBlocks = Math.sqrt(dx*dx + dy*dy) / gridSize;
            const radius = buildingStats[inf.type].influenceRadius;
            if (distBlocks <= radius) {
                if (inf.type === 'park') happiness += 15;
                else if (inf.type === 'hospital') happiness += 25;
            }
        });
        const tileX = Math.floor(building.x / gridSize), tileY = Math.floor(building.y / gridSize);
        if (isConnectedToRoad(tileX, tileY)) happiness += 30;
        if (isConnectedToService(building, 'waterTower')) happiness += 10;
        const taxPenalty = taxRate > 10 ? (taxRate - 10) * 2 : 0;
        happiness -= taxPenalty;
        building.needs.happiness = Math.max(0, Math.min(100, Math.floor(happiness + 50)));
    });
}

function restartGame(){
    money = 1000.00;
    population = 0;
    buildings = [];
    mapOffset = {x:0,y:0};
    activeMode = 'move';
    buildingType = null;
    taxRate = 5;
    taxRateSlider.value = taxRate;
    taxRateDisplay.textContent = taxRate;
    totalPowerOutput = 0; totalPowerUsage = 0;
    totalWaterOutput = 0; totalWaterUsage = 0;
    updateButtonStyles();
    hideInfoModal();
}

/* -------------- Input & events -------------- */
function updateButtonStyles(){
    moveModeButton.classList.remove('mode-active');
    destroyModeButton.classList.remove('mode-active');
    Object.values(buildingButtons).forEach(btn => btn && btn.classList.remove('mode-active'));

    if (activeMode === 'move') moveModeButton.classList.add('mode-active');
    else if (activeMode === 'destroy') destroyModeButton.classList.add('mode-active');
    else if (activeMode === 'build' && buildingType) {
        if (buildingButtons[buildingType]) buildingButtons[buildingType].classList.add('mode-active');
    }
}

function setMode(newMode, newType){
    activeMode = newMode;
    if (newType) buildingType = newType;
    updateButtonStyles();
    if (newMode === 'build') { if (!isPopupMenuOpen) togglePopupMenu(); }
    else { if (isPopupMenuOpen) togglePopupMenu(); }
}

function togglePopupMenu(){
    isPopupMenuOpen = !isPopupMenuOpen;
    popupMenu.classList.toggle('show', isPopupMenuOpen);
}

/* Keyboard shortcuts mapping */
const shortcuts = {
    m: () => setMode('move', null),
    x: () => setMode('destroy', null),
    h: () => setMode('build', 'house'),
    p: () => setMode('build', 'park'),
    t: () => setMode('build', 'store'),
    i: () => setMode('build', 'industrial'),
    j: () => setMode('build', 'road'),
    o: () => setMode('build', 'hospital'),
    l: () => setMode('build', 'windTurbine'),
    a: () => setMode('build', 'waterTower'),
    g: () => guideModal.classList.add('modal-show'),
    r: () => restartGame(),
    k: () => showReport()
};

/* Attach event listeners (single place) */
function attachListeners(){
    window.addEventListener('keydown', e => {
        const key = (e.key || '').toLowerCase();
        keys[key] = true;
        const action = shortcuts[key];
        if (action) { e.preventDefault(); action(); }
    });
    window.addEventListener('keyup', e => { keys[(e.key||'').toLowerCase()] = false; });

    const allMobileButtons = [
        landscapeControls.up, landscapeControls.down, landscapeControls.left, landscapeControls.right,
        portraitControls.up, portraitControls.down, portraitControls.left, portraitControls.right
    ];
    allMobileButtons.forEach(btn => {
        if (!btn) return;
        btn.addEventListener('pointerdown', e => {
            e.preventDefault();
            const id = e.currentTarget.id;
            if (id.includes('up')) touchControls.up = true;
            if (id.includes('down')) touchControls.down = true;
            if (id.includes('left')) touchControls.left = true;
            if (id.includes('right')) touchControls.right = true;
        });
        btn.addEventListener('pointerup', e => {
            e.preventDefault();
            const id = e.currentTarget.id;
            if (id.includes('up')) touchControls.up = false;
            if (id.includes('down')) touchControls.down = false;
            if (id.includes('left')) touchControls.left = false;
            if (id.includes('right')) touchControls.right = false;
        });
        btn.addEventListener('pointerleave', e => {
            e.preventDefault();
            const id = e.currentTarget.id;
            if (id.includes('up')) touchControls.up = false;
            if (id.includes('down')) touchControls.down = false;
            if (id.includes('left')) touchControls.left = false;
            if (id.includes('right')) touchControls.right = false;
        });
    });

    canvas.addEventListener('click', e => {
        const rect = canvas.getBoundingClientRect();
        const scaleX = canvas.width / rect.width;
        const scaleY = canvas.height / rect.height;
        const mouseX = (e.clientX - rect.left) * scaleX;
        const mouseY = (e.clientY - rect.top) * scaleY;
        const tileX = Math.floor((mouseX + mapOffset.x) / gridSize);
        const tileY = Math.floor((mouseY + mapOffset.y) / gridSize);

        if (activeMode === 'move') {
            const clickedBuilding = findBuilding(tileX, tileY);
            if (clickedBuilding) showInfoModal(clickedBuilding);
            else hideInfoModal();
        } else if (activeMode === 'build' || activeMode === 'destroy') {
            if (activeMode === 'build') {
                const existing = findBuilding(tileX, tileY);
                const stats = buildingStats[buildingType];
                if (!stats) { showMessage('Pilih jenis bangunan terlebih dahulu.'); return; }
                const cost = stats.cost;
                if (!existing && money >= cost) {
                    const newB = {
                        id: Date.now(),
                        x: tileX * gridSize,
                        y: tileY * gridSize,
                        type: buildingType,
                        color: stats.color,
                        population: stats.populationCapacity || 0,
                        currentPatients: buildingType === 'hospital' ? 0 : null,
                        needs: { happiness: 0 },
                        workersAssigned: 0,
                        isPowered: false,
                        hasWater: false
                    };
                    if (buildingType === 'park') {
                        newB.flowers = [];
                        const flowerTypes = [
                            { stemColor:'#8B4513', headColor:'#FF6347', height:10, headSize:4, type:'circle' },
                            { stemColor:'#228B22', headColor:'#DA70D6', height:14, headSize:5, type:'square' },
                            { stemColor:'#6B8E23', headColor:'#ADD8E6', height:12, headSize:6, type:'circle' },
                            { stemColor:'#556B2F', headColor:'#FFA07A', height:16, headSize:3, type:'square' }
                        ];
                        const numFlowers = Math.floor(gridSize / 10);
                        for (let i=0;i<numFlowers;i++){
                            const f = flowerTypes[Math.floor(Math.random()*flowerTypes.length)];
                            newB.flowers.push({ x: Math.random()*(gridSize - 10)+5, y: gridSize, stemColor:f.stemColor, headColor:f.headColor, height:f.height, headSize:f.headSize, type:f.type });
                        }
                    }
                    buildings.push(newB);
                    money -= cost;
                    calculateNeeds();
                } else if (existing) showMessage('Lokasi ini sudah terisi!');
                else if (money < cost) showMessage('Uang tidak cukup!');
            } else if (activeMode === 'destroy') {
                const idx = buildings.findIndex(b => Math.floor(b.x / gridSize) === tileX && Math.floor(b.y / gridSize) === tileY);
                if (idx !== -1) {
                    const removed = buildings.splice(idx,1)[0];
                    const refund = (buildingStats[removed.type]?.cost || 0) * 0.5;
                    money += refund;
                    showMessage(`Bangunan dihancurkan! Uang dikembalikan: ${formatRupiah(refund)}.`);
                    calculateNeeds();
                }
            }
        }
    });

    taxRateSlider.addEventListener('input', e => {
        taxRate = parseInt(e.target.value);
        taxRateDisplay.textContent = taxRate;
        calculateNeeds();
    });

    buildMenuButton.addEventListener('click', togglePopupMenu);
    reportButton.addEventListener('click', showReport);
    reportModalCloseButton.addEventListener('click', hideReport);
    moveModeButton.addEventListener('click', ()=> setMode('move', null));
    destroyModeButton.addEventListener('click', ()=> setMode('destroy', null));
    restartButton.addEventListener('click', restartGame);

    guideButton.addEventListener('click', ()=> guideModal.classList.add('modal-show'));
    guideModalCloseButton.addEventListener('click', ()=> guideModal.classList.remove('modal-show'));

    infoModalCloseButton.addEventListener('click', hideInfoModal);

    window.addEventListener('click', event => {
        if (event.target === guideModal) guideModal.classList.remove('modal-show');
        if (event.target === infoModal) hideInfoModal();
        if (event.target === reportModal) hideReport();
        if (isPopupMenuOpen && !popupMenu.contains(event.target) && !buildMenuButton.contains(event.target)) togglePopupMenu();
    });

    for (const type in buildingButtons) {
        if (buildingButtons[type]) {
            buildingButtons[type].addEventListener('click', () => setMode('build', type));
        }
    }
}

/* -------------- Main game loop -------------- */
function gameLoop(){
    // movement
    let moveX = 0, moveY = 0;
    if (keys['arrowup'] || touchControls.up) moveY -= speed;
    if (keys['arrowdown'] || touchControls.down) moveY += speed;
    if (keys['arrowleft'] || touchControls.left) moveX -= speed;
    if (keys['arrowright'] || touchControls.right) moveX += speed;
    mapOffset.x += moveX; mapOffset.y += moveY;

    // limit
    const worldSize = 5000;
    mapOffset.x = Math.max(0, Math.min(worldSize - canvas.width, mapOffset.x));
    mapOffset.y = Math.max(0, Math.min(worldSize - canvas.height, mapOffset.y));

    // population
    let totalPopulation = 0;
    buildings.forEach(b => { if (b.type === 'house') totalPopulation += (b.population || 0); });
    population = totalPopulation;

    // assign workers
    let workersAssigned = 0;
    const businessBuildings = buildings.filter(b => buildingStats[b.type] && buildingStats[b.type].workersRequired);
    businessBuildings.forEach(b => b.workersAssigned = 0);

    let availableWorkers = Math.max(0, population);
    // priority: hospitals
    for (const b of businessBuildings.filter(bb => bb.type === 'hospital')) {
        const need = buildingStats[b.type].workersRequired || 0;
        const assign = Math.min(need, availableWorkers);
        b.workersAssigned = assign;
        availableWorkers -= assign; workersAssigned += assign;
    }
    // others
    for (const b of businessBuildings.filter(bb => bb.type !== 'hospital')) {
        const need = buildingStats[b.type].workersRequired || 0;
        const assign = Math.min(need, availableWorkers);
        b.workersAssigned = assign;
        availableWorkers -= assign; workersAssigned += assign;
    }

    // power/water totals
    totalPowerOutput = 0; totalPowerUsage = 0;
    totalWaterOutput = 0; totalWaterUsage = 0;

    buildings.filter(b => b.type === 'windTurbine').forEach(p => totalPowerOutput += (buildingStats.windTurbine.powerOutput || 0));
    buildings.filter(b => b.type === 'waterTower').forEach(p => totalWaterOutput += (buildingStats.waterTower.waterOutput || 0));

    // determine powered buildings - allocate power
    let availablePower = totalPowerOutput;
    // reset
    buildings.forEach(b => { b.isPowered = false; b.hasWater = false; b.powerSold = 0; b.waterSold = 0; });

    // sort to prefer hospitals? (already assigned workers earlier)
    for (const b of buildings) {
        const stats = buildingStats[b.type] || {};
        if (stats.powerRequired && b.type !== 'hospital') {
            const tileX = Math.floor(b.x / gridSize), tileY = Math.floor(b.y / gridSize);
            if (isConnectedToRoad(tileX, tileY) && availablePower >= (stats.powerRequired || 0)) {
                b.isPowered = true;
                availablePower -= (stats.powerRequired || 0);
                totalPowerUsage += (stats.powerRequired || 0);
            } else {
                b.isPowered = false;
            }
        } else {
            // hospitals or non-power-consuming buildings
            b.isPowered = true;
        }
        // water connection
        b.hasWater = isConnectedToService(b, 'waterTower');
    }

    // compute water usage estimate (simple model)
    for (const b of buildings) {
        if (b.hasWater) {
            if (b.type === 'store') totalWaterUsage += 5;
            else if (b.type === 'industrial') totalWaterUsage += 12;
            else if (b.type === 'park') totalWaterUsage += 2;
            else if (b.type === 'house') totalWaterUsage += 2;
        }
    }

    // distribute sold power among turbines (simple split)
    let remainingPowerToSell = Math.max(0, totalPowerOutput - totalPowerUsage);
    buildings.filter(b => b.type === 'windTurbine').forEach(b => {
        const cap = buildingStats.windTurbine.powerOutput || 0;
        const sell = Math.min(cap, remainingPowerToSell);
        b.powerSold = sell;
        remainingPowerToSell -= sell;
    });

    // distribute sold water among water towers
    let remainingWaterToSell = Math.max(0, totalWaterOutput - totalWaterUsage);
    buildings.filter(b => b.type === 'waterTower').forEach(b => {
        const cap = buildingStats.waterTower.waterOutput || 0;
        const sell = Math.min(cap, remainingWaterToSell);
        b.waterSold = sell;
        remainingWaterToSell -= sell;
    });

    // finances (every incomeInterval)
    if (Date.now() - lastIncomeTime > incomeInterval) {
        let totalIncome = 0, totalExpenditure = 0;
        buildings.forEach(b => {
            const stats = buildingStats[b.type] || {};
            if (b.type === 'house') totalIncome += (b.population || 0) * incomePerPersonPerSecond * (taxRate/100);
            else if (stats.workersRequired) {
                if (b.type === 'hospital') totalIncome += (b.currentPatients || 0) * stats.treatmentCost;
                else if (b.isPowered && (b.workersAssigned || 0) > 0) totalIncome += (baseIncomePerWorker[b.type] || 0) * (b.workersAssigned || 0) * (taxRate/100);
            }
            if (stats.maintenance) totalExpenditure += stats.maintenance;
            if (b.type === 'windTurbine') totalIncome += (b.powerSold || 0) * pricePerUnitPower;
            if (b.type === 'waterTower') totalIncome += (b.waterSold || 0) * pricePerUnitWater;
        });
        money += totalIncome;
        money -= totalExpenditure;
        lastIncomeTime = Date.now();
    }

    // update UI
    moneyDisplay.textContent = formatRupiah(money);
    populationDisplay.textContent = population;
    availableWorkersDisplay.textContent = Math.max(0, population - workersAssigned);
    powerDisplay.textContent = `${totalPowerUsage} / ${totalPowerOutput}`;
    waterDisplay.textContent = `${totalWaterUsage} / ${totalWaterOutput}`;
    taxRateDisplay.textContent = taxRate;

    // draw
    ctx.fillStyle = '#f8fafc';
    ctx.fillRect(0,0,canvas.width,canvas.height);

    // grid lines
    ctx.strokeStyle = '#94a3b8'; ctx.lineWidth = 1;
    const cols = Math.ceil(canvas.width / gridSize) + 1;
    const rows = Math.ceil(canvas.height / gridSize) + 1;
    for (let x=0;x<cols;x++){
        const drawX = (x*gridSize) - (mapOffset.x % gridSize);
        ctx.beginPath();
        ctx.moveTo(drawX, 0);
        ctx.lineTo(drawX, canvas.height);
        ctx.stroke();
    }
    for (let y=0;y<rows;y++){
        const drawY = (y*gridSize) - (mapOffset.y % gridSize);
        ctx.beginPath();
        ctx.moveTo(0, drawY);
        ctx.lineTo(canvas.width, drawY);
        ctx.stroke();
    }

    // draw buildings
    for (const b of buildings) {
        const drawX = b.x - mapOffset.x;
        const drawY = b.y - mapOffset.y;
        if (drawX + gridSize < 0 || drawX > canvas.width || drawY + gridSize < 0 || drawY > canvas.height) continue;
        ctx.fillStyle = b.color || '#ddd';
        if (b.type === 'house') drawHouse(drawX, drawY, gridSize, gridSize, b.color);
        else if (b.type === 'store') drawStore(drawX, drawY, gridSize, gridSize, b.color);
        else if (b.type === 'industrial') drawIndustrial(drawX, drawY, gridSize, gridSize, b.color);
        else if (b.type === 'park') drawPark(drawX, drawY, gridSize, gridSize, b.flowers || []);
        else if (b.type === 'hospital') drawHospital(drawX, drawY, gridSize, gridSize, b.color);
        else if (b.type === 'windTurbine') drawWindTurbine(drawX, drawY, gridSize, gridSize);
        else if (b.type === 'waterTower') drawWaterTower(drawX, drawY, gridSize, gridSize, b.color);
        else if (b.type === 'road') drawRoad(b.x, b.y, gridSize, gridSize, b.color);
        else {
            ctx.fillRect(drawX, drawY, gridSize, gridSize);
            ctx.strokeStyle = '#334155';
            ctx.strokeRect(drawX, drawY, gridSize, gridSize);
        }
    }

    // update info modal if open
    updateInfoModal();

    requestAnimationFrame(gameLoop);
}

/* -------------- Initialize -------------- */
function resizeCanvasAndStart(){
    canvas.width = Math.min(800, window.innerWidth - 40);
    canvas.height = canvas.width * 0.75;
}
function init(){
    attachListeners();
    resizeCanvasAndStart();
    window.addEventListener('resize', () => { resizeCanvasAndStart(); });
    restartGame();
    // start loops
    requestAnimationFrame(gameLoop);
    drawWaterTower(ctx, 50, 50, 100, 150);
}
document.addEventListener('DOMContentLoaded', init);
</script>
</body>
</html>
