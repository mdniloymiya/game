# game
snake game description
https://mdniloymiya.github.io/game/
<!DOCTYPE html>
<html>
<head>
    <title>Snake Game with Controls</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        h1 {
            color: #333;
            margin-bottom: 10px;
        }
        canvas {
            background: #000;
            border: 3px solid #333;
            margin-bottom: 15px;
        }
        .score-container {
            font-size: 20px;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .controls-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-bottom: 15px;
        }
        .arrow-row {
            display: flex;
            justify-content: center;
        }
        .control-btn {
            width: 60px;
            height: 60px;
            margin: 5px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 24px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            user-select: none;
        }
        .control-btn:hover {
            background-color: #45a049;
        }
        .control-btn:active {
            background-color: #3e8e41;
            transform: scale(0.95);
        }
        .start-btn {
            padding: 10px 25px;
            background-color: #2196F3;
            color: white;
            border: none;
            border-radius: 5px;
            font-size: 18px;
            cursor: pointer;
            margin-top: 10px;
        }
        .start-btn:hover {
            background-color: #0b7dda;
        }
        .game-over {
            color: red;
            font-size: 24px;
            font-weight: bold;
            margin-top: 10px;
            display: none;
        }
        .instructions {
            margin-top: 15px;
            color: #555;
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Snake Game</h1>
    <div class="score-container">Score: <span id="score">0</span></div>
    <canvas id="gameCanvas" width="400" height="400"></canvas>
    
    <div class="controls-container">
        <div class="arrow-row">
            <button class="control-btn" id="upBtn">↑</button>
        </div>
        <div class="arrow-row">
            <button class="control-btn" id="leftBtn">←</button>
            <button class="control-btn" id="downBtn">↓</button>
            <button class="control-btn" id="rightBtn">→</button>
        </div>
    </div>
    
    <button class="start-btn" id="startBtn">Start Game</button>
    <div class="game-over" id="gameOver">Game Over!</div>
    <div class="instructions">
        <p>Use arrow buttons below or keyboard arrow keys to control the snake</p>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreDisplay = document.getElementById('score');
        const startBtn = document.getElementById('startBtn');
        const gameOverDisplay = document.getElementById('gameOver');
        const upBtn = document.getElementById('upBtn');
        const downBtn = document.getElementById('downBtn');
        const leftBtn = document.getElementById('leftBtn');
        const rightBtn = document.getElementById('rightBtn');

        const box = 20; // Size of each grid box
        let snake = [];
        let direction = null;
        let nextDirection = null;
        let food = {};
        let score = 0;
        let game = null;
        let gameRunning = false;

        // Initialize snake
        function initSnake() {
            snake = [
                {x: 9 * box, y: 10 * box},
                {x: 8 * box, y: 10 * box},
                {x: 7 * box, y: 10 * box}
            ];
            direction = "RIGHT";
            nextDirection = "RIGHT";
        }

        // Create food at random position
        function createFood() {
            food = {
                x: Math.floor(Math.random() * 20) * box,
                y: Math.floor(Math.random() * 20) * box
            };
            
            // Make sure food doesn't appear on snake
            for (let i = 0; i < snake.length; i++) {
                if (food.x === snake[i].x && food.y === snake[i].y) {
                    createFood();
                    return;
                }
            }
        }

        // Draw everything on canvas
        function draw() {
            // Clear canvas
            ctx.fillStyle = "black";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw snake
            for (let i = 0; i < snake.length; i++) {
                ctx.fillStyle = (i === 0) ? "lime" : "green";
                ctx.fillRect(snake[i].x, snake[i].y, box, box);
                ctx.strokeStyle = "black";
                ctx.strokeRect(snake[i].x, snake[i].y, box, box);
            }

            // Draw food
            ctx.fillStyle = "red";
            ctx.fillRect(food.x, food.y, box, box);
        }

        // Game over function
        function gameOver() {
            gameRunning = false;
            clearInterval(game);
            gameOverDisplay.style.display = "block";
        }

        // Main game function
        function playGame() {
            if (!gameRunning) return;
            
            // Update direction (to prevent instant reverse)
            if (nextDirection === "LEFT" && direction !== "RIGHT") direction = "LEFT";
            if (nextDirection === "UP" && direction !== "DOWN") direction = "UP";
            if (nextDirection === "RIGHT" && direction !== "LEFT") direction = "RIGHT";
            if (nextDirection === "DOWN" && direction !== "UP") direction = "DOWN";

            // Move snake
            let snakeX = snake[0].x;
            let snakeY = snake[0].y;

            if (direction === "LEFT") snakeX -= box;
            if (direction === "UP") snakeY -= box;
            if (direction === "RIGHT") snakeX += box;
            if (direction === "DOWN") snakeY += box;

            // Check if snake hit wall
            if (snakeX < 0 || snakeX >= canvas.width || snakeY < 0 || snakeY >= canvas.height) {
                gameOver();
                return;
            }

            // Check if snake hit itself
            for (let i = 1; i < snake.length; i++) {
                if (snakeX === snake[i].x && snakeY === snake[i].y) {
                    gameOver();
                    return;
                }
            }

            // Check if snake ate food
            if (snakeX === food.x && snakeY === food.y) {
                score++;
                scoreDisplay.textContent = score;
                createFood();
            } else {
                // Remove tail if no food eaten
                snake.pop();
            }

            // Add new head
            const newHead = {x: snakeX, y: snakeY};
            snake.unshift(newHead);

            draw();
        }

        // Keyboard controls
        document.addEventListener('keydown', function(event) {
            if (!gameRunning) return;
            
            if (event.keyCode === 37) nextDirection = "LEFT";
            else if (event.keyCode === 38) nextDirection = "UP";
            else if (event.keyCode === 39) nextDirection = "RIGHT";
            else if (event.keyCode === 40) nextDirection = "DOWN";
        });

        // Button controls
        upBtn.addEventListener('click', function() {
            if (gameRunning) nextDirection = "UP";
        });
        downBtn.addEventListener('click', function() {
            if (gameRunning) nextDirection = "DOWN";
        });
        leftBtn.addEventListener('click', function() {
            if (gameRunning) nextDirection = "LEFT";
        });
        rightBtn.addEventListener('click', function() {
            if (gameRunning) nextDirection = "RIGHT";
        });

        // Start game
        startBtn.addEventListener('click', function() {
            initSnake();
            createFood();
            score = 0;
            scoreDisplay.textContent = score;
            gameOverDisplay.style.display = "none";
            gameRunning = true;
            
            if (game) clearInterval(game);
            game = setInterval(playGame, 100);
        });

        // Initial draw
        initSnake();
        createFood();
        draw();
    </script>
</body>
</html>
