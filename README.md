<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CT241 - DASHBOARD SÉCURISÉ</title>
    <style>
        :root {
            --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4;
            --bg: #f8f9fa; --card-bg: #ffffff; --text-main: #2c3e50; --text-muted: #7f8c8d;
        }
        body { font-family: 'Inter', sans-serif; background: var(--bg); color: var(--text-main); margin: 0; padding: 0; }

        /* ÉCRAN DE VERROUILLAGE */
        #lock-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: var(--text-main); display: flex; align-items: center; justify-content: center; z-index: 10000;
        }
        .lock-box { background: white; padding: 30px; border-radius: 20px; text-align: center; width: 85%; max-width: 320px; border-top: 8px solid var(--gabon-jaune); }
        .lock-box input { width: 100%; padding: 15px; margin: 15px 0; border: 1px solid #ddd; border-radius: 10px; font-size: 18px; text-align: center; box-sizing: border-box; }
        .btn-unlock { width: 100%; padding: 15px; background: var(--gabon-vert); color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }

        /* CONTENU (Masqué par défaut) */
        #dashboard-content { display: none; padding: 20px; }
        
        .app-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; border-bottom: 2px solid #ddd; padding-bottom: 15px; }
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 15px; margin-bottom: 30px; }
        .kpi-card { background: var(--card-bg); padding: 20px; border-radius: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); border-top: 4px solid #eee; }
        .kpi-card.green { border-top-color: var(--gabon-vert); }
        .kpi-card.blue { border-top-color: var(--gabon-bleu); }
        .kpi-card div { font-size: 22px; font-weight: 800; margin-top: 8px; }
        
        .controls { display: flex; gap: 10px; margin-bottom: 20px; }
        .btn-filter { flex: 1; padding: 10px; border: 1px solid #ddd; background: white; border-radius: 8px; cursor: pointer; font-weight: 600; }
        .btn-filter.active { background: var(--text-main); color: white; }

        .data-panel { background: var(--card-bg); border-radius: 12px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); overflow: hidden; }
        table { width: 100%; border-collapse: collapse; }
        th { background: #f9f9f9; text-align: left; padding: 12px 20px; font-size: 11px; color: var(--text-muted); }
        td { padding: 15px 20px; border-bottom: 1px solid #f1f1f1; font-size: 14px; }
        .amount { text-align: right; font-weight: 800; color: var(--gabon-vert); }
        .id-label { font-family: monospace; background: #f1f3f5; padding: 2px 6px; border-radius: 4px; font-weight: bold; font-size: 12px; }

        @media (max-width: 600px) { .kpi-grid { grid-template-columns: 1fr 1fr; } }
    </style>
</head>
<body>

    <div id="lock-screen">
        <div class="lock-box">
            <h2 style="margin:0; color: var(--text-main);">🔐 ACCÈS PRIVÉ</h2>
            <p style="font-size: 12px; color: gray;">Entrez le code administrateur</p>
            <input type="password" id="admin-code" placeholder="Code Secret">
            <button class="btn-unlock" onclick="verifierCode()">ACCÉDER AUX COMPTES</button>
            <p id="lock-error" style="color:red; font-size: 12px; margin-top: 10px; display:none;">Code incorrect !</p>
        </div>
    </div>

    <div id="dashboard-content">
        <div class="app-header">
            <h1>CT241 <span style="color:var(--gabon-vert)">FINANCE</span></h1>
            <div style="font-size: 12px; font-weight: bold; color: var(--gabon-bleu);">ADMINISTRATEUR</div>
        </div>

        <div class="controls">
            <button class="btn-filter active" onclick="setFilter('all', this)">Global</button>
            <button class="btn-filter" onclick="setFilter('today', this)">Aujourd'hui</button>
            <button class="btn-filter" onclick="setFilter('month', this)">Mois</button>
        </div>

        <div class="kpi-grid">
            <div class="kpi-card green"><span>Encaissé (Cash)</span><div id="val-encaissé">0</div></div>
            <div class="kpi-card blue"><span>Missions</span><div id="val-count">0</div></div>
            <div class="kpi-card" style="border-top-color:var(--gabon-jaune)"><span>En Attente</span><div id="val-attente">0</div></div>
            <div class="kpi-card"><span>Moyenne</span><div id="val-avg">0</div></div>
        </div>

        <div class="data-panel">
            <table>
                <thead><tr><th>REF / DATE</th><th>CLIENT</th><th style="text-align:right">COMMISSION</th></tr></thead>
                <tbody id="table-body"></tbody>
            </table>
        </div>
        <button onclick="window.print()" style="width:100%; padding:15px; margin-top:20px; border-radius:10px; border:none; background:#34495e; color:white; font-weight:bold;">IMPRIMER LE BILAN</button>
    </div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
    import { getDatabase, ref, onValue } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js";
    import { getAuth, signInWithEmailAndPassword } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-auth.js";

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
    const auth = getAuth(app);
    let filterMode = 'all';

    window.verifierCode = () => {
        const code = document.getElementById('admin-code').value;
        if(code === "2410") {
            // REMPLACE PAR TON EMAIL ET MOT DE PASSE ADMIN ICI
            // Cela force l'App Compta à avoir les mêmes droits que l'App 1
            signInWithEmailAndPassword(auth, "admin@ct241.ga", "TON_MOT_DE_PASSE")
            .then(() => {
                document.getElementById('lock-screen').style.display = 'none';
                document.getElementById('dashboard-content').style.display = 'block';
                chargerDonnees();
            })
            .catch(err => {
                alert("Erreur de synchronisation : " + err.message);
            });
        } else {
            document.getElementById('lock-error').style.display = 'block';
        }
    };

    function chargerDonnees() {
        // Cette fonction "écoute" la base de données en temps réel
        onValue(ref(db, 'missions'), (snap) => {
            const data = snap.val();
            const body = document.getElementById('table-body');
            let totalCash = 0, totalWait = 0, count = 0;
            body.innerHTML = "";
            const today = new Date().toLocaleDateString('fr-FR');

            if (data) {
                Object.keys(data).reverse().forEach(key => {
                    const m = data[key];
                    if(filterMode === 'today' && m.date !== today) return;
                    
                    if(m.etape === 3) { // Uniquement les missions encaissées
                        totalCash += m.com; count++;
                        body.innerHTML += `<tr><td><span class="id-label">${m.id}</span></td><td><strong>${m.nom}</strong></td><td class="amount">${m.com.toLocaleString()} F</td></tr>`;
                    } else if (m.etape === 2) {
                        totalWait += m.com;
                    }
                });
            }
            document.getElementById('val-encaissé').innerText = totalCash.toLocaleString() + " F";
            document.getElementById('val-attente').innerText = totalWait.toLocaleString() + " F";
            document.getElementById('val-count').innerText = count;
            document.getElementById('val-avg').innerText = count > 0 ? Math.round(totalCash/count).toLocaleString() + " F" : "0 F";
        });
    }

    window.setFilter = (mode, btn) => {
        filterMode = mode;
        document.querySelectorAll('.btn-filter').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        chargerDonnees();
    };
</script>

    function chargerDonnees() {
        onValue(ref(db, 'missions'), (snap) => {
            const data = snap.val();
            const body = document.getElementById('table-body');
            let totalCash = 0, totalWait = 0, count = 0;
            body.innerHTML = "";
            const today = new Date().toLocaleDateString('fr-FR');

            if (data) {
                Object.keys(data).reverse().forEach(key => {
                    const m = data[key];
                    if(filterMode === 'today' && m.date !== today) return;
                    
                    if(m.etape === 3) {
                        totalCash += m.com; count++;
                        body.innerHTML += `<tr><td><span class="id-label">${m.id}</span><br><small>${m.heure}</small></td><td><strong>${m.nom}</strong><br><small>${m.date}</small></td><td class="amount">${m.com.toLocaleString()} F</td></tr>`;
                    } else if (m.etape === 2) {
                        totalWait += m.com;
                    }
                });
            }
            document.getElementById('val-encaissé').innerText = totalCash.toLocaleString() + " F";
            document.getElementById('val-attente').innerText = totalWait.toLocaleString() + " F";
            document.getElementById('val-count').innerText = count;
            document.getElementById('val-avg').innerText = count > 0 ? Math.round(totalCash/count).toLocaleString() + " F" : "0 F";
        });
    }
</script>
</body>
</html>
