<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Live Dashboard Performa Gov & Energy 2026</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #f8fafc; }
        .chart-container { position: relative; height: 260px; width: 100%; }
    </style>
</head>
<body class="text-slate-800">

    <header class="bg-slate-900 text-white shadow-md sticky top-0 z-50 px-6 py-4 flex flex-col md:flex-row justify-between items-center gap-4">
        <div class="flex items-center gap-3">
            <div class="bg-indigo-600 p-2 rounded-lg text-white">
                <i class="fa-solid fa-chart-line text-xl"></i>
            </div>
            <div>
                <h1 class="text-xl font-bold tracking-tight">Gov & Energy Live Dashboard</h1>
                <p class="text-xs text-slate-400">Koneksi Real-time Google Sheets (Tahun 2026)</p>
            </div>
        </div>
        
        <div class="flex flex-wrap gap-3 items-center w-full md:w-auto justify-end">
            <div class="flex items-center gap-2 bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700 text-xs">
                <span id="statusDot" class="w-2 h-2 bg-amber-500 rounded-full animate-pulse"></span>
                <span id="updateStatus" class="text-slate-300 font-medium">Menghubungkan data...</span>
            </div>
            <div class="flex flex-col min-w-[130px]">
                <select id="filterMonth" class="bg-slate-800 text-white border border-slate-700 rounded-lg px-3 py-1.5 text-sm focus:outline-none" onchange="updateDashboard()">
                    <option value="ALL">Semua Bulan</option>
                    <option value="JAN">Januari</option>
                    <option value="FEB">Februari</option>
                    <option value="MAR">Maret</option>
                    <option value="APR">April</option>
                    <option value="MAY">Mei</option>
                </select>
            </div>
            <div class="flex flex-col min-w-[170px]">
                <select id="filterProject" class="bg-slate-800 text-white border border-slate-700 rounded-lg px-3 py-1.5 text-sm focus:outline-none" onchange="updateDashboard()">
                    <option value="ALL">Semua Proyek</option>
                </select>
            </div>
        </div>
    </header>

    <main class="p-4 md:p-6 max-w-[1600px] mx-auto space-y-6">
        
        <div id="errorAlert" class="hidden bg-rose-50 border border-rose-200 text-rose-700 p-4 rounded-xl text-sm flex flex-col gap-2 shadow-xs">
            <div class="flex items-center gap-2 font-bold text-rose-800">
                <i class="fa-solid fa-triangle-exclamation"></i> Terjadi Masalah Sinkronisasi Data GSheets
            </div>
            <p id="errorMsg">Pesan Eror Mendetail Akan Muncul di Sini...</p>
        </div>

        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-4">
            <div class="bg-white p-4 rounded-xl border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs text-slate-500 font-semibold uppercase tracking-wider">Total Trafik</p>
                    <h3 id="kpiTotal" class="text-2xl font-bold mt-1 text-slate-900">-</h3>
                </div>
                <div class="bg-indigo-50 p-3 rounded-xl text-indigo-600"><i class="fa-solid fa-cloud-arrow-down text-xl"></i></div>
            </div>
            <div class="bg-white p-4 rounded-xl border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs text-slate-500 font-semibold uppercase tracking-wider">Trafik Voice</p>
                    <h3 id="kpiVoice" class="text-2xl font-bold mt-1 text-emerald-600">-</h3>
                </div>
                <div class="bg-emerald-50 p-3 rounded-xl text-emerald-600"><i class="fa-solid fa-phone text-xl"></i></div>
            </div>
            <div class="bg-white p-4 rounded-xl border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs text-slate-500 font-semibold uppercase tracking-wider">Trafik Digital</p>
                    <h3 id="kpiDigital" class="text-2xl font-bold mt-1 text-blue-600">-</h3>
                </div>
                <div class="bg-blue-50 p-3 rounded-xl text-blue-600"><i class="fa-solid fa-circle-nodes text-xl"></i></div>
            </div>
            <div class="bg-white p-4 rounded-xl border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs text-slate-500 font-semibold uppercase tracking-wider">Rata-rata SCR</p>
                    <h3 id="kpiSCR" class="text-2xl font-bold mt-1 text-amber-600">-</h3>
                </div>
                <div class="bg-amber-50 p-3 rounded-xl text-amber-600"><i class="fa-solid fa-star text-xl"></i></div>
            </div>
            <div class="bg-white p-4 rounded-xl border border-slate-200 flex items-center justify-between">
                <div>
                    <p class="text-xs text-slate-500 font-semibold uppercase tracking-wider">Target Tercapai</p>
                    <h3 id="kpiTarget" class="text-2xl font-bold mt-1 text-purple-600">-</h3>
                </div>
                <div class="bg-purple-50 p-3 rounded-xl text-purple-600"><i class="fa-solid fa-circle-check text-xl"></i></div>
            </div>
        </div>

        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
            <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-xs">
                <h4 class="text-sm font-bold text-slate-700 mb-4"><i class="fa-solid fa-chart-line text-indigo-500 mr-2"></i>Tren Peningkatan Trafik Bulanan</h4>
                <div class="chart-container"><canvas id="chartTrend"></canvas></div>
            </div>
            <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-xs">
                <h4 class="text-sm font-bold text-slate-700 mb-4"><i class="fa-solid fa-chart-pie text-emerald-500 mr-2"></i>Persentase Interaksi Interval Trafik</h4>
                <div class="chart-container"><canvas id="chartComposition"></canvas></div>
            </div>
            <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-xs">
                <h4 class="text-sm font-bold text-slate-700 mb-4"><i class="fa-solid fa-chart-bar text-amber-500 mr-2"></i>Rata-rata Success Rate (SCR) per Proyek</h4>
                <div class="chart-container"><canvas id="chartSCRProject"></canvas></div>
            </div>
            <div class="bg-white p-4 rounded-xl border border-slate-200 shadow-xs">
                <h4 class="text-sm font-bold text-slate-700 mb-4"><i class="fa-solid fa-chart-simple text-blue-500 mr-2"></i>Distribusi Total Trafik per Proyek</h4>
                <div class="chart-container"><canvas id="chartVolumeProject"></canvas></div>
            </div>
        </div>

        <div class="bg-white rounded-xl border border-slate-200 shadow-xs overflow-hidden">
            <div class="p-4 bg-slate-50 border-b border-slate-200 flex flex-col sm:flex-row justify-between items-center gap-4">
                <h4 class="text-sm font-bold text-slate-700">Tabel Detail Transaksi Live</h4>
                <button onclick="exportToCSV()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs px-4 py-2 rounded-lg font-medium transition cursor-pointer">
                    <i class="fa-solid fa-file-excel mr-2"></i>Export CSV
                </button>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-100 text-[11px] font-bold uppercase text-slate-600 border-b border-slate-200">
                            <th class="px-4 py-3">Tanggal</th>
                            <th class="px-4 py-3">Proyek</th>
                            <th class="px-4 py-3">Bulan</th>
                            <th class="px-4 py-3 text-right">Trafik Voice</th>
                            <th class="px-4 py-3 text-right">Trafik Digital</th>
                            <th class="px-4 py-3 text-right">Total Trafik</th>
                            <th class="px-4 py-3">Success Rate (SCR)</th>
                            <th class="px-4 py-3">Status</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="divide-y divide-slate-100 text-sm"></tbody>
                </table>
            </div>
            <div class="p-3 bg-slate-50 border-t border-slate-100 flex justify-between items-center text-xs text-slate-500">
                <div id="tableCounter">Menampilkan 0 dari 0 data</div>
                <div class="flex gap-2">
                    <button onclick="prevPage()" id="btnPrev" class="px-2 py-1 bg-white border border-slate-200 rounded-sm hover:bg-slate-50 disabled:opacity-50">Prev</button>
                    <button onclick="nextPage()" id="btnNext" class="px-2 py-1 bg-white border border-slate-200 rounded-sm hover:bg-slate-50 disabled:opacity-50">Next</button>
                </div>
            </div>
        </div>
    </main>

    <script>
        // TAUTAN DATA FIX MENGGUNAKAN EKSPOR DATAVALUE CSV YANG DISETUJUI GOOGLE
        const csvUrl = 'https://docs.google.com/spreadsheets/d/1wWPvnNoOL44xBU393gF4j9DM873BP8ZSh0K71fYHKD8/pub?gid=0&single=true&output=csv';

        let rawData = [];
        let filteredData = [];
        let currentPage = 1;
        const pageSize = 10;
        let chartTrend, chartComposition, chartSCRProject, chartVolumeProject;
        let projectListInitialized = false;

        const projectTargetsFallback = { 'BMKG': 0.95, 'CC POLRI (Mabes)': 0.9, 'KEJAGUNG': 0.95, 'KEMENKOP': 0.9 };

        async function fetchLiveGSheets() {
            setLoadingState(true, "Sinkronisasi...");
            hideError();

            Papa.parse(csvUrl, {
                download: true,
                header: true,
                skipEmptyLines: true,
                dynamicTyping: true,
                complete: function(results) {
                    if (!results.data || results.data.length === 0) {
                        showError("File Google Sheets berhasil terhubung, tetapi datanya kosong. Pastikan Anda memilih sheet yang berisi baris data transaksi pada pengaturan 'Publish to Web'.");
                        setLoadingState(false, "Data Kosong", false);
                        return;
                    }
                    parseAndCleanData(results.data);
                },
                error: function(err) {
                    showError("Gagal mengunduh CSV. Hal ini biasanya karena link spreadsheet belum di-'Publish to Web' atau setelan tipenya bukan Comma-Separated Values (.csv). Deteksi Sistem: " + JSON.stringify(err));
                    setLoadingState(false, "Gagal Sinkronisasi", false);
                }
            });
        }

        function parseAndCleanData(rawRows) {
            try {
                rawData = rawRows.map((row, index) => {
                    // Normalisasi nama kolom (mengatasi huruf besar/kecil campuran di GSheets)
                    let projectVal = row['PROJECT'] || row['Project'] || row['project'];
                    let monthVal = row['MONTH'] || row['Month'] || row['month'];
                    let dateVal = row['DATE'] || row['Date'] || row['date'];
                    
                    let voiceKey = row['Call Offered (Inbound)'] || row['Call Offered'] || 0;
                    let voiceVal = typeof voiceKey === 'string' 
                        ? parseInt(voiceKey.replace(/,/g, '')) || 0 
                        : parseInt(voiceKey) || 0;

                    let digitalKey = row['Total Digital'] || row['Total digital'] || 0;
                    let digitalVal = parseInt(digitalKey) || 0;
                    
                    let scrKey = row['SCR'] || row['scr'] || 0;
                    let scrRaw = parseFloat(scrKey) || 0;
                    let scrClean = scrRaw > 1 ? scrRaw / 100 : scrRaw;

                    let targetKey = row['Target SCR'] || row['Target scr'] || 0;
                    let targetRaw = parseFloat(targetKey) || 0;
                    let targetClean = targetRaw === 0 ? (projectTargetsFallback[projectVal] || 0.95) : targetRaw;

                    return {
                        project: projectVal,
                        month: monthVal,
                        date: dateVal,
                        voice: voiceVal,
                        digital: digitalVal,
                        total: voiceVal + digitalVal,
                        scr: scrClean,
                        target: targetClean
                    };
                }).filter(item => item.project !== undefined && item.project !== null && item.project !== "");

                if (rawData.length === 0) {
                    showError("Kolom 'PROJECT' tidak ditemukan di baris pertama file Google Sheets Anda. Harap pastikan nama header kolom di GSheets ditulis dengan huruf besar/kapital.");
                    setLoadingState(false, "Kolom Tidak Valid", false);
                    return;
                }

                if (!projectListInitialized) {
                    initProjectDropdown();
                }

                updateDashboard();
                const now = new Date();
                setLoadingState(false, "Terhubung: " + now.toLocaleTimeString('id-ID'), true);
            } catch (e) {
                showError("Gagal memproses struktur baris tabel: " + e.message);
                setLoadingState(false, "Eror Pemrosesan", false);
            }
        }

        function initProjectDropdown() {
            const projects = [...new Set(rawData.map(item => item.project))].sort();
            const projectSelect = document.getElementById('filterProject');
            
            // Bersihkan dropdown kecuali opsi pertama
            projectSelect.innerHTML = '<option value="ALL">Semua Proyek</option>';
            
            projects.forEach(proj => {
                if(proj) {
                    const opt = document.createElement('option');
                    opt.value = proj; opt.textContent = proj;
                    projectSelect.appendChild(opt);
                }
            });
            projectListInitialized = true;
        }

        function updateDashboard() {
            const selectedMonth = document.getElementById('filterMonth').value;
            const selectedProject = document.getElementById('filterProject').value;

            filteredData = rawData.filter(item => {
                const matchMonth = (selectedMonth === 'ALL' || item.month === selectedMonth);
                const matchProject = (selectedProject === 'ALL' || item.project === selectedProject);
                return matchMonth && matchProject;
            });

            let totalTrafik = 0, totalVoice = 0, totalDigital = 0, sumSCR = 0, countSCR = 0, targetTercapai = 0, rowsWithTarget = 0;

            filteredData.forEach(item => {
                totalTrafik += item.total;
                totalVoice += item.voice;
                totalDigital += item.digital;
                if (item.scr > 0) {
                    sumSCR += item.scr;
                    countSCR++;
                    if (item.scr >= item.target) targetTercapai++;
                    rowsWithTarget++;
                }
            });

            document.getElementById('kpiTotal').textContent = totalTrafik.toLocaleString('id-ID');
            document.getElementById('kpiVoice').textContent = totalVoice.toLocaleString('id-ID');
            document.getElementById('kpiDigital').textContent = totalDigital.toLocaleString('id-ID');
            document.getElementById('kpiSCR').textContent = countSCR > 0 ? ((sumSCR / countSCR) * 100).toFixed(2) + '%' : '0%';
            document.getElementById('kpiTarget').textContent = rowsWithTarget > 0 ? ((targetTercapai / rowsWithTarget) * 100).toFixed(1) + '%' : '0%';

            renderTable();
            renderCharts();
        }

        function renderTable() {
            const tbody = document.getElementById('tableBody');
            tbody.innerHTML = '';

            const start = (currentPage - 1) * pageSize;
            const end = start + pageSize;
            const pageData = filteredData.slice(start, end);

            if(pageData.length === 0) {
                tbody.innerHTML = `<tr><td colspan="8" class="text-center py-6 text-slate-400">Tidak ada data transaksi ditemukan</td></tr>`;
                document.getElementById('tableCounter').textContent = "Menampilkan 0 dari 0 data";
                return;
            }

            pageData.forEach(item => {
                const tr = document.createElement('tr');
                const scrPct = (item.scr * 100).toFixed(1);
                const isTercapai = item.scr >= item.target;
                const badgeClass = isTercapai ? "bg-emerald-50 text-emerald-700 border-emerald-200" : "bg-rose-50 text-rose-700 border-rose-200";

                tr.innerHTML = `
                    <td class="px-4 py-3 font-mono text-xs">${item.date || '-'}</td>
                    <td class="px-4 py-3 font-semibold">${item.project}</td>
                    <td class="px-4 py-3"><span class="px-2 py-0.5 bg-slate-100 rounded text-xs">${item.month}</span></td>
                    <td class="px-4 py-3 text-right font-mono">${item.voice.toLocaleString('id-ID')}</td>
                    <td class="px-4 py-3 text-right font-mono">${item.digital.toLocaleString('id-ID')}</td>
                    <td class="px-4 py-3 text-right font-semibold text-indigo-600 font-mono">${item.total.toLocaleString('id-ID')}</td>
                    <td class="px-4 py-3">
                        <div class="flex items-center gap-2">
                            <span class="w-10 text-xs font-mono">${scrPct}%</span>
                            <div class="w-20 bg-slate-100 h-2 rounded-full overflow-hidden hidden sm:block border border-slate-200">
                                <div class="${isTercapai?'bg-emerald-500':'bg-amber-500'} h-full" style="width: ${Math.min(scrPct,100)}%"></div>
                            </div>
                        </div>
                    </td>
                    <td class="px-4 py-3"><span class="px-2 py-0.5 border rounded-full text-xs font-medium ${badgeClass}">${isTercapai?'Achieved':'Below Target'}</span></td>
                `;
                tbody.appendChild(tr);
            });

            document.getElementById('tableCounter').textContent = `Menampilkan ${start + 1}-${Math.min(end, filteredData.length)} dari ${filteredData.length} data`;
            document.getElementById('btnPrev').disabled = currentPage === 1;
            document.getElementById('btnNext').disabled = end >= filteredData.length;
        }

        function prevPage() { if (currentPage > 1) { currentPage--; renderTable(); } }
        function nextPage() { if ((currentPage * pageSize) < filteredData.length) { currentPage++; renderTable(); } }

        function renderCharts() {
            const monthsOrder = ['JAN', 'FEB', 'MAR', 'APR', 'MAY'];
            const trendVoice = Array(5).fill(0), trendDigital = Array(5).fill(0), trendTotal = Array(5).fill(0);

            filteredData.forEach(item => {
                const idx = monthsOrder.indexOf(item.month);
                if (idx !== -1) {
                    trendVoice[idx] += item.voice;
                    trendDigital[idx] += item.digital;
                    trendTotal[idx] += item.total;
                }
            });

            if (chartTrend) chartTrend.destroy();
            if (chartComposition) chartComposition.destroy();
            if (chartSCRProject) chartSCRProject.destroy();
            if (chartVolumeProject) chartVolumeProject.destroy();

            chartTrend = new Chart(document.getElementById('chartTrend').getContext('2d'), {
                type: 'line',
                data: {
                    labels: ['Januari', 'Februari', 'Maret', 'April', 'Mei'],
                    datasets: [
                        { label: 'Total Trafik', data: trendTotal, borderColor: '#6366f1', backgroundColor: 'rgba(99, 102, 241, 0.05)', fill: true, tension: 0.2 },
                        { label: 'Voice Inbound', data: trendVoice, borderColor: '#10b981', fill: false, borderDash: [5,5] },
                        { label: 'Digital Omnichannel', data: trendDigital, borderColor: '#3b82f6', fill: false, borderDash: [2,2] }
                    ]
                },
                options: { responsive: true, maintainAspectRatio: false }
            });

            let sumV = trendVoice.reduce((a,b)=>a+b, 0), sumD = trendDigital.reduce((a,b)=>a+b, 0);
            chartComposition = new Chart(document.getElementById('chartComposition').getContext('2d'), {
                type: 'doughnut',
                data: {
                    labels: ['Voice Inbound', 'Digital Omnichannel'],
                    datasets: [{ data: [sumV, sumD], backgroundColor: ['#10b981', '#3b82f6'] }]
                },
                options: { responsive: true, maintainAspectRatio: false }
            });

            const projMap = {};
            filteredData.forEach(item => {
                if(!projMap[item.project]) projMap[item.project] = { total: 0, scrSum: 0, scrCount: 0 };
                projMap[item.project].total += item.total;
                if(item.scr > 0) { projMap[item.project].scrSum += item.scr; projMap[item.project].scrCount++; }
            });

            const projLabels = Object.keys(projMap);
            const projTotals = projLabels.map(p => projMap[p].total);
            const projSCRs = projLabels.map(p => projMap[p].scrCount > 0 ? (projMap[p].scrSum / projMap[p].scrCount) * 100 : 0);

            chartSCRProject = new Chart(document.getElementById('chartSCRProject').getContext('2d'), {
                type: 'bar',
                data: { labels: projLabels, datasets: [{ label: 'Rata-rata SCR (%)', data: projSCRs, backgroundColor: 'rgba(245, 158, 11, 0.85)' }] },
                options: { responsive: true, maintainAspectRatio: false, scales: { y: { min:0, max:100 } } }
            });

            chartVolumeProject = new Chart(document.getElementById('chartVolumeProject').getContext('2d'), {
                type: 'bar',
                data: { labels: projLabels, datasets: [{ label: 'Total Volume', data: projTotals, backgroundColor: 'rgba(59, 130, 246, 0.85)' }] },
                options: { responsive: true, maintainAspectRatio: false }
            });
        }

        function exportToCSV() {
            if (filteredData.length === 0) return alert("Data kosong");
            let csv = "Tanggal,Proyek,Bulan,Voice,Digital,Total,SCR,Target\n";
            filteredData.forEach(item => {
                csv += `${item.date || '-'},"${item.project}",${item.month},${item.voice},${item.digital},${item.total},${(item.scr*100).toFixed(2)}%,${(item.target*100).toFixed(0)}%\n`;
            });
            const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            link.href = URL.createObjectURL(blob);
            link.setAttribute("download", `Live_Performance_Export.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }

        function setLoadingState(isLoading, message, isSuccess = null) {
            const statusText = document.getElementById('updateStatus');
            const statusDot = document.getElementById('statusDot');
            statusText.textContent = message;
            
            if (isLoading) {
                statusDot.className = "w-2 h-2 bg-amber-500 rounded-full animate-pulse";
            } else if (isSuccess === true) {
                statusDot.className = "w-2 h-2 bg-emerald-500 rounded-full";
            } else if (isSuccess === false) {
                statusDot.className = "w-2 h-2 bg-rose-500 rounded-full";
            }
        }

        function showError(message) {
            document.getElementById('errorAlert').classList.remove('hidden');
            document.getElementById('errorMsg').textContent = message;
        }

        function hideError() {
            document.getElementById('errorAlert').classList.add('hidden');
        }

        fetchLiveGSheets();
        setInterval(fetchLiveGSheets, 30000);
    </script>
</body>
</html>
