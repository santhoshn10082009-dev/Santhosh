# Santhosh
A real-time, online multiplayer Hand Cricket game built with HTML, CSS, and JavaScript. Features a 2-player mode synced via Firebase Realtime Database, a strategic 'Duck' limit, and a mobile-responsive UI.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Hand Cricket Pro</title>
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    <style>
        :root {
            --bg: #1a1a2e;
            --card: #16213e;
            --accent: #e94560;
            --text: #ffffff;
            --yellow: #f1c40f;
        }

        body { 
            background: var(--bg); 
            color: var(--text); 
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; 
            text-align: center;
            margin: 0;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }

        .container { 
            width: 90%;
            max-width: 400px; 
            padding: 20px; 
            background: var(--card); 
            border-radius: 25px; 
            box-shadow: 0 15px 35px rgba(0,0,0,0.5);
            border: 1px solid rgba(255,255,255,0.1);
        }

        .score-board { 
            background: rgba(0,0,0,0.3); 
            padding: 15px; 
            border-radius: 15px; 
            margin-bottom: 20px; 
            border-left: 5px solid var(--accent);
        }

        .duck-tracker { 
            color: var(--yellow); 
            font-size: 0.85rem; 
            margin-top: 8px;
            font-weight: bold;
        }

        .art-display { 
            font-size: 3rem; 
            display: flex; 
            justify-content: space-around; 
            margin: 25px 0;
            background: rgba(255,255,255,0.05);
            padding: 10px;
            border-radius: 15px;
        }

        .label { font-size: 0.8rem; color: #aaa; margin-bottom: 5px; }

        .controls { 
            display: grid; 
            grid-template-columns: repeat(4, 1fr); 
            gap: 8px; 
        }

        button { 
            padding: 12px 5px; 
            font-size: 1.2rem; 
            cursor: pointer; 
            border-radius: 12px; 
            border: none; 
            background: var(--accent); 
            color: white; 
            font-weight: bold;
            transition: transform 0.1s, background 0.2s;
            -webkit-tap-highlight-color: transparent;
        }

        button:active { transform: scale(0.9); background: #c62842; }
        
        button:disabled { 
            background: #333; 
            color: #666; 
            cursor: not-allowed; 
            transform: none;
        }

        #message { height: 20px; color: var(--yellow); margin-bottom: 15px; font-weight: 500; }

        .hidden { display: none; }

        .result-screen h1 { color: var(--accent); margin-bottom: 30px; }
    </style>
</head>
<body>

<div class="container" id="main-container">
    <div id="game-play">
        <h2 style="margin-top: 0; color: var(--accent);">🏏 HAND CRICKET</h2>
        
        <div class="score-board">
            <div style="font-size: 1.6rem; font-weight: bold;">
                <span id="score">0</span> 
                <span style="font-size: 0.9rem; color: #888;">Runs</span>
            </div>
            <div style="font-size: 0.9rem; color: #ccc;">Target: <span id="target">N/A</span></div>
            <div class="duck-tracker" id="duck-msg">🦆 Ducks Left: 2</div>
        </div>

        <div class="art-display">
            <div><div class="label">YOU</div><span id="p1-hand">⬜</span></div>
            <div><div class="label">CPU</div><span id="cpu-hand">⬜</span></div>
        </div>

        <div id="message">Choose your move!</div>

        <div class="controls" id="btn-grid">
            </div>
    </div>

    <div id="result-screen" class="hidden">
        <h1 id="winner-text"></h1>
        <button onclick="location.reload()" style="width: 100%; padding: 20px;">PLAY AGAIN</button>
    </div>
</div>

<script>
    const HANDS = {0:"✊", 1:"☝️", 2:"✌️", 3:"👌", 4:"🤚", 5:"🖐️", 6:"👍", 7:"👈", 8:"🔫", 9:"🤙", 10:"🤘"};

    let score = 0;
    let target = null;
    let isFirstInnings = true;
    let ducksUsed = 0;
    const MAX_DUCKS = 2;

    function createButtons() {
        const grid = document.getElementById('btn-grid');
        grid.innerHTML = '';
        for (let i = 0; i <= 10; i++) {
            let btn = document.createElement('button');
            btn.innerText = i;
            btn.id = `btn-${i}`;
            btn.onclick = () => playTurn(i);
            grid.appendChild(btn);
        }
    }

    function playTurn(playerMove) {
        const cpuMove = Math.floor(Math.random() * 11);
        
        // Update Hand Art
        document.getElementById('p1-hand').innerText = HANDS[playerMove];
        document.getElementById('cpu-hand').innerText = HANDS[cpuMove];

        // Duck (0) Logic
        if (playerMove === 0) {
            ducksUsed++;
            updateDuckUI();
        }

        // Gameplay Logic
        if (playerMove === cpuMove) {
            handleOut();
        } else {
            let runs = (playerMove === 0) ? cpuMove : playerMove;
            score += runs;
            document.getElementById('score').innerText = score;
            document.getElementById('message').innerText = `Scored ${runs}!`;

            // Check if Chaser wins
            if (!isFirstInnings && score >= target) {
                endGame("YOU WIN! 🏆");
            }
        }
    }

    function updateDuckUI() {
        const remaining = MAX_DUCKS - ducksUsed;
        document.getElementById('duck-msg').innerText = `🦆 Ducks Left: ${remaining}`;
        if (remaining <= 0) {
            const duckBtn = document.getElementById('btn-0');
            duckBtn.disabled = true;
            duckBtn.style.opacity = "0.3";
        }
    }

    function handleOut() {
        if (isFirstInnings) {
            alert("OUT! 💥 You scored " + score);
            target = score + 1;
            score = 0;
            isFirstInnings = false;
            ducksUsed = 0; // Reset ducks for chase
            
            document.getElementById('target').innerText = target;
            document.getElementById('score').innerText = "0";
            document.getElementById('message').innerText = "Target: " + target;
            
            // Reset Button 0
            const duckBtn = document.getElementById('btn-0');
            duckBtn.disabled = false;
            duckBtn.style.opacity = "1";
            updateDuckUI();
        } else {
            if (score < target - 1) endGame("CPU WINS! 🤖");
            else if (score === target - 1) endGame("IT'S A DRAW! 🤝");
        }
    }

    function endGame(msg) {
        document.getElementById('game-play').classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        document.getElementById('winner-text').innerText = msg;
    }

    createButtons();
</script>
</body>
</html>
