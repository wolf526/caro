<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cờ Caro AI</title>
    <style>
        body {
            text-align: center;
            font-family: Arial, sans-serif;
        }
        h1 {
            color: #ff4081;
        }
        .board {
            display: grid;
            grid-template-columns: repeat(15, 40px);
            grid-template-rows: repeat(15, 40px);
            gap: 2px;
            margin: 20px auto;
            width: fit-content;
        }
        .cell {
            width: 40px;
            height: 40px;
            background-color: #f0f0f0;
            border: 1px solid #ccc;
            font-size: 24px;
            text-align: center;
            line-height: 40px;
            cursor: pointer;
        }
        .cell.taken {
            cursor: not-allowed;
        }
    </style>
</head>
<body>
    <h1>Chơi Cờ Caro Với AI 🤖</h1>
    <div class="board" id="board"></div>
    <h2 id="status">Lượt của bạn (X)</h2>

    <script>
        const SIZE = 15;
        const PLAYER = 'X';
        const AI = 'O';
        let board = Array(SIZE).fill().map(() => Array(SIZE).fill(null));

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
            let row = event.target.dataset.row;
            let col = event.target.dataset.col;

            if (board[row][col] || checkWin(PLAYER) || checkWin(AI)) return;

            board[row][col] = PLAYER;
            event.target.textContent = PLAYER;
            event.target.classList.add("taken");

            if (checkWin(PLAYER)) {
                document.getElementById("status").textContent = "Bạn thắng! 🎉";
                return;
            }

            document.getElementById("status").textContent = "Lượt của AI...";
            setTimeout(aiMove, 500);
        }

        function aiMove() {
            let bestMove = findBestMove();
            if (!bestMove) return;

            let [row, col] = bestMove;
            board[row][col] = AI;

            let cell = document.querySelector(`[data-row='${row}'][data-col='${col}']`);
            cell.textContent = AI;
            cell.classList.add("taken");

            if (checkWin(AI)) {
                document.getElementById("status").textContent = "AI thắng! 🤖";
            } else {
                document.getElementById("status").textContent = "Lượt của bạn (X)";
            }
        }

        function findBestMove() {
            let availableMoves = [];
            for (let i = 0; i < SIZE; i++) {
                for (let j = 0; j < SIZE; j++) {
                    if (!board[i][j]) availableMoves.push([i, j]);
                }
            }
            return availableMoves.length ? availableMoves[Math.floor(Math.random() * availableMoves.length)] : null;
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

        createBoard();
    </script>
</body>
</html>
