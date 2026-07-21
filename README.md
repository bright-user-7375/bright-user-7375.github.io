<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Manillen - Slimme AI</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #2b7a0b;
            color: white;
            text-align: center;
            margin: 0;
            padding: 20px;
        }
        #game-board {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 20px;
            margin-top: 20px;
        }
        .hand {
            display: flex;
            gap: 10px;
            min-height: 120px;
            padding: 10px;
            background: rgba(0, 0, 0, 0.2);
            border-radius: 10px;
        }
        #table {
            width: 400px;
            height: 180px;
            background: rgba(0, 0, 0, 0.4);
            border-radius: 90px;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 15px;
            border: 2px solid #fff;
            padding: 10px;
        }
        .played-card-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 5px;
        }
        .player-label {
            font-size: 12px;
            background: rgba(0,0,0,0.6);
            padding: 2px 6px;
            border-radius: 4px;
        }
        .card {
            width: 70px;
            height: 100px;
            background-color: white;
            border-radius: 5px;
            box-shadow: 2px 2px 5px rgba(0,0,0,0.3);
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 24px;
            font-weight: bold;
            cursor: pointer;
            user-select: none;
            transition: transform 0.2s, opacity 0.2s;
        }
        .card:hover { transform: translateY(-10px); }
        .card.disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }
        .card.on-table:hover { transform: none; cursor: default; }
        .red { color: #d32f2f; }
        .black { color: #212121; }
        .info-panel {
            background: #fff;
            color: #333;
            padding: 15px;
            border-radius: 8px;
            max-width: 500px;
            margin: 0 auto;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .scoreboard {
            display: flex;
            justify-content: space-around;
            background: #eee;
            padding: 10px;
            border-radius: 5px;
            font-weight: bold;
            font-size: 18px;
        }
        .us { color: #2e7d32; }
        .them { color: #c62828; }
        button {
            padding: 10px 15px;
            font-size: 14px;
            cursor: pointer;
            background-color: #fbc02d;
            border: none;
            border-radius: 5px;
            font-weight: bold;
            align-self: center;
        }
        button:hover { background-color: #f9a825; }
        #trump-display { font-size: 20px; font-weight: bold; }
    </style>
</head>
<body>
    <h1>Manillen</h1>
    <div class="info-panel">
        <div class="scoreboard">
            <div class="us">Wij (Jij + Partner): <span id="score-us">0</span> pnt</div>
            <div class="them">Zij (Tegenstanders): <span id="score-them">0</span> pnt</div>
        </div>
        <div id="trump-display">Troef: Nog niet gekozen</div>
        <p id="status-text" style="font-weight: bold; font-size: 16px;">Klik op 'Nieuw Spel' om te beginnen.</p>
        <button onclick="startGame()">Nieuw Spel</button>
    </div>
    <div id="game-board">
        <div>
            <p>De Tafel</p>
            <div id="table"></div>
        </div>
        <div>
            <p>Jouw Hand</p>
            <div id="player-hand" class="hand"></div>
        </div>
    </div>
    <script>
        const suits = ['♠', '♥', '♦', '♣'];
        const ranks = [
            { name: '7', value: 0, power: 1 }, { name: '8', value: 0, power: 2 }, { name: '9', value: 0, power: 3 },
            { name: 'J', value: 1, power: 4 }, { name: 'Q', value: 2, power: 5 }, { name: 'K', value: 3, power: 6 },
            { name: 'A', value: 4, power: 7 }, { name: '10', value: 5, power: 8 }
        ];
        const playerNames = ["Jij", "Linker AI", "Je Partner", "Rechter AI"];
        let deck = [];
        let hands = [[], [], [], []]; 
        let tableCards = []; 
        let trumpSuit = '';
        let currentPlayer = 0; 
        let scoreUs = 0;
        let scoreThem = 0;
        let trickInProgress = false;
        function createAndShuffleDeck() {
            deck = [];
            for (let suit of suits) {
                for (let rank of ranks) {
                    deck.push({ suit: suit, rank: rank.name, value: rank.value, power: rank.power });
                }
            }
            for (let i = deck.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [deck[i], deck[j]] = [deck[j], deck[i]];
            }
        }
        function startGame() {
            createAndShuffleDeck();
            trumpSuit = suits[Math.floor(Math.random() * suits.length)];
            const trumpColor = (trumpSuit === '♥' || trumpSuit === '♦') ? 'red' : 'black';
            document.getElementById('trump-display').innerHTML = `Troef: <span class="${trumpColor}">${trumpSuit}</span>`;
            hands[0] = deck.slice(0, 8);
            hands[1] = deck.slice(8, 16);
            hands[2] = deck.slice(16, 24);
            hands[3] = deck.slice(24, 32);
            hands[0].sort((a, b) => a.suit.localeCompare(b.suit) || a.power - b.power);
            scoreUs = 0; scoreThem = 0;
            tableCards = [];
            currentPlayer = 0;
            trickInProgress = true;
            updateScoreboard();
            renderTable();
            playTurn();
        }
        function isValidPlay(cardToPlay, hand, tableOnlyCards, trump) {
            if (tableOnlyCards.length === 0) return true;
            const leadSuit = tableOnlyCards[0].suit;
            const hasLeadSuit = hand.some(c => c.suit === leadSuit);
            const hasTrump = hand.some(c => c.suit === trump);
            if (hasLeadSuit) return cardToPlay.suit === leadSuit;
            if (hasTrump) return cardToPlay.suit === trump;
            return true;
        }
        // Helper functie om te berekenen wie er momenteel de winnende kaart heeft liggen
        function getWinningPlay(plays, trump) {
            if (plays.length === 0) return null;
            let leadSuit = plays[0].card.suit;
            let winningPlay = plays[0];
            for (let i = 1; i < plays.length; i++) {
                let play = plays[i];
                let winIsTrump = winningPlay.card.suit === trump;
                let playIsTrump = play.card.suit === trump;
                if (playIsTrump && !winIsTrump) {
                    winningPlay = play; 
                } else if (playIsTrump && winIsTrump) {
                    if (play.card.power > winningPlay.card.power) winningPlay = play; 
                } else if (!playIsTrump && !winIsTrump) {
                    if (play.card.suit === leadSuit) {
                        if (winningPlay.card.suit !== leadSuit || play.card.power > winningPlay.card.power) {
                            winningPlay = play; 
                        }
                    }
                }
            }
            return winningPlay;
        }
        // Logica voor de AI om de best mogelijke kaart te kiezen
        function chooseSmartCard(validCards, aiPlayerIndex) {
            if (tableCards.length === 0) {
                // Eerste speler: speel gewoon hoogste power kaart uit (basis strategie)
                return validCards.reduce((prev, curr) => (prev.power > curr.power) ? prev : curr);
            }
            const winningPlay = getWinningPlay(tableCards, trumpSuit);
            const partnerIndex = (aiPlayerIndex + 2) % 4;
            const isPartnerWinning = (winningPlay.player === partnerIndex);
            if (isPartnerWinning) {
                // Partner wint: gooi hoogste punten (waarde) om hem te spekken, bij gelijke punten de laagste power
                return validCards.reduce((prev, curr) => {
                    if (curr.value > prev.value) return curr;
                    if (curr.value === prev.value && curr.power < prev.power) return curr;
                    return prev;
                });
            } else {
                // Tegenstander wint: kunnen we de slag winnen met een van onze geldige kaarten?
                let winningCards = validCards.filter(c => {
                    let testTable = [...tableCards, { card: c, player: aiPlayerIndex }];
                    return getWinningPlay(testTable, trumpSuit).player === aiPlayerIndex;
                });
                if (winningCards.length > 0) {
                    // We kunnen winnen! Speel de ZWAKSTE kaart die toch sterk genoeg is om te winnen
                    return winningCards.reduce((prev, curr) => (prev.power < curr.power) ? prev : curr);
                } else {
                    // We kunnen NIET winnen. Gooi de kaart weg met de minste punten en minste power (vuilbak)
                    return validCards.reduce((prev, curr) => {
                        if (curr.value < prev.value) return curr;
                        if (curr.value === prev.value && curr.power < prev.power) return curr;
                        return prev;
                    });
                }
            }
        }
        function playTurn() {
            if (tableCards.length === 4) {
                setTimeout(resolveTrick, 1500); 
                return;
            }
            if (currentPlayer === 0) {
                document.getElementById('status-text').innerText = "Jij bent aan de beurt. Speel een kaart.";
                renderHand();
                trickInProgress = false; 
                return;
            }
            document.getElementById('status-text').innerText = `${playerNames[currentPlayer]} denkt na...`; 
            setTimeout(() => {
                let aiHand = hands[currentPlayer];
                let justTableCards = tableCards.map(t => t.card);
                let validCards = aiHand.filter(c => isValidPlay(c, aiHand, justTableCards, trumpSuit));
                // Roep de nieuwe slimme hersens aan!
                let chosenCard = chooseSmartCard(validCards, currentPlayer);
                hands[currentPlayer] = aiHand.filter(c => c !== chosenCard);
                tableCards.push({ card: chosenCard, player: currentPlayer });
                renderTable();
                currentPlayer = (currentPlayer + 1) % 4;
                playTurn();
            }, 1000); 
        }
        function playCard(index) {
            if (trickInProgress || currentPlayer !== 0 || tableCards.length >= 4) return;
            const cardToPlay = hands[0][index];
            const justTableCards = tableCards.map(t => t.card);
            if (!isValidPlay(cardToPlay, hands[0], justTableCards, trumpSuit)) {
                alert("Ongeldige zet! Bekennen of troeven is verplicht.");
                return;
            }
            trickInProgress = true;
            const playedCard = hands[0].splice(index, 1)[0];
            tableCards.push({ card: playedCard, player: 0 });
            renderHand();
            renderTable();
            currentPlayer = 1; 
            playTurn();
        }
        function resolveTrick() {
            const winningPlay = getWinningPlay(tableCards, trumpSuit);
            let trickPoints = tableCards.reduce((sum, play) => sum + play.card.value, 0);
            const winnerIndex = winningPlay.player;
            if (winnerIndex === 0 || winnerIndex === 2) {
                scoreUs += trickPoints;
            } else {
                scoreThem += trickPoints;
            }
            document.getElementById('status-text').innerText = `${playerNames[winnerIndex]} wint de slag met ${trickPoints} punten!`;
            updateScoreboard();
            currentPlayer = winnerIndex;
            setTimeout(() => {
                tableCards = []; 
                renderTable();
                if (hands[0].length === 0 && hands[1].length === 0) {
                    document.getElementById('status-text').innerText = `Ronde afgelopen! Wij: ${scoreUs} | Zij: ${scoreThem}`;
                } else {
                    playTurn(); 
                }
            }, 2500); 
        }
        function renderHand() {
            const handDiv = document.getElementById('player-hand');
            handDiv.innerHTML = '';
            const justTableCards = tableCards.map(t => t.card);
            hands[0].forEach((card, index) => {
                const cardEl = document.createElement('div');
                const isRed = card.suit === '♥' || card.suit === '♦';
                const canPlay = isValidPlay(card, hands[0], justTableCards, trumpSuit);
                cardEl.className = `card ${isRed ? 'red' : 'black'} ${canPlay && !trickInProgress && currentPlayer === 0 ? '' : 'disabled'}`;
                cardEl.innerText = `${card.rank}${card.suit}`;
                cardEl.onclick = () => { if(canPlay) playCard(index); };
                handDiv.appendChild(cardEl);
            });
        }
        function renderTable() {
            const tableDiv = document.getElementById('table');
            tableDiv.innerHTML = '';
            tableCards.forEach(play => {
                const container = document.createElement('div');
                container.className = 'played-card-container';
                const cardEl = document.createElement('div');
                const isRed = play.card.suit === '♥' || play.card.suit === '♦';
                cardEl.className = `card on-table ${isRed ? 'red' : 'black'}`;
                cardEl.innerText = `${play.card.rank}${play.card.suit}`;
                const labelEl = document.createElement('div');
                labelEl.className = 'player-label';
                labelEl.innerText = playerNames[play.player];
                container.appendChild(cardEl);
                container.appendChild(labelEl);
                tableDiv.appendChild(container);
            });
        }
        function updateScoreboard() {
            document.getElementById('score-us').innerText = scoreUs;
            document.getElementById('score-them').innerText = scoreThem;
        }
    </script>
</body>
</html>

