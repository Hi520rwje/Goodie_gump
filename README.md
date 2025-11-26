# Goodie Jump

By Ayden Neely

HTML Code:
```
<!DOCTYPE html>
<html>
<head>
    <title>Doodle Jump</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(to bottom, #87CEEB, #E0F7FA);
            font-family: 'Arial', sans-serif;
            overflow: hidden;
        }
        
        #gameContainer {
            position: relative;
            width: 400px;
            height: 600px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
            border-radius: 10px;
            overflow: hidden;
        }
        
        #gameCanvas {
            background: linear-gradient(to bottom, #87CEEB, #E0F7FA);
        }
        
        #score {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 24px;
            font-weight: bold;
            color: #333;
            z-index: 10;
        }
        
        #gameOver {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.7);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 20;
        }
        
        #restartBtn {
            margin-top: 20px;
            padding: 10px 20px;
            font-size: 18px;
            background: #4CAF50;
            border: none;
            border-radius: 5px;
            color: white;
            cursor: pointer;
        }
        
        #startScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.7);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            z-index: 20;
        }
        
        #startBtn {
            margin-top: 20px;
            padding: 10px 20px;
            font-size: 18px;
            background: #4CAF50;
            border: none;
            border-radius: 5px;
            color: white;
            cursor: pointer;
        }
        
        #instructions {
            text-align: center;
            max-width: 80%;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="400" height="600"></canvas>
        <div id="score">Score: 0</div>
        <div id="gameOver">
            <h2>Game Over!</h2>
            <p id="finalScore">Score: 0</p>
            <button id="restartBtn">Play Again</button>
        </div>
        <div id="startScreen">
            <h2>Doodle Jump</h2>
            <div id="instructions">
                <p>Use LEFT and RIGHT arrow keys to move</p>
                <p>Avoid falling off the platforms</p>
                <p>Jump on platforms to go higher</p>
            </div>
            <button id="startBtn">Start Game</button>
        </div>
    </div>

    <script>
        // Game variables
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const gameOverScreen = document.getElementById('gameOver');
        const finalScoreElement = document.getElementById('finalScore');
        const restartBtn = document.getElementById('restartBtn');
        const startScreen = document.getElementById('startScreen');
        const startBtn = document.getElementById('startBtn');
        
        // Game state
        let gameRunning = false;
        let score = 0;
        let cameraY = 0;
        
        // Player properties
        const player = {
            x: canvas.width / 2 - 25,
            y: canvas.height - 100,
            width: 50,
            height: 50,
            velocityX: 0,
            velocityY: 0,
            speed: 5,
            jumpForce: 15,
            isJumping: false
        };
        
        // Platform class
        class Platform {
            constructor(x, y, width, height, color) {
                this.x = x;
                this.y = y;
                this.width = width;
                this.height = height;
                this.color = color;
            }
            
            draw() {
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, this.width, this.height);
                
                // Add platform details
                ctx.fillStyle = '#8B4513';
                ctx.fillRect(this.x, this.y, this.width, 5);
            }
        }
        
        // Create initial platforms
        let platforms = [];
        function createPlatforms() {
            platforms = [];
            
            // Ground platform
            platforms.push(new Platform(0, canvas.height - 20, canvas.width, 20, '#8B4513'));
            
            // Generate fewer platforms (10 instead of 20)
            for (let i = 0; i < 10; i++) {
                const x = Math.random() * (canvas.width - 100);
                const y = canvas.height - 100 - (i * 60);
                const width = 100;
                const height = 20;
                const color = i % 3 === 0 ? '#228B22' : '#32CD32';
                platforms.push(new Platform(x, y, width, height, color));
            }
        }
        
        // Draw player
        function drawPlayer() {
            // Draw doodle character
            ctx.fillStyle = '#FF6B6B';
            ctx.beginPath();
            ctx.arc(player.x + player.width/2, player.y + player.height/2, player.width/2, 0, Math.PI * 2);
            ctx.fill();
            
            // Draw eyes
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.arc(player.x + player.width/3, player.y + player.height/3, 5, 0, Math.PI * 2);
            ctx.arc(player.x + 2*player.width/3, player.y + player.height/3, 5, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(player.x + player.width/3, player.y + player.height/3, 2, 0, Math.PI * 2);
            ctx.arc(player.x + 2*player.width/3, player.y + player.height/3, 2, 0, Math.PI * 2);
            ctx.fill();
            
            // Draw mouth
            ctx.beginPath();
            ctx.arc(player.x + player.width/2, player.y + 2*player.height/3, 8, 0, Math.PI);
            ctx.stroke();
        }
        
        // Update game state
        function update() {
            if (!gameRunning) return;
            
            // Apply gravity
            player.velocityY += 0.8;
            
            // Update player position
            player.x += player.velocityX;
            player.y += player.velocityY;
            
            // Boundary checks
            if (player.x < 0) player.x = 0;
            if (player.x + player.width > canvas.width) player.x = canvas.width - player.width;
            
            // Check if player falls off the screen
            if (player.y > canvas.height) {
                gameOver();
                return;
            }
            
            // Platform collision
            player.isJumping = true;
            for (let i = 0; i < platforms.length; i++) {
                const p = platforms[i];
                
                // Check if player is above platform and falling
                if (player.velocityY > 0 &&
                    player.y + player.height <= p.y + 10 &&
                    player.y + player.height + player.velocityY >= p.y &&
                    player.x + player.width > p.x &&
                    player.x < p.x + p.width) {
                    
                    player.velocityY = -player.jumpForce;
                    player.isJumping = false;
                    
                    // Increase score
                    score += 5;
                    scoreElement.textContent = `Score: ${score}`;
                    
                    // Move camera up when player jumps on platforms
                    if (player.y < canvas.height / 2) {
                        cameraY += canvas.height / 2 - player.y;
                        player.y = canvas.height / 2;
                    }
                    
                    break;
                }
            }
            
            // Move platforms down when player jumps up
            if (player.y < canvas.height / 2) {
                const diff = canvas.height / 2 - player.y;
                for (let i = 0; i < platforms.length; i++) {
                    platforms[i].y += diff;
                }
                player.y = canvas.height / 2;
            }
            
            // Generate new platforms at the top
            if (platforms[platforms.length - 1].y > 0) {
                const x = Math.random() * (canvas.width - 100);
                const y = platforms[platforms.length - 1].y - 60;
                const width = 100;
                const height = 20;
                const color = platforms.length % 3 === 0 ? '#228B22' : '#32CD32';
                platforms.push(new Platform(x, y, width, height, color));
            }
            
            // Remove platforms that are off screen
            platforms = platforms.filter(p => p.y < canvas.height);
        }
        
        // Draw everything
        function draw() {
            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // Draw background
            ctx.fillStyle = '#87CEEB';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Draw clouds
            ctx.fillStyle = 'rgba(255, 255, 255, 0.8)';
            ctx.beginPath();
            ctx.arc(100, 80, 30, 0, Math.PI * 2);
            ctx.arc(130, 70, 35, 0, Math.PI * 2);
            ctx.arc(160, 80, 30, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.beginPath();
            ctx.arc(300, 120, 30, 0, Math.PI * 2);
            ctx.arc(330, 110, 35, 0, Math.PI * 2);
            ctx.arc(360, 120, 30, 0, Math.PI * 2);
            ctx.fill();
            
            // Draw platforms
            for (let i = 0; i < platforms.length; i++) {
                platforms[i].draw();
            }
            
            // Draw player
            drawPlayer();
        }
        
        // Game loop
        function gameLoop() {
            update();
            draw();
            if (gameRunning) {
                requestAnimationFrame(gameLoop);
            }
        }
        
        // Start game
        function startGame() {
            gameRunning = true;
            score = 0;
            cameraY = 0;
            player.x = canvas.width / 2 - 25;
            player.y = canvas.height - 100;
            player.velocityX = 0;
            player.velocityY = 0;
            scoreElement.textContent = `Score: ${score}`;
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            createPlatforms();
            gameLoop();
        }
        
        // Game over
        function gameOver() {
            gameRunning = false;
            finalScoreElement.textContent = `Score: ${score}`;
            gameOverScreen.style.display = 'flex';
        }
        
        // Event listeners
        document.addEventListener('keydown', (e) => {
            if (!gameRunning) return;
            
            if (e.key === 'ArrowLeft') {
                player.velocityX = -player.speed;
            } else if (e.key === 'ArrowRight') {
                player.velocityX = player.speed;
            }
        });
        
        document.addEventListener('keyup', (e) => {
            if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
                player.velocityX = 0;
            }
        });
        
        restartBtn.addEventListener('click', startGame);
        startBtn.addEventListener('click', startGame);
        
        // Initialize
        createPlatforms();
    </script>
</body>
</html>
```
