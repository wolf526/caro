<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cờ Caro</title>
    <style>
        body { text-align: center; font-family: Arial, sans-serif; }
        h1 { color: #ff4081; }
        .board { display: grid; grid-template-columns: repeat(15, 40px); grid-template-rows: repeat(15, 40px); gap: 2px; margin: 20px auto; width: fit-content; }
        .cell { width: 40px; height: 40px; background-color: #f0f0f0; border: 1px solid #ccc; font-size: 24px; text-align: center; line-height: 40px; cursor: pointer; }
        .cell.taken { cursor: not-allowed; }
        .scoreboard { font-size: 20px; margin-top: 20px; }
        button { padding: 10px 20px; font-size: 18px; margin-top: 20px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Chơi Cờ Caro Với AI</h1>
    <div class="scoreboard">
        <span>Người chơi (X): <strong id="playerScore">0</strong></span> |
        <span>AI (O): <strong id="aiScore">0</strong></span>
    </div>
    <h2 id="status">Lượt của bạn (X)</h2>
    <div class="board" id="board"></div>
    <button onclick="resetGame()">Chơi Lại</button>

    <script>
        const SIZE = 15;
        const PLAYER = 'X';
        const AI = 'O';
        let board, aiFirstMove, playerTurn;
        let playerScore = 0;
        let aiScore = 0;

        function initGame() {
            board = Array(SIZE).fill().map(() => Array(SIZE).fill(null));
            aiFirstMove = true;
            playerTurn = true;
            createBoard();
            document.getElementById("status").textContent = "Lượt của bạn (X)";
        }

        function createBoard() {
            const boardElement = document.getElementById("board");
            boardElement.innerHTML = "";
            for (let i = 0; i < SIZE; i++) {
                for (let j = 0; j < SIZE; j++) {
                    const cell = document.createElement("div");
                    cell.classList.add("cell");
                    cell.dataset.row = i;
                    cell.dataset.col = j;
                    cell.addEventListener("click", playerMove);
                    boardElement.appendChild(cell);
                }
            }
        }

        function playerMove(event) {
            if (!playerTurn) return;

            let row = event.target.dataset.row;
            let col = event.target.dataset.col;

            if (board[row][col] || checkWin(PLAYER) || checkWin(AI)) return;

            board[row][col] = PLAYER;
            event.target.textContent = PLAYER;
            event.target.classList.add("taken");

            if (checkWin(PLAYER)) {
                document.getElementById("status").textContent = "Bạn thắng! 🎉";
                playerScore++;
                updateScore();
                playerTurn = false;
                return;
            }

            playerTurn = false;
            document.getElementById("status").textContent = "Lượt của AI...";
            setTimeout(aiMove, 500);
        }

        function aiMove() {
            let bestMove = findBestMove();
            if (!bestMove) return;

            let [row, col] = bestMove;
            board[row][col] = AI;
            aiFirstMove = false;

            let cell = document.querySelector(`[data-row='${row}'][data-col='${col}']`);
            cell.textContent = AI;
            cell.classList.add("taken");

            if (checkWin(AI)) {
                document.getElementById("status").textContent = "AI thắng! 🤖";
                aiScore++;
                updateScore();
                return;
            }

            playerTurn = true;
            document.getElementById("status").textContent = "Lượt của bạn (X)";
        }

        function findBestMove() {
            if (aiFirstMove) {
                return [Math.floor(SIZE / 2), Math.floor(SIZE / 2)];
            }

            let bestScore = -Infinity;
            let bestMove = null;

            for (let i = 0; i < SIZE; i++) {
                for (let j = 0; j < SIZE; j++) {
                    if (!board[i][j]) {
                        let score = evaluateMove(i, j);
                        if (score > bestScore) {
                            bestScore = score;
                            bestMove = [i, j];
                        }
                    }
                }
            }
            return bestMove;
        }

        function evaluateMove(row, col) {
            let score = 0;
            score += checkLineScore(row, col, 1, 0);
            score += checkLineScore(row, col, 0, 1);
            score += checkLineScore(row, col, 1, 1);
            score += checkLineScore(row, col, 1, -1);
            return score;
        }

        function checkLineScore(row, col, dRow, dCol) {
            let playerCount = 0, aiCount = 0, emptyCount = 0;
            
            for (let i = -4; i <= 4; i++) {
                let r = row + i * dRow, c = col + i * dCol;
                if (r >= 0 && c >= 0 && r < SIZE && c < SIZE) {
                    if (board[r][c] === PLAYER) playerCount++;
                    if (board[r][c] === AI) aiCount++;
                    if (!board[r][c]) emptyCount++;
                }
            }

            if (playerCount === 4 && emptyCount >= 1) return 100000;
            if (aiCount === 4 && emptyCount >= 1) return 10000;
            if (playerCount === 3 && emptyCount >= 1) return 9999;
            if (aiCount === 3 && emptyCount >= 1) return 9000;
            if (playerCount === 2 && emptyCount >= 2) return 800;
            if (aiCount === 2 && emptyCount >= 2) return 700;

            return 0;
        }

        function checkWin(player) {
            for (let i = 0; i < SIZE; i++) {
                for (let j = 0; j < SIZE; j++) {
                    if (checkDirection(i, j, 1, 0, player) || 
                        checkDirection(i, j, 0, 1, player) || 
                        checkDirection(i, j, 1, 1, player) || 
                        checkDirection(i, j, 1, -1, player)) {
                        return true;
                    }
                }
            }
            return false;
        }

        function checkDirection(row, col, dRow, dCol, player) {
            let count = 0;
            for (let i = 0; i < 5; i++) {
                let r = row + i * dRow, c = col + i * dCol;
                if (r < 0 || c < 0 || r >= SIZE || c >= SIZE || board[r][c] !== player) break;
                count++;
            }
            return count === 5;
        }

        function updateScore() {
            document.getElementById("playerScore").textContent = playerScore;
            document.getElementById("aiScore").textContent = aiScore;
        }

        function resetGame() {
            initGame();
        }

        initGame();
    </script>
</body>
</html>
