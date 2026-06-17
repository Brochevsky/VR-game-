<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Игры для малышей</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; user-select: none; touch-action: none; }
        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background: #1a2a3a;
            font-family: 'Segoe UI', Arial, sans-serif;
        }

        /* Блокируем вертикальный режим */
        #rotateMessage {
            position: fixed;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.95);
            color: white;
            display: none;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 9999;
            font-size: 24px;
            text-align: center;
            padding: 20px;
        }
        #rotateMessage .icon {
            font-size: 80px;
            margin-bottom: 20px;
            animation: rotatePhone 2s ease-in-out infinite;
        }
        @keyframes rotatePhone {
            0% { transform: rotate(0deg); }
            50% { transform: rotate(-90deg); }
            100% { transform: rotate(0deg); }
        }

        /* Основное меню — сверху, но в ландшафте */
        #menu {
            background: #2a4a5a;
            padding: 8px 10px;
            display: flex;
            justify-content: center;
            gap: 8px;
            flex-wrap: nowrap;
            border-bottom: 3px solid #ffcc44;
            flex-shrink: 0;
            z-index: 100;
            width: 100%;
            position: relative;
        }
        #menu button {
            background: #ffcc44;
            border: none;
            border-radius: 30px;
            padding: 8px 14px;
            font-size: 14px;
            font-weight: bold;
            color: #1a2a3a;
            cursor: pointer;
            transition: 0.2s;
            box-shadow: 0 3px 0 #b38822;
            touch-action: manipulation;
            flex: 1;
            max-width: 180px;
            white-space: nowrap;
        }
        #menu button:active {
            transform: translateY(3px);
            box-shadow: 0 0 0 #b38822;
        }
        #menu button.active {
            background: #88ddff;
            box-shadow: 0 3px 0 #44aadd;
        }

        /* Игровой контейнер — на весь остаток экрана */
        #gameContainer {
            flex: 1;
            position: relative;
            overflow: hidden;
            background: #87CEEB;
            width: 100%;
            height: calc(100% - 50px); /* отнимаем высоту меню */
        }

        canvas {
            display: block;
            width: 100% !important;
            height: 100% !important;
            background: transparent;
            touch-action: none;
        }

        #hint {
            position: absolute;
            bottom: 15px;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0,0,0,0.6);
            color: white;
            padding: 8px 20px;
            border-radius: 30px;
            font-size: 16px;
            pointer-events: none;
            white-space: nowrap;
            backdrop-filter: blur(4px);
            border: 1px solid rgba(255,255,255,0.2);
            z-index: 10;
        }

        #scoreDisplay {
            position: absolute;
            top: 15px;
            right: 20px;
            background: rgba(0,0,0,0.5);
            color: #ffdd44;
            padding: 6px 16px;
            border-radius: 20px;
            font-size: 22px;
            font-weight: bold;
            pointer-events: none;
            backdrop-filter: blur(4px);
            border: 1px solid rgba(255,255,255,0.2);
            z-index: 10;
        }

        #winOverlay {
            position: absolute;
            top: 0; left: 0; right: 0; bottom: 0;
            background: rgba(0,0,0,0.4);
            display: none;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            z-index: 50;
            backdrop-filter: blur(4px);
        }
        #winOverlay.show { display: flex; }
        #winOverlay h1 {
            font-size: 56px;
            color: #ffdd44;
            text-shadow: 0 0 40px rgba(255,220,68,0.6);
            animation: bounce 0.8s ease infinite alternate;
        }
        #winOverlay p {
            font-size: 28px;
            color: white;
            margin-top: 10px;
            text-shadow: 0 0 20px rgba(0,0,0,0.5);
        }
        @keyframes bounce {
            0% { transform: scale(1); }
            100% { transform: scale(1.1); }
        }

        @media (max-width: 700px) {
            #menu button { font-size: 12px; padding: 6px 10px; }
            #hint { font-size: 13px; padding: 5px 14px; bottom: 10px; }
            #scoreDisplay { font-size: 18px; padding: 4px 12px; top: 10px; right: 12px; }
            #winOverlay h1 { font-size: 38px; }
            #winOverlay p { font-size: 20px; }
            #rotateMessage { font-size: 18px; }
            #rotateMessage .icon { font-size: 60px; }
        }
        @media (max-height: 500px) {
            #menu { padding: 4px 8px; }
            #menu button { font-size: 11px; padding: 4px 8px; }
            #hint { font-size: 11px; padding: 3px 10px; bottom: 5px; }
            #scoreDisplay { font-size: 14px; padding: 2px 10px; top: 5px; right: 8px; }
        }
    </style>
</head>
<body>

    <!-- СООБЩЕНИЕ О ПОВОРОТЕ -->
    <div id="rotateMessage">
        <div class="icon">📱</div>
        <div>Пожалуйста, поверни телефон<br><strong>горизонтально</strong></div>
    </div>

    <div id="menu">
        <button id="mode1" class="active">🦎 Корми ящерицу</button>
        <button id="mode2">🎈 Лопни 20</button>
        <button id="mode3">☁️ Шарики в небо</button>
    </div>

    <div id="gameContainer">
        <canvas id="gameCanvas"></canvas>
        <div id="scoreDisplay">⭐ 0</div>
        <div id="hint">👆 Тапай или тащи пальцем</div>
        <div id="winOverlay">
            <h1>🎉 ТЫ МОЛОДЕЦ!</h1>
            <p>Собрал 20 шариков!</p>
        </div>
    </div>

    <script>
        // ============================================================
        //  БЛОКИРОВКА ОРИЕНТАЦИИ (ТОЛЬКО ЛАНДШАФТ)
        // ============================================================
        const rotateMessage = document.getElementById('rotateMessage');

        function checkOrientation() {
            // Если высота больше ширины — телефон вертикально (показываем предупреждение)
            if (window.innerHeight > window.innerWidth) {
                rotateMessage.style.display = 'flex';
                document.body.style.overflow = 'hidden';
            } else {
                rotateMessage.style.display = 'none';
                document.body.style.overflow = 'hidden';
            }
        }

        window.addEventListener('load', checkOrientation);
        window.addEventListener('resize', checkOrientation);

        // Принудительно блокируем ландшафт
        try {
            if (screen.orientation && screen.orientation.lock) {
                screen.orientation.lock('landscape').catch(() => {});
            }
        } catch(e) {}

        // ============================================================
        //  ЗВУКИ
        // ============================================================
        class SoundFX {
            constructor() {
                this.ctx = null;
                this.enabled = true;
                try {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                } catch(e) {
                    this.enabled = false;
                }
            }
            resume() {
                if (this.ctx && this.ctx.state === 'suspended') this.ctx.resume();
            }
            slurp() {
                if (!this.enabled || !this.ctx) return;
                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(800, this.ctx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(200, this.ctx.currentTime + 0.15);
                    gain.gain.setValueAtTime(0.2, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.15);
                    osc.start(this.ctx.currentTime);
                    osc.stop(this.ctx.currentTime + 0.15);
                } catch(e) {}
            }
            pop() {
                if (!this.enabled || !this.ctx) return;
                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.connect(gain);
                    gain.connect(this.ctx.destination);
                    osc.type = 'square';
                    osc.frequency.setValueAtTime(600, this.ctx.currentTime);
                    osc.frequency.exponentialRampToValueAtTime(100, this.ctx.currentTime + 0.08);
                    gain.gain.setValueAtTime(0.15, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + 0.08);
                    osc.start(this.ctx.currentTime);
                    osc.stop(this.ctx.currentTime + 0.08);
                } catch(e) {}
            }
            win() {
                if (!this.enabled || !this.ctx) return;
                try {
                    const notes = [523, 659, 784, 1047, 784, 659, 523];
                    for (let i = 0; i < notes.length; i++) {
                        const osc = this.ctx.createOscillator();
                        const gain = this.ctx.createGain();
                        osc.connect(gain);
                        gain.connect(this.ctx.destination);
                        osc.type = 'sine';
                        osc.frequency.setValueAtTime(notes[i], this.ctx.currentTime + i * 0.12);
                        gain.gain.setValueAtTime(0.15, this.ctx.currentTime + i * 0.12);
                        gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + i * 0.12 + 0.15);
                        osc.start(this.ctx.currentTime + i * 0.12);
                        osc.stop(this.ctx.currentTime + i * 0.12 + 0.15);
                    }
                } catch(e) {}
            }
            colorChange() {
                if (!this.enabled || !this.ctx) return;
                try {
                    const notes = [880, 1100];
                    for (let i = 0; i < notes.length; i++) {
                        const osc = this.ctx.createOscillator();
                        const gain = this.ctx.createGain();
                        osc.connect(gain);
                        gain.connect(this.ctx.destination);
                        osc.type = 'sine';
                        osc.frequency.setValueAtTime(notes[i], this.ctx.currentTime + i * 0.1);
                        gain.gain.setValueAtTime(0.12, this.ctx.currentTime + i * 0.1);
                        gain.gain.exponentialRampToValueAtTime(0.01, this.ctx.currentTime + i * 0.1 + 0.12);
                        osc.start(this.ctx.currentTime + i * 0.1);
                        osc.stop(this.ctx.currentTime + i * 0.1 + 0.12);
                    }
                } catch(e) {}
            }
        }

        const sound = new SoundFX();

        // ============================================================
        //  ОСНОВНЫЕ НАСТРОЙКИ
        // ============================================================
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('gameContainer');
        const scoreDisplay = document.getElementById('scoreDisplay');
        const hint = document.getElementById('hint');
        const winOverlay = document.getElementById('winOverlay');

        let W, H;

        function resize() {
            W = container.clientWidth;
            H = container.clientHeight;
            canvas.width = W;
            canvas.height = H;
        }
        window.addEventListener('resize', resize);
        resize();

        // ============================================================
        //  СОСТОЯНИЕ ИГРЫ
        // ============================================================
        let currentMode = 1;
        let score = 0;
        let dragTarget = null;
        let dragOffsetX = 0, dragOffsetY = 0;
        let frameId = null;

        // ============================================================
        //  РЕЖИМ 1 — ЯЩЕРИЦА
        // ============================================================
        let lizard = {
            x: 0, y: 0,
            mouthX: 0, mouthY: 0,
            tongueOut: false,
            tongueTimer: 0,
            color: '#4CAF50',
            happy: false,
            happyTimer: 0,
            eatCount: 0,
            colorChangeTimer: 0,
            colorChanging: false
        };
        let flies = [];

        const LIZARD_COLORS = [
            '#4CAF50', '#e74c3c', '#3498db', '#f39c12',
            '#9b59b6', '#1abc9c', '#e67e22', '#e84393',
            '#00b894', '#6c5ce7', '#fd79a8', '#fdcb6e'
        ];

        function initLizard() {
            lizard.x = W * 0.4;
            lizard.y = H * 0.6;
            lizard.mouthX = lizard.x + 45;
            lizard.mouthY = lizard.y - 10;
            lizard.tongueOut = false;
            lizard.tongueTimer = 0;
            lizard.color = '#4CAF50';
            lizard.happy = false;
            lizard.happyTimer = 0;
            lizard.eatCount = 0;
            lizard.colorChanging = false;
            lizard.colorChangeTimer = 0;
            flies = [];
            for (let i = 0; i < 5; i++) {
                flies.push({
                    x: W * 0.6 + Math.random() * W * 0.3,
                    y: 50 + Math.random() * (H * 0.5),
                    r: 22,
                    vx: (Math.random() - 0.5) * 2,
                    vy: (Math.random() - 0.5) * 2,
                    alive: true,
                    emoji: '🪰'
                });
            }
        }

        function updateLizard() {
            for (let f of flies) {
                if (!f.alive) continue;
                f.x += f.vx;
                f.y += f.vy;
                if (f.x < W * 0.3 || f.x > W - 30) f.vx *= -1;
                if (f.y < 30 || f.y > H * 0.6) f.vy *= -1;
            }
            if (lizard.tongueOut) {
                lizard.tongueTimer++;
                if (lizard.tongueTimer > 30) {
                    lizard.tongueOut = false;
                    lizard.tongueTimer = 0;
                }
            }
            if (lizard.happy) {
                lizard.happyTimer++;
                if (lizard.happyTimer > 40) {
                    lizard.happy = false;
                    lizard.happyTimer = 0;
                }
            }
            if (lizard.colorChanging) {
                lizard.colorChangeTimer++;
                if (lizard.colorChangeTimer > 30) {
                    lizard.colorChanging = false;
                    lizard.colorChangeTimer = 0;
                }
            }
        }

        function drawLizard() {
            const l = lizard;
            ctx.save();

            let currentColor = l.color;
            if (l.colorChanging) {
                const flash = Math.sin(l.colorChangeTimer * 0.5) > 0;
                if (flash) currentColor = '#ffffff';
            }

            ctx.shadowColor = 'rgba(0,0,0,0.15)';
            ctx.shadowBlur = 20;

            // Хвост
            ctx.beginPath();
            ctx.moveTo(l.x - 45, l.y + 5);
            ctx.quadraticCurveTo(l.x - 80, l.y - 15, l.x - 100, l.y + 10);
            ctx.quadraticCurveTo(l.x - 80, l.y + 25, l.x - 45, l.y + 10);
            ctx.fillStyle = currentColor;
            ctx.fill();
            ctx.strokeStyle = '#2E7D32';
            ctx.lineWidth = 1.5;
            ctx.stroke();

            // Тело
            ctx.beginPath();
            ctx.ellipse(l.x, l.y, 50, 32, 0, 0, Math.PI * 2);
            ctx.fillStyle = currentColor;
            ctx.fill();
            ctx.strokeStyle = '#2E7D32';
            ctx.lineWidth = 2;
            ctx.stroke();

            // Голова
            ctx.beginPath();
            ctx.ellipse(l.x + 45, l.y - 5, 28, 24, 0.2, 0, Math.PI * 2);
            ctx.fillStyle = currentColor;
            ctx.fill();
            ctx.stroke();

            // Глаза
            ctx.shadowBlur = 0;
            // Левый глаз
            ctx.beginPath();
            ctx.ellipse(l.x + 52, l.y - 16, 10, 11, 0, 0, Math.PI * 2);
            ctx.fillStyle = 'white';
            ctx.fill();
            ctx.strokeStyle = '#2E7D32';
            ctx.lineWidth = 1.5;
            ctx.stroke();
            ctx.beginPath();
            ctx.arc(l.x + 57, l.y - 18, 5, 0, Math.PI * 2);
            ctx.fillStyle = '#1a1a2e';
            ctx.fill();
            ctx.beginPath();
            ctx.arc(l.x + 58, l.y - 19, 2, 0, Math.PI * 2);
            ctx.fillStyle = 'white';
            ctx.fill();

            // Правый глаз
            ctx.beginPath();
            ctx.ellipse(l.x + 38, l.y - 18, 9, 10, 0, 0, Math.PI * 2);
            ctx.fillStyle = 'white';
            ctx.fill();
            ctx.stroke();
            ctx.beginPath();
            ctx.arc(l.x + 43, l.y - 20, 4.5, 0, Math.PI * 2);
            ctx.fillStyle = '#1a1a2e';
            ctx.fill();
            ctx.beginPath();
            ctx.arc(l.x + 44, l.y - 21, 1.8, 0, Math.PI * 2);
            ctx.fillStyle = 'white';
            ctx.fill();

            // Рот
            ctx.beginPath();
            ctx.arc(l.x + 47, l.y + 2, 12, 0.1, Math.PI - 0.1);
            ctx.strokeStyle = '#2E7D32';
            ctx.lineWidth = 2;
            ctx.stroke();

            // Язык
            if (l.tongueOut) {
                ctx.beginPath();
                ctx.moveTo(l.x + 54, l.y + 4);
                ctx.quadraticCurveTo(l.x + 80, l.y - 12, l.x + 95, l.y + 2);
                ctx.strokeStyle = '#e74c3c';
                ctx.lineWidth = 5;
                ctx.stroke();
                ctx.beginPath();
                ctx.arc(l.x + 95, l.y + 2, 6, 0, Math.PI * 2);
                ctx.fillStyle = '#e74c3c';
                ctx.fill();
            }

            // Лапки
            ctx.fillStyle = currentColor;
            ctx.strokeStyle = '#2E7D32';
            ctx.lineWidth = 1.5;
            const paws = [
                [-30, 30, -0.2],
                [-12, 32, 0.2],
                [12, 32, -0.2],
                [30, 30, 0.2]
            ];
            for (let [dx, dy, rot] of paws) {
                ctx.beginPath();
                ctx.ellipse(l.x + dx, l.y + dy, 8, 5, rot, 0, Math.PI * 2);
                ctx.fill();
                ctx.stroke();
                for (let i = -1; i <= 1; i++) {
                    ctx.beginPath();
                    ctx.arc(l.x + dx + i * 5, l.y + dy + 6, 3, 0, Math.PI * 2);
                    ctx.fill();
                    ctx.stroke();
                }
            }

            // Чешуйки
            ctx.shadowBlur = 0;
            for (let i = 0; i < 6; i++) {
                const angle = i / 6 * Math.PI * 2;
                const cx = l.x + Math.cos(angle) * 28;
                const cy = l.y + Math.sin(angle) * 16;
                ctx.beginPath();
                ctx.arc(cx, cy, 5, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(46, 125, 50, 0.25)';
                ctx.fill();
            }

            ctx.restore();

            // Мухи
            for (let f of flies) {
                if (!f.alive) continue;
                ctx.save();
                ctx.font = `${f.r * 2}px Arial`;
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.shadowColor = 'rgba(0,0,0,0.15)';
                ctx.shadowBlur = 15;
                ctx.fillText(f.emoji, f.x, f.y);
                ctx.restore();
            }

            // Звёзды счастья
            if (l.happy) {
                ctx.save();
                ctx.font = '40px Arial';
                ctx.textAlign = 'center';
                ctx.shadowColor = 'rgba(255,220,68,0.5)';
                ctx.shadowBlur = 30;
                ctx.fillText('⭐', l.x - 60, l.y - 50);
                ctx.fillText('✨', l.x + 80, l.y - 60);
                ctx.fillText('🌟', l.x + 10, l.y - 70);
                ctx.restore();
            }

            // Счётчик
            ctx.save();
            ctx.font = '22px Arial';
            ctx.textAlign = 'center';
            ctx.fillStyle = 'rgba(0,0,0,0.5)';
            ctx.fillText('🐛 Съедено: ' + lizard.eatCount, l.x, l.y + 60);
            ctx.restore();
        }

        function changeLizardColor() {
            let newColor;
            let attempts = 0;
            do {
                const idx = Math.floor(Math.random() * LIZARD_COLORS.length);
                newColor = LIZARD_COLORS[idx];
                attempts++;
            } while (newColor === lizard.color && attempts < 20);
            lizard.color = newColor;
            lizard.colorChanging = true;
            lizard.colorChangeTimer = 0;
            sound.colorChange();
            lizard.happy = true;
            lizard.happyTimer = 0;
        }

        function checkEatFly(mx, my) {
            const l = lizard;
            const mouthDist = Math.hypot(mx - l.mouthX, my - (l.mouthY + 4));
            if (mouthDist < 45) {
                let closest = null;
                let closestDist = Infinity;
                for (let f of flies) {
                    if (!f.alive) continue;
                    const d = Math.hypot(mx - f.x, my - f.y);
                    if (d < closestDist) {
                        closestDist = d;
                        closest = f;
                    }
                }
                if (closest && closestDist < 65) {
                    closest.alive = false;
                    sound.slurp();
                    lizard.tongueOut = true;
                    lizard.tongueTimer = 0;
                    lizard.happy = true;
                    lizard.happyTimer = 0;
                    lizard.eatCount++;
                    score++;
                    updateScore();

                    if (lizard.eatCount % 10 === 0) {
                        changeLizardColor();
                    }

                    setTimeout(() => {
                        flies.push({
                            x: W * 0.6 + Math.random() * W * 0.3,
                            y: 50 + Math.random() * (H * 0.5),
                            r: 22,
                            vx: (Math.random() - 0.5) * 2,
                            vy: (Math.random() - 0.5) * 2,
                            alive: true,
                            emoji: '🪰'
                        });
                    }, 800);
                    return true;
                }
            }
            return false;
        }

        // ============================================================
        //  РЕЖИМ 2 — 20 ШАРИКОВ
        // ============================================================
        let balloons2 = [];
        let winMode2 = false;

        function initMode2() {
            balloons2 = [];
            winMode2 = false;
            winOverlay.classList.remove('show');
            for (let i = 0; i < 15; i++) {
                balloons2.push({
                    x: 30 + Math.random() * (W - 60),
                    y: 30 + Math.random() * (H - 80),
                    r: 25 + Math.random() * 22,
                    color: `hsl(${Math.random() * 360}, 90%, 60%)`,
                    vy: 0.3 + Math.random() * 0.5,
                    popped: false
                });
            }
        }

        function updateMode2() {
            for (let b of balloons2) {
                if (b.popped) continue;
                b.y -= b.vy * 0.3;
                if (b.y < -30) b.y = H + 20;
            }
            if (score >= 20 && !winMode2) {
                winMode2 = true;
                sound.win();
                winOverlay.classList.add('show');
                setTimeout(() => {
                    score = 0;
                    updateScore();
                    winOverlay.classList.remove('show');
                    initMode2();
                }, 3000);
            }
        }

        function drawMode2() {
            for (let b of balloons2) {
                if (b.popped) continue;
                ctx.save();
                ctx.shadowColor = 'rgba(0,0,0,0.1)';
                ctx.shadowBlur = 12;
                ctx.beginPath();
                ctx.ellipse(b.x, b.y, b.r, b.r * 1.15, 0, 0, Math.PI * 2);
                ctx.fillStyle = b.color;
                ctx.fill();
                ctx.strokeStyle = 'rgba(255,255,255,0.3)';
                ctx.lineWidth = 2;
                ctx.stroke();
                ctx.shadowBlur = 0;
                ctx.beginPath();
                ctx.ellipse(b.x - b.r * 0.25, b.y - b.r * 0.2, b.r * 0.2, b.r * 0.1, -0.5, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(255,255,255,0.5)';
                ctx.fill();
                ctx.beginPath();
                ctx.moveTo(b.x, b.y + b.r * 1.15);
                ctx.lineTo(b.x + 3, b.y + b.r * 1.5);
                ctx.strokeStyle = '#666';
                ctx.lineWidth = 1.5;
                ctx.stroke();
                ctx.restore();
            }
        }

        function popBalloon2(x, y) {
            for (let b of balloons2) {
                if (b.popped) continue;
                const d = Math.hypot(x - b.x, y - b.y);
                if (d < b.r + 10) {
                    b.popped = true;
                    sound.pop();
                    score++;
                    updateScore();
                    setTimeout(() => {
                        if (!winMode2) {
                            b.popped = false;
                            b.x = 30 + Math.random() * (W - 60);
                            b.y = H + 20;
                            b.color = `hsl(${Math.random() * 360}, 90%, 60%)`;
                            b.r = 25 + Math.random() * 22;
                            b.vy = 0.3 + Math.random() * 0.5;
                        }
                    }, 300);
                    return true;
                }
            }
            return false;
        }

        // ============================================================
        //  РЕЖИМ 3 — ШАРИКИ В НЕБО
        // ============================================================
        let balloons3 = [];

        function initMode3() {
            balloons3 = [];
            for (let i = 0; i < 20; i++) {
                balloons3.push({
                    x: Math.random() * W,
                    y: H + 30 + Math.random() * 100,
                    r: 22 + Math.random() * 20,
                    color: `hsl(${Math.random() * 360}, 85%, 65%)`,
                    vy: 0.8 + Math.random() * 1.5,
                    vx: (Math.random() - 0.5) * 0.8,
                    popped: false
                });
            }
        }

        function updateMode3() {
            for (let b of balloons3) {
                if (b.popped) continue;
                b.y -= b.vy;
                b.x += b.vx;
                if (b.y < -40) {
                    b.y = H + 30;
                    b.x = Math.random() * W;
                    b.color = `hsl(${Math.random() * 360}, 85%, 65%)`;
                    b.r = 22 + Math.random() * 20;
                }
                if (b.x < 10 || b.x > W - 10) b.vx *= -1;
            }
        }

        function drawMode3() {
            for (let b of balloons3) {
                if (b.popped) continue;
                ctx.save();
                ctx.shadowColor = 'rgba(0,0,0,0.08)';
                ctx.shadowBlur = 10;
                ctx.beginPath();
                ctx.ellipse(b.x, b.y, b.r, b.r * 1.1, 0, 0, Math.PI * 2);
                ctx.fillStyle = b.color;
                ctx.fill();
                ctx.strokeStyle = 'rgba(255,255,255,0.25)';
                ctx.lineWidth = 1.5;
                ctx.stroke();
                ctx.shadowBlur = 0;
                ctx.beginPath();
                ctx.ellipse(b.x - b.r * 0.2, b.y - b.r * 0.2, b.r * 0.2, b.r * 0.1, -0.5, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(255,255,255,0.4)';
                ctx.fill();
                ctx.restore();
            }
        }

        function popBalloon3(x, y) {
            for (let b of balloons3) {
                if (b.popped) continue;
                const d = Math.hypot(x - b.x, y - b.y);
                if (d < b.r + 10) {
                    b.popped = true;
                    sound.pop();
                    score++;
                    updateScore();
                    setTimeout(() => {
                        b.popped = false;
                        b.y = H + 30;
                        b.x = Math.random() * W;
                        b.color = `hsl(${Math.random() * 360}, 85%, 65%)`;
                        b.r = 22 + Math.random() * 20;
                        b.vy = 0.8 + Math.random() * 1.5;
                        b.vx = (Math.random() - 0.5) * 0.8;
                    }, 200);
                    return true;
                }
            }
            return false;
        }

        // ============================================================
        //  ОБЩИЕ ФУНКЦИИ
        // ============================================================
        function updateScore() {
            scoreDisplay.textContent = '⭐ ' + score;
        }

        function getPos(e) {
            const rect = canvas.getBoundingClientRect();
            const touch = e.touches ? e.touches[0] : e;
            return {
                x: (touch.clientX - rect.left) * (canvas.width / rect.width),
                y: (touch.clientY - rect.top) * (canvas.height / rect.height)
            };
        }

        // ============================================================
        //  ОБРАБОТЧИКИ
        // ============================================================
        function handleStart(e) {
            e.preventDefault();
            sound.resume();
            const pos = getPos(e);
            const x = pos.x, y = pos.y;

            if (currentMode === 1) {
                for (let f of flies) {
                    if (!f.alive) continue;
                    const d = Math.hypot(x - f.x, y - f.y);
                    if (d < f.r + 15) {
                        dragTarget = f;
                        dragOffsetX = f.x - x;
                        dragOffsetY = f.y - y;
                        return;
                    }
                }
                checkEatFly(x, y);
            } else if (currentMode === 2) {
                popBalloon2(x, y);
            } else if (currentMode === 3) {
                popBalloon3(x, y);
            }
        }

        function handleMove(e) {
            e.preventDefault();
            const pos = getPos(e);
            const x = pos.x, y = pos.y;

            if (currentMode === 1 && dragTarget) {
                dragTarget.x = x + dragOffsetX;
                dragTarget.y = y + dragOffsetY;
                checkEatFly(dragTarget.x, dragTarget.y);
            }
        }

        function handleEnd(e) {
            e.preventDefault();
            if (currentMode === 1 && dragTarget) {
                dragTarget = null;
            }
        }

        canvas.addEventListener('touchstart', handleStart, { passive: false });
        canvas.addEventListener('touchmove', handleMove, { passive: false });
        canvas.addEventListener('touchend', handleEnd, { passive: false });
        canvas.addEventListener('mousedown', handleStart);
        canvas.addEventListener('mousemove', (e) => { if (e.buttons === 1) handleMove(e); });
        canvas.addEventListener('mouseup', handleEnd);

        // ============================================================
        //  ПЕРЕКЛЮЧЕНИЕ РЕЖИМОВ
        // ============================================================
        function setMode(mode) {
            currentMode = mode;
            score = 0;
            updateScore();
            winOverlay.classList.remove('show');
            dragTarget = null;
            document.querySelectorAll('#menu button').forEach(b => b.classList.remove('active'));
            document.getElementById('mode' + mode).classList.add('active');

            if (mode === 1) {
                hint.textContent = '👆 Тяни муху пальцем к ящерице в рот!';
                initLizard();
            } else if (mode === 2) {
                hint.textContent = '👆 Лопай шарики! Собери 20 для победы!';
                initMode2();
            } else if (mode === 3) {
                hint.textContent = '☁️ Лопай шарики, которые летят в небо!';
                initMode3();
            }
        }

        document.getElementById('mode1').addEventListener('click', () => setMode(1));
        document.getElementById('mode2').addEventListener('click', () => setMode(2));
        document.getElementById('mode3').addEventListener('click', () => setMode(3));

        // ============================================================
        //  ГЛАВНЫЙ ЦИКЛ
        // ============================================================
        function gameLoop() {
            if (currentMode === 1) {
                updateLizard();
            } else if (currentMode === 2) {
                updateMode2();
            } else if (currentMode === 3) {
                updateMode3();
            }

            ctx.clearRect(0, 0, W, H);

            if (currentMode === 1) {
                // --- Фон для ящерицы (ландшафтный) ---
                const grad = ctx.createLinearGradient(0, 0, 0, H);
                grad.addColorStop(0, '#87CEEB');
                grad.addColorStop(0.45, '#A8D5BA');
                grad.addColorStop(1, '#6B8E6B');
                ctx.fillStyle = grad;
                ctx.fillRect(0, 0, W, H);
                // Солнце справа
                ctx.save();
                ctx.shadowColor = 'rgba(255,220,50,0.3)';
                ctx.shadowBlur = 50;
                ctx.beginPath();
                ctx.arc(W - 70, 70, 50, 0, Math.PI * 2);
                ctx.fillStyle = '#FFD700';
                ctx.fill();
                ctx.restore();
                drawLizard();
            } else if (currentMode === 2) {
                ctx.fillStyle = '#87CEEB';
                ctx.fillRect(0, 0, W, H);
                ctx.save();
                ctx.fillStyle = 'rgba(255,255,255,0.4)';
                for (let i = 0; i < 5; i++) {
                    const cx = (i * 200 + 40) % W;
                    const cy = 30 + i * 15 % 50;
                    ctx.beginPath();
                    ctx.arc(cx, cy, 45, 0, Math.PI * 2);
                    ctx.arc(cx + 40, cy - 12, 38, 0, Math.PI * 2);
                    ctx.arc(cx - 35, cy + 5, 32, 0, Math.PI * 2);
                    ctx.fill();
                }
                ctx.restore();
                drawMode2();
            } else if (currentMode === 3) {
                const grad = ctx.createLinearGradient(0, 0, 0, H);
                grad.addColorStop(0, '#1a2a6c');
                grad.addColorStop(0.5, '#b21f1f');
                grad.addColorStop(1, '#fdbb2d');
                ctx.fillStyle = grad;
                ctx.fillRect(0, 0, W, H);
                ctx.fillStyle = 'rgba(255,255,255,0.3)';
                for (let i = 0; i < 40; i++) {
                    ctx.beginPath();
                    ctx.arc((i * 137 + 50) % W, (i * 97 + 30) % (H * 0.4), 1.8, 0, Math.PI * 2);
                    ctx.fill();
                }
                drawMode3();
            }

            frameId = requestAnimationFrame(gameLoop);
        }

        // ============================================================
        //  СТАРТ
        // ============================================================
        setMode(1);
        gameLoop();

        console.log('🎮 Игры для малышей (горизонтальный режим) запущены!');
    </script>
</body>
</html>
