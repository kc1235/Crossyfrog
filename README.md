<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Frogger-style Game</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            margin: 0;
            padding: 0;
            height: 100vh;
        }
        #gameContainer {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        #gameCanvas {
            border: 2px solid #333;
            border-radius: 10px;
        }
        #score {
            font-size: 24px;
            margin-top: 10px;
        }
        #controls {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-top: 20px;
        }
        .control-btn {
            width: 60px;
            height: 60px;
            font-size: 24px;
            border: none;
            border-radius: 50%;
            background-color: #4CAF50;
            color: white;
            cursor: pointer;
        }
        .control-btn:active {
            background-color: #45a049;
        }
        @media (max-height: 600px) {
            .control-btn {
                width: 40px;
                height: 40px;
                font-size: 18px;
            }
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas"></canvas>
        <div id="score">Score: 0</div>
        <div id="controls">
            <button class="control-btn" id="leftBtn">←</button>
            <button class="control-btn" id="upBtn">↑</button>
            <button class="control-btn" id="rightBtn">→</button>
            <div></div>
            <button class="control-btn" id="downBtn">↓</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');

        let score = 0;
        let scale = 1;
        let wheelRotation = 0;

        const player = {
            x: 200,
            y: 380,
            width: 30,
            height: 30,
            color: '#0f0'
        };

        const cars = [
            {x: 0, y: 100, width: 60, height: 30, speed: 2, color: 'red'},
            {x: 200, y: 160, width: 60, height: 30, speed: -3, color: 'blue'},
            {x: 100, y: 220, width: 60, height: 30, speed: 1.5, color: 'yellow'},
            {x: 300, y: 280, width: 60, height: 30, speed: -2.5, color: 'purple'}
        ];

        function resizeCanvas() {
            const containerWidth = window.innerWidth * 0.9;
            const containerHeight = window.innerHeight * 0.7;
            const containerRatio = containerWidth / containerHeight;
            const gameRatio = 400 / 400;

            if (containerRatio > gameRatio) {
                canvas.height = containerHeight;
                canvas.width = containerHeight * gameRatio;
            } else {
                canvas.width = containerWidth;
                canvas.height = containerWidth / gameRatio;
            }

            scale = canvas.width / 400;
            ctx.scale(scale, scale);
        }

        function drawPlayer() {
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.arc(player.x + player.width / 2, player.y + player.height / 2, player.width / 2, 0, Math.PI * 2);
            ctx.fill();

            // Eyes
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.arc(player.x + player.width / 3, player.y + player.height / 3, 5, 0, Math.PI * 2);
            ctx.arc(player.x + 2 * player.width / 3, player.y + player.height / 3, 5, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(player.x + player.width / 3, player.y + player.height / 3, 2, 0, Math.PI * 2);
            ctx.arc(player.x + 2 * player.width / 3, player.y + player.height / 3, 2, 0, Math.PI * 2);
            ctx.fill();

            // Mouth
            ctx.beginPath();
            ctx.arc(player.x + player.width / 2, player.y + 2 * player.height / 3, 8, 0, Math.PI);
            ctx.stroke();

            // Hands
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.arc(player.x - 5, player.y + player.height / 2, 5, 0, Math.PI * 2);
            ctx.arc(player.x + player.width + 5, player.y + player.height / 2, 5, 0, Math.PI * 2);
            ctx.fill();

            // Feet
            ctx.beginPath();
            ctx.arc(player.x + 5, player.y + player.height + 5, 5, 0, Math.PI * 2);
            ctx.arc(player.x + player.width - 5, player.y + player.height + 5, 5, 0, Math.PI * 2);
            ctx.fill();
        }

        function drawWheel(x, y, rotation) {
            ctx.save();
            ctx.translate(x, y);
            ctx.rotate(rotation);
            ctx.beginPath();
            ctx.arc(0, 0, 7, 0, Math.PI * 2);
            ctx.moveTo(0, -7);
            ctx.lineTo(0, 7);
            ctx.moveTo(-7, 0);
            ctx.lineTo(7, 0);
            ctx.strokeStyle = 'white';
            ctx.stroke();
            ctx.restore();
        }

        function drawCar(car) {
            // Car body
            ctx.fillStyle = car.color;
            ctx.fillRect(car.x, car.y, car.width, car.height);
            
            // Car top
            ctx.fillStyle = car.color;
            ctx.fillRect(car.x + 10, car.y - 15, car.width - 20, 15);

            // Windows
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(car.x + 15, car.y - 12, 10, 12);
            ctx.fillRect(car.x + car.width - 25, car.y - 12, 10, 12);

            // Wheels
            ctx.fillStyle = 'black';
            drawWheel(car.x + 15, car.y + car.height, wheelRotation);
            drawWheel(car.x + car.width - 15, car.y + car.height, wheelRotation);
        }

        function drawCars() {
            cars.forEach(car => drawCar(car));
        }

        function drawGround() {
            // Start area
            ctx.fillStyle = '#90EE90';
            ctx.fillRect(0, 400 - 50, 400, 50);

            // End area
            ctx.fillStyle = '#90EE90';
            ctx.fillRect(0, 0, 400, 50);

            // Middle areas
            ctx.fillStyle = '#98FB98';
            ctx.fillRect(0, 50, 400, 50);
            ctx.fillRect(0, 170, 400, 50);
            ctx.fillRect(0, 290, 400, 50);
        }

        function drawRoads() {
            ctx.fillStyle = '#333';
            for (let i = 1; i < 5; i++) {
                ctx.fillRect(0, i * 60 + 40, 400, 50);
            }

            // Draw lane markings
            ctx.strokeStyle = 'white';
            ctx.setLineDash([20, 20]);
            for (let i = 1; i < 5; i++) {
                ctx.beginPath();
                ctx.moveTo(0, i * 60 + 65);
                ctx.lineTo(400, i * 60 + 65);
                ctx.stroke();
            }
            ctx.setLineDash([]);
        }

        function drawBarriers() {
            ctx.fillStyle = '#8B4513';
            ctx.fillRect(0, 45, 400, 5);
            ctx.fillRect(0, 165, 400, 5);
            ctx.fillRect(0, 285, 400, 5);
            ctx.fillRect(0, 350, 400, 5);
        }

        function moveCars() {
            cars.forEach(car => {
                car.x += car.speed;
                if (car.x > 400) {
                    car.x = -car.width;
                } else if (car.x < -car.width) {
                    car.x = 400;
                }
            });
            wheelRotation += 0.1;
        }

        function checkCollision() {
            return cars.some(car =>
                player.x < car.x + car.width &&
                player.x + player.width > car.x &&
                player.y < car.y + car.height &&
                player.y + player.height > car.y
            );
        }

        function gameLoop() {
            ctx.clearRect(0, 0, 400, 400);
            drawGround();
            drawRoads();
            drawBarriers();
            drawPlayer();
            drawCars();
            moveCars();

            if (checkCollision()) {
                alert('Game Over! Score: ' + score);
                player.y = 380;
                score = 0;
            }

            if (player.y <= 0) {
                score++;
                scoreElement.textContent = 'Score: ' + score;
                player.y = 380;
            }

            requestAnimationFrame(gameLoop);
        }

        function movePlayer(direction) {
            switch(direction) {
                case 'up':
                    if (player.y > 0) player.y -= 20;
                    break;
                case 'down':
                    if (player.y < 400 - player.height) player.y += 20;
                    break;
                case 'left':
                    if (player.x > 0) player.x -= 20;
                    break;
                case 'right':
                    if (player.x < 400 - player.width) player.x += 20;
                    break;
            }
        }

        document.addEventListener('keydown', (event) => {
            switch(event.key) {
                case 'ArrowUp': movePlayer('up'); break;
                case 'ArrowDown': movePlayer('down'); break;
                case 'ArrowLeft': movePlayer('left'); break;
                case 'ArrowRight': movePlayer('right'); break;
            }
        });

        // Touch controls
        document.getElementById('upBtn').addEventListener('click', () => movePlayer('up'));
        document.getElementById('downBtn').addEventListener('click', () => movePlayer('down'));
        document.getElementById('leftBtn').addEventListener('click', () => movePlayer('left'));
        document.getElementById('rightBtn').addEventListener('click', () => movePlayer('right'));

        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();
        gameLoop();
    </script>
</body>
</html>
