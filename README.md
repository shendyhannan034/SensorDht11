<!DOCTYPE html>

<html lang="id">
<head>
    <meta charset="UTF-8">
    <title>Sensor DHT11</title>

    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <style>
        body {
            margin: 0;
            font-family: Arial;
            display: flex;
        }

        /* SIDEBAR */
        .sidebar {
            width: 220px;
            background: #292929;
            color: #292929;
            height: 300vh;
            padding: 20px;
        }

            .sidebar h1 {
                color: #ffffff;
            }

        /* MENU */
        .menu-item {
            padding: 12px;
            margin: 10px 0;
            background: #969696;
            border-radius: 10px;
            cursor: pointer;
            transition: 0.3s;
        }

            .menu-item:hover {
                background: #121212;
                color: #f9f9f9;
            }

            .menu-item.active {
                background: #121212;
                color: #f9f9f9;
            }

        /* MAIN */
        .main {
            flex: 1;
            padding: 20px;
            background: #f9f9f9;
        }

        /* PAGE */
        .page {
            display: none;
            color: #000000;
        }

            .page.active {
                display: block;
            }

        /* CARD */
        .card {
            background: #292929;
            padding: 40px;
            margin: 20px;
            border-radius: 30px;
            display: inline-block;
            width: 200px;
            text-align: center;
            font-size: 25px;
            color: yellow;
        }
    </style>

</head>

<body>

    <!-- SIDEBAR -->

    <div class="sidebar">
        <h1>DASHBOARD</h1>

        <div class="menu-item active" onclick="showPage('dashboard', this)">Parameter</div>
        <div class="menu-item" onclick="showPage('grafik', this)">Grafik</div>
        <div class="menu-item" onclick="showPage('riwayat', this)">Riwayat</div>
    </div>

    <!-- MAIN -->

    <div class="main">

        <!-- DASHBOARD -->

        <div id="dashboard" class="page active">
            <h2>Parameter</h2>

            <p class="card">Suhu<br><span id="suhu">Loading...</span> °C</p>
            <p class="card">Kelembapan<br><span id="kelembapan">Loading...</span></p>


        </div>

        <!-- GRAFIK -->

        <div id="grafik" class="page">
            <h3>Grafik Suhu</h3>
            <canvas id="suhuChart"></canvas>

            <h3>Grafik Kelembapan</h3>
            <canvas id="kelembapanChart"></canvas>

        </div>

        <!-- RIWAYAT -->

        <div id="riwayat" class="page">
            <h2>Riwayat Data</h2>


            <table border="1" width="100%">
                <thead>
                    <tr>
                        <th>Tanggal & Waktu</th>
                        <th>Suhu °C</th>
                        <th>Kelembapan</th>
                    </tr>

                </thead>
                <tbody id="table"></tbody>
            </table>


        </div>

    </div>


    <script>

        const urlLive = "https://sensordht11-c9561-default-rtdb.asia-southeast1.firebasedatabase.app/DHT11/live.json";
        const urlHistory = "https://sensordht11-c9561-default-rtdb.asia-southeast1.firebasedatabase.app/DHT11/history.json";
        // ================= NAVIGASI =================
        function showPage(page, el) {

            document.querySelectorAll(".page").forEach(p => p.classList.remove("active"));
            document.getElementById(page).classList.add("active");

            document.querySelectorAll(".menu-item").forEach(m => m.classList.remove("active"));
            el.classList.add("active");
        }

        // ================= SIMULASI GRAFIK =================
        let dataSuhu = [];
        let dataKelembapan = [];
        let labelWaktu = [];

        const suhuChart = new Chart(document.getElementById("suhuChart"), {
            type: "line",
            data: {
                labels: labelWaktu,
                datasets: [{
                    label: "Suhu",
                    data: dataSuhu,
                    borderWidth: 2
                }]
            }
        });

        const kelembapanChart = new Chart(document.getElementById("kelembapanChart"), {
            type: "line",
            data: {
                labels: labelWaktu,
                datasets: [{
                    label: "Kelembapan",
                    data: dataKelembapan,
                    borderWidth: 2
                }]
            }
        });

        // ================= SISTEM AMBIL DATA =================
        function ambilLive() {
            fetch(urlLive)
                .then(res => res.json())
                .then(data => {

                    if (!data) return;

                    // PARAMETER
                    document.getElementById("suhu").innerText = data.suhu;
                    document.getElementById("kelembapan").innerText = data.kelembapan;

                    // GRAFIK
                    let time = new Date().toLocaleTimeString();

                    labelWaktu.push(time);
                    dataSuhu.push(data.suhu);
                    dataKelembapan.push(data.kelembapan);

                    if (labelWaktu.length > 10) {
                        labelWaktu.shift();
                        dataSuhu.shift();
                        dataKelembapan.shift();
                    }

                    suhuChart.update();
                    kelembapanChart.update();
                });
        }

        function ambilHistory() {
            fetch(urlHistory)
                .then(res => res.json())
                .then(data => {

                    if (!data) return;

                    let table = document.getElementById("table");
                    table.innerHTML = "";

                    Object.keys(data).sort().forEach(key => {
                        let item = data[key];

                        let row = `
                    <tr>
                        <td>${key}</td>
                        <td>${item.suhu}</td>
                        <td>${item.kelembapan}</td>
                    </tr>
                `;

                        table.innerHTML += row;
                    });
                });
        }

        setInterval(ambilLive, 2000);     // realtime
        setInterval(ambilHistory, 10000); // riwayat
        ambilLive();
        ambilHistory();
    </script>

</body>
</html>
