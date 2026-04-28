<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Plushie Studio</title>
    <style>
        :root { --pink: #ff85a2; --bg: #fff0f5; }
        body { margin: 0; background: var(--bg); font-family: 'Comic Sans MS', cursive; display: flex; touch-action: none; overflow: hidden; height: 100vh; }
        
        #toolbar {
            width: 100px; background: white; border-right: 4px solid var(--pink);
            display: flex; flex-direction: column; align-items: center; padding: 10px; gap: 10px;
            overflow-y: auto;
        }
        .swatch {
            width: 70px; height: 70px; border-radius: 15px; border: 3px solid #eee;
            display: flex; align-items: center; justify-content: center; font-size: 30px; cursor: pointer;
        }
        .swatch.active { border-color: var(--pink); box-shadow: 0 0 15px var(--pink); }

        #workspace { flex-grow: 1; position: relative; display: flex; align-items: center; justify-content: center; }
        canvas { background: white; border-radius: 40px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); width: 90%; height: 80%; }
        
        #hint { position: absolute; top: 20px; color: var(--pink); font-weight: bold; text-align: center; width: 100%; pointer-events: none; }
    </style>
</head>
<body>

<div id="toolbar">
    <p style="font-size: 12px; color: var(--pink);">FABRIC</p>
    <div class="swatch active" style="background:#ffb6c1" onclick="setPart('body', '#ffb6c1', this)"></div>
    <div class="swatch" style="background:#b0e0e6" onclick="setPart('body', '#b0e0e6', this)"></div>
    
    <p style="font-size: 12px; color: var(--pink);">EYES</p>
    <div class="swatch" onclick="setPart('eyes', '👀', this)">👀</div>
    <div class="swatch" onclick="setPart('eyes', '✨', this)">✨</div>

    <p style="font-size: 12px; color: var(--pink);">DECOR</p>
    <div class="swatch" onclick="setPart('acc', '💝', this)">💝</div>
    <div class="swatch" onclick="setPart('acc', '👑', this)">👑</div>
</div>

<div id="workspace">
    <div id="hint">Pinch or Drag to Mold the Shape!</div>
    <canvas id="plushieCanvas"></canvas>
</div>

<script>
    const canvas = document.getElementById('plushieCanvas');
    const ctx = canvas.getContext('2d');
    
    // Game State
    let parts = {
        body: { x: 0, y: 0, w: 150, h: 180, color: '#ffb6c1', type: 'shape' },
        eyes: { x: 0, y: -40, w: 100, h: 60, content: '👀', type: 'text' },
        acc: { x: 0, y: -120, w: 80, h: 80, content: '💝', type: 'text' }
    };
    
    let activePart = 'body';
    let isDragging = false;
    let lastTouchDist = 0;

    function resize() {
        canvas.width = canvas.offsetWidth;
        canvas.height = canvas.offsetHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    function setPart(category, value, el) {
        activePart = category;
        if (parts[category].type === 'shape') parts[category].color = value;
        else parts[category].content = value;
        
        document.querySelectorAll('.swatch').forEach(s => s.classList.remove('active'));
        el.classList.add('active');
    }

    // --- TOUCH CONTROLS ---
    canvas.addEventListener('touchstart', (e) => {
        isDragging = true;
        if (e.touches.length === 2) {
            lastTouchDist = Math.hypot(e.touches[0].pageX - e.touches[1].pageX, e.touches[0].pageY - e.touches[1].pageY);
        }
    });

    canvas.addEventListener('touchmove', (e) => {
        if (!isDragging) return;
        e.preventDefault();

        const p = parts[activePart];

        // MOLDING: One finger drag to stretch
        if (e.touches.length === 1) {
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            const mouseX = touch.clientX - rect.left - canvas.width/2;
            const mouseY = touch.clientY - rect.top - canvas.height/2;
            
            p.w = Math.abs(mouseX - p.x) * 2;
            p.h = Math.abs(mouseY - p.y) * 2;
        } 
        
        // MOLDING: Two finger pinch to scale both
        if (e.touches.length === 2) {
            const dist = Math.hypot(e.touches[0].pageX - e.touches[1].pageX, e.touches[0].pageY - e.touches[1].pageY);
            const delta = dist / lastTouchDist;
            p.w *= delta;
            p.h *= delta;
            lastTouchDist = dist;
        }
    }, { passive: false });

    canvas.addEventListener('touchend', () => { isDragging = false; });

    // --- DRAW LOOP ---
    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        ctx.save();
        ctx.translate(canvas.width/2, canvas.height/2);

        // Draw Body
        ctx.fillStyle = parts.body.color;
        ctx.beginPath();
        ctx.ellipse(parts.body.x, parts.body.y, parts.body.w/2, parts.body.h/2, 0, 0, Math.PI*2);
        ctx.fill();
        
        // Draw Eyes
        ctx.textAlign = "center";
        ctx.font = `${parts.eyes.h}px Arial`;
        ctx.fillText(parts.eyes.content, parts.eyes.x, parts.eyes.y + (parts.eyes.h/3));

        // Draw Accessory
        ctx.font = `${parts.acc.h}px Arial`;
        ctx.fillText(parts.acc.content, parts.acc.x, parts.acc.y + (parts.acc.h/3));

        // Highlight Active Part
        ctx.strokeStyle = varPink = "#ff85a2";
        ctx.setLineDash([5, 5]);
        const active = parts[activePart];
        ctx.strokeRect(active.x - active.w/2, active.y - active.h/2, active.w, active.h);

        ctx.restore();
        requestAnimationFrame(draw);
    }
    draw();
</script>
</body>
</html>
