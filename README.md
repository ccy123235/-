<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>贪吃蛇</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; touch-action: none; }
        body {
            background: #1a1a2e; color: #fff; font-family: Arial, sans-serif;
            display: flex; flex-direction: column; align-items: center;
            justify-content: center; min-height: 100vh; padding: 12px;
            user-select: none; -webkit-user-select: none;
        }
        .score { font-size: 22px; margin-bottom: 10px; color: #4cc9f0; font-weight: bold; }
        canvas {
            background: #0c0c1a; border: 2px solid #4cc9f0; border-radius: 8px;
            display: block; width: 92vw; max-width: 420px; height: auto;
        }
        .controls {
            display: grid; grid-template-columns: 60px 60px 60px;
            gap: 8px; margin-top: 18px; justify-content: center;
        }
        .controls button {
            background: #0f3460; color: #4cc9f0; border: 1px solid #4cc9f0;
            border-radius: 10px; font-size: 22px; padding: 14px 0;
            font-weight: bold; cursor: pointer;
        }
        .controls button:active { background: #4cc9f0; color: #16213e; }
        .controls .up    { grid-column: 2; }
        .controls .left  { grid-column: 1; grid-row: 2; }
        .controls .down  { grid-column: 2; grid-row: 2; }
        .controls .right { grid-column: 3; grid-row: 2; }
        .hint { font-size: 13px; color: #888; margin-top: 12px; text-align: center; }
    </style>
</head>
<body>
    <div class="score">得分：<span id="score">0</span></div>
    <canvas id="game" width="400" height="400"></canvas>

    <div class="controls">
        <button class="up"    onclick="setDir(0,-1)">▲</button>
        <button class="left"  onclick="setDir(-1,0)">◀</button>
        <button class="down"  onclick="setDir(0,1)">▼</button>
        <button class="right" onclick="setDir(1,0)">▶</button>
    </div>
    <div class="hint">也可以直接在屏幕上滑动控制方向</div>

<script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');

    const GRID = 20;          // 20x20 格子
    const CELL = canvas.width / GRID;
    let snake, food, dir, nextDir, score, gameOver, timer;

    function init() {
        snake = [{x: 10, y: 10}, {x: 9, y: 10}, {x: 8, y: 10}];
        dir = {x: 1, y: 0};
        nextDir = {x: 1, y: 0};
        score = 0;
        gameOver = false;
        scoreEl.textContent = '0';
        placeFood();
        clearInterval(timer);
        timer = setInterval(tick, 150);
        draw();
    }

    function placeFood() {
        while (true) {
            const fx = Math.floor(Math.random() * GRID);
            const fy = Math.floor(Math.random() * GRID);
            if (!snake.some(s => s.x === fx && s.y === fy)) {
                food = {x: fx, y: fy};
                break;
            }
        }
    }

    function setDir(x, y) {
        // 不能原地掉头
        if (x === -dir.x && y === -dir.y) return;
        nextDir = {x, y};
    }

    function tick() {
        if (gameOver) return;
        dir = nextDir;
        const head = {x: snake[0].x + dir.x, y: snake[0].y + dir.y};

        // 撞墙或撞自己
        if (
            head.x < 0 || head.x >= GRID ||
            head.y < 0 || head.y >= GRID ||
            snake.some(s => s.x === head.x && s.y === head.y)
        ) {
            gameOver = true;
            clearInterval(timer);
            draw();
            setTimeout(() => {
                alert('游戏结束！得分：' + score);
                init();
            }, 50);
            return;
        }

        snake.unshift(head);

        if (head.x === food.x && head.y === food.y) {
            score++;
            scoreEl.textContent = score;
            placeFood();
        } else {
            snake.pop();
        }

        draw();
    }

    function draw() {
        ctx.fillStyle = '#0c0c1a';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // 食物
        ctx.fillStyle = '#e94560';
        ctx.beginPath();
        ctx.arc(food.x * CELL + CELL/2, food.y * CELL + CELL/2, CELL/2 - 2, 0, Math.PI * 2);
        ctx.fill();

        // 蛇身
        snake.forEach((s, i) => {
            ctx.fillStyle = i === 0 ? '#4cc9f0' : '#2a9d8f';
            ctx.fillRect(s.x * CELL + 1, s.y * CELL + 1, CELL - 2, CELL - 2);
        });

        if (gameOver) {
            ctx.fillStyle = 'rgba(0,0,0,0.6)';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#fff';
            ctx.font = 'bold 24px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('游戏结束', canvas.width/2, canvas.height/2);
        }
    }

    // 滑动控制
    let touchX = 0, touchY = 0;
    canvas.addEventListener('touchstart', e => {
        const t = e.touches[0];
        touchX = t.clientX;
        touchY = t.clientY;
    }, {passive: true});

    canvas.addEventListener('touchmove', e => {
        e.preventDefault();
        const t = e.touches[0];
        const dx = t.clientX - touchX;
        const dy = t.clientY - touchY;
        if (Math.abs(dx) < 20 && Math.abs(dy) < 20) return;

        if (Math.abs(dx) > Math.abs(dy)) {
            setDir(dx > 0 ? 1 : -1, 0);
        } else {
            setDir(0, dy > 0 ? 1 : -1);
        }
        touchX = t.clientX;
        touchY = t.clientY;
    }, {passive: false});

    init();
</script>
</body>
</html># -
贪吃蛇
