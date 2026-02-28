<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CT241 - DASHBOARD COMPTABLE</title>
    <style>
        :root {
            --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4;
            --bg: #f8f9fa; --card-bg: #ffffff; --text-main: #2c3e50; --text-muted: #7f8c8d;
        }
        body { font-family: 'Inter', -apple-system, sans-serif; background: var(--bg); color: var(--text-main); margin: 0; padding: 20px; }
        
        /* HEADER UTILITAIRE */
        .app-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; border-bottom: 2px solid #ddd; padding-bottom: 15px; }
        .app-header h1 { font-size: 24px; margin: 0; font-weight: 800; letter-spacing: -1px; }
        .app-header .status { font-size: 12px; background: #e1f5fe; color: var(--gabon-bleu); padding: 5px 12px; border-radius: 20px; font-weight: bold; }

        /* GRILLE DE KPI (Indicateurs Clés) */
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 15px; margin-bottom: 30px; }
        .kpi-card { background: var(--card-bg); padding: 20px; border-radius: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); border-top: 4px solid #eee; }
        .kpi-card.green { border-top-color: var(--gabon-vert); }
        .kpi-card.blue { border-top-color: var(--gabon-bleu); }
        .kpi-card.yellow { border-top-color: var(--gabon-jaune); }
        .kpi-card span { font-size: 11px; color: var(--text-muted); text-transform: uppercase; font-weight: bold; }
        .kpi-card div { font-size: 22px; font-weight: 800; margin-top: 8px; }

        /* FILTRES */
        .controls { display: flex; gap: 10px; margin-bottom: 20px; }
        .btn-filter { flex: 1; padding: 10px; border: 1px solid #ddd; background: white; border-radius: 8px; cursor: pointer; font-weight: 600; font-size: 13px; }
        .btn-filter.active { background: var(--text-main); color: white; border-color: var(--text-main); }

        /* TABLEAU DE BORD */
        .data-panel { background: var(--card-bg); border-radius: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); overflow: hidden; }
        .data-panel-header { padding: 15px 20px; background: #fdfdfd; border-bottom: 1px solid #eee; font-weight: bold; display: flex; justify-content: space-between; }
        
        table { width: 100%; border-collapse: collapse; }
        th { background: #f9f9f9; text-align: left; padding: 12px 20px; font-size: 11px; color: var(--text-muted); border-bottom: 1px solid #eee; }
        td { padding: 15px 20px; border-bottom: 1px solid #f1f1f1; font-size: 14px; }
        .id-label { font-family: monospace; background: #f1f3f5; padding: 2px 6px; border-radius: 4px; font-weight: bold; font-size: 12px; }
        .amount { text-align: right; font-weight: 800; color: var(--gabon-vert); }
        .timestamp { font-size: 11px; color: var(--text-muted); display: block; }

        .btn-print { width: 100%; margin-top: 20px; padding: 15px; background: #34495e; color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: 0.2s; }
        .btn-print:hover { background: #2c3e50; }

        @media (max-width: 600px) {
            .kpi-grid { grid-template-columns: 1fr 1fr; }
        }
    </style>
</head>
<body>

    <div class="app-header">
        <h1>CT241 <span style="color:var(--gabon-vert)">FINANCE</span></h1>
        <div class="status">● LIVE GABON</div>
    </div>

    <div class="controls">
        <button class="btn-filter active" id="btn-all">Global</button>
        <button class="btn-filter" id="btn-today">Aujourd'hui</button>
        <button class="btn-filter" id="btn-month">Mois en cours</button>
    </div>

    <div class="kpi-grid">
        <div class="kpi-card green">
            <span>Encaissé (Cash)</span>
            <div id="val-encaissé">0</div>
        </div>
        <div class="kpi-card yellow">
            <span>En Attente</span>
            <div id="val-attente">0</div>
        </div>
        <div class="kpi-card blue">
            <span>Missions</span>
            <div id="val-count">0</div>
        </div>
        <div class="kpi-card">
            <span>Moyenne/Client</span>
            <div id="val-avg">0</div>
        </div>
    </div>

    <div class="data-panel">
        <div class="data-panel-header">
            <span>Journal des Recettes</span>
            <span id="label-filter" style="color:var(--gabon-bleu)">TOUT</span>
        </div>
        <table id="main-table">
            <thead>
                <tr>
                    <th>REFERENCE</th>
                    <th>CLIENT</th>
                    <th style="text-align:right">COMMISSION</th>
                </tr>
            </thead>
            <tbody id="table-body">
                </tbody>
        </table>
    </div>

    <button class="btn-print" onclick="window.print()">Générer Rapport Comptable (PDF)</button>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
    import { getDatabase, ref, onValue } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js";

    const firebaseConfig = {
        apiKey: "AIzaSyAPCKRy9NTo4X8nn8YpxAbPtX8SlKj-7sQ",
        authDomain: "cashtransfert-21.firebaseapp.com",
        databaseURL: "https://cashtransfert-21-default-rtdb.firebaseio.com",
        projectId: "cashtransfert-21",
        storageBucket: "cashtransfert-21.firebasestorage.app",
        messagingSenderId: "564831743134",
        appId: "1:564831743134:web:c22a1f53707f0f1dd9df8f"
    };

    const app = initializeApp(firebaseConfig);
    const db = getDatabase(app);
    let filterMode = 'all';

    const updateUI = (data) => {
        const body = document.getElementById('table-body');
        let totalCash = 0, totalWait = 0, count = 0;
        body.innerHTML = "";

        const today = new Date().toLocaleDateString('fr-FR');
        const thisMonth = new Date().getMonth();

        if (data) {
            Object.keys(data).reverse().forEach(key => {
                const m = data[key];
                
                // Filtres
                if(filterMode === 'today' && m.date !== today) return;
                if(filterMode === 'month') {
                    const mMonth = parseInt(m.date.split('/')[1]) - 1;
                    if(mMonth !== thisMonth) return;
                }

                // Calculs
                if(m.etape === 3) {
                    totalCash += m.com;
                    count++;
                    body.innerHTML += `
                        <tr>
                            <td><span class="id-label">${m.id}</span><span class="timestamp">${m.heure} | ${m.date}</span></td>
                            <td><strong>${m.nom}</strong></td>
                            <td class="amount">${m.com.toLocaleString()}</td>
                        </tr>
                    `;
                } else if (m.etape === 2) {
                    totalWait += m.com;
                }
            });
        }

        document.getElementById('val-encaissé').innerText = totalCash.toLocaleString() + " F";
        document.getElementById('val-attente').innerText = totalWait.toLocaleString() + " F";
        document.getElementById('val-count').innerText = count;
        document.getElementById('val-avg').innerText = count > 0 ? Math.round(totalCash/count).toLocaleString() + " F" : "0 F";
    };

    // Events
    const load = () => onValue(ref(db, 'missions'), (snap) => updateUI(snap.val()));
    
    document.getElementById('btn-all').onclick = (e) => { filterMode = 'all'; toggleBtn(e.target); load(); };
    document.getElementById('btn-today').onclick = (e) => { filterMode = 'today'; toggleBtn(e.target); load(); };
    document.getElementById('btn-month').onclick = (e) => { filterMode = 'month'; toggleBtn(e.target); load(); };

    function toggleBtn(target) {
        document.querySelectorAll('.btn-filter').forEach(b => b.classList.remove('active'));
        target.classList.add('active');
        document.getElementById('label-filter').innerText = target.innerText.toUpperCase();
    }

    load();
</script>
</body>
</html>
