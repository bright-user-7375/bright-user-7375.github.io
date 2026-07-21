<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Manillen Basis</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #2b7a0b; /* Casino groen */
            color: white;
            text-align: center;
            margin: 0;
            padding: 20px;
        }
        #game-board {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 40px;
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
            width: 300px;
            height: 200px;
            background: rgba(0, 0, 0, 0.4);
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            gap: 15px;
            border: 2px solid #fff;
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
            transition: transform 0.2s;
        }
        .card:hover {
            transform: translateY(-10px);
        }
        .red { color: #d32f2f; }
        .black { color: #212121; }
        .info-panel {
            background: #fff;
            color: #333;
            padding: 15px;
            border-radius: 8px;
            max-width: 400px;
            margin: 0 auto;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #fbc02d;
            border: none;
            border-radius: 5px;
            font-weight: bold;
        }
        button:hover { background-color: #f9a825; }
    </style>
</head>
<body>

    <h1>Manillen</h1>
    
    <div class="info-panel">
        <h3 style="margin-top: 0;">Spel Status</h3>
        <p id="status-text">Klik op 'Deel Kaarten' om te beginnen.</p>
        <button onclick="startGame()">Deel Kaarten</button>
    </div>

    <div id="game-board">
        <div>
            <p>Tafel (Gespeelde kaarten)</p>
            <div id="table"></div>
        </div>

        <div>
            <p>Jouw Hand (Speler 1)</p>
            <div id="player-hand" class="hand"></div>
        </div>
    </div>

    <script>
        const suits = ['♠️', '♥️', '♦️', '♣️'];
        // Specifieke Manillen volgorde (Laag naar hoog: 7, 8, 9, Boer, Dame, Heer, Aas, 10/Manille)
        const ranks = [
            { name: '7', value: 0 },
            { name: '8', value: 0 },
            { name: '9', value: 0 },
            { name: 'J', value: 1 },
            { name: 'Q', value: 2 },
            { name: 'K', value: 3 },
            { name: 'A', value: 4 },
            { name: '10', value: 5 } // De Manille!
        ];

        let deck = [];
        let playerHand = [];
        let tableCards = [];

        function createDeck() {
            deck = [];
            for (let suit of suits) {
                for (let rank of ranks) {
                    deck.push({ suit: suit, rank: rank.name, value: rank.value });
                }
            }
        }

        function shuffleDeck() {
            for (let i = deck.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [deck[i], deck[j]] = [deck[j], deck[i]];
            }
        }

        function startGame() {
            createDeck();
            shuffleDeck();
            
            // In een echt spel krijgen 4 spelers elk 8 kaarten.
            // Voor nu delen we alleen de hand van de speler uit.
            playerHand = deck.splice(0, 8);
            tableCards = []; // Maak de tafel leeg
            
            document.getElementById('status-text').innerText = "Jij bent aan de beurt. Klik op een kaart om te spelen.";
            document.getElementById('table').innerHTML = '';
            
            // Sorteer hand op kleur voor overzicht (optioneel, maar handig)
            playerHand.sort((a, b) => a.suit.localeCompare(b.suit));
            
            renderHand();
        }

        function renderHand() {
            const handDiv = document.getElementById('player-hand');
            handDiv.innerHTML = ''; // Maak leeg

            playerHand.forEach((card, index) => {
                const cardEl = document.createElement('div');
                cardEl.className = card ${card.suit === '♥️' || card.suit === '♦️' ? 'red' : 'black'};
                cardEl.innerText = ${card.rank}${card.suit};
                
                // Klik evenement om de kaart te spelen
                cardEl.onclick = () => playCard(index);
                
                handDiv.appendChild(cardEl);
            });
        }

        function playCard(index) {
            if (tableCards.length >= 4) {
                alert("De slag is al vol! (Dit moet nog opgeruimd worden in de logica)");
                return;
            }

            // Haal de kaart uit de hand en leg hem op tafel
            const playedCard = playerHand.splice(index, 1)[0];
            tableCards.push(playedCard);
            
            renderHand();
            renderTable();

            if(tableCards.length === 1) {
                 document.getElementById('status-text').innerText = "Je hebt " + playedCard.rank + playedCard.suit + " gespeeld. Nu moeten de andere spelers (AI) nog toegevoegd worden.";
            }
        }

        function renderTable() {
            const tableDiv = document.getElementById('table');
            tableDiv.innerHTML = '';

            tableCards.forEach(card => {
                const cardEl = document.createElement('div');
                cardEl.className = card ${card.suit === '♥️' || card.suit === '♦️' ? 'red' : 'black'};
                cardEl.innerText = ${card.rank}${card.suit};
                // Zorg dat kaarten op tafel niet meer klikbaar zijn
                cardEl.style.cursor = 'default';
                cardEl.style.transform = 'none'; 
                tableDiv.appendChild(cardEl);
            });
        }
    </script>
</body>
</html>

