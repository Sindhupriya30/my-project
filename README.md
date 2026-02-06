# my-project
A Memory Matching Game challenges players to find pairs of hidden cards by recalling their positions. Flip two cards at a time, remember patterns, and match all pairs to win. Simple yet addictive, it boosts focus, sharpens memory, and offers a fun, customizable brain-training experience.
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Memory Match - Custom Grids</title>
    <style>
        /* --- CSS STYLES --- */
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background-color: #2c3e50;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: white;
        }

        .game-container {
            background: #34495e;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.5);
            text-align: center;
            max-width: 800px;
            width: 95%;
        }

        h1 { margin: 10px 0 20px; }

        /* Controls Section */
        .controls {
            margin-bottom: 20px;
            display: flex;
            justify-content: center;
            gap: 8px;
            flex-wrap: wrap;
        }

        .btn {
            padding: 8px 16px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            transition: transform 0.2s, background 0.3s;
            color: white;
            font-size: 0.9em;
        }

        .btn:active { transform: scale(0.95); }
        .btn-restart { background-color: #e74c3c; }
        
        /* Level Buttons */
        .btn-level { background-color: #3498db; }
        .btn-level:hover { background-color: #2980b9; }

        .stats {
            display: flex;
            justify-content: space-around;
            margin-bottom: 15px;
            font-size: 1.2em;
            background: rgba(0,0,0,0.2);
            padding: 10px;
            border-radius: 8px;
        }

        /* THE DYNAMIC GRID */
        .game-board {
            display: grid;
            gap: 8px;
            margin: 0 auto;
            justify-content: center;
            perspective: 1000px;
        }

        /* CARD STYLING */
        .card {
            background-color: transparent;
            cursor: pointer;
            position: relative;
        }

        .card-inner {
            position: relative;
            width: 100%;
            height: 100%;
            text-align: center;
            transition: transform 0.6s;
            transform-style: preserve-3d;
        }

        .card.flipped .card-inner { transform: rotateY(180deg); }

        .card-face {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }

        .card-back { background-color: #3498db; color: #fff; font-weight: bold; }
        .card-front { background-color: #ecf0f1; transform: rotateY(180deg); color: #333; }

        .card.matched .card-back, .card.matched .card-front {
            border: 2px solid #2ecc71;
            opacity: 0.6;
        }

        /* WIN MODAL */
        .modal {
            display: none;
            position: fixed;
            z-index: 100;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0,0,0,0.8);
            backdrop-filter: blur(5px);
        }

        .modal-content {
            background-color: #fff;
            color: #333;
            margin: 15% auto;
            padding: 30px;
            border-radius: 10px;
            width: 90%;
            max-width: 400px;
            text-align: center;
        }
        
        .modal-content button {
            background-color: #2ecc71;
            margin-top: 15px;
        }
    </style>
</head>
<body>

    <div class="game-container">
        <h1>Memory Match</h1>
        
        <div class="controls">
            <button class="btn btn-level" onclick="startGame(4, 4)">4x4 (16)</button>
            <button class="btn btn-level" onclick="startGame(5, 4)">5x4 (20)</button>
            <button class="btn btn-level" onclick="startGame(5, 6)">5x6 (30)</button>
            <button class="btn btn-level" onclick="startGame(6, 6)">6x6 (36)</button>
            <button class="btn btn-level" onclick="startGame(8, 8)">8x8 (64)</button>
            <button class="btn btn-restart" onclick="restartCurrentGame()">Restart</button>
        </div>

        <div class="stats">
            <span id="moves">Moves: 0</span>
            <span id="timer">Time: 0s</span>
        </div>

        <div class="game-board" id="game-board">
            </div>
    </div>

    <div id="win-modal" class="modal">
        <div class="modal-content">
            <h2>🎉 Stage Cleared! 🎉</h2>
            <p>Time: <span id="final-time"></span> | Moves: <span id="final-moves"></span></p>
            <button class="btn" onclick="winModal.style.display='none'; restartCurrentGame()">Play Again</button>
        </div>
    </div>

    <script>
        // --- JAVASCRIPT LOGIC ---
        
        const allIcons = [
            '🍎','🍌','🍇','🍉','🍓','🍒','🍍','🥝',
            '🥑','🌽','🥕','🥦','🥨','🍔','🍕','🌭',
            '🐶','🐱','🐭','🐹','🐰','🦊','🐻','🐼',
            '⚽','🏀','🏈','🎾','🏐','🎱','🏓','🏸' 
        ]; // 32 pairs max (64 cards)

        let gameCards = [];
        let flippedCards = [];
        let lockBoard = false;
        let moves = 0;
        let matches = 0;
        let timer = null;
        let time = 0;
        
        // Store current config to allow restart
        let currentRows = 4;
        let currentCols = 4;

        const gameBoard = document.getElementById('game-board');
        const movesDisplay = document.getElementById('moves');
        const timerDisplay = document.getElementById('timer');
        const winModal = document.getElementById('win-modal');

        function shuffle(array) {
            let currentIndex = array.length, randomIndex;
            while (currentIndex != 0) {
                randomIndex = Math.floor(Math.random() * currentIndex);
                currentIndex--;
                [array[currentIndex], array[randomIndex]] = [array[randomIndex], array[currentIndex]];
            }
            return array;
        }

        function startTimer() {
            stopTimer();
            time = 0;
            timerDisplay.textContent = `Time: 0s`;
            timer = setInterval(() => {
                time++;
                timerDisplay.textContent = `Time: ${time}s`;
            }, 1000);
        }

        function stopTimer() {
            clearInterval(timer);
            timer = null;
        }

        // --- NEW: Function now accepts Rows and Cols ---
        function startGame(rows, cols) {
            currentRows = rows;
            currentCols = cols;

            stopTimer();
            moves = 0;
            matches = 0;
            time = 0;
            movesDisplay.textContent = `Moves: 0`;
            timerDisplay.textContent = `Time: 0s`;
            flippedCards = [];
            lockBoard = false;
            winModal.style.display = 'none';

            // 1. Calculate needed pairs
            const totalCards = rows * cols;
            const pairsNeeded = totalCards / 2;

            // 2. Select icons
            const selectedIcons = allIcons.slice(0, pairsNeeded);
            const cardPairs = [...selectedIcons, ...selectedIcons];
            gameCards = shuffle(cardPairs);

            // 3. Setup CSS Grid Columns
            gameBoard.innerHTML = '';
            // This line ensures the grid has the exact number of columns requested
            gameBoard.style.gridTemplateColumns = `repeat(${cols}, 1fr)`;

            // 4. Dynamic Sizing Logic
            // We adjust card size based on how crowded the grid is (using columns as the metric)
            let cardSize = 80;
            let fontSize = 2.5;

            if (cols === 4) { cardSize = 80; fontSize = 2.5; }
            if (cols === 5 || cols === 6) { cardSize = 60; fontSize = 2; }
            if (cols >= 8) { cardSize = 45; fontSize = 1.5; }

            // 5. Generate Cards
            gameCards.forEach(icon => {
                const cardElement = document.createElement('div');
                cardElement.classList.add('card');
                cardElement.dataset.icon = icon;
                
                cardElement.style.width = `${cardSize}px`;
                cardElement.style.height = `${cardSize}px`;

                cardElement.innerHTML = `
                    <div class="card-inner">
                        <div class="card-face card-back" style="font-size: ${fontSize}em">?</div>
                        <div class="card-face card-front" style="font-size: ${fontSize}em">${icon}</div>
                    </div>
                `;
                cardElement.addEventListener('click', handleCardClick);
                gameBoard.appendChild(cardElement);
            });
        }

        function restartCurrentGame() {
            startGame(currentRows, currentCols);
        }

        function handleCardClick() {
            if (time === 0 && !timer) startTimer();
            if (lockBoard || this.classList.contains('matched') || this.classList.contains('flipped')) return;

            this.classList.add('flipped');
            flippedCards.push(this);

            if (flippedCards.length === 2) {
                lockBoard = true;
                moves++;
                movesDisplay.textContent = `Moves: ${moves}`;
                checkForMatch();
            }
        }

        function checkForMatch() {
            const [card1, card2] = flippedCards;
            if (card1.dataset.icon === card2.dataset.icon) {
                card1.classList.add('matched');
                card2.classList.add('matched');
                matches++;
                flippedCards = [];
                lockBoard = false;
                
                const totalPairs = (currentRows * currentCols) / 2;
                if (matches === totalPairs) gameOver();
            } else {
                setTimeout(() => {
                    card1.classList.remove('flipped');
                    card2.classList.remove('flipped');
                    flippedCards = [];
                    lockBoard = false;
                }, 1000);
            }
        }

        function gameOver() {
            stopTimer();
            document.getElementById('final-time').textContent = `${time}s`;
            document.getElementById('final-moves').textContent = moves;
            winModal.style.display = 'block';
        }

        // Initialize with standard 4x4
        startGame(4, 4);
    </script>
</body>
</html>
