<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>CT241 - GESTION & LOGISTIQUE</title>
    <style>
        :root {
            --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4;
            --danger: #e74c3c; --dark: #1a1a1a; --light: #f8f9fa;
        }
        body { font-family: 'Segoe UI', sans-serif; background: var(--light); margin: 0; padding: 10px; }
        
        #auth-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: var(--dark); display: flex; align-items: center; justify-content: center; z-index: 9999;
        }
        .login-card {
            background: white; padding: 25px; border-radius: 15px; width: 85%; max-width: 320px;
            text-align: center; border-top: 8px solid var(--gabon-jaune);
        }
        input { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; outline: none; }
        input:focus { border-color: var(--gabon-bleu); }
        .btn-login { width: 100%; padding: 14px; background: var(--gabon-vert); color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; transition: 0.3s; }
        .btn-login:disabled { background: #95a5a6; cursor: not-allowed; }

        #main-app { display: none; max-width: 800px; margin: auto; background: white; border-radius: 12px; padding: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #eee; padding-bottom: 10px; margin-bottom: 15px; }
        
        nav { display: flex; gap: 5px; margin-bottom: 15px; }
        nav button { flex: 1; padding: 12px; border: none; border-radius: 8px; background: #eee; font-weight: bold; font-size: 11px; transition: 0.3s; cursor: pointer; }
        nav button.active { background: var(--gabon-vert); color: white; box-shadow: 0 2px 5px rgba(0,158,96,0.3); }

        .form-box { background: #f9f9f9; padding: 15px; border-radius: 10px; margin-bottom: 20px; border: 1px solid #ddd; }
        #mComDisplay { background: #e3f2fd; font-weight: bold; border: 1px solid var(--gabon-bleu); color: var(--gabon-bleu); text-align: center; }
        
        .card { border: 1px solid #ddd; padding: 15px; border-radius: 10px; margin-bottom: 15px; border-left: 6px solid var(--gabon-bleu); position: relative; }
        .card.done { border-left-color: var(--gabon-vert); background: #f0fff4; }
        
        .btn-wa { background: #25D366; color: white; border: none; padding: 14px; border-radius: 10px; width: 100%; font-weight: bold; cursor: pointer; margin-top: 10px; }
        .btn-del { color: var(--danger); border: none; background: none; font-weight: bold; font-size: 20px; cursor: pointer; position: absolute; top: 10px; right: 10px; }
        
        .section { display: none; }
        .active-sec { display: block; }
        
        .stat-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 15px; }
        .stat-item { background: #1a1a1a; padding: 15px; border-radius: 10px; color: white; text-align: center; }
        
        #login-error { color: var(--danger); font-size: 12px; margin-top: 10px; display: none; font-weight: bold; }
    </style>
</head>
<body>

    <div id="auth-screen">
        <div class="login-card">
            <h2 style="color:var(--gabon-vert); margin:0">CT241 GABON</h2>
            <p style="font-size: 12px; color: #666; margin-bottom: 20px;">Gestion Pro & Comptabilité</p>
            <input type="email" id="login-email" placeholder="Email (ex: admin@ct241.com)">
            <input type="password" id="login-pass" placeholder="Mot de passe">
            <button class="btn-login" id="btnConnect">SE CONNECTER</button>
            <div id="login-error">Accès refusé. Vérifiez vos identifiants.</div>
        </div>
    </div>

    <div id="main-app">
        <header>
            <div>
                <h3 style="margin:0; color:var(--gabon-vert)">CT241 GESTION</h3>
                <small id="userDisplay" style="font-size: 10px; color: #777;"></small>
            </div>
            <button id="btnOut" style="font-size:10px; color:var(--danger); background:none; border:none; font-weight:bold; cursor:pointer">DÉCONNEXION</button>
        </header>

        <nav id="adminNav">
            <button onclick="ouvrir('saisie')" id="t-saisie" class="active">SAISIE</button>
            <button onclick="ouvrir('taches')" id="t-taches">MISSIONS</button>
            <button onclick="ouvrir('reçus')" id="t-reçus">SUIVI & CAISSE</button>
        </nav>

        <!-- ONGLET SAISIE (ADMIN) -->
        <div id="sec-saisie" class="section active-sec">
            <div class="form-box">
                <input type="text" id="mNom" placeholder="Nom du Client">
                <input type="tel" id="mTel" placeholder="Numéro du Client">
                <input type="text" id="mLieu" placeholder="Localisation / Quartier">
                <input type="number" id="mRetrait" placeholder="Montant Cash (FCFA)">
                <div style="margin-top:5px; font-size: 11px; color: #555;">Commission Direction (Gain Net) :</div>
                <input type="text" id="mComDisplay" value="390 FCFA" readonly>
                <button onclick="lancerMission()" style="width:100%; padding:14px; background:var(--gabon-bleu); color:white; border:none; border-radius:8px; font-weight:bold; margin-top:10px; cursor:pointer">CRÉER LA MISSION</button>
            </div>
        </div>

        <!-- ONGLET MISSIONS (LIVREURS) -->
        <div id="sec-taches" class="section">
            <h4 style="margin: 0 0 10px 0;">Missions disponibles</h4>
            <div id="list-taches"></div>
        </div>

        <!-- ONGLET SUIVI & COMPTABILITÉ -->
        <div id="sec-reçus" class="section">
            <div class="stat-grid">
                <div class="stat-item">
                    <small style="color: var(--gabon-jaune)">PAIES LIVREURS (800F/u)</small><br>
                    <b id="totalComLivreur" style="font-size:18px">0</b> <small>F</small>
                </div>
                <div class="stat-item" style="border: 1px solid var(--gabon-vert)">
                    <small style="color: var(--gabon-vert)">PROFIT CT241 (390F/u)</small><br>
                    <b id="totalProfitAdmin" style="font-size:18px; color: var(--gabon-vert)">0</b> <small>F</small>
                </div>
            </div>
            
            <input type="text" id="searchInput" oninput="filtrerRecus()" style="margin-top:15px" placeholder="🔍 Rechercher (Client, Livreur...)">
            <div id="list-reçus" style="margin-top:10px"></div>

            <button class="btn-wa" onclick="shareWA()">📲 BILAN WHATSAPP</button>
        </div>
    </div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
    import { getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-auth.js";
    import { getDatabase, ref, push, onValue, update, remove, set } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js";

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
    const auth = getAuth(app);
    const db = getDatabase(app);
    let userNow = "";
    let missionsLocales = [];

    // Connexion avec gestion d'état visuel
    const btnConnect = document.getElementById('btnConnect');
    const loginErr = document.getElementById('login-error');

    btnConnect.onclick = async () => {
        const email = document.getElementById('login-email').value.trim();
        const pass = document.getElementById('login-pass').value;
        
        if(!email || !pass) return;

        btnConnect.disabled = true;
        btnConnect.innerText = "CONNEXION EN COURS...";
        loginErr.style.display = 'none';

        try {
            await signInWithEmailAndPassword(auth, email, pass);
        } catch (error) {
            console.error(error);
            loginErr.style.display = 'block';
            btnConnect.disabled = false;
            btnConnect.innerText = "SE CONNECTER";
        }
    };

    document.getElementById('btnOut').onclick = () => signOut(auth);

    onAuthStateChanged(auth, (u) => {
        if(u) {
            userNow = u.email;
            document.getElementById('userDisplay').innerText = userNow;
            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('main-app').style.display = 'block';
            
            // Réinitialiser le bouton si besoin
            btnConnect.disabled = false;
            btnConnect.innerText = "SE CONNECTER";

            // Si c'est un livreur, on masque l'administration
            if(userNow.includes('livreur')) {
                document.getElementById('t-saisie').style.display = 'none';
                document.getElementById('t-reçus').style.display = 'none';
                ouvrir('taches');
            } else {
                document.getElementById('t-saisie').style.display = 'block';
                document.getElementById('t-reçus').style.display = 'block';
            }
            ecouter();
        } else {
            document.getElementById
