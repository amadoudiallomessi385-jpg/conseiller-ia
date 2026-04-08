<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    
    <!-- Paramètres pour l'écran d'accueil -->
    <title>GUERRIER AI</title>
    <meta name="apple-mobile-web-app-title" content="GUERRIER">
    <meta name="application-name" content="GUERRIER">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    <link rel="icon" href="https://flaticon.com">

    <style>
        :root { --gold: #ffd700; --blood: #8b0000; --dark: #0d0d0d; }
        body { background: var(--dark); color: var(--gold); font-family: sans-serif; height: 100vh; margin: 0; display: flex; flex-direction: column; }
        header { background: #1a1a1a; padding: 15px; text-align: center; border-bottom: 2px solid var(--gold); box-shadow: 0 5px 15px rgba(0,0,0,0.5); }
        #chat-box { flex: 1; overflow-y: auto; padding: 20px; display: flex; flex-direction: column; gap: 15px; }
        .msg { padding: 12px 18px; border-radius: 20px; max-width: 85%; animation: fadeIn 0.3s ease; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; } }
        .user { background: var(--blood); color: white; align-self: flex-end; border: 1px solid #ff4500; }
        .gemini { background: #1e1e1e; color: var(--gold); align-self: flex-start; border: 1px solid var(--gold); border-left: 5px solid var(--gold); }
        .input-area { padding: 15px; background: #151515; display: flex; gap: 10px; border-top: 2px solid var(--blood); }
        input { flex: 1; background: #000; border: 1px solid var(--gold); border-radius: 25px; padding: 12px 18px; color: #fff; outline: none; }
        button { background: var(--gold); color: #000; border: none; padding: 10px 20px; border-radius: 25px; font-weight: bold; cursor: pointer; }
    </style>
</head>
<body>

<header><h2>⚔️ CONSEIL DU GUERRIER</h2></header>

<div id="chat-box"></div>

<div class="input-area">
    <input type="text" id="user-input" placeholder="Ton ordre, Seigneur..." autocomplete="off">
    <button onclick="appelerGemini()">ENVOYER</button>
</div>

<script>
    // REMPLACE "TA_CLE_ICI" PAR TA VRAIE CLÉ API GOOGLE
    const API_KEY = "AIzaSyAeevVNMx0BlYcwXFuB4Vc6AcKoAJoNZfQ"; 
    const chatBox = document.getElementById('chat-box');
    const userInput = document.getElementById('user-input');

    window.onload = () => {
        const history = JSON.parse(localStorage.getItem('warrior_chat')) || [{role: 'gemini', text: 'Prêt pour la bataille. Quel est ton ordre ?'}];
        history.forEach(m => afficherMessage(m.role, m.text));
    };

    function afficherMessage(role, text) {
        const div = document.createElement('div');
        div.className = `msg ${role}`;
        div.innerText = text;
        chatBox.appendChild(div);
        chatBox.scrollTop = chatBox.scrollHeight;
    }

    async function appelerGemini() {
        const msg = userInput.value.trim();
        if (!msg) return;

        afficherMessage('user', msg);
        userInput.value = "";

        const loading = document.createElement('div');
        loading.className = "msg gemini";
        loading.innerText = "Analyse tactique en cours...";
        chatBox.appendChild(loading);

        try {
            const response = await fetch(`https://googleapis.com{API_KEY}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ contents: [{ parts: [{ text: "Réponds comme un général de guerre antique, sage et autoritaire : " + msg }] }] })
            });
            const data = await response.json();
            const reply = data.candidates[0].content.parts[0].text;
            chatBox.removeChild(loading);
            afficherMessage('gemini', reply);
            sauvegarder(msg, reply);
        } catch (e) {
            loading.innerText = "Erreur de forge ! La communication est rompue.";
        }
    }

    function sauvegarder(u, g) {
        let history = JSON.parse(localStorage.getItem('warrior_chat')) || [];
        history.push({role: 'user', text: u}, {role: 'gemini', text: g});
        localStorage.setItem('warrior_chat', JSON.stringify(history.slice(-20)));
    }
</script>
</body>
</html>
