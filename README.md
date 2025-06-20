# Retro-Pixel-hub- <!DOCTYPE html>
<html>
<head>
    <title>5-in-1 Retro Games</title>
    <meta charset="UTF-8">
    <style>
        body {
            font-family: 'Courier New', monospace;
            background: #0f0f23;
            color: #00ff00;
            text-align: center;
            margin: 0;
            padding: 20px;
        }
        header {
            margin-bottom: 20px;
            text-shadow: 0 0 5px #00ff00;
        }
        .game-buttons {
            display: flex;
            justify-content: center;
            gap: 10px;
            flex-wrap: wrap;
            margin-bottom: 20px;
        }
        button {
            background: #003300;
            color: #00ff00;
            border: 1px solid #00ff00;
            padding: 10px 15px;
            cursor: pointer;
            font-family: inherit;
        }
        button:hover {
            background: #005500;
        }
        .game-container {
            margin: 0 auto;
            position: relative;
        }
        canvas {
            display: block;
            margin: 0 auto;
            background: #000;
            border: 2px solid #00ff00;
        }
        .score {
            margin: 10px 0;
            font-size: 1.2em;
        }
    </style>
</head>
<body>
    <header>
        <h1>🔵 5-in-1 Retro Games 🔴</h1>
    </header>

    <div class="game-buttons">
        <button onclick="startGame('snake')">Змейка</button>
        <button onclick="startGame('tetris')">Тетрис</button>
        <button onclick="startGame('pong')">Понг</button>
        <button onclick="startGame('flappy')">Flappy Bird</button>
        <button onclick="startGame('breakout')">Арканоид</button>
    </div>

    <div class="score" id="score">Счёт: 0</div>

    <div class="game-container">
        <canvas id="gameCanvas"></canvas>
    </div>

    <script>
        // Общие переменные
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        let currentGame = null;
        let score = 0;

        // Размеры холста
        function resizeCanvas() {
            const size = Math.min(window.innerWidth - 40, 600);
            canvas.width = size;
            canvas.height = size;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // ===== ЗМЕЙКА =====
        function initSnake() {
            const gridSize = 20;
            let snake = [{x: 10, y: 10}];
            let food = {x: 5, y: 5};
            let dx = 0, dy = 0;
            let speed = 150;

            function gameLoop() {
                // Движение
                const head = {x: snake[0].x + dx, y: snake[0].y + dy};
                snake.unshift(head);
                
                // Проверка на съедение еды
                if (head.x === food.x && head.y === food.y) {
                    generateFood();
                    score += 10;
                    speed = Math.max(50, speed - 5);
                } else {
                    snake.pop();
                }

                // Проверка столкновений
                if (isCollision()) {
                    gameOver();
                    return;
                }

                // Отрисовка
                drawGame();
                setTimeout(gameLoop, speed);
            }

            function drawGame() {
                // Очистка
                ctx.fillStyle = '#000';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                // Змейка
                ctx.fillStyle = '#00ff00';
                snake.forEach(segment => {
                    ctx.fillRect(
                        segment.x * gridSize, 
                        segment.y * gridSize, 
                        gridSize - 1, 
                        gridSize - 1
                    );
                });
                
                // Еда
                ctx.fillStyle = '#ff0000';
                ctx.fillRect(
                    food.x * gridSize, 
                    food.y * gridSize, 
                    gridSize - 1, 
                    gridSize - 1
                );
                
                updateScore();
            }

            function generateFood() {
                food = {
                    x: Math.floor(Math.random() * (canvas.width / gridSize)),
                    y: Math.floor(Math.random() * (canvas.height / gridSize))
                };
            }

            function isCollision() {
                const head = snake[0];
                // Стены
                if (
                    head.x < 0 || 
                    head.y < 0 || 
                    head.x >= canvas.width / gridSize || 
                    head.y >= canvas.height / gridSize
                ) return true;
                
                // Сама в себя
                for (let i = 1; i < snake.length; i++) {
                    if (head.x === snake[i].x && head.y === snake[i].y) {
                        return true;
                    }
                }
                
                return false;
            }

            function gameOver() {
                alert(`Игра окончена! Счёт: ${score}`);
                resetGame();
            }

            function resetGame() {
                snake = [{x: 10, y: 10}];
                dx = dy = 0;
                score = 0;
                speed = 150;
                generateFood();
            }

            // Управление
            document.addEventListener('keydown', e => {
                if (e.key === 'ArrowUp' && dy === 0) { dx = 0; dy = -1; }
                if (e.key === 'ArrowDown' && dy === 0) { dx = 0; dy = 1; }
                if (e.key === 'ArrowLeft' && dx === 0) { dx = -1; dy = 0; }
                if (e.key === 'ArrowRight' && dx === 0) { dx = 1; dy = 0; }
            });

            generateFood();
            gameLoop();
        }

        // ===== ТЕТРИС =====
        function initTetris() {
            const gridSize = 30;
            const cols = Math.floor(canvas.width / gridSize);
            const rows = Math.floor(canvas.height / gridSize);
            
            let grid = Array(rows).fill().map(() => Array(cols).fill(0));
            let currentPiece = null;
            let gameSpeed = 500;
            let dropCounter = 0;
            let lastTime = 0;

            // Фигуры тетриса
            const pieces = [
                [[1, 1, 1, 1]],                           // I
                [[1, 1], [1, 1]],                         // O
                [[0, 1, 0], [1, 1, 1]],                   // T
                [[1, 1, 0], [0, 1, 1]],                   // Z
                [[0, 1, 1], [1, 1, 0]],                   // S
                [[1, 0, 0], [1, 1, 1]],                   // L
                [[0, 0, 1], [1, 1, 1]]                    // J
            ];

            function createPiece() {
                const piece = pieces[Math.floor(Math.random() * pieces.length)];
                return {
                    shape: piece,
                    pos: {x: Math.floor(cols / 2) - Math.floor(piece[0].length / 2), y: 0}
                };
            }

            function draw() {
                // Очистка
                ctx.fillStyle = '#000';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                // Сетка
                ctx.strokeStyle = '#333';
                for (let y = 0; y < rows; y++) {
                    for (let x = 0; x < cols; x++) {
                        ctx.strokeRect(x * gridSize, y * gridSize, gridSize, gridSize);
                    }
                }
                
                // Фигура
                ctx.fillStyle = '#00f';
                if (currentPiece) {
                    currentPiece.shape.forEach((row, y) => {
                        row.forEach((value, x) => {
                            if (value) {
                                ctx.fillRect(
                                    (currentPiece.pos.x + x) * gridSize,
                                    (currentPiece.pos.y + y) * gridSize,
                                    gridSize - 1,
                                    gridSize - 1
                                );
                            }
                        });
                    });
                }
                
                // Стаканные фигуры
                grid.forEach((row, y) => {
                    row.forEach((value, x) => {
                        if (value) {
                            ctx.fillStyle = '#f00';
                            ctx.fillRect(
                                x * gridSize,
                                y * gridSize,
                                gridSize - 1,
                                gridSize - 1
                            );
                        }
                    });
                });
            }

            function update(time = 0) {
                const deltaTime = time - lastTime;
                lastTime = time;
                
                dropCounter += deltaTime;
                if (dropCounter > gameSpeed) {
                    movePiece(0, 1);
                    dropCounter = 0;
                }
                
                draw();
                requestAnimationFrame(update);
            }

            function movePiece(dx, dy) {
                if (!currentPiece) return;
                
                const newX = currentPiece.pos.x + dx;
                const newY = currentPiece.pos.y + dy;
                
                if (!collision(newX, newY, currentPiece.shape)) {
                    currentPiece.pos.x = newX;
                    currentPiece.pos.y = newY;
                    return true;
                }
                
                // Фигура достигла дна
                if (dy > 0) {
                    mergePiece();
                    clearLines();
                    currentPiece = createPiece();
                    
                    // Проверка проигрыша
                    if (collision(currentPiece.pos.x, currentPiece.pos.y, currentPiece.shape)) {
                        alert(`Игра окончена! Счёт: ${score}`);
                        resetGame();
                    }
                }
                
                return false;
            }

            function collision(x, y, shape) {
                for (let py = 0; py < shape.length; py++) {
                    for (let px = 0; px < shape[py].length; px++) {
                        if (shape[py][px] === 0) continue;
                        
                        const newX = x + px;
                        const newY = y + py;
                        
                        if (
                            newX < 0 || 
                            newX >= cols || 
                            newY >= rows || 
                            (newY >= 0 && grid[newY][newX])
                        ) {
                            return true;
                        }
                    }
                }
                return false;
            }

            function mergePiece() {
                currentPiece.shape.forEach((row, y) => {
                    row.forEach((value, x) => {
                        if (value) {
                            const gy = currentPiece.pos.y + y;
                            const gx = currentPiece.pos.x + x;
                            if (gy >= 0) {
                                grid[gy][gx] = 1;
                            }
                        }
                    });
                });
            }

            function clearLines() {
                let linesCleared = 0;
                
                for (let y = rows - 1; y >= 0; y--) {
                    if (grid[y].every(cell => cell === 1)) {
                        grid.splice(y, 1);
                        grid.unshift(Array(cols).fill(0));
                        linesCleared++;
                        y++; // Проверить снова эту позицию
                    }
                }
                
                if (linesCleared > 0) {
                    score += linesCleared * 100;
                    updateScore();
                    gameSpeed = Math.max(100, gameSpeed - 20);
                }
            }

            function rotatePiece() {
                if (!currentPiece) return;
                
                const rotated = [];
                for (let x = 0; x < currentPiece.shape[0].length; x++) {
                    const newRow = [];
                    for (let y = currentPiece.shape.length - 1; y >= 0; y--) {
                        newRow.push(currentPiece.shape[y][x]);
                    }
                    rotated.push(newRow);
                }
                
                if (!collision(currentPiece.pos.x, currentPiece.pos.y, rotated)) {
                    currentPiece.shape = rotated;
                }
            }

            function resetGame() {
                grid = Array(rows).fill().map(() => Array(cols).fill(0));
                currentPiece = createPiece();
                score = 0;
                gameSpeed = 500;
                updateScore();
            }

            // Управление
            document.addEventListener('keydown', e => {
                if (e.key === 'ArrowLeft') movePiece(-1, 0);
                if (e.key === 'ArrowRight') movePiece(1, 0);
                if (e.key === 'ArrowDown') movePiece(0, 1);
                if (e.key === 'ArrowUp') rotatePiece();
            });

            currentPiece = createPiece();
            update();
        }

        // ===== ПОНГ =====
        function initPong() {
            const paddleWidth = 100;
            const paddleHeight = 15;
            const ballSize = 10;
            
            let paddleX = (canvas.width - paddleWidth) / 2;
            let ballX = canvas.width / 2;
            let ballY = canvas.height / 2;
            let ballSpeedX = 5;
            let ballSpeedY = -5;
            let gameRunning = true;

            function draw() {
                // Очистка
                ctx.fillStyle = '#000';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                // Платформа
                ctx.fillStyle = '#00ff00';
                ctx.fillRect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
                
                // Мяч
                ctx.fillStyle = '#ff0000';
                ctx.beginPath();
                ctx.arc(ballX, ballY, ballSize, 0, Math.PI * 2);
                ctx.fill();
            }

            function update() {
                if (!gameRunning) return;
                
                // Движение мяча
                ballX += ballSpeedX;
                ballY += ballSpeedY;
                
                // Отскок от стен
                if (ballX <= ballSize || ballX >= canvas.width - ballSize) {
                    ballSpeedX = -ballSpeedX;
                }
                
                if (ballY <= ballSize) {
                    ballSpeedY = -ballSpeedY;
                }
                
                // Столкновение с платформой
                if (
                    ballY >= canvas.height - paddleHeight - ballSize &&
                    ballX >= paddleX &&
                    ballX <= paddleX + paddleWidth
                ) {
                    ballSpeedY = -ballSpeedY;
                    score += 10;
                    updateScore();
                }
                
                // Проигрыш
                if (ballY >= canvas.height) {
                    gameRunning = false;
                    alert(`Игра окончена! Счёт: ${score}`);
                    resetGame();
                }
                
                draw();
                requestAnimationFrame(update);
            }

            function resetGame() {
                paddleX = (canvas.width - paddleWidth) / 2;
                ballX = canvas.width / 2;
                ballY = canvas.height / 2;
                ballSpeedX = 5 * (Math.random() > 0.5 ? 1 : -1);
                ballSpeedY = -5;
                score = 0;
                gameRunning = true;
                updateScore();
                update();
            }

            // Управление
            document.addEventListener('mousemove', e => {
                const rect = canvas.getBoundingClientRect();
                paddleX = e.clientX - rect.left - paddleWidth / 2;
                paddleX = Math.max(0, Math.min(paddleX, canvas.width - paddleWidth));
            });

            resetGame();
        }

        // ===== FLAPPY BIRD =====
        function initFlappy() {
            const birdSize = 20;
            const gap = 150;
            const pipeWidth = 60;
            
            let birdY = canvas.height / 2;
            let velocity = 0;
            let gravity = 0.5;
            let pipes = [];
            let frameCount = 0;
            let gameRunning = true;

            function draw() {
                // Очистка
                ctx.fillStyle = '#87CEEB';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                // Птица
                ctx.fillStyle = '#FFD700';
                ctx.beginPath();
                ctx.arc(canvas.width / 4, birdY, birdSize, 0, Math.PI * 2);
                ctx.fill();
                
                // Трубы
                ctx.fillStyle = '#228B22';
                pipes.forEach(pipe => {
                    ctx.fillRect(pipe.x, 0, pipeWidth, pipe.top);
                    ctx.fillRect(pipe.x, pipe.bottom, pipeWidth, canvas.height - pipe.bottom);
                });
            }

            function update() {
                if (!gameRunning) return;
                
                // Гравитация
                velocity += gravity;
                birdY += velocity;
                
                // Генерация труб
                frameCount++;
                if (frameCount % 100 === 0) {
                    const topHeight = Math.floor(Math.random() * (canvas.height - gap - 100)) + 50;
                    pipes.push({
                        x: canvas.width,
                        top: topHeight,
                        bottom: topHeight + gap,
                        passed: false
                    });
                }
                
                // Движение труб
                pipes.forEach(pipe => {
                    pipe.x -= 2;
                    
                    // Проверка прохождения трубы
                    if (pipe.x + pipeWidth < canvas.width / 4 && !pipe.passed) {
                        pipe.passed = true;
                        score += 1;
                        updateScore();
                    }
                });
                
                // Удаление труб за экраном
                pipes = pipes.filter(pipe => pipe.x + pipeWidth > 0);
                
                // Проверка столкновений
                if (
                    birdY - birdSize < 0 || 
                    birdY + birdSize > canvas.height ||
                    pipes.some(pipe => 
                        canvas.width / 4 + birdSize > pipe.x && 
                        canvas.width / 4 - birdSize < pipe.x + pipeWidth && 
                        (birdY - birdSize < pipe.top || birdY + birdSize > pipe.bottom)
                    )
                ) {
                    gameRunning = false;
                    alert(`Игра окончена! Счёт: ${score}`);
                    resetGame();
                }
                
                draw();
                requestAnimationFrame(update);
            }

            function resetGame() {
                birdY = canvas.height / 2;
                velocity = 0;
                pipes = [];
                frameCount = 0;
                score = 0;
                gameRunning = true;
                updateScore();
                update();
            }

            // Управление
            document.addEventListener('click', () => {
                if (gameRunning) {
                    velocity = -10;
                } else {
                    resetGame();
                }
            });

            resetGame();
        }

        // ===== АРКАНОИД =====
        function initBreakout() {
            const brickRowCount = 5;
            const brickColumnCount = 10;
            const brickWidth = 75;
            const brickHeight = 20;
            const brickPadding = 10;
            const brickOffsetTop = 60;
            const brickOffsetLeft = 30;
            const paddleHeight = 15;
            const paddleWidth = 100;
            const ballRadius = 10;
            
            let paddleX = (canvas.width - paddleWidth) / 2;
            let ballX = canvas.width / 2;
            let ballY = canvas.height / 2;
            let ballSpeedX = 4;
            let ballSpeedY = -4;
            let gameRunning = true;
            
            // Создание кирпичей
            const bricks = [];
            for (let c = 0; c < brickColumnCount; c++) {
                bricks[c] = [];
                for (let r = 0; r < brickRowCount; r++) {
                    bricks[c][r] = { x: 0, y: 0, status: 1 };
                }
            }

            function draw() {
                // Очистка
                ctx.fillStyle = '#000';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                // Кирпичи
                for (let c = 0; c < brickColumnCount; c++) {
                    for (let r = 0; r < brickRowCount; r++) {
                        if (bricks[c][r].status === 1) {
                            const brickX = c * (brickWidth + brickPadding) + brickOffsetLeft;
                            const brickY = r * (brickHeight + brickPadding) + brickOffsetTop;
                            bricks[c][r].x = brickX;
                            bricks[c][r].y = brickY;
                            
                            ctx.fillStyle = '#0095DD';
                            ctx.fillRect(brickX, brickY, brickWidth, brickHeight);
                        }
                    }
                }
                
                // Платформа
                ctx.fillStyle = '#00ff00';
                ctx.fillRect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
                
                // Мяч
                ctx.fillStyle = '#ff0000';
                ctx.beginPath();
                ctx.arc(ballX, ballY, ballRadius, 0, Math.PI * 2);
                ctx.fill();
            }

            function update() {
                if (!gameRunning) return;
                
                // Движение мяча
                ballX += ballSpeedX;
                ballY += ballSpeedY;
                
                // Отскок от стен
                if (ballX <= ballRadius || ballX >= canvas.width - ballRadius) {
                    ballSpeedX = -ballSpeedX;
                }
                
                if (ballY <= ballRadius) {
                    ballSpeedY = -ballSpeedY;
                }
                
                // Столкновение с платформой
                if (
                    ballY >= canvas.height - paddleHeight - ballRadius &&
                    ballX >= paddleX &&
                    ballX <= paddleX + paddleWidth
                ) {
                    ballSpeedY = -ballSpeedY;
                }
                
                // Проигрыш
                if (ballY >= canvas.height) {
                    gameRunning = false;
                    alert(`Игра окончена! Счёт: ${score}`);
                    resetGame();
                }
                
                // Проверка столкновений с кирпичами
                for (let c = 0; c < brickColumnCount; c++) {
                    for (let r = 0; r < brickRowCount; r++) {
                        const brick = bricks[c][r];
                        if (brick.status === 1) {
                            if (
                                ballX > brick.x &&
                                ballX < brick.x + brickWidth &&
                                ballY > brick.y &&
                                ballY < brick.y + brickHeight
                            ) {
                                ballSpeedY = -ballSpeedY;
                                brick.status = 0;
                                score += 10;
                                updateScore();
                            }
                        }
                    }
                }
                
                draw();
                requestAnimationFrame(update);
            }

            function resetGame() {
                // Сброс кирпичей
                for (let c = 0; c < brickColumnCount; c++) {
                    for (let r = 0; r < brickRowCount; r++) {
                        bricks[c][r].status = 1;
                    }
                }
                
                paddleX = (canvas.width - paddleWidth) / 2;
                ballX = canvas.width / 2;
                ballY = canvas.height / 2;
                ballSpeedX = 4 * (Math.random() > 0.5 ? 1 : -1);
                ballSpeedY = -4;
                score = 0;
                gameRunning = true;
                updateScore();
                update();
            }

            // Управление
            document.addEventListener('mousemove', e => {
                const rect = canvas.getBoundingClientRect();
                paddleX = e.clientX - rect.left - paddleWidth / 2;
                paddleX = Math.max(0, Math.min(paddleX, canvas.width - paddleWidth));
            });

            resetGame();
        }

        // Общие функции
        function updateScore() {
            scoreElement.textContent = `Счёт: ${score}`;
        }

        function startGame(game) {
            // Остановка текущей игры
            if (currentGame && typeof currentGame === 'function') {
                document.removeEventListener('keydown', currentGame.keyboardHandler);
                document.removeEventListener('mousemove', currentGame.mouseHandler);
                document.removeEventListener('click', currentGame.clickHandler);
            }
            
            // Сброс счёта
            score = 0;
            updateScore();
            
            // Запуск новой игры
            switch (game) {
                case 'snake':
                    currentGame = { keyboardHandler: null };
                    initSnake();
                    break;
                case 'tetris':
                    currentGame = { keyboardHandler: null };
                    initTetris();
                    break;
                case 'pong':
                    currentGame = { mouseHandler: null };
                    initPong();
                    break;
                case 'flappy':
                    currentGame = { clickHandler: null };
                    initFlappy();
                    break;
                case 'breakout':
                    currentGame = { mouseHandler: null };
                    initBreakout();
                    break;
            }
        }

        // Запускаем первую игру
        startGame('snake');
    </script>
</body>
</html>
<!DOCTYPE html>
<html>
<head>
    <title>5-in-1 Retro Games | Mobile Controls</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            font-family: 'Courier New', monospace;
            background: #0f0f23;
            color: #00ff00;
            text-align: center;
            margin: 0;
            padding: 10px;
            touch-action: manipulation;
            overflow-x: hidden;
        }
        header {
            margin-bottom: 15px;
            text-shadow: 0 0 5px #00ff00;
        }
        .game-buttons {
            display: flex;
            justify-content: center;
            gap: 8px;
            flex-wrap: wrap;
            margin-bottom: 15px;
        }
        button {
            background: #003300;
            color: #00ff00;
            border: 1px solid #00ff00;
            padding: 8px 12px;
            cursor: pointer;
            font-family: inherit;
            font-size: 14px;
            border-radius: 4px;
        }
        button:hover {
            background: #005500;
        }
        .game-container {
            margin: 0 auto;
            position: relative;
            max-width: 100%;
        }
        canvas {
            display: block;
            margin: 0 auto;
            background: #000;
            border: 2px solid #00ff00;
            max-width: 100%;
            height: auto;
        }
        .score {
            margin: 10px 0;
            font-size: 18px;
        }
        /* Мобильное управление */
        .controls {
            display: none;
            margin-top: 15px;
        }
        .control-btn {
            background: rgba(0, 51, 0, 0.7);
            color: #00ff00;
            border: 1px solid #00ff00;
            padding: 12px;
            font-size: 20px;
            border-radius: 6px;
            user-select: none;
        }
        .control-row {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-bottom: 10px;
        }
        @media (max-width: 768px) {
            .controls {
                display: block;
            }
            button {
                padding: 10px 15px;
                font-size: 16px;
            }
        }
    </style>
</head>
<body>
    <header>
        <h1>🎮 5 Retro Games</h1>
    </header>

    <div class="game-buttons">
        <button onclick="startGame('snake')">Змейка</button>
        <button onclick="startGame('tetris')">Тетрис</button>
        <button onclick="startGame('pong')">Понг</button>
        <button onclick="startGame('flappy')">Flappy</button>
        <button onclick="startGame('breakout')">Арканоид</button>
    </div>

    <div class="score" id="score">Счёт: 0</div>

    <div class="game-container">
        <canvas id="gameCanvas"></canvas>
    </div>

    <!-- Мобильное управление -->
    <div class="controls" id="controls">
        <div class="control-row">
            <div class="control-btn" id="up">↑</div>
        </div>
        <div class="control-row">
            <div class="control-btn" id="left">←</div>
            <div class="control-btn" id="down">↓</div>
            <div class="control-btn" id="right">→</div>
        </div>
        <div class="control-row">
            <div class="control-btn" id="action">A</div>
            <div class="control-btn" id="rotate">B</div>
        </div>
    </div>

    <script>
        // Общие переменные
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const controls = document.getElementById('controls');
        let currentGame = null;
        let score = 0;
        let isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);

        // Размеры холста
        function resizeCanvas() {
            const size = Math.min(window.innerWidth - 40, 500);
            canvas.width = size;
            canvas.height = size;
        }
        resizeCanvas();
        window.addEventListener('resize', resizeCanvas);

        // Мобильное управление
        const controlButtons = {
            up: document.getElementById('up'),
            down: document.getElementById('down'),
            left: document.getElementById('left'),
            right: document.getElementById('right'),
            action: document.getElementById('action'),
            rotate: document.getElementById('rotate')
        };

        // Показываем управление только на мобильных
        if (isMobile) {
            controls.style.display = 'block';
        }

        // ===== ЗМЕЙКА =====
        function initSnake() {
            const gridSize = 20;
            let snake = [{x: 10, y: 10}];
            let food = {x: 5, y: 5};
            let dx = 0, dy = 0;
            let speed = 150;

            // Мобильное управление
            const setupControls = () => {
                controlButtons.up.onpointerdown = () => { if (dy === 0) { dx = 0; dy = -1; } };
                controlButtons.down.onpointerdown = () => { if (dy === 0) { dx = 0; dy = 1; } };
                controlButtons.left.onpointerdown = () => { if (dx === 0) { dx = -1; dy = 0; } };
                controlButtons.right.onpointerdown = () => { if (dx === 0) { dx = 1; dy = 0; } };
                
                // Отключение других кнопок
                controlButtons.action.onpointerdown = null;
                controlButtons.rotate.onpointerdown = null;
            };

            // Клавиатура
            const keyboardHandler = (e) => {
                if (e.key === 'ArrowUp' && dy === 0) { dx = 0; dy = -1; }
                if (e.key === 'ArrowDown' && dy === 0) { dx = 0; dy = 1; }
                if (e.key === 'ArrowLeft' && dx === 0) { dx = -1; dy = 0; }
                if (e.key === 'ArrowRight' && dx === 0) { dx = 1; dy = 0; }
            };

            setupControls();
            document.addEventListener('keydown', keyboardHandler);

            // Остальной код змейки (как в предыдущем примере)
            function gameLoop() {
                const head = {x: snake[0].x + dx, y: snake[0].y + dy};
                snake.unshift(head);
                
                if (head.x === food.x && head.y === food.y) {
                    generateFood();
                    score += 10;
                    speed = Math.max(50, speed - 5);
                } else {
                    snake.pop();
                }

                if (isCollision()) {
                    gameOver();
                    return;
                }

                drawGame();
                setTimeout(gameLoop, speed);
            }

            function drawGame() {
                ctx.fillStyle = '#000';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                ctx.fillStyle = '#00ff00';
                snake.forEach(segment => {
                    ctx.fillRect(
                        segment.x * gridSize, 
                        segment.y * gridSize, 
                        gridSize - 1, 
                        gridSize - 1
                    );
                });
                
                ctx.fillStyle = '#ff0000';
                ctx.fillRect(
                    food.x * gridSize, 
                    food.y * gridSize, 
                    gridSize - 1, 
                    gridSize - 1
                );
                
                updateScore();
            }

            function generateFood() {
                food = {
                    x: Math.floor(Math.random() * (canvas.width / gridSize)),
                    y: Math.floor(Math.random() * (canvas.height / gridSize))
                };
            }

            function isCollision() {
                const head = snake[0];
                if (
                    head.x < 0 || 
                    head.y < 0 || 
                    head.x >= canvas.width / gridSize || 
                    head.y >= canvas.height / gridSize
                ) return true;
                
                for (let i = 1; i < snake.length; i++) {
                    if (head.x === snake[i].x && head.y === snake[i].y) {
                        return true;
                    }
                }
                
                return false;
            }

            function gameOver() {
                alert(`Игра окончена! Счёт: ${score}`);
                resetGame();
            }

            function resetGame() {
                snake = [{x: 10, y: 10}];
                dx = dy = 0;
                score = 0;
                speed = 150;
                generateFood();
                setupControls();
                gameLoop();
            }

            generateFood();
            gameLoop();

            // Возвращаем обработчик для удаления при смене игры
            return { keyboardHandler };
        }

        // ===== ТЕТРИС =====
        function initTetris() {
            // ... (код тетриса из предыдущего примера)
            // Добавляем мобильное управление:
            const setupControls = () => {
                controlButtons.left.onpointerdown = () => movePiece(-1, 0);
                controlButtons.right.onpointerdown = () => movePiece(1, 0);
                controlButtons.down.onpointerdown = () => movePiece(0, 1);
                controlButtons.rotate.onpointerdown = () => rotatePiece();
                
                // Отключение ненужных кнопок
                controlButtons.up.onpointerdown = null;
                controlButtons.action.onpointerdown = null;
            };

            setupControls();
            
            // Возвращаем обработчики для удаления
            return { 
                keyboardHandler: (e) => {
                    if (e.key === 'ArrowLeft') movePiece(-1, 0);
                    if (e.key === 'ArrowRight') movePiece(1, 0);
                    if (e.key === 'ArrowDown') movePiece(0, 1);
                    if (e.key === 'ArrowUp') rotatePiece();
                },
                touchHandler: setupControls
            };
        }

        // ===== ПОНГ =====
        function initPong() {
            // ... (код понга из предыдущего примера)
            // Управление для мобильных:
            const touchHandler = (e) => {
                const rect = canvas.getBoundingClientRect();
                paddleX = e.touches[0].clientX - rect.left - paddleWidth / 2;
                paddleX = Math.max(0, Math.min(paddleX, canvas.width - paddleWidth));
            };

            if (isMobile) {
                canvas.ontouchmove = touchHandler;
            }

            // Возвращаем обработчики для удаления
            return { 
                mouseHandler: (e) => {
                    const rect = canvas.getBoundingClientRect();
                    paddleX = e.clientX - rect.left - paddleWidth / 2;
                    paddleX = Math.max(0, Math.min(paddleX, canvas.width - paddleWidth));
                },
                touchHandler
            };
        }

        // ===== FLAPPY BIRD =====
        function initFlappy() {
            // ... (код flappy bird из предыдущего примера)
            // Управление для мобильных:
            const tapHandler = () => {
                if (gameRunning) {
                    velocity = -10;
                } else {
                    resetGame();
                }
            };

            if (isMobile) {
                canvas.ontouchstart = tapHandler;
                controlButtons.action.onpointerdown = tapHandler;
            }

            // Возвращаем обработчики для удаления
            return { 
                clickHandler: tapHandler,
                touchHandler: tapHandler
            };
        }

        // ===== АРКАНОИД =====
        function initBreakout() {
            // ... (код арканоида из предыдущего примера)
            // Управление для мобильных:
            const touchHandler = (e) => {
                const rect = canvas.getBoundingClientRect();
                paddleX = e.touches[0].clientX - rect.left - paddleWidth / 2;
                paddleX = Math.max(0, Math.min(paddleX, canvas.width - paddleWidth));
            };

            if (isMobile) {
                canvas.ontouchmove = touchHandler;
            }

            // Возвращаем обработчики для удаления
            return { 
                mouseHandler: (e) => {
                    const rect = canvas.getBoundingClientRect();
                    paddleX = e.clientX - rect.left - paddleWidth / 2;
                    paddleX = Math.max(0, Math.min(paddleX, canvas.width - paddleWidth));
                },
                touchHandler
            };
        }

        // Общие функции
        function updateScore() {
            scoreElement.textContent = `Счёт: ${score}`;
        }

        function startGame(game) {
            // Остановка текущей игры
            if (currentGame) {
                ['keyboardHandler', 'mouseHandler', 'clickHandler', 'touchHandler'].forEach(handler => {
                    if (currentGame[handler]) {
                        if (handler === 'keyboardHandler') {
                            document.removeEventListener('keydown', currentGame[handler]);
                        } else if (handler === 'mouseHandler') {
                            document.removeEventListener('mousemove', currentGame[handler]);
                        } else if (handler === 'clickHandler') {
                            document.removeEventListener('click', currentGame[handler]);
                        } else if (handler === 'touchHandler') {
                            canvas.ontouchmove = null;
                            canvas.ontouchstart = null;
                        }
                    }
                });
            }
            
            // Сброс счёта
            score = 0;
            updateScore();
            
            // Настройка кнопок управления
            controlButtons.up.onpointerdown = null;
            controlButtons.down.onpointerdown = null;
            controlButtons.left.onpointerdown = null;
            controlButtons.right.onpointerdown = null;
            controlButtons.action.onpointerdown = null;
            controlButtons.rotate.onpointerdown = null;
            
            // Запуск новой игры
            switch (game) {
                case 'snake':
                    currentGame = initSnake();
                    break;
                case 'tetris':
                    currentGame = initTetris();
                    break;
                case 'pong':
                    currentGame = initPong();
                    break;
                case 'flappy':
                    currentGame = initFlappy();
                    break;
                case 'breakout':
                    currentGame = initBreakout();
                    break;
            }
        }

        // Запускаем первую игру
        startGame('snake');
    </script>
</body>
</html>
