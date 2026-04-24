<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Samurai Slice</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #1a1a1a;
            overflow: hidden;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            touch-action: none; /* Prevent scrolling and pull-to-refresh on touch devices */
            user-select: none;
            -webkit-user-select: none;
        }
        #gameCanvas {
            display: block;
            width: 100%;
            height: 100%;
        }
        #uiLayer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        .screen {
            display: none;
            flex-direction: column;
            align-items: center;
            background: rgba(0, 0, 0, 0.85);
            padding: 50px;
            border-radius: 20px;
            pointer-events: auto;
            text-align: center;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            border: 2px solid #444;
        }
        .screen.active {
            display: flex;
            animation: fadeIn 0.3s ease-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; transform: scale(0.9); }
            to { opacity: 1; transform: scale(1); }
        }
        h1 {
            color: #fff;
            margin-top: 0;
            font-size: 3rem;
            text-shadow: 0 4px 10px rgba(0,0,0,0.5);
            letter-spacing: 2px;
        }
        button {
            background: linear-gradient(135deg, #e74c3c, #c0392b);
            color: white;
            border: none;
            padding: 15px 40px;
            font-size: 1.5rem;
            font-weight: bold;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 6px 15px rgba(231, 76, 60, 0.4);
            transition: transform 0.1s, box-shadow 0.2s;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        button:hover {
            transform: scale(1.05);
            box-shadow: 0 8px 20px rgba(231, 76, 60, 0.6);
        }
        button:active {
            transform: scale(0.95);
        }
        #scoreDisplay {
            position: absolute;
            top: 25px;
            left: 25px;
            color: white;
            font-size: 2.5rem;
            font-weight: bold;
            text-shadow: 0 4px 8px rgba(0,0,0,0.8);
            display: none;
            pointer-events: none;
        }
        .subtitle {
            color: #aaa;
            font-size: 1.2rem;
            margin-bottom: 30px;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    
    <div id="scoreDisplay">Score: <span id="scoreValue">0</span></div>

    <div id="uiLayer">
        <div id="startScreen" class="screen active">
            <h1>SAMURAI SLICE</h1>
            <p class="subtitle">Swipe to slice fruits. Avoid the bombs!</p>
            <button id="startBtn">Start Game</button>
        </div>
        
        <div id="gameOverScreen" class="screen">
            <h1>GAME OVER</h1>
            <p style="color: white; font-size: 1.8rem; margin: 10px 0 30px 0;">Final Score: <span id="finalScore" style="color: #f1c40f; font-weight: bold;">0</span></p>
            <button id="restartBtn">Play Again</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const scoreValue = document.getElementById('scoreValue');
        const finalScore = document.getElementById('finalScore');
        const startBtn = document.getElementById('startBtn');
        const restartBtn = document.getElementById('restartBtn');

        // Audio System
        let audioCtx;
        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
        }

        function playSound(type) {
            if (!audioCtx) return;
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            
            if (type === 'slice') {
                oscillator.type = 'triangle';
                oscillator.frequency.setValueAtTime(800, audioCtx.currentTime);
                oscillator.frequency.exponentialRampToValueAtTime(1500, audioCtx.currentTime + 0.1);
                gainNode.gain.setValueAtTime(0.3, audioCtx.currentTime);
                gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1);
                oscillator.start();
                oscillator.stop(audioCtx.currentTime + 0.1);
            } else if (type === 'bomb') {
                oscillator.type = 'sawtooth';
                oscillator.frequency.setValueAtTime(150, audioCtx.currentTime);
                oscillator.frequency.exponentialRampToValueAtTime(10, audioCtx.currentTime + 0.5);
                gainNode.gain.setValueAtTime(0.5, audioCtx.currentTime);
                gainNode.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5);
                oscillator.start();
                oscillator.stop(audioCtx.currentTime + 0.5);
            }
        }

        // Game Configuration & State
        let gameState = 'START'; // START, PLAYING, GAMEOVER
        let score = 0;
        let objects = []; 
        let particles = [];
        let trail = [];
        let isSwiping = false;
        const gravity = 0.2;
        let spawnTimer = 0;
        let spawnInterval = 90;
        let difficultyMultiplier = 1;
        let animationFrameId;

        // Ensure canvas matches screen resolution
        function resize() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resize);
        resize();

        // Input Handling (Mouse & Touch)
        function handleStart(e) {
            if (gameState !== 'PLAYING') return;
            isSwiping = true;
            trail = [];
            addTrailPoint(e);
        }

        function handleMove(e) {
            if (!isSwiping || gameState !== 'PLAYING') return;
            addTrailPoint(e);
            checkCollisions();
        }

        function handleEnd() {
            isSwiping = false;
        }

        function addTrailPoint(e) {
            let x, y;
            if (e.touches && e.touches.length > 0) {
                x = e.touches[0].clientX;
                y = e.touches[0].clientY;
            } else {
                x = e.clientX;
                y = e.clientY;
            }
            trail.push({x, y, age: 0});
            if (trail.length > 8) trail.shift();
        }

        canvas.addEventListener('mousedown', handleStart);
        canvas.addEventListener('mousemove', handleMove);
        window.addEventListener('mouseup', handleEnd);
        
        canvas.addEventListener('touchstart', (e) => { e.preventDefault(); handleStart(e); }, {passive: false});
        canvas.addEventListener('touchmove', (e) => { e.preventDefault(); handleMove(e); }, {passive: false});
        window.addEventListener('touchend', handleEnd);

        // Entities
        class GameObject {
            constructor() {
                // Determine type
                this.isBomb = Math.random() < 0.2; // 20% bombs
                this.type = this.isBomb ? 'bomb' : 'fruit';
                this.radius = this.isBomb ? 35 : Math.random() * 20 + 25;
                
                const fruitColors = ['#e74c3c', '#2ecc71', '#f1c40f', '#9b59b6', '#e67e22', '#1abc9c'];
                this.color = this.isBomb ? '#111' : fruitColors[Math.floor(Math.random() * fruitColors.length)];

                // Spawn position (bottom of screen)
                this.x = Math.random() * (canvas.width * 0.7) + (canvas.width * 0.15);
                this.y = canvas.height + this.radius + 10;
                
                // Velocity based on canvas dimensions to ensure visibility
                this.vx = (Math.random() - 0.5) * (canvas.width * 0.01) * difficultyMultiplier;
                
                // Calculate jump velocity required to reach 60% to 90% of screen height
                let jumpHeight = canvas.height * (Math.random() * 0.3 + 0.6);
                this.vy = -(Math.sqrt(2 * gravity * jumpHeight)) * Math.min(difficultyMultiplier, 1.3);
                
                this.markedForDeletion = false;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.vy += gravity;

                if (this.y - this.radius > canvas.height && this.vy > 0) {
                    this.markedForDeletion = true;
                }
            }

            draw(ctx) {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.closePath();

                if (this.isBomb) {
                    // Bomb details (Fuse & Cap)
                    ctx.fillStyle = '#333';
                    ctx.fillRect(this.x - 8, this.y - this.radius - 6, 16, 10);
                    
                    ctx.beginPath();
                    ctx.moveTo(this.x, this.y - this.radius - 6);
                    ctx.quadraticCurveTo(this.x + 15, this.y - this.radius - 20, this.x + 25, this.y - this.radius - 10);
                    ctx.strokeStyle = '#e74c3c';
                    ctx.lineWidth = 3;
                    ctx.stroke();

                    // Spark
                    if (Math.random() > 0.5) {
                        ctx.beginPath();
                        ctx.arc(this.x + 25, this.y - this.radius - 10, 4, 0, Math.PI * 2);
                        ctx.fillStyle = '#f1c40f';
                        ctx.fill();
                    }

                    // Gloss
                    ctx.beginPath();
                    ctx.arc(this.x - this.radius * 0.3, this.y - this.radius * 0.3, this.radius * 0.3, 0, Math.PI*2);
                    ctx.fillStyle = 'rgba(255,255,255,0.15)';
                    ctx.fill();
                } else {
                    // Fruit details (Gloss & Stem)
                    ctx.beginPath();
                    ctx.arc(this.x, this.y - this.radius, 4, 0, Math.PI*2);
                    ctx.fillStyle = '#8e44ad';
                    ctx.fill();

                    ctx.beginPath();
                    ctx.arc(this.x - this.radius * 0.3, this.y - this.radius * 0.3, this.radius * 0.3, 0, Math.PI*2);
                    ctx.fillStyle = 'rgba(255,255,255,0.3)';
                    ctx.fill();
                }
            }
        }

        class JuiceParticle {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.radius = Math.random() * 6 + 2;
                this.color = color;
                this.vx = (Math.random() - 0.5) * 12;
                this.vy = (Math.random() - 0.5) * 12;
                this.life = 1;
                this.decay = Math.random() * 0.02 + 0.02;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.vy += gravity;
                this.life -= this.decay;
            }

            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.life);
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.restore();
            }
        }

        class HalfFruit {
             constructor(x, y, radius, color, vx, vy, angle) {
                this.x = x;
                this.y = y;
                this.radius = radius;
                this.color = color;
                this.vx = vx;
                this.vy = vy;
                this.angle = angle;
                this.life = 1;
                this.decay = 0.015;
                this.rotationSpeed = (Math.random() - 0.5) * 0.3;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.vy += gravity;
                this.angle += this.rotationSpeed;
                this.life -= this.decay;
            }

            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.life);
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                
                // Outer rind
                ctx.beginPath();
                ctx.arc(0, 0, this.radius, 0, Math.PI);
                ctx.fillStyle = this.color;
                ctx.fill();
                
                // Inner flesh
                ctx.beginPath();
                ctx.arc(0, 0, this.radius - 4, 0, Math.PI);
                ctx.fillStyle = '#fff9c4'; 
                ctx.fill();
                
                ctx.restore();
            }
        }

        class SparkleParticle {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.size = Math.random() * 15 + 10;
                this.vx = (Math.random() - 0.5) * 6;
                this.vy = (Math.random() - 0.5) * 6;
                this.life = 1;
                this.decay = Math.random() * 0.04 + 0.04;
                this.angle = Math.random() * Math.PI * 2;
                this.rotationSpeed = (Math.random() - 0.5) * 0.4;
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.angle += this.rotationSpeed;
                this.life -= this.decay;
            }

            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.life);
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                
                ctx.beginPath();
                ctx.moveTo(0, -this.size);
                ctx.quadraticCurveTo(this.size*0.2, -this.size*0.2, this.size, 0);
                ctx.quadraticCurveTo(this.size*0.2, this.size*0.2, 0, this.size);
                ctx.quadraticCurveTo(-this.size*0.2, this.size*0.2, -this.size, 0);
                ctx.quadraticCurveTo(-this.size*0.2, -this.size*0.2, 0, -this.size);
                
                ctx.fillStyle = '#ffffff';
                ctx.fill();
                
                ctx.scale(0.5, 0.5);
                ctx.fillStyle = '#f1c40f';
                ctx.fill();
                
                ctx.restore();
            }
        }

        class FireParticle {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.radius = Math.random() * 20 + 10;
                
                let angle = Math.random() * Math.PI * 2;
                let speed = Math.random() * 8 + 2;
                this.vx = Math.cos(angle) * speed;
                this.vy = Math.sin(angle) * speed;
                
                this.life = 1;
                this.decay = Math.random() * 0.03 + 0.02;
                
                const colors = ['#fff59d', '#ffb300', '#e65100', '#bf360c', '#333333'];
                this.color = colors[Math.floor(Math.random() * colors.length)];
            }

            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.radius *= 0.95;
                this.life -= this.decay;
            }

            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.max(0, this.life);
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.restore();
            }
        }

        // Logic
        function spawnRoutine() {
            spawnTimer++;
            if (spawnTimer > spawnInterval) {
                let count = Math.floor(Math.random() * 3) + 1; // 1 to 3 objects
                for(let i=0; i<count; i++) {
                    objects.push(new GameObject());
                }
                spawnTimer = 0;
            }
        }

        function createExplosion(x, y, color, radius) {
            // Juice
            for (let i = 0; i < 15; i++) {
                particles.push(new JuiceParticle(x, y, color));
            }

            // Sparkles
            for (let i = 0; i < 8; i++) {
                particles.push(new SparkleParticle(x, y));
            }
            
            // Two halves split perpendicularly to a random slash angle
            let slashAngle = Math.random() * Math.PI * 2;
            let splitVelocity = 4;
            let nx = Math.cos(slashAngle) * splitVelocity;
            let ny = Math.sin(slashAngle) * splitVelocity;
            
            particles.push(new HalfFruit(x, y, radius, color, nx, ny, slashAngle));
            particles.push(new HalfFruit(x, y, radius, color, -nx, -ny, slashAngle + Math.PI));
        }

        function createBombExplosion(x, y) {
            for (let i = 0; i < 60; i++) {
                particles.push(new FireParticle(x, y));
            }
        }

        function lineIntersectsCircle(p1, p2, center, radius) {
            let dx = p2.x - p1.x;
            let dy = p2.y - p1.y;
            let lenSquare = dx * dx + dy * dy;
            if (lenSquare === 0) return false;

            let dot = (((center.x - p1.x) * dx) + ((center.y - p1.y) * dy)) / lenSquare;
            
            let closestX = p1.x + (dot * dx);
            let closestY = p1.y + (dot * dy);

            if (dot < 0 || dot > 1) {
                let dist1 = Math.hypot(center.x - p1.x, center.y - p1.y);
                let dist2 = Math.hypot(center.x - p2.x, center.y - p2.y);
                return dist1 <= radius || dist2 <= radius;
            }

            let distToLineSquare = (closestX - center.x) ** 2 + (closestY - center.y) ** 2;
            return distToLineSquare <= radius * radius;
        }

        function checkCollisions() {
            if (trail.length < 2) return;
            let p1 = trail[trail.length - 2];
            let p2 = trail[trail.length - 1];

            for (let i = objects.length - 1; i >= 0; i--) {
                let obj = objects[i];
                if (lineIntersectsCircle(p1, p2, {x: obj.x, y: obj.y}, obj.radius)) {
                    if (obj.type === 'fruit') {
                        score += 10;
                        scoreValue.innerText = score;
                        createExplosion(obj.x, obj.y, obj.color, obj.radius);
                        playSound('slice');
                        objects.splice(i, 1);
                        
                        // Progressive difficulty scaling
                        if (score % 100 === 0) {
                            difficultyMultiplier += 0.15;
                            spawnInterval = Math.max(30, spawnInterval - 10);
                        }
                    } else if (obj.type === 'bomb') {
                        createBombExplosion(obj.x, obj.y);
                        objects.splice(i, 1);
                        gameOver();
                    }
                }
            }
        }

        function startGame() {
            initAudio();
            gameState = 'PLAYING';
            score = 0;
            scoreValue.innerText = score;
            objects = [];
            particles = [];
            trail = [];
            difficultyMultiplier = 1;
            spawnInterval = 90;
            spawnTimer = 0;
            
            startScreen.classList.remove('active');
            gameOverScreen.classList.remove('active');
            scoreDisplay.style.display = 'block';
            
            cancelAnimationFrame(animationFrameId);
            gameLoop();
        }

        let flashAlpha = 0;

        function gameOver() {
            gameState = 'GAMEOVER';
            playSound('bomb');
            
            flashAlpha = 1;

            finalScore.innerText = score;
            setTimeout(() => {
                gameOverScreen.classList.add('active');
                scoreDisplay.style.display = 'none';
            }, 1500); // 1.5 seconds delay so we see the explosion!
        }

        function gameLoop() {
            // Clear Frame with slight trail effect (motion blur background)
            ctx.fillStyle = '#1a1a1a';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            if (gameState === 'PLAYING') {
                spawnRoutine();
            }

            // Draw and update entities
            for (let i = objects.length - 1; i >= 0; i--) {
                objects[i].update();
                objects[i].draw(ctx);
                if (objects[i].markedForDeletion) {
                    objects.splice(i, 1);
                }
            }

            for (let i = particles.length - 1; i >= 0; i--) {
                particles[i].update();
                particles[i].draw(ctx);
                if (particles[i].life <= 0) {
                    particles.splice(i, 1);
                }
            }

            // Draw Samurai Sword Trail
            if (gameState === 'PLAYING') {
                if (trail.length > 1) {
                    ctx.beginPath();
                    ctx.moveTo(trail[0].x, trail[0].y);
                    for (let i = 1; i < trail.length; i++) {
                        ctx.lineTo(trail[i].x, trail[i].y);
                    }
                    
                    ctx.shadowBlur = 15;
                    ctx.shadowColor = '#00f3ff';
                    ctx.strokeStyle = '#ffffff';
                    ctx.lineWidth = 6;
                    ctx.lineCap = 'round';
                    ctx.lineJoin = 'round';
                    ctx.stroke();
                    
                    // Reset shadow to avoid affecting other elements
                    ctx.shadowBlur = 0;
                    
                    // Trail fade logic
                    if (!isSwiping) {
                        trail.shift();
                        trail.shift(); // fade out faster
                    }
                } else if (!isSwiping && trail.length > 0) {
                    trail = []; // instantly clear remainder
                }
            }

            // Flash effect for bomb explosion
            if (flashAlpha > 0) {
                ctx.fillStyle = `rgba(255, 255, 255, ${flashAlpha})`;
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                flashAlpha -= 0.05;
            }

            animationFrameId = requestAnimationFrame(gameLoop);
        }

        // Bind UI buttons
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', startGame);

    </script>
</body>
</html>
