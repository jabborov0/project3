# project3
> Ⓔ︎Ⓛ︎Ⓑ︎Ⓔ︎Ⓚ︎:
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Neon Tic-Tac-Toe v2.0</title>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        :root {
            --bg: #050505;
            --cell: #1a1a1a;
            --x-color: #ff007f;
            --o-color: #00f2ff;
            --shadow: 0 0 20px;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Orbitron', sans-serif; }

        body {
            background: var(--bg);
            color: white;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }

        .menu { margin-bottom: 30px; text-align: center; }
        .mode-btn {
            background: transparent;
            border: 2px solid #444;
            color: #aaa;
            padding: 10px 20px;
            margin: 5px;
            cursor: pointer;
            border-radius: 8px;
            transition: 0.3s;
        }
        .mode-btn.active {
            border-color: var(--o-color);
            color: white;
            box-shadow: var(--shadow) var(--o-color);
        }

        #status {
            font-size: 24px;
            margin-bottom: 20px;
            text-transform: uppercase;
            letter-spacing: 2px;
            height: 30px;
        }

        .grid {
            display: grid;
            grid-template-columns: repeat(3, 110px);
            grid-template-rows: repeat(3, 110px);
            gap: 15px;
        }

        .cell {
            background: var(--cell);
            border-radius: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 50px;
            cursor: pointer;
            transition: 0.2s;
            border: 1px solid #333;
        }

        .cell:hover { background: #252525; }

        .cell.x { color: var(--x-color); text-shadow: var(--shadow) var(--x-color); }
        .cell.o { color: var(--o-color); text-shadow: var(--shadow) var(--o-color); }

        .cell.winner {
            animation: pulse 0.5s infinite alternate;
            background: #333;
        }

        @keyframes pulse {
            from { transform: scale(1); opacity: 1; }
            to { transform: scale(1.05); opacity: 0.8; }
        }

        .reset-btn {
            margin-top: 40px;
            padding: 12px 35px;
            background: white;
            color: black;
            border: none;
            border-radius: 30px;
            font-weight: bold;
            cursor: pointer;
            text-transform: uppercase;
            transition: 0.3s;
        }
        .reset-btn:hover { transform: scale(1.1); box-shadow: 0 0 20px white; }

        .overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.9);
            display: none; flex-direction: column; align-items: center; justify-content: center;
            z-index: 10;
        }
        .overlay h1 { font-size: 50px; margin-bottom: 20px; text-align: center; padding: 0 20px;}
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
</head>
<body>

    <div class="menu">
        <button class="mode-btn active" onclick="setMode('pvp', this)">DO'ST BILAN</button>
        <button class="mode-btn" onclick="setMode('pve', this)">BOT BILAN</button>
    </div>

    <div id="status">X NAVBATI</div>

    <div class="grid" id="board"></div>

    <button class="reset-btn" onclick="resetGame()">Qayta boshlash</button>

    <div class="overlay" id="overlay">
        <h1 id="winnerMsg">G'ALABA!</h1>
        <button class="reset-btn" onclick="resetGame()">YANA O'YNASH</button>
    </div>

> Ⓔ︎Ⓛ︎Ⓑ︎Ⓔ︎Ⓚ︎:
<script>
        let board = Array(9).fill(null);
        let currentPlayer = 'X';
        let gameActive = true;
        let gameMode = 'pvp';
        const winConditions = [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],
            [0, 3, 6], [1, 4, 7], [2, 5, 8],
            [0, 4, 8], [2, 4, 6]
        ];

        const boardElement = document.getElementById('board');
        const statusDisplay = document.getElementById('status');
        const overlay = document.getElementById('overlay');
        const winnerMsg = document.getElementById('winnerMsg');

        function setMode(mode, btn) {
            gameMode = mode;
            document.querySelectorAll('.mode-btn').forEach(b => b.classList.remove('active'));
            btn.classList.add('active');
            resetGame();
        }

        function createBoard() {
            boardElement.innerHTML = '';
            board.forEach((cell, index) => {
                const cellElement = document.createElement('div');
                cellElement.classList.add('cell');
                cellElement.dataset.index = index;
                cellElement.addEventListener('click', handleCellClick);
                boardElement.appendChild(cellElement);
            });
        }

        function handleCellClick(e) {
            const index = e.target.dataset.index;
            if (board[index] || !gameActive) return;

            makeMove(index, currentPlayer);

            if (gameActive && gameMode === 'pve' && currentPlayer === 'O') {
                boardElement.style.pointerEvents = 'none'; // Bot o'ylayotganda bosishni taqiqlash
                setTimeout(() => {
                    botMove();
                    boardElement.style.pointerEvents = 'auto';
                }, 500);
            }
        }

        function makeMove(index, player) {
            board[index] = player;
            const cell = boardElement.children[index];
            cell.innerText = player;
            cell.classList.add(player.toLowerCase());

            if (checkWin()) {
                endGame(false);
            } else if (board.every(cell => cell !== null)) {
                endGame(true);
            } else {
                currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
                statusDisplay.innerText = ${currentPlayer} NAVBATI;
            }
        }

        function botMove() {
            if (!gameActive) return;
            
            let move = findBestMove('O');
            if (move === null) move = findBestMove('X');
            if (move === null && board[4] === null) move = 4;
            if (move === null) {
                const emptyCells = board.map((v, i) => v === null ? i : null).filter(v => v !== null);
                move = emptyCells[Math.floor(Math.random() * emptyCells.length)];
            }

            if (move !== null) makeMove(move, 'O');
        }

        function findBestMove(player) {
            for (let condition of winConditions) {
                const [a, b, c] = condition;
                const vals = [board[a], board[b], board[c]];
                if (vals.filter(v => v === player).length === 2 && vals.includes(null)) {
                    return condition[vals.indexOf(null)];
                }
            }
            return null;
        }

        function checkWin() {
            for (let condition of winConditions) {
                const [a, b, c] = condition;
                if (board[a] && board[a] === board[b] && board[a] === board[c]) {
                    highlightWinner([a, b, c]);
                    return true;
                }
            }
            return false;
        }

        function highlightWinner(indices) {
            indices.forEach(i => boardElement.children[i].classList.add('winner'));
        }

        function endGame(draw) {

> Ⓔ︎Ⓛ︎Ⓑ︎Ⓔ︎Ⓚ︎:
gameActive = false;
            setTimeout(() => {
                overlay.style.display = 'flex';
                if (draw) {
                    winnerMsg.innerText = "DURANG!";
                    winnerMsg.style.color = "white";
                } else {
                    winnerMsg.innerText = ${currentPlayer} G'ALABA QOZONDI!;
                    winnerMsg.style.color = currentPlayer === 'X' ? 'var(--x-color)' : 'var(--o-color)';
                }
            }, 500);
        }

        function resetGame() {
            board = Array(9).fill(null);
            currentPlayer = 'X';
            gameActive = true;
            statusDisplay.innerText = "X NAVBATI";
            overlay.style.display = 'none';
            boardElement.style.pointerEvents = 'auto';
            createBoard();
        }

        createBoard();
    </script>
</body>
</html>
