<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>CT241 - Commander & Suivre</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <style>
        :root { --gabon-vert: #009E60; --gabon-jaune: #FCD116; --gabon-bleu: #3A75C4; }
        .bg-gabon-vert { background-color: var(--gabon-vert); }
        #map { height: 280px; width: 100%; border-radius: 16px; margin-top: 12px; display: none; z-index: 10; border: 2px solid #e5e7eb; }
        .pulse { animation: pulse-animation 2s infinite; }
        @keyframes pulse-animation { 0% { opacity: 1; } 50% { opacity: 0.4; } 100% { opacity: 1; } }
        .section { animation: fadeIn 0.3s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
    </style>
</head>
<body class="bg-gray-50 text-gray-900 font-sans">

    <!-- HEADER -->
    <header class="bg-white sticky top-0 z-50 shadow-sm border-b-4 border-gabon-jaune p-4 flex justify-between items-center">
        <div class="flex items-center gap-2" onclick="location.reload()" style="cursor: pointer;">
            <div class="w-10 h-10 bg-gabon-vert rounded-xl flex items-center justify-center text-white font-black shadow-sm">CT</div>
            <div>
                <h1 class="font-black text-xl leading-none tracking-tighter text-gray-800">CT241</h1>
                <span class="text-[10px] font-bold text-gabon-vert uppercase tracking-widest">Gabon Cash</span>
            </div>
        </div>
        <div class="flex gap-2">
            <button onclick="showSection('commander')" class="bg-gray-100 text-gray-600 px-4 py-2 rounded-full text-[10px] font-black uppercase shadow-sm active:scale-95 transition-all">Commander</button>
            <button onclick="showSection('suivi')" class="bg-blue-600 text-white px-4 py-2 rounded-full text-[10px] font-black uppercase shadow-md active:scale-95 transition-all">Suivre</button>
        </div>
    </header>

    <main class="max-w-md mx-auto p-4 pb-24">
        
        <!-- SECTION 1 : PASSER COMMANDE -->
        <div id="sec-commander" class="section">
            <div class="bg-white rounded-[2rem] shadow-2xl p-6 border border-gray-100 overflow-hidden relative">
                <h2 class="text-2xl font-black mb-1 text-gray-800 tracking-tighter">Nouvelle Commande</h2>
                <p class="text-gray-400 text-[10px] mb-6 uppercase font-bold tracking-widest">Recevez votre cash à domicile</p>

                <div class="space-y-4">
                    <div>
                        <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Nom complet</label>
                        <input type="text" id="cNom" class="w-full p-4 bg-gray-50 border-2 border-gray-100 focus:border-green-500 rounded-2xl outline-none transition-all font-bold" placeholder="Ex: Jean-Marc">
                    </div>
                    
                    <div class="grid grid-cols-2 gap-4">
                        <div>
                            <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Téléphone</label>
                            <input type="tel" id="cTel" class="w-full p-4 bg-gray-50 border-2 border-gray-100 focus:border-green-500 rounded-2xl outline-none font-bold" placeholder="077...">
                        </div>
                        <div>
                            <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Quartier</label>
                            <input type="text" id="cLieu" class="w-full p-4 bg-gray-50 border-2 border-gray-100 focus:border-green-500 rounded-2xl outline-none font-bold" placeholder="Lieu dit">
                        </div>
                    </div>

                    <div>
                        <label class="text-[10px] font-black text-gray-400 uppercase mb-1 block ml-1">Montant Cash souhaité</label>
                        <div class="relative">
                            <input type="number" id="cMontant" oninput="calculerFrais()" class="w-full p-6 bg-gray-100 border-2 border-transparent focus:bg-white focus:border-green-500 rounded-3xl outline-none font-black text-3xl text-green-700 shadow-inner" placeholder="0">
                            <span class="absolute right-6 top-7 text-gray-400 font-black">FCFA</span>
                        </div>
                    </div>

                    <!-- BLOC RÉSUMÉ COMPTABILITÉ -->
                    <div class="bg-gray-900 rounded-[1.5rem] p-5 text-white shadow-xl">
                        <div class="space-y-1 mb-3 border-b border-gray-800 pb-2">
                            <div class="flex justify-between text-[10px] font-bold text-gray-400 uppercase tracking-tighter">
                                <span>Service Client (Frais)</span>
                                <span class="text-blue-400">190 F</span>
                            </div>
                            <div class="flex justify-between text-[10px] font-bold text-gray-400 uppercase tracking-tighter">
                                <span>Frais de Livraison</span>
                                <span class="text-blue-400">1 000 F</span>
                            </div>
                        </div>
                        <div class="flex justify-between items-end">
                            <span class="font-bold text-xs text-gray-400 uppercase leading-none">Total à payer<br><span class="text-[9px] lowercase italic">Cash + Frais</span></span>
                            <span id="totalDisplay" class="font-black text-3xl leading-none text-white tracking-tighter">0 F</span>
                        </div>
                    </div>

                    <button onclick="validerCommande()" id="btnCommander" class="w-full bg-gabon-vert text-white font-black py-5 rounded-2xl shadow-xl active:scale-95 hover:brightness-110 transition-all text-xl mt-2 uppercase tracking-tighter">
                        Confirmer la commande
                    </button>
                </div>
            </div>
        </div>

        <!-- SECTION 2 : SUIVI GPS -->
        <div id="sec-suivi" class="section hidden">
            <div class="bg-white rounded-[2rem] shadow-xl p-6 mb-6 border border-blue-50">
                <p class="text-[10px] font-black text-blue-600 uppercase mb-3 ml-1 tracking-widest text-center">Rechercher une livraison</p>
                <div class="flex gap-2">
                    <input type="tel" id="searchTel" placeholder="Votre numéro 07..." class="flex-1 p-4 bg-gray-50 border-2 border-gray-100 rounded-2xl outline-none focus:border-blue-500 font
