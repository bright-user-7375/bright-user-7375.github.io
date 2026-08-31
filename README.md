<html lang="nl">
<head>
    <meta charset="UTF-8">
    <title>Manillen voor 2 Spelers</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #0b6623; color: white; text-align: center; margin: 0; padding: 20px; }
        #game-container { max-width: 800px; margin: 0 auto; background: rgba(0, 0, 0, 0.3); padding: 20px; border-radius: 10px; position: relative; }
        .hand { display: flex; justify-content: center; gap: 10px; min-height: 120px; margin: 15px 0; flex-wrap: wrap; }
        .card { width: 70px; height: 105px; background: white; color: black; border-radius: 5px; display: flex; flex-direction: column; justify-content: space-between; padding: 5px; font-weight: bold; cursor: pointer; box-shadow: 2px 2px 5px rgba(0,0,0,0.5); user-select: none; transition: transform 0.2s; }
        .card:hover { transform: translateY(-5px); }
        .card.red { color: red; }
        .card.back { background: #b22222; border: 2px solid white; }
        #table-area { height: 130px; border: 2px dashed rgba(255,255,255,0.5); border-radius: 8px; display: flex; justify-content: center; align-items: center; gap: 20px; margin: 20px 0; background: rgba(0,0,0,0.1); }
        .info-panel { display: flex; justify-content: space-around; background: rgba(0,0,0,0.4); padding: 10px; border-radius: 5px; margin-bottom: 15px; }
        button { background: #ffcc00; border: none; padding: 10px 20px; font-size: 16px; font-weight: bold; cursor: pointer; border-radius: 5px; margin: 5px; }
        button:hover { background: #e6b800; }
        
        #menu-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0, 0, 0, 0.6); border-radius: 10px; display: flex; flex-direction: column; justify-content: center; align-items: center; z-index: 10; }
        .menu-btn-container { display: flex; gap: 10px; margin-top: 15px; flex-wrap: wrap; justify-content: center; }
        .suit-btn { background: white; color: black; padding: 15px 25px; font-size: 18px; }
        .suit-btn.red { color: red; }
        .pass-btn { background: #b22222; color: white; padding: 15px 25px; font-size: 18px; }
        .pass-btn:hover { background: #8b0000; }
        
        .match-banner { background: #ffcc00; color: black; font-weight: bold; padding: 8px; border-radius: 5px; margin-bottom: 15px; font-size: 18px; }
    </style>
</head>
<body>
<div id="game-container">
    <!-- GEFIXT: Het dubbele woord 'Computer' is hier nu wegehaald -->
    <div class="match-banner">
        WEDSTRIJD STAND (EERSTE TOT 50): Jij: <span id="totaal-speler">0</span> | Computer: <span id="totaal-comp">0</span>
    </div>

    <!-- HIER STAAT HET VERNIEUWDE KEUZEMENU -->
    <div id="menu-overlay">
        <h2 id="menu-title">Kies Troef of Pass</h2>
        <p id="menu-text">Jij mag als eerste de troef bepalen voor dit spel.</p>
        <div id="menu-buttons" class="menu-btn-container">
            <!-- Dynamisch gevuld via Javascript -->
        </div>
    </div>

    <h1>Manillen (2 Spelers)</h1>
    <div class="info-panel">
        <div>Troef: <span id="troef-display" style="font-weight:bold; color:#ffcc00;">-</span> <span id="multi-display" style="color:#ff4444; font-weight:bold; margin-left:5px;"></span></div>
        <div>Dit spel Speler: <span id="score-speler">0</span></div>
        <div>Dit spel Computer: <span id="score-comp">0</span></div>
        <div>In deck: <span id="deck-count">32</span></div>
    </div>
    <div>Computer Hand:</div>
    <div id="computer-hand" class="hand"></div>
    <div id="table-area"><span>Leg een kaart...</span></div>
    <div>Jouw Hand:</div>
    <div id="player-hand" class="hand"></div>
    <p id="message"></p>
    <button id="next-btn" style="display:none;" onclick="startVolgendeSlag()">Volgende Slag</button>
    <button id="restart-btn" style="display:none; background: #ff4444; color: white;" onclick="initGame()">Volgende Spel</button>
</div>
<script>
    const suits = ['Harten', 'Ruiten', 'Klaveren', 'Schoppen'];
    const suitSymbols = {'Harten': '♥', 'Ruiten': '♦', 'Klaveren': '♣', 'Schoppen': '♠'};
    const values = [
        { name: '7', rank: 1, points: 0 }, { name: '8', rank: 2, points: 0 }, { name: '9', rank: 3, points: 0 },
        { name: 'Boer', rank: 4, points: 1 }, { name: 'Dame', rank: 5, points: 2 }, { name: 'Heer', rank: 6, points: 3 },
        { name: 'Aas', rank: 7, points: 4 }, { name: '10', rank: 8, points: 5 }
    ];

    let totaalSpeler = 0;
    let totaalComputer = 0;

    let deck = [], playerHand = [], computerHand = [], currentTroef = '';
    let playerPoints = 0, computerPoints = 0, turn = 'player', wieBegonDeSlag = 'player';
    let tableCardPlayer = null, tableCardComp = null, leadingSuit = null;
    
    // NIEUW: Houdt bij wie troef koos en wat de inzetvermenigvuldiger is
    let wieKloosTroef = 'player';
    let scoreMultiplier = 1; 

    function initGame() {
        deck = []; playerPoints = 0; computerPoints = 0; scoreMultiplier = 1;
        if (totaalSpeler >= 50 || totaalComputer >= 50) { totaalSpeler = 0; totaalComputer = 0; }

        suits.forEach(suit => { values.forEach(val => { deck.push({ suit, name: val.name, rank: val.rank, points: val.points }); }); });
        deck.sort(() => Math.random() - 0.5);
        playerHand = deck.splice(0, 8); computerHand = deck.splice(0, 8);
        updateUI();

        document.getElementById('score-speler').innerText = playerPoints;
        document.getElementById('score-comp').innerText = computerPoints;
        document.getElementById('totaal-speler').innerText = totaalSpeler;
        document.getElementById('totaal-comp').innerText = totaalComputer;
        document.getElementById('deck-count').innerText = deck.length;
        document.getElementById('multi-display').innerText = "";
        
        // FIX: Verberg de volgende/herstartknop ALTIJD direct bij de start van een nieuw spel
        document.getElementById('next-btn').style.display = 'none';
        document.getElementById('restart-btn').style.display = 'none';
        
        turn = 'player'; wieBegonDeSlag = 'player';
        toonTroefMenu();
    }

    function toonTroefMenu() {
        document.getElementById('menu-overlay').style.display = 'flex';
        document.getElementById('menu-title').innerText = "Kies Troef of Pass";
        document.getElementById('menu-text').innerText = "Jij mag de troef bepalen voor dit spel.";
        
        const container = document.getElementById('menu-buttons');
        container.innerHTML = `
            <button class="suit-btn red" onclick="kiesTroef('Harten')">Harten ♥</button>
            <button class="suit-btn red" onclick="kiesTroef('Ruiten')">Ruiten ♦</button>
            <button class="suit-btn" onclick="kiesTroef('Klaveren')">Klaveren ♣</button>
            <button class="suit-btn" onclick="kiesTroef('Schoppen')">Schoppen ♠</button>
            <button class="pass-btn" onclick="kiesTroef('Pass')">Passen (Computer kiest)</button>
        `;
    }

    function kiesTroef(keuze) {
        if (keuze === 'Pass') {
            currentTroef = suits[Math.floor(Math.random() * suits.length)];
            wieKloosTroef = 'computer';
            setMessage(`Jij paste. De computer heeft ${currentTroef} gekozen.`);
            setTimeout(faseTegenSpeler, 800);
        } else {
            currentTroef = keuze;
            wieKloosTroef = 'player';
            setMessage(`Je hebt ${currentTroef} als troef gekozen.`);
            setTimeout(faseTegenComputer, 800);
        }
        document.getElementById('troef-display').innerText = `${currentTroef} ${suitSymbols[currentTroef]}`;
    }

    // De computer mag reageren op JOUW troefkeuze
    function faseTegenComputer() {
        // AI check: als de computer 3 of meer hoge kaarten (rank 7 of 8) heeft, gaat hij "Tegen"
        let hogeKaarten = computerHand.filter(c => c.rank >= 7 || c.suit === currentTroef);
        
        if (hogeKaarten.length >= 4) {
            scoreMultiplier = 2;
            document.getElementById('multi-display').innerText = "(TEGEN! ×2)";
            setMessage(`De computer roept: TEGEN! De inzet verdubbelt.`);
            // Geef speler kans op hertegen
            faseHertegenSpeler();
        } else {
            startHetSpel();
        }
    }

    // Jij mag reageren op de troefkeuze van de computer
    function faseTegenSpeler() {
        document.getElementById('menu-overlay').style.display = 'flex';
        document.getElementById('menu-title').innerText = "Computer koos troef!";
        document.getElementById('menu-text').innerText = `De computer heeft ${currentTroef} gekozen. Wil jij "Tegen" roepen?`;
        
        const container = document.getElementById('menu-buttons');
        container.innerHTML = `
            <button class="pass-btn" style="background:#0b6623;" onclick="reageerTegen(true)">TEGEN (×2)</button>
            <button class="suit-btn" onclick="reageerTegen(false)">Passen (Normale inzet)</button>
        `;
    }

    function reageerTegen(wilTegen) {
        if (wilTegen) {
            scoreMultiplier = 2;
            document.getElementById('multi-display').innerText = "(TEGEN! ×2)";
            setMessage(`Jij roept TEGEN! De computer overweegt hertegen...`);
            
            // Computer AI checkt voor Hertegen (als hand super sterk is)
            let superSterk = computerHand.filter(c => c.rank === 8 || (c.suit === currentTroef && c.rank >= 6));
            if (superSterk.length >= 3) {
                setTimeout(() => {
                    scoreMultiplier = 4;
                    document.getElementById('multi-display').innerText = "(HERTEGEN!! ×4)";
                    setMessage(`De computer roept: HERTEGEN!! De punten tellen maal 4!`);
                    startHetSpel();
                }, 1000);
            } else {
                setTimeout(startHetSpel, 800);
            }
        } else {
            startHetSpel();
        }
    }

    function faseHertegenSpeler() {
        document.getElementById('menu-overlay').style.display = 'flex';
        document.getElementById('menu-title').innerText = "De computer ging Tegen!";
        document.getElementById('menu-text').innerText = `De computer daagt je uit. Wil jij "Hertegen" roepen voor x4 punten?`;
        
        const container = document.getElementById('menu-buttons');
        container.innerHTML = `
            <button class="pass-btn" style="background:#ffcc00; color:black;" onclick="reageerHertegen(true)">HERTEGEN (×4)</button>
            <button class="suit-btn" onclick="reageerHertegen(false)">Passen</button>
        `;
    }

    function reageerHertegen(wilHertegen) {
        if (wilHertegen) {
            scoreMultiplier = 4;
            document.getElementById('multi-display').innerText = "(HERTEGEN!! ×4)";
            setMessage(`Jij roept HERTEGEN! De inzet staat op vierdubbel!`);
        }
        startHetSpel();
    }

    function startHetSpel() {
        document.getElementById('menu-overlay').style.display = 'none';
        updateUI();
    }

    function deelKaarten() {
        playerHand = deck.splice(0, 8); computerHand = deck.splice(0, 8);
        document.getElementById('score-speler').innerText = playerPoints;
        document.getElementById('score-comp').innerText = computerPoints;
        document.getElementById('deck-count').innerText = deck.length;
        document.getElementById('next-btn').style.display = 'none';
        updateUI();
    }

    function getGeldigeKaarten(hand, tegenstanderKaart) {
        // Als je zelf de eerste kaart legt, mag je alles spelen
        if (!tegenstanderKaart || !leadingSuit) {
            return hand;
        }

        // 1. Probeer de gevraagde kleur te volgen
        let dezelfdeKleur = hand.filter(c => c.suit === leadingSuit);
        if (dezelfdeKleur.length > 0) {
            // VERPLICHT OVERSTAG GAAN: Als je een hogere kaart hebt in dezelfde kleur, MOET je die spelen
            let hogereKaarten = dezelfdeKleur.filter(c => c.rank > tegenstanderKaart.rank);
            if (hogereKaarten.length > 0) {
                return hogereKaarten; // Je bent verplicht om hoger te spelen
            }
            return dezelfdeKleur; // Je kan niet hoger, maar moet wel de kleur volgen
        }

        // 2. Kan je de kleur niet volgen? Dan moet je verplicht troeven (kopen)
        let troefKaarten = hand.filter(c => c.suit === currentTroef);
        if (troefKaarten.length > 0) {
            // Als de tegenstander al troef heeft gelegd, moet je proberen over te troeven
            if (tegenstanderKaart.suit === currentTroef) {
                let hogereTroeven = troefKaarten.filter(c => c.rank > tegenstanderKaart.rank);
                if (hogereTroeven.length > 0) {
                    return hogereTroeven; // Verplicht overtroeven
                }
            }
            return troefKaarten; // Gewoon kopen
        }

        // 3. Geen kleur en geen troef? Dan mag je alles wegsmijten (brieven)
        return hand;
    }

    function playCard(index) {
        if (turn !== 'player' || tableCardPlayer !== null) return;
        const card = playerHand[index];
        const geldige = getGeldigeKaarten(playerHand, tableCardComp);
        if (!geldige.some(c => c.suit === card.suit && c.name === card.name)) {
            setMessage('Ongeldige zet! Volg de kleur of koop met troef.');
            return;
        }
        if (!tableCardPlayer && !tableCardComp) { leadingSuit = card.suit; wieBegonDeSlag = 'player'; }
        tableCardPlayer = card; playerHand.splice(index, 1); updateUI();
        if (tableCardComp === null) { turn = 'computer'; setTimeout(computerTurn, 600); } else { evalSlag(); }
    }

    function computerTurn() {
        if (computerHand.length === 0) return;
        
        // Haal alle kaarten op die de computer MAG leggen volgens de volgplicht
        let geldigeKaarten = getGeldigeKaarten(computerHand, tableCardPlayer);
        if (!geldigeKaarten || geldigeKaarten.length === 0) geldigeKaarten = computerHand;
        
        let gekozenKaart = null;

        // SCENARIO 1: De computer opent de slag (mag als eerste leggen)
        if (tableCardPlayer === null) {
            // Sorteer van hoge rank naar lage rank
            let gesorteerd = [...geldigeKaarten].sort((a, b) => b.rank - a.rank);
            
            // Probeer eerst een hoge niet-troef kaart te spelen om punten te pakken
            let hogeNietTroef = gesorteerd.find(c => c.suit !== currentTroef && c.rank >= 7);
            if (hogeNietTroef) {
                gekozenKaart = hogeNietTroef;
            } else {
                // Anders gewoon de hoogste kaart die hij heeft
                gekozenKaart = gesorteerd[0];
            }
            
            leadingSuit = gekozenKaart.suit;
            wieBegonDeSlag = 'computer';
        } 
        // SCENARIO 2: De computer moet reageren op jouw kaart
        else {
            // Filter de kaarten die jouw kaart kunnen verslaan
            let winnendeKaarten = geldigeKaarten.filter(c => {
                if (c.suit === tableCardPlayer.suit) return c.rank > tableCardPlayer.rank;
                if (c.suit === currentTroef && tableCardPlayer.suit !== currentTroef) return true;
                return false;
            });

            if (winnendeKaarten.length > 0) {
                // Tactisch: Win de slag met een zo LAAG mogelijke kaart (zuinig spelen)
                winnendeKaarten.sort((a, b) => a.rank - b.rank);
                gekozenKaart = winnendeKaarten[0];
            } else {
                // Hij kan niet winnen: Gooi een zo laag mogelijke kaart weg (brieven)
                // Sorteer op punten (0 punten eerst), daarna op rank (laagste rank eerst)
                let verliezendeKaarten = [...geldigeKaarten].sort((a, b) => {
                    if (a.points !== b.points) return a.points - b.points;
                    return a.rank - b.rank;
                });
                gekozenKaart = verliezendeKaarten[0];
            }
        }

        // Zoek de exacte index in de echte hand om de kaart correct te verwijderen
        const echteIndex = computerHand.findIndex(c => c.suit === gekozenKaart.suit && c.name === gekozenKaart.name);
        
        tableCardComp = computerHand[echteIndex];
        computerHand.splice(echteIndex, 1);
        updateUI();

        if (tableCardPlayer === null) {
            turn = 'player';
            setMessage('De computer is uitgekomen. Jouw beurt!');
        } else {
            evalSlag();
        }
    }

    function evalSlag() {
        if (!tableCardPlayer || !tableCardComp) return;
        let slagWinnaar = 'player', pValue = tableCardPlayer.rank, cValue = tableCardComp.rank;
        if (tableCardPlayer.suit === tableCardComp.suit) {
            if (cValue > pValue) slagWinnaar = 'computer';
        } else {
            if (tableCardComp.suit === currentTroef) slagWinnaar = 'computer';
            else if (tableCardPlayer.suit === currentTroef) slagWinnaar = 'player';
            else slagWinnaar = wieBegonDeSlag;
        }
        let puntenSlag = (tableCardPlayer.points || 0) + (tableCardComp.points || 0);
        if (slagWinnaar === 'player') { playerPoints += puntenSlag; turn = 'player'; setMessage(`Jij wint (+${puntenSlag} ptn)!`); }
        else { computerPoints += puntenSlag; turn = 'computer'; setMessage(`Computer wint (+${puntenSlag} ptn)!`); }
        document.getElementById('score-speler').innerText = playerPoints;
        document.getElementById('score-comp').innerText = computerPoints;
        updateUI();
    }

    function startVolgendeSlag() {
        tableCardPlayer = null; tableCardComp = null; leadingSuit = null;
        document.getElementById('next-btn').style.display = 'none';
        document.getElementById('restart-btn').style.display = 'none';

        if (playerHand.length === 0 && computerHand.length === 0) {
            if (deck.length > 0) {
                setMessage('Volgende 16 kaarten worden gedeeld...');
                document.getElementById('table-area').innerHTML = '<span>Nieuwe kaarten onderweg...</span>';
                setTimeout(() => { deelKaarten(); if (turn === 'computer') setTimeout(computerTurn, 600); else setMessage('Nieuwe kaarten! Jij mag uitkomen.'); }, 1000);
                return;
            } else {
                // EINDE VAN DE 32 KAARTEN: Bereken de score met de scoreMultiplier (×2 of ×4)
                let verdiendePunten = 0;
                
                if (playerPoints > 30) {
                    verdiendePunten = (playerPoints - 30) * scoreMultiplier;
                    totaalSpeler += verdiendePunten;
                    setMessage(`Spel voorbij! Jij behaalde ${playerPoints} punten. Met de vermenigvuldiger (×${scoreMultiplier}) krijg jij ${verdiendePunten} punten op het scorebord!`);
                } else if (computerPoints > 30) {
                    // FIX: De scoreMultiplier wordt nu ook correct toegepast op de computer!
                    verdiendePunten = (computerPoints - 30) * scoreMultiplier;
                    totaalComputer += verdiendePunten;
                    setMessage(`Spel voorbij! Computer behaalde ${computerPoints} punten. Met de vermenigvuldiger (×${scoreMultiplier}) krijgt de computer ${verdiendePunten} punten op het scorebord!`);
                } else {
                    setMessage(`Spel voorbij! Exact 30-30 gelijkspel. Niemand krijgt punten.`);
                }

                // Update direct de grote wedstrijd stand bovenin
                document.getElementById('totaal-speler').innerText = totaalSpeler;
                document.getElementById('totaal-comp').innerText = totaalComputer;

                // CHECK TOERNOOI WINNAAR (Eerste tot 50)
                if (totaalSpeler >= 50) {
                    setMessage(`HOERA! Je hebt de kaap van 50 punten bereikt en wint de volledige wedstrijd met ${totaalSpeler} vs ${totaalComputer}!`);
                    document.getElementById('restart-btn').innerText = "Nieuwe Wedstrijd Starten";
                } else if (totaalComputer >= 50) {
                    setMessage(`Helaas! De computer heeft de kaap van 50 punten bereikt en wint de wedstrijd met ${totaalComputer} vs ${totaalSpeler}.`);
                    document.getElementById('restart-btn').innerText = "Nieuwe Wedstrijd Starten";
                } else {
                    document.getElementById('restart-btn').innerText = "Volgende Spel Spelen";
                }
                
                document.getElementById('restart-btn').style.display = 'inline-block';
                updateUI(); 
                return;
            }
        }
        updateUI();
        if (turn === 'computer') setTimeout(computerTurn, 600);
    }

    function createCardElement(card, isHidden = false, onClick = null) {
        const div = document.createElement('div');
        if (isHidden) { div.className = 'card back'; return div; }
        div.className = `card ${(card.suit === 'Harten' || card.suit === 'Ruiten') ? 'red' : ''}`;
        div.innerHTML = `<div>${card.name}<br>${suitSymbols[card.suit]}</div><div style="font-size:24px; text-align:center;">${suitSymbols[card.suit]}</div><div style="text-align:right;">${card.name}</div>`;
        if (onClick) div.onclick = onClick;
        return div;
    }

    function updateUI() {
        const pHandEl = document.getElementById('player-hand'); pHandEl.innerHTML = '';
        playerHand.forEach((card, index) => { pHandEl.appendChild(createCardElement(card, false, () => playCard(index))); });
        const cHandEl = document.getElementById('computer-hand'); cHandEl.innerHTML = '';
        computerHand.forEach(() => { cHandEl.appendChild(createCardElement(null, true)); });
        const tableArea = document.getElementById('table-area'); tableArea.innerHTML = '';
        
        if (tableCardPlayer) tableArea.appendChild(createCardElement(tableCardPlayer));
        if (tableCardComp) tableArea.appendChild(createCardElement(tableCardComp));
        
        if (tableCardPlayer && tableCardComp) { 
            document.getElementById('next-btn').style.display = 'inline-block'; 
        } else {
            document.getElementById('next-btn').style.display = 'none';
            if (!tableCardPlayer && !tableCardComp) {
                if (playerHand.length > 0 || computerHand.length > 0 || deck.length > 0) {
                    const span = document.createElement('span'); 
                    span.innerText = turn === 'player' ? 'Jouw beurt om uit te komen' : 'Computer denkt...';
                    tableArea.appendChild(span);
                } else {
                    tableArea.innerHTML = '<span>Spel afgelopen</span>';
                }
            }
        }
        document.getElementById('deck-count').innerText = deck.length;
    }

    function setMessage(msg) { document.getElementById('message').innerText = msg; }
    window.onload = initGame;
</script>
</body>
</html>
