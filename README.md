<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CT241 - FINANCE & DASHBOARD</title>
    <style>
        :root {
            --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4;
            --bg: #f4f7f6; --card-bg: #ffffff; --text-main: #2d3436; --text-muted: #636e72;
        }
        body { font-family: 'Inter', -apple-system, sans-serif; background: var(--bg); color: var(--text-main); margin: 0; padding: 0; }

        /* ÉCRAN DE VERROUILLAGE */
        #lock-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: #1e272e; display: flex; align-items: center; justify-content: center; z-index: 10000;
        }
        .lock-box { background: white; padding: 30px; border-radius: 20px; text-align: center; width: 85%; max-width: 320px; border-top: 10px solid var(--gabon-jaune); box-shadow: 0 10px 25px rgba(0,0,0,0.3); }
        .lock-box input { width: 100%; padding: 15px; margin: 15px 0; border: 2px solid #eee; border-radius: 10px; font-size: 18px; text-align: center; box-sizing: border-box; outline: none; transition: 0.3s; }
        .lock-box input:focus { border-color: var(--gabon-bleu); }
        .btn-unlock { width: 100%; padding: 15px; background: var(--gabon-vert); color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; font-size: 16px; }

        /* DASHBOARD CONTENT */
        #dashboard-content { display: none; padding: 15px; max-width: 800px; margin: auto; }
        
        .header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; border-bottom: 2px solid #ddd; padding-bottom: 10px; }
        .header h1 { font-size: 20px; margin: 0; color: var(--gabon-vert); font-weight: 800; }

        /* KPI CARDS */
        .kpi-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 20px; }
        .kpi-card { background: var(--card-bg); padding: 15px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.05); border-left: 5px solid #ddd; }
        .kpi-card.green { border-left-color: var(--gabon-vert); }
        .kpi-card.yellow { border-left-color: var(--gabon-jaune); }
        .kpi-card span { font-size: 10px; color: var(--text-muted); text-transform: uppercase; font-weight: bold; }
        .kpi-card div { font-size: 18px; font-weight: 800; margin-top: 5px; }

        /* FILTRES */
        .filters { display: flex; gap: 8px; margin-bottom: 20px; }
        .btn-f { flex: 1; padding: 10px; border: none; background: #e2e8f0; border-radius: 8px; cursor: pointer; font-weight: bold; font-size: 12px; transition: 0.2s; }
        .btn-f.active { background: var(--text-main); color: white; }

        /* TABLEAU */
        .panel { background: white; border-radius: 15px; overflow: hidden; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }
        table { width: 100%; border-collapse: collapse; }
        th { background: #f8fafc; text-align: left; padding: 12px 15px; font-size: 10px; color: var(--text-muted); border-bottom: 1px solid #eee; }
        td { padding: 12px 15px; border-bottom: 1px solid #f1f5f9; font-size: 13px; vertical-align: top; }
        
        .id-label { font-family: monospace; background: #2d3436; color: var(--gabon-jaune); padding: 2px 6px; border-radius: 4px; font-weight: bold; font-size: 11px; }
        .date-info { font-size: 10px; color: #95a5a6; display: block; margin-top: 4px; line-height: 1.2; }
        .amount { text-align: right; font-weight: 800; color: var(--gabon-vert); font-size: 14px; }
        .lieu-info { font-size: 11px; color: var(--gabon-bleu); font-weight: 500; }

        .btn-print { width: 100%; margin-top: 20px; padding: 15px; background: #2d3436; color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }

        @media print { .filters, .btn-print, #lock-screen { display: none; } }
    </style>
</head>
<body>

    <div id="lock-screen">
        <div class="lock-box">
            <h2 style="margin:0; font-size: 18px;">CT241 FINANCE 🔐</h2>
            <p id="status-text" style="font-size: 11px; color: #7f8c8d; margin-top: 5px;">Entrez le code d'accès</p>
            <input type="password" id="admin-code" placeholder="Code secret" autocomplete="off">
            <button class="btn-unlock" id="unlock-btn">OUVRIR LE BILAN</button>
            <p id="lock-error" style="color:red; font-size: 11px; margin-top: 10px; display:none; font-weight: bold;">CODE INCORRECT</p>
        </div>
    </div>

    <div id="dashboard-content">
        <div class="header">
            <h1>CT241 <span style="color:var(--text-main)">COMPTA</span></h1>
            <div style="font-size: 10px; background: #e1f5fe; padding: 4px 8px; border-radius: 5px; color: var(--gabon-bleu); font-weight: bold;">LIVE SYNC</div>
        </div>

        <div class="filters">
            <button class="btn-f active" id="f-all">Global</button>
            <button class="btn-f" id="f-today">Aujourd'hui</button>
            <button class="btn-f" id="f-month">Ce Mois</button>
        </div>

        <div class="kpi-grid">
            <div class="kpi-card green">
                <span>Cash en Main ✅</span>
                <div id="val-encaissé">0 F</div>
            </div>
            <div class="kpi-card yellow">
                <span>En Attente ⏳</span>
                <div id="val-attente">0 F</div>
            </div>
            <div class="kpi-card">
                <span>Missions Payées</span>
                <div id="val-count">0</div>
            </div>
            <div class="kpi-card">
                <span>Moyenne/Client</span>
                <div id="val-avg">0 F</div>
            </div>
        </div>

        <div class="panel">
            <table>
                <thead>
                    <tr>
                        <th>MISSION / DATE</th>
                        <th>DÉTAILS CLIENT</th>
                        <th style="text-align:right">MONTANT</th>
                    </tr>
                </thead>
                <tbody id="table-body">
                </tbody>
            </table>
        </div>

        <button class="btn-print" onclick="window.print()">📥 TÉLÉCHARGER LE RAPPORT PDF</button>
    </div>

<script>
    // Variable pour stocker la fonction de chargement Firebase
    let startSync = null;

    function verifierDirect() {
        const val = document.getElementById('admin-code').value.trim();
        const error = document.getElementById('lock-error');
        
        // Le code est "2410"
        if (val === "2410") {
            document.getElementById('lock-screen').style.display = 'none';
            document.getElementById('dashboard-content').style.display = 'block';
            if (typeof startSync === "function") startSync();
        } else {
            error.style.display = 'block';
            document.getElementById('admin-code').value = "";
        }
    }

    document.addEventListener('DOMContentLoaded', () => {
        const btn = document.getElementById('unlock-btn');
        const input = document.getElementById('admin-code');
        
        btn.addEventListener('click', verifierDirect);
        input.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') verifierDirect();
        });
        input.focus();
    });
</script>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
    import { getDatabase, ref, onValue } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js";

    const firebaseConfig = {
        apiKey: "AIzaSyAPCKRy9NTo4X8nn8YpxAbPtX8SlKj-7sQ",
        auth
