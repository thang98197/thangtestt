const canvas = document.getElementById('fireworks');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

const particles = [];

function Particle(x, y, color) {
    this.x = x;
    this.y = y;
    this.color = color;
    this.vx = Math.random() * 10 - 5;
    this.vy = Math.random() * 10 - 5;
    this.alpha = 1;
    this.size = Math.random() * 5 + 2;
}

Particle.prototype.update = function() {
    this.x += this.vx;
    this.y += this.vy;
    this.alpha -= 0.02;
    this.size *= 0.98;
};

Particle.prototype.draw = function() {
    ctx.globalAlpha = this.alpha;
    ctx.fillStyle = this.color;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
    ctx.fill();
};

function createFirework(x, y) {
    const colors = ['#ff0040', '#ff4081', '#ff80ab', '#ffff00', '#00ffff', '#8000ff'];
    for (let i = 0; i < 50; i++) {
        const color = colors[Math.floor(Math.random() * colors.length)];
        particles.push(new Particle(x, y, color));
    }
}

function animate() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    particles.forEach((particle, index) => {
        particle.update();
        particle.draw();
        if (particle.alpha <= 0) {
            particles.splice(index, 1);
        }
    });
    requestAnimationFrame(animate);
}

animate();

document.getElementById('messageForm').addEventListener('submit', function(e) {
    e.preventDefault();
    const name = document.getElementById('senderName').value.trim();
    const msg = document.getElementById('message').value.trim();
    if (name && msg) {
        const text = `Tên: ${name}\nLời nhắn: ${msg}`;
        fetch(`https://api.telegram.org/bot8568502284:AAHrFHaMeOMW8plyPo0hnlpytC7hl37ionw/sendMessage`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                chat_id: '1111320262',
                text: text
            })
        })
        .then(response => response.json())
        .then(data => {
            if (data.ok) {
                alert('Tin nhắn đã được gửi thành công!');
            } else {
                alert('Lỗi gửi tin nhắn: ' + data.description);
            }
        })
        .catch(error => {
            alert('Lỗi kết nối: ' + error.message);
        });
        this.reset();
    }
});

function launchFireworks() {
    const x = Math.random() * canvas.width;
    const y = Math.random() * canvas.height / 2;
    createFirework(x, y);
    setTimeout(() => createFirework(Math.random() * canvas.width, Math.random() * canvas.height / 2), 200);
    setTimeout(() => createFirework(Math.random() * canvas.width, Math.random() * canvas.height / 2), 400);
}

const snakeCanvas = document.getElementById('snakeCanvas');
const snakeCtx = snakeCanvas.getContext('2d');
const tileSize = 20;
const tileCount = 20;
let snake = [];
let velocity = { x: 0, y: 0 };
let apple = { x: 10, y: 10 };
let score = 0;
let gameInterval = null;

function resetSnakeGame() {
    snake = [{ x: 10, y: 10 }];
    velocity = { x: 1, y: 0 };
    score = 0;
    placeApple();
    updateScore();
    drawSnake();
}

function placeApple() {
    apple.x = Math.floor(Math.random() * tileCount);
    apple.y = Math.floor(Math.random() * tileCount);
    if (snake.some(segment => segment.x === apple.x && segment.y === apple.y)) {
        placeApple();
    }
}

function updateScore() {
    document.getElementById('score').textContent = `Điểm: ${score}`;
}

function drawSnake() {
    snakeCtx.fillStyle = '#111';
    snakeCtx.fillRect(0, 0, snakeCanvas.width, snakeCanvas.height);

    snakeCtx.fillStyle = '#ff4d4d';
    snakeCtx.fillRect(apple.x * tileSize, apple.y * tileSize, tileSize - 2, tileSize - 2);

    snakeCtx.fillStyle = '#00ff6a';
    snake.forEach(segment => {
        snakeCtx.fillRect(segment.x * tileSize, segment.y * tileSize, tileSize - 2, tileSize - 2);
    });
}

function updateSnake() {
    const head = {
        x: snake[0].x + velocity.x,
        y: snake[0].y + velocity.y
    };

    if (
        head.x < 0 ||
        head.x >= tileCount ||
        head.y < 0 ||
        head.y >= tileCount ||
        snake.some((segment, index) => index > 0 && segment.x === head.x && segment.y === head.y)
    ) {
        clearInterval(gameInterval);
        gameInterval = null;
        alert('Game over! Điểm của bạn: ' + score);
        return;
    }

    snake.unshift(head);

    if (head.x === apple.x && head.y === apple.y) {
        score += 10;
        updateScore();
        placeApple();
    } else {
        snake.pop();
    }

    drawSnake();
}

document.getElementById('startGame').addEventListener('click', function() {
    if (gameInterval) {
        clearInterval(gameInterval);
    }
    resetSnakeGame();
    gameInterval = setInterval(updateSnake, 120);
});

window.addEventListener('keydown', function(e) {
    if (!gameInterval) return;
    switch (e.key) {
        case 'ArrowUp':
            if (velocity.y === 1) return;
            velocity = { x: 0, y: -1 };
            break;
        case 'ArrowDown':
            if (velocity.y === -1) return;
            velocity = { x: 0, y: 1 };
            break;
        case 'ArrowLeft':
            if (velocity.x === 1) return;
            velocity = { x: -1, y: 0 };
            break;
        case 'ArrowRight':
            if (velocity.x === -1) return;
            velocity = { x: 1, y: 0 };
            break;
    }
});

window.addEventListener('resize', () => {
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
});