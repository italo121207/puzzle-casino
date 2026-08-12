# puzzle-casino
index.html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Puzzle Casino</title>

<style>

* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    min-height: 100vh;
    background:
        radial-gradient(circle at top, #55208c 0%, #170923 45%, #07030b 100%);
    color: white;
    font-family: Arial, sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 15px;
}

.game {
    width: 100%;
    max-width: 430px;
    background: rgba(12, 7, 20, .97);
    border: 2px solid #9b55ff;
    border-radius: 25px;
    padding: 18px;
    box-shadow: 0 0 35px rgba(155, 85, 255, .35);
}

header {
    text-align: center;
    margin-bottom: 15px;
}

header h1 {
    font-size: 32px;
    margin-bottom: 5px;
}

header p {
    color: #bdb2c9;
    font-size: 14px;
}

.stats {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 8px;
    margin-bottom: 15px;
}

.stat {
    background: #21112f;
    border-radius: 12px;
    padding: 10px 5px;
    text-align: center;
}

.stat small {
    display: block;
    color: #aaa;
    font-size: 11px;
}

.stat strong {
    display: block;
    margin-top: 4px;
    font-size: 17px;
}

#credits {
    color: #ffd84d;
}

#score {
    color: #62e7ff;
}

#combo {
    color: #ff70dc;
}

#board {
    width: 100%;
    aspect-ratio: 1 / 1;
    display: grid;
    grid-template-columns: repeat(8, 1fr);
    gap: 5px;
    padding: 8px;
    background: #0c0612;
    border-radius: 17px;
    border: 1px solid #43245e;
}

.tile {
    border: none;
    border-radius: 9px;
    font-size: clamp(20px, 7vw, 34px);
    cursor: pointer;
    background: #261433;
    transition: transform .12s, filter .12s;
    user-select: none;
}

.tile:hover {
    filter: brightness(1.25);
}

.tile:active {
    transform: scale(.88);
}

.selected {
    outline: 3px solid white;
    transform: scale(.88);
}

.matching {
    animation: pop .25s ease-in-out;
}

@keyframes pop {
    0% {
        transform: scale(1);
    }

    50% {
        transform: scale(1.25);
        filter: brightness(2);
    }

    100% {
        transform: scale(0);
    }
}

.controls {
    display: flex;
    gap: 8px;
    margin-top: 15px;
}

button.control {
    flex: 1;
    border: none;
    padding: 13px 8px;
    border-radius: 12px;
    color: white;
    background: linear-gradient(135deg, #7137b8, #a634b8);
    font-weight: bold;
    cursor: pointer;
}

button.control:active {
    transform: scale(.96);
}

.message {
    min-height: 45px;
    margin-top: 12px;
    background: #170d22;
    border-radius: 12px;
    padding: 13px;
    text-align: center;
    font-weight: bold;
    color: #e7d9f4;
}

.rules {
    margin-top: 14px;
    background: #130a1c;
    padding: 13px;
    border-radius: 13px;
    color: #aaa;
    font-size: 12px;
    line-height: 1.5;
}

.rules strong {
    color: white;
}

.footer {
    text-align: center;
    margin-top: 12px;
    font-size: 11px;
    color: #6e6475;
}

</style>
</head>

<body>

<div class="game">

<header>
    <h1>🧩 Puzzle Casino</h1>
    <p>Combine peças e conquiste créditos fictícios</p>
</header>

<div class="stats">

    <div class="stat">
        <small>CRÉDITOS</small>
        <strong id="credits">1000</strong>
    </div>

    <div class="stat">
        <small>PONTOS</small>
        <strong id="score">0</strong>
    </div>

    <div class="stat">
        <small>COMBO</small>
        <strong id="combo">x1</strong>
    </div>

</div>

<div id="board"></div>

<div class="controls">

    <button class="control" onclick="newGame()">
        🔄 Novo jogo
    </button>

    <button class="control" onclick="shuffleBoard()">
        🔀 Embaralhar
    </button>

</div>

<div class="message" id="message">
    🧩 Toque em duas peças vizinhas para trocar!
</div>

<div class="rules">

    <strong>Como jogar:</strong><br>

    Troque duas peças vizinhas.
    Forme linhas ou colunas com 3 ou mais símbolos iguais.
    Quanto maior a combinação, maior a pontuação.

    <br><br>

    <strong>Combinações:</strong><br>

    3 peças → +30 pontos<br>
    4 peças → +60 pontos<br>
    5+ peças → +100 pontos<br>
    Combo → bônus adicional

</div>

<div class="footer">
    Protótipo educativo • Créditos sem valor monetário
</div>

</div>

<script>

/* =========================
   CONFIGURAÇÕES
========================= */

const SIZE = 8;

const SYMBOLS = [
    "💎",
    "⭐",
    "🍒",
    "🍋",
    "🍊",
    "🍉"
];

let board = [];

let credits = 1000;

let score = 0;

let combo = 1;

let selected = null;

let busy = false;


/* =========================
   ELEMENTOS
========================= */

const boardElement =
    document.getElementById("board");

const creditsElement =
    document.getElementById("credits");

const scoreElement =
    document.getElementById("score");

const comboElement =
    document.getElementById("combo");

const messageElement =
    document.getElementById("message");


/* =========================
   ATUALIZAR INTERFACE
========================= */

function updateStats() {

    creditsElement.textContent =
        credits;

    scoreElement.textContent =
        score;

    comboElement.textContent =
        "x" + combo;

}


/* =========================
   SÍMBOLO ALEATÓRIO
========================= */

function randomSymbol() {

    return SYMBOLS[
        Math.floor(
            Math.random() * SYMBOLS.length
        )
    ];

}


/* =========================
   CRIAR TABULEIRO
========================= */

function createBoard() {

    board = [];

    for (let row = 0; row < SIZE; row++) {

        board[row] = [];

        for (let col = 0; col < SIZE; col++) {

            let symbol;

            do {

                symbol = randomSymbol();

            } while (
                createsInitialMatch(
                    row,
                    col,
                    symbol
                )
            );

            board[row][col] = symbol;

        }

    }

}


/* =========================
   EVITAR COMBOS INICIAIS
========================= */

function createsInitialMatch(row, col, symbol) {

    if (
        col >= 2 &&
        board[row][col - 1] === symbol &&
        board[row][col - 2] === symbol
    ) {
        return true;
    }

    if (
        row >= 2 &&
        board[row - 1][col] === symbol &&
        board[row - 2][col] === symbol
    ) {
        return true;
    }

    return false;

}


/* =========================
   DESENHAR TABULEIRO
========================= */

function renderBoard() {

    boardElement.innerHTML = "";

    for (let row = 0; row < SIZE; row++) {

        for (let col = 0; col < SIZE; col++) {

            const tile =
                document.createElement("button");

            tile.className = "tile";

            tile.textContent =
                board[row][col] || "";

            tile.dataset.row = row;
            tile.dataset.col = col;

            tile.onclick = () =>
                selectTile(row, col);

            boardElement.appendChild(tile);

        }

    }

}


/* =========================
   SELECIONAR PEÇA
========================= */

function selectTile(row, col) {

    if (busy) return;

    const tile =
        getTile(row, col);

    if (selected === null) {

        selected = { row, col };

        tile.classList.add("selected");

        messageElement.textContent =
            "Agora escolha uma peça vizinha.";

        return;

    }

    if (
        selected.row === row &&
        selected.col === col
    ) {

        tile.classList.remove("selected");

        selected = null;

        return;

    }

    if (
        !areNeighbors(
            selected.row,
            selected.col,
            row,
            col
        )
    ) {

        messageElement.textContent =
            "⚠️ Escolha uma peça vizinha.";

        return;

    }

    const first = selected;

    selected = null;

    swapTiles(
        first.row,
        first.col,
        row,
        col
    );

}


/* =========================
   VERIFICAR VIZINHOS
========================= */

function areNeighbors(r1, c1, r2, c2) {

    const distance =
        Math.abs(r1 - r2) +
        Math.abs(c1 - c2);

    return distance === 1;

}


/* =========================
   TROCAR PEÇAS
========================= */

async function swapTiles(
    r1,
    c1,
    r2,
    c2
) {

    busy = true;

    const firstTile =
        getTile(r1, c1);

    if (firstTile) {
        firstTile.classList.remove(
            "selected"
        );
    }

    const temp =
        board[r1][c1];

    board[r1][c1] =
        board[r2][c2];

    board[r2][c2] =
        temp;

    renderBoard();

    const matches =
        findMatches();

    if (matches.length === 0) {

        await delay(250);

        const tempBack =
            board[r1][c1];

        board[r1][c1] =
            board[r2][c2];

        board[r2][c2] =
            tempBack;

        renderBoard();

        combo = 1;

        updateStats();

        messageElement.textContent =
            "❌ Nenhuma combinação. Tente outra troca.";

        busy = false;

        return;

    }

    await resolveMatches();

    busy = false;

}


/* =========================
   ENCONTRAR COMBINAÇÕES
========================= */

function findMatches() {

    const matches = new Set();

    /* LINHAS */

    for (let row = 0; row < SIZE; row++) {

        let start = 0;

        for (
            let col = 1;
            col <= SIZE;
            col++
        ) {

            const current =
                col < SIZE
                    ? board[row][col]
                    : null;

            const previous =
                board[row][start];

            if (
                current === previous &&
                current !== null
            ) {
                continue;
            }

            const length =
                col - start;

            if (length >= 3) {

                for (
                    let i = start;
                    i < col;
                    i++
                ) {

                    matches.add(
                        `${row},${i}`
                    );

                }

            }

            start = col;

        }

    }


    /* COLUNAS */

    for (let col = 0; col < SIZE; col++) {

        let start = 0;

        for (
            let row = 1;
            row <= SIZE;
            row++
        ) {

            const current =
                row < SIZE
                    ? board[row][col]
                    : null;

            const previous =
                board[start][col];

            if (
                current === previous &&
                current !== null
            ) {
                continue;
            }

            const length =
                row - start;

            if (length >= 3) {

                for (
                    let i = start;
                    i < row;
                    i++
                ) {

                    matches.add(
                        `${i},${col}`
                    );

                }

            }

            start = row;

        }

    }

    return [...matches];

}


/* =========================
   PROCESSAR COMBOS
========================= */

async function resolveMatches() {

    let chain = 0;

    while (true) {

        const matches =
            findMatches();

        if (matches.length === 0) {
            break;
        }

        chain++;

        combo = chain;

        updateStats();

        highlightMatches(matches);

        await delay(300);

        const amount =
            matches.length;

        let points = 0;

        if (amount === 3) {

            points = 30;

        } else if (amount === 4) {

            points = 60;

        } else {

            points = 100;

        }

        const bonus =
            (combo - 1) * 20;

        points += bonus;

        score += points;

        credits += points;

        updateStats();

        messageElement.textContent =
            `🎉 +${points} pontos! Combo x${combo}`;

        removeMatches(matches);

        await delay(200);

        collapseBoard();

        renderBoard();

        await delay(250);

    }

    combo = 1;

    updateStats();

}


/* =========================
   DESTACAR COMBOS
========================= */

function highlightMatches(matches) {

    matches.forEach(key => {

        const [row, col] =
            key.split(",").map(Number);

        const tile =
            getTile(row, col);

        if (tile) {

            tile.classList.add(
                "matching"
            );

        }

    });

}


/* =========================
   REMOVER PEÇAS
========================= */

function removeMatches(matches) {

    matches.forEach(key => {

        const [row, col] =
            key.split(",").map(Number);

        board[row][col] = null;

    });

}


/* =========================
   FAZER PEÇAS CAÍREM
========================= */

function collapseBoard() {

    for (let col = 0; col < SIZE; col++) {

        const remaining = [];

        for (
            let row = SIZE - 1;
            row >= 0;
            row--
        ) {

            if (
                board[row][col] !== null
            ) {

                remaining.push(
                    board[row][col]
                );

            }

        }

        for (
            let row = SIZE - 1;
            row >= 0;
            row--
        ) {

            const index =
                SIZE - 1 - row;

            board[row][col] =
                remaining[index] ||
                randomSymbol();

        }

    }

}


/* =========================
   EMBARALHAR
========================= */

function shuffleBoard() {

    if (busy) return;

    for (let row = 0; row < SIZE; row++) {

        for (let col = 0; col < SIZE; col++) {

            board[row][col] =
                randomSymbol();

        }

    }

    renderBoard();

    combo = 1;

    updateStats();

    messageElement.textContent =
        "🔀 Tabuleiro embaralhado!";

}


/* =========================
   NOVO JOGO
========================= */

function newGame() {

    if (busy) return;

    credits = 1000;

    score = 0;

    combo = 1;

    selected = null;

    createBoard();

    renderBoard();

    updateStats();

    messageElement.textContent =
        "🧩 Novo jogo iniciado!";

}


/* =========================
   PEGAR PEÇA
========================= */

function getTile(row, col) {

    return document.querySelector(
        `.tile[data-row="${row}"][data-col="${col}"]`
    );

}


/* =========================
   DELAY
========================= */

function delay(ms) {

    return new Promise(
        resolve => setTimeout(resolve, ms)
    );

}


/* =========================
   INICIAR
========================= */

newGame();

</script>

</body>
</html>